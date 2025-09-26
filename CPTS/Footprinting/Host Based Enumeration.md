# File Sharing
## FTP
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1066)
Port ; 21

[Commands](https://web.archive.org/web/20230326204635/https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/) [Status Codes](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes) 
the /etc/ftpusers is a black list file containing all the users that cannot login to ftp
### Download files
we can download files from the ftp server using the get command like :
```shell
ftp> ls
...
-rwxrwxrwx    1 ftp      ftp            41 Sep 18 15:58 Important Notes.txt
226 Directory send OK.


ftp> get Important\ Notes.txt

226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)

# local

ls | grep Notes.txt

'Important Notes.txt'
```
we can download all files at once using the wget utility like :
```shell
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
...
FINISHED --2021-09-19 14:45:58--
Total wall clock time: 0,03s
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```
### Upload files
now we want to check if we can upload files to the server. we can use the `put` command to upload a file to the server, NOTE; the file must be in the same directory where you started the ftp process or else provide the full path:
```shell
ftp> put testupload.txt 

---> STOR testupload.txt
226 Transfer complete.

ftp> ls

---> LIST
...
-rw-------    1 1002     133             0 Sep 15 14:57 testupload.txt
```
### SSL Connection
Sometimes the service runs with SSL/TLS encryption, thus we can't use tools like `ncat` or `telnet` to connect to the ftp server. We can however use `openssl` using the following syntax: 
```shell
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```
## SMB
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1067)
Port ; 445

The Server Message Block is used to share files between machines. It is also cross platform with SMB in windows and Samba in linux. you can list available shares in smb using smbclient:
### List shares
```shell
smbclient -L \\\\192.168.122.5\\custom_share
Password for [MYGROUP\vulner]:

	Sharename       Type      Comment
	---------       ----      -------
	homes           Disk      Home Directories
	print$          Disk      Printer Drivers
	custom_share    Disk      Hello there
	IPC$            IPC       IPC Service (Samba 4.17.12-Debian)
	nobody          Disk      Home Directories
```
### Connect to shares
if the share has the `usershare allow guests` option set to `yes` we can connect to it like:
```shell
smbclient -N \\\\192.168.122.5\\custom_share
OR
smbclient -N //192.168.122.5/custom_share
```
#### Download files from shares
we can use the `get` command to download a files to our system:
```shell
smb: \> ls
  .                                   D        0  Tue Jul  1 06:33:42 2025
  ..                                  D        0  Tue Jul  1 06:33:19 2025
  testing_smb                         N       53  Tue Jul  1 06:34:06 2025

		81000912 blocks of size 1024. 74793592 blocks available
smb: \> get testing_smb 
getting file \testing_smb of size 53 as testing_smb (25.9 KiloBytes/sec) (average 25.9 KiloBytes/sec)
smb: \> !cat testing_smb 
Hello there hacker, I hope you are having a nice day
```
notice that I've executed a system command on my machine using the !\<cmd\> format
### View status
If you've gained a foothold into the system it's worth checking the status of the smb server as it may hold usernames and/or IPs:
```shell
root@playground:/home# smbstatus

Samba version 4.17.12-Debian
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
2831    nobody       nogroup      192.168.122.1 (ipv4:192.168.122.1:50386)  SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
custom_share 2831    192.168.122.1 Mon Jun 30 11:42:25 PM 2025 EDT  -            -
```
### Information gathering
rpcclient [man page](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) has all available functions in the COMMANDS section

when running nmap we are not given much information about the target smb server, we can in this case utilize a tool called `rpcclient`, as with `smbclient` we can use it to connect to one of the smb shares anonymously and gather more information:
```shell
rpcclient -U "" 192.168.122.5
Password for [MYGROUP\]: <BLANK>
rpcclient $> srvinfo
	PLAYGROUND     Wk Sv PrQ Unx NT SNT Samba 4.17.12-Debian
	platform_id     :	500
	os version      :	6.1
	server type     :	0x809a03
```
we can also query users using the `queryuser <RID>` but as you can see, we need the RID for that user. This is a great way to do user enumeration by trying multiple RIDs, the [samdump.py](https://github.com/fortra/impacket/blob/master/examples/samrdump.py) script does this for us. Other then `rpcclient` there are other tools that automate the process like [SMBMap](https://github.com/ShawnDEvans/smbmap) and [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) also [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) but the latter won't automatically start enumerating, you need to specify what you want
## NFS
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1069)
Port ; 2049 TCP

Setup guide [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)
The Network File System is similar to SMB but it's not cross platform, as it's for linux/unix only. You can view all available shares in the target machine using this command:
```shell
showmount -e 10.129.14.128
```
if you want to mount all available shares use this command:
```shell
mkdir target-NFS
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
```
to mount a single share use this command:
```sh
sudo mount 10.129.202.5:<SHARE_NAME> ./LOCAL_DESTINATION
```
NOTICE: the dot specifies the current directory, as mount uses absolute path
# DNS
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1069)
Port ; 53 

The Domain Name System is responsible for converting hostnames (e.g. google.com) to IP addresses (e.g. 172.217.17.46). There are several types of DNS servers that are used world wide, see the picture below;
![[Pasted image 20250702065137.png]] When querying DNS servers, the output is called a record. Here is a list of DNS records;
![[Pasted image 20250702070352.png]]
### Footprinting the Service
We can use the '@' character to query information about the dns server with dig:
```shell
dig ns inlanefreight.htb @10.129.14.128 # The IP of the DNS server

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
```
if the version entry exist on the server, we can fetch it;
```shell
dig CH TXT version.bind 10.129.120.85

;; ANSWER SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1"

;; ADDITIONAL SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1-Debian"
```
we can use the option `any` to request the server to disclose all available info;
```
dig any inlanefreight.htb @10.129.14.128

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
```
we notice that the server version doesn't appear in the output, thus we should always check it manually
#### Zone transfer
To do a zone transfer, we can use the following command;
```shell
dig axfr inlanefreight.htb @10.129.14.128
```
one tool that might help you enumerating the DNS server is [DNSenum](https://github.com/fwaeytens/dnsenum)
#### Subdomain enumeration
We can enumerate subdomains on the target using `subbrute` by adding the nameserver's ip address to a list and feeding it to the tool ;
```shell
python subbrute.py  inlanefreight.htb  -r resolver.txt -s /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

cat resolver.txt
ns.inlanefreight.htb
```
>[!note]
>Add the domain's IP to your hosts file

#### Answers Commands
These are the commands I used to get the answers, view the source on top;
Q1 ; `dig ns inlanefreight.htb @<IP>`
Q2 ; `dig axfr inlanefreight.htb @<IP>` then `dig axfr internal.inlanefreight.htb @<IP>`
Q3 ; using the previous two commands check every entry to get the answer
Q4 ; using dnsenum you have to brute force one of the subdomains available thru zone transfer, the key is to use multiple wordlists and before that, to determine the right subdomain to bruteforce use `dig ns <domain> @<IP> +short` if you get an answer bruteforce.
# Email
![[Pasted image 20250724013412.png]]
## SMTP
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1072)
Commands [samlogic](https://www.samlogic.net/articles/smtp-commands-reference.htm)
Port ; 25

We can connect to SMTP on port 25 using `telnet` like ;
```shell
telnet <IP> <PORT>
```
after the connection, we should start a session, to do so we use the HELO command with the FQDN of the mail server of the target like;
```shell
HELO mail1.inlanefreight.htb
# OR
EHLO mail1
```
the commands `VRFY`, `EXPN`, and `RCPT` can be used to enumerate usernames on the system.
>[!note]
>often this won't work, to make sure you're not getting false positives try with an invalid username

```shell
VRFY asldkjfhasdpofihjaewrg

252 2.0.0 asldkjfhasdpofihjaewrg

VRFY root

252 2.0.0 root
```
in the above example it's not possible to enumerate usernames because we will always get the same answer.
```shell
VRFY root

252 2.0.0 root

VRFY asdkj

550 5.1.1 <asdkj>: Recipient address rejected: User unknown in local recipient table
```
this time though we can enumerate usernames cause we have different responses. `EXPN` is similar to `VRFY`, except that when used with a distribution list, it will list all users on that list. This can be a bigger problem than the `VRFY` command since sites often have an alias such as "all". There are two tools you can use to enumerate an smtp server for usernames;
- [smtp-user-enum](https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum)
- metasploit `scanner/smtp/smtp_enum`

>[!hint] 
>`Sometimes we may have to work through a web proxy. We can also make this web proxy connect to the SMTP server. The command that we would send would then look something like this: CONNECT 10.129.14.128:25 HTTP/1.0. you can use BurpSuite`

## IMAP / POP3
`Internet Message Access Protocol` and `Post Office Protocol` work together and with the `Simple Mail Transfer Protocol`, IMAP and POP3 receive the email in different ways. It's out of scope for this paper, while SMTP transfer - sends - the email.
### Commands
Below is a list of `IMAP` commands;
![[Pasted image 20250703183038.png]]
and another list for `POP3` commands;
![[Pasted image 20250703183103.png]]
### Bad Settings
these are some dangerous settings that shouldn't be present on the server;
![[Pasted image 20250703183144.png]]
### Connection
If we have valid credentials for those services, we can login using `curl` or `openssl`
```shell
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v
```
using `openssl` with pop3
```shell
openssl s_client -connect <IP>:pop3s
```
using `openssl` with imap
```shell
openssl s_client -connect <IP>:imaps
```
#### User Enumeration
We can use `POP3`'s `USER` command to enumerate users ;
```shell
USER julio

-ERR


USER john

+OK
```
# SNMP
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1075)
Ports ; UDP161 ,UDP162(trap)

Simple Network Management Protocol is used to manage and monitor devices on the network. There are three versions of SNMP, v1, v2c and v3, the most used versions are v1 and v2c - the 'c' stands for community-based - but the problem with these two versions is that they don't have any authentication but version 2c have what is called a `community string` which is basically like a token to communicate between two devices
### Footprinting
Tools ; 
- snmpwalk
- braa
- onesixtyone
`snmpwalk` is used to query the OIDs with there information, the syntax for the `snmpwalk` command is like:
```shell
snmpwalk -v <VERSION(1,2c)> -c <COMMUNITY_STRING> <TARGET_IP>
```
`onesixtyone` is used to bruteforce the community strings, the syntax for the command is like:
```
onesixtyone -c /usr/share/Seclists/Discovery/SNMP/snmp.txt <TARGET_IP>
```
often when community strings are bound to specific addresses, they are named after the host, sometimes symbols are added for complexity. we can create custom wordlists with rules and mutations using [crunch](https://secf00tprint.github.io/blog/passwords/crunch/advanced/en).
`braa` is used to bruteforce the OIDs, we use this tool after obtaining the community strings. The syntax is as follows:
```
braa <COMMUNITY_STRING>@<TARGET_IP>:<START_OF_OID>*
braa public@10.10.10.9:.1.5.3.*
```
# Databases
## MSSQL
Port ; 1433
DMBS for windows. The SSMS -SQL Server Management Studio - is a client used to connect to the database server, thus you may find credentials on the same target that has this client. You can connect to MSSQL using many clients:
- [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15)
- [SQL Server PowerShell](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15)
- [HeidiSQL](https://www.heidisql.com/)
- [SQLPro](https://www.macsqlclient.com/)
- [Impacket's mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)
### Footprinting
We can use `nmap` with the `ms-sql` script to enumerate the service, the syntax is like:
```shell
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <TARGET_IP>
```
there is also an auxiliary module in metasploit which has nicer output:
```shell
msf6 > scanner/mssql/mssql_ping
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts <TARGET_IP>
```
### Connection
We can use [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) to connect to the mssql server, but we should have valid credentials, the syntax of the tool is like:
```shell
python3 mssqlclient.py <USERNAME>@10.129.201.248 -windows-auth

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
```
## Oracle TNS
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/2117)
Port ; 1521 TCP

`Oracle Transparent Network Substrate` server is a communication protocol that facilitates the communication between Oracle databases and applications over the network.
### Enumeration
Using Nmap we can check some information from the service port:
```shell
sudo nmap -p1521 -sV <TARGET_IP>
```
If we want to connect to an oracle database we must provide a `SID` or System Identifier, we can use multiple tools to brute force SIDs. but we will use `nmap` and `odat` :
```shell
sudo nmap -p1521 -sV <TARGET_IP> --script oracle-sid-brute

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-06 11:01 EST
Nmap scan report for 10.129.204.235
Host is up (0.0044s latency).

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute: 
|_  XE
```
Or with odat `Oracle Database Attacking Tool` :
```shell
sudo odat passwordguesser -s <TARGET_IP> -d XE

[1] (10.129.205.19:1521): Searching valid accounts on the 10.129.205.19 server, port 1521

[+] Valid credentials found: scott/tiger. Continue...             
[+] Accounts found on 10.129.205.19:1521/sid:XE: 
scott/tiger
```
In the above example we specified an action `password guesser`, use `--help` to view all available actions.
### Connection
Once we have the required information, we can login to the database using `sqlplus` :
```shell
sqlplus scott/tiger@<TARGET_ID>/XE
...

SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SCOTT                          CONNECT                        NO  YES NO
SCOTT                          RESOURCE                       NO  YES NO
```
we can also try escalating privileges and login as sysadmin:
```shell
sqlplus scott/tiger@<TARGET_ID>/XE as sysdba
...

SQL> select * from user_role_privs;

USERNAME                       GRANTED_ROLE                   ADM DEF OS_
------------------------------ ------------------------------ --- --- ---
SYS                            ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS                            APEX_ADMINISTRATOR_ROLE        YES YES NO
SYS                            AQ_ADMINISTRATOR_ROLE          YES YES NO
```
After gaining access, we can use the [sqlplus commands cheat sheet](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985) to enumerate the database :
```shell
SQL> select name, password from sys.user$;

NAME                           PASSWORD
------------------------------ ------------------------------
SYS                            FBA343E7D6C8BC9D
PUBLIC
CONNECT
```
we can try to upload a reverse shell to the server, but before that we should upload a normal file to check if there is any firewalls.
```shell
echo "Oracle File Upload Test" > testing.txt
odat utlfile -s <TARGET_IP> -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```
here we are dealing with a windows machine, hence the `C:\\` in the start of the path.
# IPMI
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1245)

`Intelligent Platform Management Interface` is used to access machines motherboards in case the sysadmin had to do a task that requires physical access or in case of an outage. This protocol is the closest thing to being physically present but in reality you are remotely connecting to it. you will find this protocol running on port 623UDP and it's available once you get internally into a network.
### Footprinting
After gaining access to the internal network you can check for IPMI using nmap on port 623UDP with the help of the [ipmi-version](https://nmap.org/nsedoc/scripts/ipmi-version.html) script:
```shell
sudo nmap -A --script ipmi-version -p623 -sU 10.129.202.5 
...
Host is up (0.17s latency).

PORT    STATE SERVICE  VERSION
623/udp open  asf-rmcp
| ipmi-version: 
|   Version: 
|     IPMI-2.0
|   UserAuth: password, md5, md2, null
|   PassAuth: auth_msg, auth_user, non_null_user
|_  Level: 1.5, 2.0
```
You can also use metasploit to fingerprint the service using the  [IPMI Information Discovery (auxiliary/scanner/ipmi/ipmi_version)](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/):
```shell
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > run

[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
### RAKP Hash Grabbing
The IPMI protocol uses another protocol for authentication by the name of RAKP. One critical flaw about this protocol is that before a client wants to authenticate to the service, the service sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place, thus you can get the hash of any VALID user on the system and try to crack it using `john` or `hashcat`. You can get the hash of the user using the [IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes/) in Metasploit:
```shell
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set RHOSTS 10.129.202.5
RHOSTS => 10.129.202.5
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set OUTPUT_JOHN_FILE /home/vulner/htb/hash.john
OUTPUT_JOHN_FILE => /home/vulner/htb/hash.john
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
[+] 10.129.202.5:623 - IPMI - Hash found: admin:b22e36078408000086d6d81d534cbcb06a27f320141f4b7c7011c08b381c2948c0c7916919154ea9a123456789abcdefa123456789abcdef140561646d696e:d1320a5a2f405605461aced4f869becf3caa2e42
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Cracking with john:
```shell
➜  john hash.john 
Warning: detected hash type "RAKP", but the string is also recognized as "RAKP-opencl"
....
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
trinity          (10.129.202.5 admin)
...
```
### IPMI Default Creds
The below credentials are set by default in the most used vendors:
- Vendor  / username / passowrd
-  Dell iDRAC / root / calvin
- HP iLO / Administrator / randomized 8-character string consisting of numbers and uppercase letters
- Supermicro IPMI / ADMIN / ADMIN
If you encountered the HP iLO IPMI you can use rules in `hashcat` to bruteforce the password:
```shell
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```
# Rsync
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1240)

Rsync is used to transfer files between the local and remote machines. It's strong point is in the algorithm which transfers the messing parts of a file to the destination instead of the whole file. It runs on port 873 and can be configured to use `ssh` established connections to transfer files securely. There are many ways this service can be abused, on of them is by listing the contents a shared folder on the target system, some times we don't need credentials to do this, and some times we do. 
### Footprinting
We can use nmap to look for the Rsync service on port 873 like:
```shell
sudo nmap -sV -p873 <TARGET_IP>

Host is up (0.0058s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```
We can then probe the service to check for accessible shares:
```shell
nc -nv <TARGET_IP> 873

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools # <SHARE_NAME>
```
after confirming the share name, we can use `rsync` to list the its contents:
```shell
rsync -av --list-only rsync://<TARGET_IP>/<SHARE_NAME>

receiving incremental file list
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh
```
we can then sync the files to our machine:
```
rsync -av rsync://<TARGET_IP>/<TARGET_SHARE>
```
if `rsync` is configured to use ssh we can add the `-e ssh` options or `-e "ssh -p2222"` option for none standard ports, this [guide](https://phoenixnap.com/kb/how-to-rsync-over-ssh) explains the syntax further.
## R-Services
Source [HackTheBox](https://academy.hackthebox.com/module/112/section/1240)

this service is well covered and there is no need to shorten it futher.