## File Sharing

### FTP

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1066) | **Port:** 21

**References:** [Commands](https://web.archive.org/web/20230326204635/https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/) | [Status Codes](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes)

The `/etc/ftpusers` file is a blacklist containing all users that cannot login to FTP.

#### Downloading Files

```
ftp> get Important\ Notes.txt
```

Download all files at once using `wget`:
```
wget -m --no-passive ftp://anonymous:anonymous@<TARGET_IP>
```

#### Uploading Files

> [!note] 
> The file must be in the same directory where you started the FTP process, or provide the full path.

```
ftp> put testupload.txt
```

#### SSL Connection

When the service runs with SSL/TLS encryption, tools like `ncat` or `telnet` won't work. Use `openssl` instead:

```
openssl s_client -connect <TARGET_IP>:21 -starttls ftp
```

---
### SMB

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1067) | **Port:** 445

Server Message Block is used to share files between machines. It is cross-platform — SMB on Windows and Samba on Linux.

#### Listing Shares



```
smbclient -L \\\\<TARGET_IP>\\<SHARE_NAME>
```

#### Connecting to a Share

If the share has `usershare allow guests` set to `yes`:



```
smbclient -N \\\\<TARGET_IP>\\<SHARE_NAME>
# OR
smbclient -N //<TARGET_IP>/<SHARE_NAME>
```

#### Downloading Files



```
smb: \> get <FILENAME>
```

> [!note] 
> You can run local system commands from within the SMB session using `!<cmd>` (e.g. `!cat file.txt`).

#### Viewing Server Status

Useful after gaining a foothold — may reveal usernames and IPs:



```
smbstatus
```

#### Information Gathering

**rpcclient** — connect anonymously and query server info:



```
rpcclient -U "" <TARGET_IP>
rpcclient $> srvinfo
```

Use `queryuser <RID>` to query individual users. The [samdump.py](https://github.com/fortra/impacket/blob/master/examples/samrdump.py) script automates RID enumeration.

Other tools that automate SMB enumeration:

- [SMBMap](https://github.com/ShawnDEvans/smbmap)
- [enum4linux-ng](https://github.com/cddmp/enum4linux-ng)
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) — requires specifying what to enumerate explicitly

---

### NFS

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1069) | **Port:** 2049 TCP

**Setup Guide:** [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)

Network File System is similar to SMB but Linux/Unix only.

#### Listing Available Shares



```
showmount -e <TARGET_IP>
```

#### Mounting All Shares



```
mkdir target-NFS
sudo mount -t nfs <TARGET_IP>:/ ./target-NFS/ -o nolock
```

#### Mounting a Single Share



```
sudo mount -t nfs <TARGET_IP>:<SHARE_NAME> ./<LOCAL_DESTINATION>
```

> [!note] 
> The `.` specifies the current directory. `mount` requires an absolute path.

---

## DNS

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1069) | **Port:** 53

The Domain Name System converts hostnames (e.g. `google.com`) to IP addresses (e.g. `172.217.17.46`).

#### Footprinting

Query the DNS server's nameserver records:



```
dig ns <DOMAIN> @<TARGET_IP>
```

Fetch the server version if the entry exists:



```
dig CH TXT version.bind <TARGET_IP>
```

Request all available info:



```
dig any <DOMAIN> @<TARGET_IP>
```

> [!note] 
> The server version may not appear in the `any` output — always check it manually.

#### Zone Transfer



```
dig axfr <DOMAIN> @<TARGET_IP>
```

Tool: [DNSenum](https://github.com/fwaeytens/dnsenum)

#### Subdomain Enumeration

Add the nameserver's IP to a resolvers file, then run `subbrute`:



```
python subbrute.py <DOMAIN> -r resolver.txt -s /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

> [!note] 
> Add the domain's IP to your `/etc/hosts` file before enumerating.

---

## Email

### SMTP

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1072) | **Port:** 25

**Commands Reference:** [samlogic](https://www.samlogic.net/articles/smtp-commands-reference.htm)

#### Connection



```
telnet <TARGET_IP> 25
```

Start a session after connecting:



```
EHLO mail1.inlanefreight.htb
```

#### User Enumeration

The `VRFY`, `EXPN`, and `RCPT TO` commands can be used to enumerate users. Only useful when the server returns **different responses** for valid vs. invalid users.

> [!note] 
> Always test with an invalid username first to confirm the server distinguishes between valid and invalid responses, otherwise you'll get false positives.

`EXPN` is similar to `VRFY` but also expands distribution lists — aliases like `all` can expose many usernames at once.

Automated tools:

- [smtp-user-enum](https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum)
- Metasploit: `scanner/smtp/smtp_enum`

> [!hint] 
> If working through a web proxy, use `CONNECT <TARGET_IP>:25 HTTP/1.0`. BurpSuite can assist with this.

---

### IMAP / POP3

**Port:** IMAP 143 / 993 (SSL) | POP3 110 / 995 (SSL)

`Internet Message Access Protocol` and `Post Office Protocol` work alongside SMTP. SMTP sends/transfers email; IMAP and POP3 handle receiving it.

#### Connecting

IMAP with curl:



```
curl -k 'imaps://<TARGET_IP>' --user <USER>:<PASS> -v
```

POP3 with openssl:



```
openssl s_client -connect <TARGET_IP>:pop3s
```

IMAP with openssl:



```
openssl s_client -connect <TARGET_IP>:imaps
```

#### User Enumeration via POP3



```
USER <USERNAME>
```

A `+OK` response indicates the user exists; `-ERR` means they don't.

---

## SNMP

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1075) | **Ports:** UDP 161, UDP 162 (trap)

Simple Network Management Protocol is used to manage and monitor network devices. Versions v1 and v2c lack authentication — v2c uses a `community string` as a basic token. v3 adds proper authentication.

#### Footprinting

Query OIDs with `snmpwalk`:



```
snmpwalk -v <VERSION> -c <COMMUNITY_STRING> <TARGET_IP>
```

Bruteforce community strings with `onesixtyone`:



```
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt <TARGET_IP>
```

> [!note] 
> Community strings are sometimes named after the host, occasionally with added symbols. Use [crunch](https://secf00tprint.github.io/blog/passwords/crunch/advanced/en) to generate custom wordlists with mutations.

Bruteforce OIDs with `braa` (after obtaining a community string):



```
braa <COMMUNITY_STRING>@<TARGET_IP>:<OID_PREFIX>*
```

---

## Databases

### MSSQL

**Port:** 1433 | Windows DBMS

The SQL Server Management Studio (SSMS) client may store credentials on the same host — worth checking after gaining a foothold.

#### Footprinting

With nmap scripts:



```
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmd,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <TARGET_IP>
```

With Metasploit (cleaner output):



```
use scanner/mssql/mssql_ping
set RHOSTS <TARGET_IP>
run
```

#### Connection

Using [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py):



```
python3 mssqlclient.py <USERNAME>@<TARGET_IP> -windows-auth
```

---

### Oracle TNS

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/2117) | **Port:** 1521 TCP

Oracle Transparent Network Substrate facilitates communication between Oracle databases and applications over the network.

#### Footprinting

Basic service scan:



```
sudo nmap -p1521 -sV <TARGET_IP>
```

Bruteforce SIDs with nmap:



```
sudo nmap -p1521 -sV <TARGET_IP> --script oracle-sid-brute
```

Bruteforce credentials with `odat`:



```
sudo odat passwordguesser -s <TARGET_IP> -d <SID>
```

> [!note] 
> Use `odat --help` to view all available actions.

#### Connection

Login with `sqlplus`:



```
sqlplus <USER>/<PASS>@<TARGET_IP>/<SID>
```

Attempt privilege escalation as sysdba:



```
sqlplus <USER>/<PASS>@<TARGET_IP>/<SID> as sysdba
```

Reference: [sqlplus commands cheat sheet](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985)

#### File Upload

Test for firewall restrictions before uploading a :



```
echo "Oracle File Upload Test" > testing.txt
odat utlfile -s <TARGET_IP> -d <SID> -U <USER> -P <PASS> --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

> [!note] 
> The `C:\\` path indicates a Windows target.

---

## IPMI

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1245) | **Port:** UDP 623

Intelligent Platform Management Interface provides out-of-band management access to a machine's motherboard — the closest thing to physical presence. Available once you gain internal network access.

#### Footprinting

With nmap:



```
sudo nmap -A --script ipmi-version -p623 -sU <TARGET_IP>
```

With Metasploit:



```
use auxiliary/scanner/ipmi/ipmi_version
set RHOSTS <TARGET_IP>
run
```

#### RAKP Hash Grabbing

A critical flaw in the RAKP authentication protocol causes the service to send a salted SHA1/MD5 hash of the user's password **before** authentication completes. This means you can retrieve the hash for any valid user and crack it offline.

Grab hashes with Metasploit:



```
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS <TARGET_IP>
set OUTPUT_JOHN_FILE /home/<USER>/hash.john
run
```

Crack with John:



```
john hash.john
```

Crack HP iLO hashes (8-char uppercase+digits) with hashcat rules:



```
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

#### Default Credentials

|Vendor|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|Random 8-char (uppercase + digits)|
|Supermicro IPMI|ADMIN|ADMIN|

---

## Rsync

**Source:** [HackTheBox](https://academy.hackthebox.com/module/112/section/1240) | **Port:** 873

Rsync transfers files between local and remote machines, only syncing the changed parts of a file rather than the whole thing. It can be configured over SSH for secure transfers. Shares are sometimes accessible without credentials.

#### Footprinting

Scan for the service:



```
sudo nmap -sV -p873 <TARGET_IP>
```

List available shares:



```
nc -nv <TARGET_IP> 873
```

Then type `#list` after the connection banner.

List share contents:



```
rsync -av --list-only rsync://<TARGET_IP>/<SHARE_NAME>
```

#### Syncing Files



```
rsync -av rsync://<TARGET_IP>/<SHARE_NAME>
```

If Rsync is configured over SSH:



```
rsync -av -e ssh rsync://<TARGET_IP>/<SHARE_NAME>
# Non-standard port:
rsync -av -e "ssh -p2222" rsync://<TARGET_IP>/<SHARE_NAME>
```

Reference: [rsync over SSH guide](https://phoenixnap.com/kb/how-to-rsync-over-ssh)