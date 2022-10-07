Import ```beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1```

To see the domain ```beacon> powershell Get-Domain``` 

To get domain group member with distinguished name ```beacon> powershell Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName```, the same for the username but without the "select" statement ```beacon> powershell Get-DomainGroupMember -Identity "Domain Admins"```
