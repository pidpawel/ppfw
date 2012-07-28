PPFW ReadMePl0x
===============

Description
-----------
PPFW is a firewall solution for Linux. It is designed to fit modern desktop or server's requirements. PPFW _does_ support both **IPv4** and **IPv6**. PPFW is also highly configurable through `/etc/ppfw.conf`. All sections are optional. Default configuration _should_ be suitable for most desktops but **you** take the full reponsibility for **any** firewalls' action. This software is provided **as is** under the MIT-like license.

This software is **not** meant to be _routers'_ firewall script.

**Remember:** This software is still in development and production usage should be _heavily_ monitored.

**Notice:** This documentation is not a priority and is may not fit the code.

Requirements
------------
This list will be updated in future.
* Kernel NetFilter support
* iptables
* sysctl
* Python (This software was tested on Python 3, but _should_ work on Python 2)
* Python's configparser

Installation
------------
- Copy `ppfw.conf.example` to `/etc/ppfw.conf`.
- Move to _Configuration_ section.

Configuration
-------------
- Edit `/etc/ppfw.conf` to fit your needs.

Notes on configuration file
---------------------------
All sections in configuration file are optional. PPFW **should** work properly with no configuration file (it will use default rules) but this configuration is not highly usable.

### Desktop configuration
* Policies should be set to: input = deny, output = accept, forward = deny
* In most cases no further configuration is needed.
* To enable SSHd access from external hosts use _allow tcp from any to port ssh_ rule.
Example config:

```
[policies]
input = deny
output = accept
forward = deny
[rules]
allow tcp from any to port ssh # Allow SSHd access from any host
```

### Server configuration
If it comes to server configurations situation is more complicated. First of all you have to think what do you **really** need and allow this. All policies should be set to deny, and all rules should be „whitelisted”.

Notes on [rules] section
------------------------
Please refer to `ppfw.conf.example` for detailed description.

PPFW's rules are meant to make firewalls' configurations more human readable and shorten firewalls' deploy time. They do not cover whole iptables functionality or even 1% of it, but most common usages should be covered.

Usage/Maintenance
-----------------
1. Run `ppfw test --verbosity=4` to check whether what you specified was properly interpreted. (You need to understand iptables and „sysfs” rules)
2. Before proceeding make sure that you **have** KVM or **physical access** to your host unless you'll **probably get into trouble** if something goes wrong. This software **can shut down all the network communication**.
3. Get root if not already.
4. If everything **is okay** run `ppfw start` (PPFW _does_ understand init.d script-like arguments)
5. If ppfw is not in `/etc/init.d` already, copy it there.
6. Add ppfw to your init table.
7. If you want to update your RBL rules frequently add suitable cron rules pointing to `ppfw reload` or `/etc/init.d/ppfw reload`. Warning: during the update process host **will** be **unprotected** by those RBL rules. This process also re-downloads DNS record for hostnames' bans and exceptions.

Practical thoughts
------------------
* Make sure that nobody can modify ppfw file.
* Only user that has to be able to read `/etc/ppfw.conf` is root, so chmod it to 600, and even ACL.

