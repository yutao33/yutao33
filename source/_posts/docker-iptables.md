---
title: docker block ip address
date: 2019-06-17 13:54:00
mathjax: true
tags: [docker, iptable, ipset]
---

```sh
# Create the ipset list
ipset -N china hash:net

# Pull the latest IP set for China
wget -P . http://www.ipdeny.com/ipblocks/data/countries/cn.zone

# Add each IP address from the downloaded list into the ipset 'china'
for i in $(cat ./cn.zone ); do ipset -A china $i; done

# Restore iptables
/sbin/iptables-restore < /etc/iptables.firewall.rules



iptables -I DOCKER-USER -m set --match-set china dst -s 172.17.0.2 -j RETURN
iptables -I DOCKER-USER -s 172.17.0.2 -j DROP
iptables -I DOCKER-USER -s 172.18.0.2 -j REJECT
iptables -I DOCKER-USER -m set --match-set us dst -s 172.18.0.2 -j REJECT
```
