# Recon
you can view the full notes on [[Active Directory - Recon|Active Directory - Recon]] 
## Initial Enumeration
Collect as much information as possible
- [[Active Directory - Recon#passive host discovery|passive host discovery]] using `Responder`, `pktmon.exe` and `tcpdump`
- [[Active Directory - Recon#active host discovery|active host discovery]] using `nmap`
- [[Active Directory - Recon#Identifying Users|identifying users]] using `kerbrute`

## Post-FootHold Enumeration
### Enumerating Security Controls
- [[Active Directory - Recon#Windows Defender|enumerating windows defender]]
- [[Active Directory - Recon#App Locker|enumerating app locker]]
- [[Active Directory - Recon#PowerShell Constrained Language Mode|PowerShell constrained language mode]]
- [[Active Directory - Recon#LAPS|enumerating LAPS]]
### Credentialed Enumeration - Linux
- [[Active Directory - Recon#nxc|enumerating groups,users and shares]] using `netexec`
- [[Active Directory - Recon#SMBMap|enumerating shares]] using `smbmap`
- [[Active Directory - Recon#rpcclient|enumerating the domain with Null session]] using `rpcclient`
- [[Active Directory - Recon#Windapsearch|enumerating groups,users and shares]] using `windapsearch`
- [[Active Directory - Recon#Bloodhound.py|collect and visualize domain data]] using `bloodhound-python`
### Credentialed Enumeration - Windows
- [[Active Directory - Recon#ActiveDirectory PowerShell Module|enumerating SPNs,groups and trust relations]] using `ActiveDirectory` PowerShell module
- [[Active Directory - Recon#PowerView|general enumeration]] using `PowerView.ps1` PowerShell module
- [[Active Directory - Recon#SharpView|general enumeration]] using `SharpView.ps1` PowerShell module
- [[Active Directory - Recon#Snaffler|harvest shares for sensitive data]] using `Snaffler.exe`
- [[Active Directory - Recon#BloodHound|collect and visualize domain data]] using `SharpHound.exe` and `BloodHound.exe`
### Living of the land - Windows
- [[Active Directory - Recon#Env Commands For Host & Network Recon|get basic host and network info]]
- [[Active Directory - Recon#Harnessing PowerShell|using powershell]]
- [[Active Directory - Recon#Checking Defenses|checking defences]] using `netsh` and `sc`
- [[Active Directory - Recon#checking logged on users|checking logged on users]]
- [[Active Directory - Recon#Network Information|fetch network information]]
- [[Active Directory - Recon#Windows Management Instrumentation (WMI)|local enumeration using WMI]]
- [[Active Directory - Recon#Net Commands|local and domain enumeration]] using `net`
- [[Active Directory - Recon#Dsquery|local and domain enumeration]] using `dsquery`

# Attacking AD
you can view the full notes on [[Attacking Active Directory|Attacking Active Directory]]

## LLMNR/NBT-NS Poisoning
- [[Attacking Active Directory#LLMNR/NBT-NS Poisoning|An example of LLMNR/NBT-NS Poisoning]]
- [[Attacking Active Directory#From Linux|LLMNR/NBT-NS Poisoning from Linux]]
- [[Attacking Active Directory#From Windows|LLMNR/NBT-NS Poisoning from Windows]]

## Password Spraying
- [[Attacking Active Directory#Enumerating & Retrieving Password Policies|Enumerating and retrieving password policies from Windows and Linux]]
- [[Attacking Active Directory#Making a Target User List|making a target user list]]
- [[Attacking Active Directory#Linux - Internal Password Spraying|Internal password spraying - Linux]]
- [[Attacking Active Directory#Windows - Internal Password Spraying|Internal password spraying - Windows]]

## Kerberoasting
- [[Attacking Active Directory#Kerberoasting - from Linux|Kerberoasting - Linux]]
- [[Attacking Active Directory#Kerberoasting - from Windows|Kerberoasting - Windows]]

## Access Control List (ACL) Abuse
- [[Attacking Active Directory#Intro|Intro about ACL abuse]]
- [[Attacking Active Directory#ACL Enumeration|ACL enumeration]]
- [[Attacking Active Directory#ACL Abuse Tactics|ACL abuse tactics]]
- 