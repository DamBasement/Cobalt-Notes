# External-Reconnaissance

Reconnaissance can happen on different levels. 
## DNS
Domain Name System (DNS) records can be a standard first step to perform to retrieve info regarding services that may be exposed to the Internet
```
$ dig targetORG.io +short
104.21.90.222
172.67.205.143
```
## WHOIS
You can follow on WHOISing the public IPs you've got to find each public IP address who it belongs to
```
$ whois 104.21.90.222
```
## DNSSCAN
Subdomains can also provide insight to other publicly available services, which could include webmail, remote access solutions such as Citrix, or a VPN.
## SPOOFCHECK
Weak email security (SPF, DMARC and DKIM) may allow us to spoof emails. [Spoofcheck](https://github.com/BishopFox/spoofcheck) is a Python tool that can verify the email security of a given domain.
```
$ ./spoofcheck.py cyberbotic.io
```
# NMAP
The nmap utility can be used to scan for open ports in an IP range.
Here is a list of ports to look for when hunting for domain controllers.
```
53/TCP and 53/UDP for DNS
88/TCP for Kerberos authentication
135/TCP and 135/UDP MS-RPC epmapper (EndPoint Mapper)
137/TCP and 137/UDP for NBT-NS
138/UDP for NetBIOS datagram service
139/TCP for NetBIOS session service
389/TCP for LDAP
636/TCP for LDAPS (LDAP over TLS/SSL)
445/TCP and 445/UDP for SMB
464/TCP and 445/UDP for Kerberos password change
3268/TCP for LDAP Global Catalog
3269/TCP for LDAP Global Catalog over TLS/SSL
```
