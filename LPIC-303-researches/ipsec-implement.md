
- **left** refers to the server that have the access to the data and network.
- **right** referes to the client that wants to connecto to that network through ipsec protocol.
```
# on both left and right
ipsec initnss
ipsec newhostkey

# list generated keys on each machine
ipsec showhostkey --list

# you can see the key on left machine
ipsec showhostkey \
  --ckaid edf4de29e0c73d04df00c3c9a2425165f8717a08 \
  --left | grep 'leftrsasigkey'

# you can see the key on right machine
ipsec showhostkey \
  --ckaid edf4de29e0c73d04df00c3c9a2425165f8717a08 \
  --right | grep 'rightrsasigkey'

# on both machines create the configuration file in `/etc/ipsec.d/host-to-host.conf` with exact info
conn tunnel
    leftid=@west
    left=192.168.55.102
    leftrsasigkey=0sAwEAAcLzCnhDBXYH1UVbFTouGScEZiA11fNEf4k2...
    rightid=@east
    right=192.168.55.101
    rightrsasigkey=0sAwEAAbAyEbQ8w9XJEYNWKkXHeNGiSFA7X6SkghDy...
    authby=rsasig

# open these udp ports on both machines in firewall:
firewall-cmd --add-port=500/udp --permanent
firewall-cmd --add-port=4500/udp --permanent
firewall-cmd --add-protocol=50 --permanent
firewall-cmd --add-protocol=51 --permanent
firewall-cmd --reload

# start ipsec service on both machines
systemctl start ipsec

# verify the connction from the left machine
ipsec verify

# on both machines add the tunnel 
ipsec auto --add tunnel

# on left bring up the tunnel
ipsec auto --up tunnel

# to check the connection run tcpdump on left and ping the left machine from right. you will see the packets are recieved in left
tcpdump -n -i enp0s8 esp or udp port 500 or udp port 4500
ping -n -c 4 192.168.55.102
```