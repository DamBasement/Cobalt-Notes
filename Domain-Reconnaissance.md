Import ```beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1```

- ```beacon> powershell Get-Domain``` See the domain 
- ```beacon> powershell Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName```Get domain group member with distinguished name
- ```beacon> powershell Get-DomainGroupMember -Identity "Domain Admins"``` same for the username but without the "select" statement
- ```powershell Get-DomainTrust```See domain trust and its direction
- ```powershell Get-DomainGroup -Identity "Domain Admins" -Domain cyberbotic.io -Properties ObjectSid``` Get SID of a target group-
- ```powershell Get-DomainSID"``` Get Domain SID 
- ```powershell Get-DomainController -Domain cyberbotic.io | select Name``` Get Domain Controller
- ```powershell Get-DomainGroupMember -Identity "Domain Admins" -Domain cyberbotic.io | select MemberName``` Find a domain administrator in the specified Domain
- ```powershell Get-DomainComputer -Domain dev-studio.com -Properties DnsHostName``` Enumerate foreign domain across the trust.
- ```beacon> powershell Get-DomainForeignGroupMember -Domain dev-studio.com``` Enumerate any groups that contain users outside of its domain and return its members.
- ```powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1120``` Get Member Name by converting its SID
