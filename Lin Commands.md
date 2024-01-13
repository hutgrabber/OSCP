# Linux Handy Commands

---

## Tools
### Gobuster
```bash
gobuster dir -u 'http://192.158.45.222/' -w /usr/share/wordlists/dirb/common.txt
```

### WP-Scan
```bash
wpscan --url http://192.168.50.244 --enumerate p --plugins-detection aggressive -o outfile
# -e to enumerate
## vp   Vulnerable plugins
## ap   All plugins
`## p    Popular plugins
## vt   Vulnerable hemes
## at   All themes
## t    Popular themes
## tt   Timthumbs
## cb   Config backups
## dbe  Db exports
# --throttle in case website crashes
# --users-list [list.txt] # check if users exist.
# --password-attack [PARAM] # wp-login, xmlrpc, xmlrpc-multicall
# --force # do not check if target is running WP or not.

```

### Feroxbuster
```bash
```

### Dirsearch
```bash
```

### John
```bash
# first find the *2john file for the password.
```

```bash
# john misc
unshadow /etc/passwd /etc/shadow > output.db
john --wordlist=/usr/share/wordlists/rockyou.txt output.db
```

---

## Commands
### Find Magic
```bash
find / -perm /4000 -type f 2>/dev/null # suid binaries
```

### Git
```bash
git log # checks the different commits made in the past

```