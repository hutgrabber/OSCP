# Windows Users & Groups
1. Enumerate using the `net group` & the `net user` commands. Add the `/domain` tag if necessary.
2. If you have perms, you can add yourself to a group or create a new user:
```PowerShell
net user /domain username pswd  /add
net group /domain "Group Name" username  /add
# remove the /domain flag to add user locally
# then use the runas.exe /user:new_usr cmd to run a shell.
# use /del instead of add to remove members
net user /domain username pswd /del
net group /domain "Administrators" username /del
```
