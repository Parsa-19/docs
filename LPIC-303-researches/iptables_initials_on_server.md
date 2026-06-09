## initial commands on every production server in iptables
```
# Loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Existing connections
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# FTP control
iptables -A INPUT -p tcp --dport 21 -m conntrack --ctstate NEW -j ACCEPT

# FTP passive ports (example)
iptables -A INPUT -p tcp --dport 50000:51000 -m conntrack --ctstate NEW -j ACCEPT

iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP
```