# Test queries
## Rate limiting
`repeat 200 dig @34.212.133.226 +short +tries=1 +time=1 p4a.net a`

## Local unbound tests
`dig p4a.net @127.0.0.1 -p 5353`

`dig sigfail.verteiltesysteme.net @127.0.0.1 -p 5353`

`dig sigok.verteiltesysteme.net @127.0.0.1 -p 5353`

# Helpful blog posts
## Bad queries
* https://upload.wikimedia.org/wikipedia/commons/3/37/Netfilter-packet-flow.svg
* https://wiki.opennic.org/opennic/tier2security
* https://forums.centos.org/viewtopic.php?t=62148

## iptables
* https://javapipe.com/blog/iptables-ddos-protection/
* https://www.hostingfuze.net/ddos-protection-with-iptables/
* https://making.pusher.com/per-ip-rate-limiting-with-iptables/
* https://www.quickpacket.com/billing/index.php?rp=/knowledgebase/26/Preventing-Your-DNS-Server-From-Becoming-a-DDoS-Attack-Source.html

# iptables rules
```
sudo iptables -t raw -I PREROUTING -p udp -m string --hex-string "|04|ripe|03|net" --algo bm --to 65535 --dport 53 -j DROP
sudo iptables -t raw -I PREROUTING -p udp -m string --hex-string "|04|ripe|03|net" --algo bm --to 65535 --dport 53 -j LOG --log-prefix "BLOCKED IPv4 ripe.net: "
sudo iptables -t raw -I PREROUTING -p udp -m string --hex-string "|03|isc|03|org" --algo bm --to 65535 --dport 53 -j DROP
sudo iptables -t raw -I PREROUTING -p udp -m string --hex-string "|03|isc|03|org" --algo bm --to 65535 --dport 53 -j LOG --log-prefix "BLOCKED IPv4 isc.org: "
sudo iptables -t raw -I PREROUTING -p udp --dport 53 -m u32 --u32 "0x28=0x0000ff00" -j DROP
sudo iptables -t raw -I PREROUTING -p udp --dport 53 -m u32 --u32 "0x28=0x0000ff00" -j LOG --log-prefix "BLOCKED IPv4 ROOT: "
sudo iptables -t raw -I PREROUTING -p tcp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 52 -j DROP
sudo iptables -t raw -I PREROUTING -p tcp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 52 -j LOG --log-prefix "BLOCKED IPv4 TCP ANY: "
sudo iptables -t raw -I PREROUTING -p udp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 40 -j DROP
sudo iptables -t raw -I PREROUTING -p udp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 40 -j LOG --log-prefix "BLOCKED IPv4 UDP ANY: "

sudo iptables -i ens5 -A INPUT -p udp -m hashlimit --hashlimit-srcmask 32 --hashlimit-mode srcip --hashlimit-above 25/sec --hashlimit-burst 50 --hashlimit-name DNSTHROTTLE --dport 53 -j LOG --log-prefix "IPv4 THROTTLED: "
sudo iptables -i ens5 -A INPUT -p udp -m hashlimit --hashlimit-srcmask 32 --hashlimit-mode srcip --hashlimit-above 25/sec --hashlimit-burst 50 --hashlimit-name DNSTHROTTLE --dport 53 -j DROP
sudo iptables -i ens5 -A INPUT -p udp -m hashlimit --hashlimit-srcmask 32 --hashlimit-mode srcip --hashlimit-upto 25/sec --hashlimit-burst 50 --hashlimit-name DNSTHROTTLE --dport 53 -j ACCEPT

sudo iptables -A OUTPUT -p icmp -m icmp --icmp-type port-unreachable -j LOG --log-prefix "BLOCKED ICMPPortUnreachable: "
sudo iptables -A OUTPUT -p icmp -m icmp --icmp-type port-unreachable -j DROP

sudo iptables-save > iptables_rules

sudo ip6tables -t raw -I PREROUTING -p udp -m string --hex-string "|04|ripe|03|net" --algo bm --to 65535 --dport 53 -j DROP
sudo ip6tables -t raw -I PREROUTING -p udp -m string --hex-string "|04|ripe|03|net" --algo bm --to 65535 --dport 53 -j LOG --log-prefix "BLOCKED IPv6 ripe.net: "
sudo ip6tables -t raw -I PREROUTING -p udp -m string --hex-string "|03|isc|03|org" --algo bm --to 65535 --dport 53 -j DROP
sudo ip6tables -t raw -I PREROUTING -p udp -m string --hex-string "|03|isc|03|org" --algo bm --to 65535 --dport 53 -j LOG --log-prefix "BLOCKED IPv6 isc.org: "
sudo ip6tables -t raw -I PREROUTING -p udp --dport 53 -m u32 --u32 "0x28=0x0000ff00" -j DROP
sudo ip6tables -t raw -I PREROUTING -p udp --dport 53 -m u32 --u32 "0x28=0x0000ff00" -j LOG --log-prefix "BLOCKED IPv6 ROOT: "
sudo ip6tables -t raw -I PREROUTING -p tcp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 52 -j DROP
sudo ip6tables -t raw -I PREROUTING -p tcp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 52 -j LOG --log-prefix "BLOCKED IPv6 TCP ANY: "
sudo ip6tables -t raw -I PREROUTING -p udp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 40 -j DROP
sudo ip6tables -t raw -I PREROUTING -p udp --dport 53 -m string --hex-string "|0000FF0001|" --algo bm --from 40 -j LOG --log-prefix "BLOCKED IPv6 UDP ANY: "

sudo ip6tables -i ens5 -A INPUT -p udp -m hashlimit --hashlimit-srcmask 32 --hashlimit-mode srcip --hashlimit-above 25/sec --hashlimit-burst 50 --hashlimit-name DNSTHROTTLE --dport 53 -j LOG --log-prefix "IPv6 THROTTLED: "
sudo ip6tables -i ens5 -A INPUT -p udp -m hashlimit --hashlimit-srcmask 32 --hashlimit-mode srcip --hashlimit-above 25/sec --hashlimit-burst 50 --hashlimit-name DNSTHROTTLE --dport 53 -j DROP
sudo ip6tables -i ens5 -A INPUT -p udp -m hashlimit --hashlimit-srcmask 32 --hashlimit-mode srcip --hashlimit-upto 25/sec --hashlimit-burst 50 --hashlimit-name DNSTHROTTLE --dport 53 -j ACCEPT

sudo ip6tables-save > ip6tables_rules

sudo apt-get install iptables-persistent
sudo iptables -L --line-num
sudo iptables -L --line-num -t raw
```

## Block a domain
```
sudo iptables -t raw -I PREROUTING -p udp --dport 53 -m string --hex-string "|05|debug|07|opendns|03|com" --algo bm --from 40 -j LOG --log-prefix "BLOCKED debug.opendns.com: "
sudo iptables -t raw -I PREROUTING -p udp --dport 53 -m string --hex-string "|05|debug|07|opendns|03|com" --algo bm --from 40 -j DROP
```

# Unorganized random notes

https://security.stackexchange.com/questions/36128/why-cant-we-block-dns-amplification-attack-by-blocking-udp-packets-or-dns-respo
And for legitimate open resolvers, have them send UDP packets as small as possible with the TC bit set ("please retry using TCP") so that amplification doesn't happen yet it doesn't break legitimate traffic.

Other mitigation techniques for resolvers include enforcing a minimum TTL (which also helps against poisoning) and a reasonable maximum payload size for responses sent using UDP. Google DNS are using a 512 bytes limit for this reason.



https://server-support.co/sysadmin/block-recursive-dns-queries-with-iptables-rate-limiting-rules/



https://www.fortinet.com/blog/threat-research/10-simple-ways-to-mitigate-dns-based-ddos-attacks.html


https://blog.cloudflare.com/a-deep-dive-into-dns-packet-sizes-why-smaller-packet-sizes-keep-the-internet-safe/




IPtables Rules

The following iptables rules should be added where appropriate to your setup. When in doubt, add them to the beginning of the INPUT table, before adding your other firewall rules.

To protect against floods from queries for isc.org:

-p udp -m string --hex-string "|00000000000103697363036f726700|" --algo bm --to 65535 --dport 53 -j DROP

To protect against floods from queries for ripe.net:

-p udp -m string --hex-string "|0000000000010472697065036e6574|" --algo bm --to 65535 --dport 53 -j DROP

To limit ANY queries per IP address, use these two lines:

-p udp --dport 53 -m string --from 50 --algo bm --hex-string '|0000FF0001|' -m recent --set --name dnsanyquery
-p udp --dport 53 -m string --from 50 --algo bm --hex-string '|0000FF0001|' -m recent --name dnsanyquery --rcheck --seconds 60 --hitcount 4 -j DROP

These rules will throttle a connection to 30 queries per minute, allowing for burst traffic of 10 queries (more information can be found at IPTablesRulesToBlockDDOSTraffic):

-p udp -m hashlimit --hashlimit-srcmask 24 --hashlimit-mode srcip --hashlimit-upto 30/m --hashlimit-burst 10 --hashlimit-name DNSTHROTTLE --dport 53 -j ACCEPT
-p udp -m udp --dport 53 -j DROP

Sometimes just replying that you refuse to answer a query can be enough to saturate your connection. The refusal packet can be a few more bytes than the original request and can add up quickly. It has also been observed that bots which abuse your DNS server will only try harder if you send a refusal packet, but if you do not send any reply (as if your service were offline), many bots will go away…

If you are using the whitelisting service, or have a specific list of IP addresses that you will will respond to, all other queries will generate a “refused” notice sent back to the requester. This rule will prevent your DNS server from actually sending the refused notices, thus giving the appearance that your DNS server has gone dead. Again, note that this is only useful for servers that use an ACL to whitelist who can talk to them!

-A OUTPUT -p udp --source-port 53 -m string --algo kmp --from 30 --to 31 --hex-string "|8105|" -j DROP

If you have a new situation and cannot stop the incoming traffic, iptables string-match rules can be used as a last resort. Keep in mind that string matching is resource-intensive and may slow down your network speed, however if you need to shut out an attack, this option is better than letting your server continue to answer the abusive queries.

First, you need to get a hex-dump of the packets you wish to block. For this, you will use tcpdump:

# tcpdump -c10 -pntxi eth0 not udp src port 53 and udp dst port 53

The option -c10 specifies that you only want to dump out 10 packets at a time. The portion after the interface specifies that we only want to look at incoming packets on UDP port 53, and ignore outgoing packets.

If your server is being attacked, you will probably see several instances of the particular query within those 10 packets. For example, while being flooded with ANY queries for the root zone, I captured the following:

IP 69.9.100.242.58100 > 216.87.84.211.53: 43581+ [1au] ANY? . (28)
        0x0000:  4500 0038 a48c 0000 f111 4e02 4509 64f2
        0x0010:  d857 54d3 e2f4 0035 0024 0000 aa3d 0100
        0x0020:  0001 0000 0000 0001 0000 ff00 0100 0029
        0x0030:  2328 0000 0000 0000
IP 99.108.65.243.18176 > 216.87.84.211.53: 40843+ [1au] ANY? . (28)
        0x0000:  4500 0038 d50a 0000 f111 2220 636c 41f3
        0x0010:  d857 54d3 4700 0035 0024 0000 9f8b 0100
        0x0020:  0001 0000 0000 0001 0000 ff00 0100 0029
        0x0030:  2328 0000 0000 0000

You will probably see examples from several different IP addresses. What you are looking for is a block of consecutive bytes that are identical between all queries. In my case, I am going to choose this section near the end:

0000 0001 0000 ff00 0100 0029 2328 0000

For the iptables rule, you want to remove all the spaces between bytes, and drop the results into a rule like this:

# iptables -I INPUT -p udp -m string --hex-string "|000000010000ff000100002923280000|" --algo bm --dport 53 -j DROP

Running this rule will immediately shut out all matching queries, preventing your server from wasting time and bandwidth trying to answer them.


