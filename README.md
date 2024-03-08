# Anti-DDoS
IPTables/NFTables DDoS Protection Rules for Linux Servers


## IPTable's DDoS rules

## This means that all invalid packets are discarded immediately before they are further processed by the system.
```
iptables -t mangle -A PREROUTING --ctstate INVALID -m conntrack -j DROP
or this:
iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP
```
## This rule drops all packets that are not “Syn” in the TCP stack.
```
iptables -t mangle -A PREROUTING -m conntrack -p tcp ! --syn --ctstate NEW -j DROP
```
## The next iptables rule blocks new packets (only SYN packets can be new packets according to the previous two rules) that use a non-common TCP MSS value. This helps block stupid SYN floods.
```
iptables -t mangle -A PREROUTING -p tcp -m conntrack --ctstate NEW -m tcpmss ! --mss 536:65535 -j DROP
```
## The next set of rules blocks packets that use fake TCP flags, i.e. H. TCP flags that legitimate packets would not use.
###
```
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,RST FIN,RST -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,ACK FIN -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,URG URG -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,PSH PSH -j DROP
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL NONE -j DROP
```
###

## Block fragmented packets.
```
iptables -t mangle -A PREROUTING -f -j DROP
```
## Limit the number of simultaneous connections per IP address.
```
iptables -A INPUT -p tcp --syn -m connlimit --connlimit-above 80 -j DROP
```
## Prevent UDP flood.
```
iptables -A INPUT -p udp -m limit --limit 150/s -j ACCEPT
iptables -A INPUT -p udp -j DROP
```
## To prevent DNS impersonation attacks (DNS spoofing), you can create specific IPTables rules to protect DNS traffic.
```
iptables -A INPUT -p udp --sport 53 -m conntrack --ctstate ESTABLISHED -j ACCEPT
iptables -A INPUT -p udp --sport 53 -j DROP
```

