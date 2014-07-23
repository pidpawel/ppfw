PPFW
====

Description
-----------
PPFW is a firewall solution for Linux.
It is designed to fit modern desktop or server's requirements.
PPFW _does_ support both **IPv4** and **IPv6**.
PPFW is also highly configurable through `/etc/ppfw.conf`.
PPFW has never been designed to be suitable for routers.

All sections are optional.
Default configuration _should_ be suitable for most desktops but **you** take the full reponsibility for **any** firewall's action.
This software is provided **as is** under the MIT-like license.

**Remember:** This software is still in development and production usage should be _heavily_ monitored.

**Notice:** This documentation is not a priority and is may not fit the code.

Requirements
------------
This list will be updated in future.
* Kernel NetFilter support
* iptables, ip6tables accordingly
* sysctl (may be disabled)
* Python 3 (Compatibility with Python 2 has been broken)
* Docopt

Installation
------------
- Copy `ppfw.conf.example` to `/etc/ppfw.conf`.
- Move to _Configuration_ section.
- Read _Usage_ section

Configuration
-------------
- Edit `/etc/ppfw.conf` to fit your needs.

Notes on configuration file
---------------------------
All sections in configuration file are optional. PPFW **should** work properly with no configuration file (it will use default rules) but this configuration is not highly usable.

### Desktop configuration
* Policies should be set to: input = deny, output = accept, forward = deny, ping = accept
* In most cases no further configuration is needed.
* To enable SSHd access from external hosts use add `ssh` under [services] section.

Example config:

```
[policies]
input = deny
output = accept
forward = deny
ping = accept
[services]
ssh
```

### Server configuration
If it comes to server configurations situation is more complicated.
First of all you have to think what do you **really** need and allow this.
All policies should be set to deny, and all rules should be "whitelisted".

Notes on [rules] section
------------------------
Please refer to `ppfw.conf.example` for detailed description.

**Warning:** in version 0.2 there is no support for [rules] section. This is (hopefully) temporary.

PPFW's rules are meant to make firewall's configuration more human readable and shorten firewall's deploy time.
They do not cover whole iptables functionality or even fraction of it, but most common usages should be covered.

Usage/Maintenance
-----------------
1. Run `ppfw --pretend --verbosity=4 start` to check whether what you specified was properly interpreted. (You need to understand iptables and „sysfs” rules)
2. Before proceeding make sure that you **have** KVM or **physical access** to your host unless you'll **probably get into trouble** if something goes wrong. This software **can shut down all the network communication**.
3. Get root if not already. (Or use sudo.)
4. If everything **is okay** run `ppfw start` (PPFW _does_ understand init.d script-like arguments - start/stop/restart)
5. If ppfw is not in `/etc/init.d` already, copy it there.
6. Add ppfw to your init table.

Practical thoughts
------------------
* Make sure that nobody can modify ppfw file. I am pretty sure that this file can be used to execute scripts under root privileges ([path] section).
* Only user that has to be able to read `/etc/ppfw.conf` is root, so chmod it to 600, and even ACL.

Example production config and log
---------------------------------
```
[core]
use_v4 = yes
use_v6 = yes
verbosity = 4

[policies]
input = reject
output = accept
forward = drop
ping = accept

[whitelist]
8.8.8.8
2001:f1d0:1:1afg::1

[blacklist]
101.98.14.137
bad.drone.eu

[services]
ssh
https
12345
```

```
> Starting.
> Pretend mode is ON. Commands will not be executed despite logging so.
>> Cleaning firewall…
>> Setting new policies…
>> Dropping invalid packets.
>> Setting loopback policy…
>> Blacklist.
>> Accepting established and related packets.
>> Whitelist.
>> Adding ICMP rules…
>> Public services.
>> Applying sysctl rules…
>>> Executing /sbin/sysctl -q -w net.ipv4.ip_forward=0
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.accept_redirects=0
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.proxy_arp=0
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.send_redirects=0
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.accept_source_route=0
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.secure_redirects=0
>>> Executing /sbin/sysctl -q -w net.ipv4.icmp_ignore_bogus_error_responses=1
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.rp_filter=1
>>> Executing /sbin/sysctl -q -w net.ipv4.tcp_syncookies=1
>>> Executing /sbin/sysctl -q -w net.ipv4.icmp_echo_ignore_broadcasts=1
>>> Executing /sbin/sysctl -q -w net.ipv4.conf.all.log_martians=1
>> Loading iptables rules…
>>>> Section cleanup
>>> Executing /sbin/iptables -F
>>> Executing /sbin/iptables -X
>>> Executing /sbin/iptables -Z
>>>> Section policies
>>> Executing /sbin/iptables -P INPUT DROP
>>> Executing /sbin/iptables -P OUTPUT ACCEPT
>>> Executing /sbin/iptables -P FORWARD DROP
>>>> Section invalid
>>> Executing /sbin/iptables -A INPUT -m state --state INVALID -j DROP
>>>> Section loopback
>>> Executing /sbin/iptables -A INPUT -i lo -j ACCEPT
>>> Executing /sbin/iptables -A OUTPUT -o lo -j ACCEPT
>>>> Section blacklist
>>> Executing /sbin/iptables -A INPUT -s bad.drone.eu -j DROP
>>> Executing /sbin/iptables -A INPUT -s 101.98.14.137 -j DROP
>>>> Section stateful
>>> Executing /sbin/iptables -A INPUT -m state --state ESTABLISHED -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -m state --state RELATED -j ACCEPT
>>>> Section whitelist
>>> Executing /sbin/iptables -A INPUT -s 2001:f1d0:1:1afg::1 -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -s 8.8.8.8 -j ACCEPT
>>>> Section icmp
>>> Executing /sbin/iptables -A INPUT -p icmp -j ACCEPT
>>>> Section services
>>> Executing /sbin/iptables -A INPUT -p tcp --dport 443 -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -p udp --dport 443 -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -p tcp --dport 22 -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -p udp --dport 22 -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -p tcp --dport 12345 -j ACCEPT
>>> Executing /sbin/iptables -A INPUT -p udp --dport 12345 -j ACCEPT
>>>> Section finish
>>> Executing /sbin/iptables -A INPUT -p tcp -j REJECT --reject-with tcp-reset
>>> Executing /sbin/iptables -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
>> Loading ip6tables rules…
>>>> Section cleanup
>>> Executing /sbin/ip6tables -F
>>> Executing /sbin/ip6tables -X
>>> Executing /sbin/ip6tables -Z
>>>> Section policies
>>> Executing /sbin/ip6tables -P INPUT DROP
>>> Executing /sbin/ip6tables -P OUTPUT ACCEPT
>>> Executing /sbin/ip6tables -P FORWARD DROP
>>>> Section invalid
>>> Executing /sbin/ip6tables -A INPUT -m state --state INVALID -j DROP
>>>> Section loopback
>>> Executing /sbin/ip6tables -A INPUT -i lo -j ACCEPT
>>> Executing /sbin/ip6tables -A OUTPUT -o lo -j ACCEPT
>>>> Section blacklist
>>> Executing /sbin/ip6tables -A INPUT -s bad.drone.eu -j DROP
>>>> Section stateful
>>> Executing /sbin/ip6tables -A INPUT -m state --state ESTABLISHED -j ACCEPT
>>> Executing /sbin/ip6tables -A INPUT -m state --state RELATED -j ACCEPT
>>>> Section whitelist
>>> Executing /sbin/ip6tables -A INPUT -s 2001:f1d0:1:1afg::1 -j ACCEPT
>>>> Section icmp
>>> Executing /sbin/ip6tables -A INPUT -p ipv6-icmp -j ACCEPT
>>>> Section services
>>> Executing /sbin/ip6tables -A INPUT -p tcp --dport 443 -j ACCEPT
>>> Executing /sbin/ip6tables -A INPUT -p udp --dport 443 -j ACCEPT
>>> Executing /sbin/ip6tables -A INPUT -p tcp --dport 22 -j ACCEPT
>>> Executing /sbin/ip6tables -A INPUT -p udp --dport 22 -j ACCEPT
>>> Executing /sbin/ip6tables -A INPUT -p tcp --dport 12345 -j ACCEPT
>>> Executing /sbin/ip6tables -A INPUT -p udp --dport 12345 -j ACCEPT
>>>> Section finish
>>> Executing /sbin/ip6tables -A INPUT -p tcp -j REJECT --reject-with tcp-reset
>>> Executing /sbin/ip6tables -A INPUT -p udp -j REJECT --reject-with icmp6-port-unreachable
> Done.
```