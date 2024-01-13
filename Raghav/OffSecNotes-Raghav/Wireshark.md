```Commands
ip.addr == <IP Address>
ip.src == <SRC IP Address> and ip.dst == <DST IP Address>
tcp.port eq <Port #> or <Protocol Name>
udp.port eq <Port #> or <Protocol Name>
arp.opcode==2 (reply)
arp.opcode==1 (request)
eth.src==80:fb:06:f0:45:d7 (Source MAC)
eth.addr==80:fb:06:f0:45:d7 (Filter for just MAC)
ip.src_host matches "\.237$" (Regex Filtering)
```

### Important Stuff to Navigate to or Use
Statistics > Protocol Hierarchy
File > Export Objects > HTTP
