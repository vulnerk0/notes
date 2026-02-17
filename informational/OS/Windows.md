# File Structure
![[Pasted image 20251021103441.png]]

>[!note]
>before the last entry, we see that the three files `System` and `System32` and `SysWOW64` are used when a program loads a DLL file without specifying an absolute path, I think we can abuse this to escalate our privileges by creating a DLL file with a reverse shell and placing it in one of these folders.

---
# File System

The currently used file systems are FAT32 (File Allocation Table) that uses 32bits and NTFS (New Technology File System). Here is a little comparison between the two:

**`Pros of FAT32:`**

- Device compatibility - it can be used on computers, digital cameras, gaming consoles, smartphones, tablets, and more.
- Operating system cross-compatibility - It works on all Windows operating systems starting from Windows 95 and is also supported by MacOS and Linux.

**`Cons of FAT32:`**

- Can only be used with files that are less than 4GB.
- No built-in data protection or file compression features.
- Must use third-party tools for file encryption.

NTFS (New Technology File System) is the default Windows file system since Windows NT 3.1. In addition to making up for the shortcomings of FAT32, NTFS also has better support for metadata and better performance due to improved data structuring.

**`Pros of NTFS:`**

- NTFS is reliable and can restore the consistency of the file system in the event of a system failure or power loss.
- Provides security by allowing us to set granular permissions on both files and folders.
- Supports very large-sized partitions.
- Has journaling built-in, meaning that file modifications (addition, modification, deletion) are logged.

**`Cons of NTFS:`**

- Most mobile devices do not support NTFS natively.
- Older media devices such as TVs and digital cameras do not offer support for NTFS storage devices.

## Permissions

The NTFS file system has basic and advanced permissions:
![[Pasted image 20251022133649.png]]
### Integrity Control Access Control List (icacls)

You can manage these permissions using the File Explorer GUI under the security tab or for more control, you can use `icacls` which is a command line utility to manage NTFS file permissions. To list permissions of a "folder" you can issue the command `icacls C:\FOLDER` :
```cmd-session
icacls c:\windows
c:\windows NT SERVICE\TrustedInstaller:(F)
           NT SERVICE\TrustedInstaller:(CI)(IO)(F)
           NT AUTHORITY\SYSTEM:(M)
           NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
           BUILTIN\Administrators:(M)
           BUILTIN\Administrators:(OI)(CI)(IO)(F)
           BUILTIN\Users:(RX)
           BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
           CREATOR OWNER:(OI)(CI)(IO)(F)
           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(RX)
           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(RX)
           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
```

There are a couple of permissions you see in the output above:
- `(CI)`: container inherit
- `(OI)`: object inherit
- `(IO)`: inherit only
- `(NP)`: do not propagate inherit
- `(I)`: permission inherited from parent container

In the above example, the `NT AUTHORITY\SYSTEM` account has object inherit, container inherit, inherit only, and full access permissions. This means that this account has full control over all file system objects in this directory and subdirectories.

Basic access permissions are as follows:

- `F` : full access
- `D` :  delete access
- `N` :  no access
- `M` :  modify access
- `RX` :  read and execute access
- `R` :  read-only access
- `W` :  write-only access

We can add and remove permissions via the command line using `icacls`. Here we are executing `icacls` in the context of a local administrator account showing the `C:\users` directory where the `joe` user does not have any write permissions.
```cmd-session
icacls c:\Users
c:\Users NT AUTHORITY\SYSTEM:(OI)(CI)(F)
         BUILTIN\Administrators:(OI)(CI)(F)
         BUILTIN\Users:(RX)
         BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
         Everyone:(RX)
         Everyone:(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files
```

Using the command `icacls c:\users /grant joe:f` we can grant the joe user full control over the directory, but given that `(oi)` and `(ci)` were not included in the command, the joe user will only have rights over the `c:\users` folder but not over the user subdirectories and files contained within them.

```cmd-session
icacls c:\users
c:\users WS01\joe:(F)
         NT AUTHORITY\SYSTEM:(OI)(CI)(F)
         BUILTIN\Administrators:(OI)(CI)(F)
         BUILTIN\Users:(RX)
         BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
         Everyone:(RX)
         Everyone:(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files
```

---
# Troubleshooting

### xfreerdp3 connection

When I tried to connect to a server using `xfreerdp3` there was an error saying connection timeout, to solve this I had to force "Network Level Authentication" or TLS. I think the problem is that `xfreerdp3` connects to the target using kerberos by default, to force the usage of `nla` use `/sec:nla`, another issue is slow connection to the servers which will take more time to connect to the machine which will result in connection timeout error. To solve add `/timeout:60000` like:
```
xfreerdp3 /v:IP /u:USER /p:PASS /timeout:60000 /cert:ignore /sec:nla or tls
```

this is a log without using `/sec:nla`:
```
[22:05:47:348] [16882:000041f4] [WARN][com.freerdp.crypto] - [tls_verify_certificate]: [DANGER] Certificate not checked, /cert:ignore in use.
[22:05:47:348] [16882:000041f4] [WARN][com.freerdp.crypto] - [tls_verify_certificate]: [DANGER] This prevents MITM attacks from being detected!
[22:05:47:348] [16882:000041f4] [WARN][com.freerdp.crypto] - [tls_verify_certificate]: [DANGER] Avoid using this unless in a secure LAN (=no internet) environment
[22:06:27:608] [16882:000041f4] [ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5glue_get_init_creds (Client 'htb-student@ATHENA.MIT.EDU' not found in Kerberos database [-1765328378])
[22:07:07:852] [16882:000041f4] [ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5glue_get_init_creds (Client 'htb-student@ATHENA.MIT.EDU' not found in Kerberos database [-1765328378])
[22:07:07:852] [16882:000041f4] [ERROR][com.freerdp.core.transport] - [transport_default_write]: BIO_should_retry returned an error: error:8000006E:system library::Connection timed out
[22:07:07:852] [16882:000041f4] [ERROR][com.freerdp.core] - [transport_default_write]: ERRCONNECT_CONNECT_TRANSPORT_FAILED [0x0002000D]
[22:07:07:852] [16882:000041f4] [ERROR][com.freerdp.core.transport] - [transport_connect_nla]: NLA begin failed
[22:07:22:872] [16882:000041f4] [ERROR][com.freerdp.core] - [freerdp_tcp_default_connect]: ERRCONNECT_CONNECT_FAILED [0x00020006]
[22:07:22:873] [16882:000041f4] [ERROR][com.freerdp.core] - [freerdp_tcp_default_connect]: failed to connect to 10.129.130.195
[22:07:22:873] [16882:000041f4] [ERROR][com.freerdp.core.nego] - [nego_connect]: Failed to connect
[22:07:22:873] [16882:000041f4] [ERROR][com.freerdp.core] - [freerdp_connect]: freerdp_post_connect failed
```

notice that it tries connecting using kerberos.

---
# Useful Resources
[Get-WmiObject](https://adamtheautomator.com/get-wmiobject/) a good cmdlet for getting more information about local and remote computers
[Enable Remote Desktop on your PC](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-allow-access) a page from microsoft explaining how to enable RDP. Might come in handy someday
[Windows Command Reference](https://download.microsoft.com/download/5/8/9/58911986-D4AD-4695-BF63-F734CD4DF8F2/ws-commands.pdf) a PDF file that covers most Windows commands with usage examples.