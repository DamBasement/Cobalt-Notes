# Use PowerView, but before.. import it! 
```Import powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1```

- Get the Domain
```
powershell Get-Domain
``` 
- Get Forest
```
powershell Get-ForestDomain
``` 
- Find domain password policy.
```
powershell Get-DomainPolicyData | select -expand SystemAccess
```
- Get Domain User (w/o -Identity to get all users)
```
powershell Get-DomainUser -Identity jking -Properties DisplayName, MemberOf | fl
``` 
- Get all computers in a Domain
```
powershell Get-DomainComputer -Properties DnsHostName | sort -Property DnsHostName
``` 
- Get all OU (organizational unit)
```
powershell Get-DomainOU -Properties Name | sort -Property Name
```
- Get all domain groups or specific domain group objects.
```
powershell Get-DomainGroup | where Name -like "*Admins*" | select SamAccountName
``` 
- Get all Group Policy Objects (GPOs)
```
powershell Get-DomainGPO -Properties DisplayName | sort -Property DisplayName
``` 
- Get all GPOs applied to a machine
```
powershell Get-DomainGPO -Properties DisplayName -ComputerIdentity wkstn-1 | sort -Property DisplayName
``` 
- Get all GPOs that modify local group membership
```
powershell Get-DomainGPOLocalGroup | select GPODisplayName, GroupName
``` 
- Get the machines where a specific domain user/group is a member of a specific local group.
```
powershell Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators | select ObjectName, GPODisplayName, ContainerName, ComputerName | fl
``` 
- Get domain group member with distinguished name
```
powershell Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName
```
- same for the username but without the "select" statement. 
```
powershell Get-DomainGroupMember -Identity "Domain Admins"
``` 
- See domain trust and its direction
```
powershell Get-DomainTrust
```
- Get SID of a target group
```
powershell Get-DomainGroup -Identity "Domain Admins" -Domain cyberbotic.io -Properties ObjectSid
``` 
- Get Domain SID 
```
powershell Get-DomainSID
``` 
- Get Domain Controller
```
powershell Get-DomainController | select Forest, Name, OSVersion | fl
``` 
- Get domain admin in the specified Domain
```
powershell Get-DomainGroupMember -Identity "Domain Admins" -Domain cyberbotic.io | select MemberName
``` 
- Get foreign domain across the trust.
```
powershell Get-DomainComputer -Domain dev-studio.com -Properties DnsHostName
``` 
- Get any groups that contain users outside of its domain and return its members.
```
powershell Get-DomainForeignGroupMember -Domain dev-studio.com
``` 
- Get Member Name by converting its SID
```
powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1120
``` 
