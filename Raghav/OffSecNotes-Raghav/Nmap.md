**Normal Ping Scan**
nmap -sn [IP]

**ICMP Echo Scan**
nmap -sn -PE [IP]

**ICMP Mask Scan**
nmap -sn -PM [IP]

**ICMP Timestamp Scan**
nmap -sn -PP [IP]

**ARP Scan without pinging**
nmap -PR -sn [IP]

**TCP SYN Ping Scan**
nmap -PS21-50 -sn [IP]
nmap -PS80,443 -sn [IP]

**TCP ACK Ping**
nmap -PA21-50 -sn [IP]
nmap -PA80,443 -sn [IP]

**UDP Ping**
nmap -PU-sn [IP]

**Query DNS**
nmap -R [IP]
nmap --dns-servers DNS_SERVER [IP]

**TCP Connect**
nmap -sT [IP]

**SYN  Connect**
nmap -sS [IP]

**UDP Scan**
nmap -sU  -F [IP]

**TCP Null Scan** 
sudo nmap -sN [IP]

**TCP FIN Scan** 
sudo nmap -sF [IP]

**TCP Xmas Scan** 
sudo nmap -sX [IP]

**TCP Maimon Scan** 
sudo nmap -sM [IP]

**TCP ACK Scan** 
sudo nmap -sA [IP]

**TCP Window Scan** 
sudo nmap -sW [IP]

**Custom TCP Scan** 
sudo nmap --scanflags URGACKPSHRSTSYNFIN [IP]

**Spoofed Source IP** 
sudo nmap -S SPOOFED_IP [IP]

**Spoofed MAC Address** 
--spoof-mac SPOOFED_MAC

**Decoy Scan** 
nmap -D DECOY_IP,ME [IP]

**Idle (Zombie) Scan** 
sudo nmap -sI ZOMBIE_IP [IP]
Fragment IP data into 8 bytes 
-f
Fragment IP data into 16 bytes 
-ff

`-sV`
determine service/version info on open ports

`-sV --version-light`
try the most likely probes (2)

`-sV --version-all`
try all available probes (9)

`-O`
detect OS

`--traceroute`
run traceroute to target

`--script=SCRIPTS`

Nmap scripts to run
`-sC` or `--script=default`

run default scripts
`-A`
equivalent to `-sV -O -sC --traceroute`

You can control the scan timing using `-T<0-5>`. `-T0` is the slowest (paranoid), while `-T5` is the fastest. According to Nmap manual page, there are six templates:
-   paranoid (0)
-   sneaky (1)
-   polite (2)
-   normal (3)
-   aggressive (4)
-   insane (5)

To avoid IDS alerts, you might consider `-T0` or `-T1`. For instance, `-T0` scans one port at a time and waits 5 minutes between sending each probe, so you can guess how long scanning one target would take to finish. If you don’t specify any timing, Nmap uses normal `-T3`. Note that `-T5` is the most aggressive in terms of speed; however, this can affect the accuracy of the scan results due to the increased likelihood of packet loss. Note that `-T4` is often used during CTFs and when learning to scan on practice targets, whereas `-T1` is often used during real engagements where stealth is more important.

Alternatively, you can choose to control the packet rate using `--min-rate <number>` and `--max-rate <number>`. For example, `--max-rate 10` or `--max-rate=10` ensures that your scanner is not sending more than ten packets per second.

Moreover, you can control probing parallelization using `--min-parallelism <numprobes>` and `--max-parallelism <numprobes>`. Nmap probes the targets to discover which hosts are live and which ports are open; probing parallelization specifies the number of such probes that can be run in parallel. For instance, `--min-parallelism=512` pushes Nmap to maintain at least 512 probes in parallel; these 512 probes are related to host discovery and open ports.