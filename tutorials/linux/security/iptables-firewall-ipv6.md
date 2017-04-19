Linux / Security / Basic ipv6 firewall with ip6tables
======

__NOTE: IT'S DANGER ZONE.__ This is expert-level settings setup. You must know what you're doing and do not blindly copy-paste commands and rules described below, otherwise you may end up with unaccessible server

Use ipv6 setting only if your server has assigned IPv6 address. To check assigned IPv6 addresses use: `/sbin/ifconfig | grep inet6`

This tutorial will set ip6tables rules to accept traffic only on `http (80)`, `https (443)`. Which is fine setup for most basic web applications. Using suggested rules you're free to add more udp/tcp ports on demand.

See also great article on ip6tables rules by [linode](https://www.linode.com/docs/security/firewalls/control-network-traffic-with-iptables).

Start with creating blank ip6tables file:
```shell
ip6tables-save > /etc/firewallv6.conf
```

Now edit exported rules:
```shell
# nano /etc/firewallv6.conf
# You should end up with something like:
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
# Allow everything on localhost (loopback)
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

# Allow all outgoing connections
-A OUTPUT -j ACCEPT

# Allow all active incoming connection to continue
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
# Allow all active outgoing connection to continue
-A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
# Drop all INVALID incoming connections
-A INPUT -m conntrack --ctstate INVALID -j DROP

# Main incoming connection rules
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT

# Drop all other incoming connections
-A INPUT -j DROP
# Reject any forwarding
-A FORWARD -j REJECT
COMMIT
```

To test rules run:
```shell
ip6tables-restore < /etc/firewallv6.conf
```

To make created rules persistent, create file `/etc/network/if-up.d/firewall`
```shell
#!/bin/sh
ip6tables-restore < /etc/firewallv6.conf
```

Make it executable:
```shell
chmod +x /etc/network/if-up.d/firewall
```
