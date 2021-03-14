---
title: Environment Setup
layout: post
author: hyprcub
tags: methodology
---
When it comes to play with [Hack The Box](https://www.hackthebox.eu/) or [Try Hack Me](https://tryhackme.com/) I like to have my environment set up conviently.

# Hack The Box

```bash
export PREFIX=~/Documents
export lhost=$(ip address show tun0 | grep "inet " | awk '{print $2}' | cut -d "/" -f 1)
export lport=443
export target=10.129.84.176
export name=luanne
sudo sh -c "sed -i '/$name/d' /etc/hosts"
sudo sh -c "echo $target $name.htb >> /etc/hosts"
mkdir -p $PREFIX/HTB/$name
cd $PREFIX/HTB/$name
```

# Try Hack Me

```bash
export PREFIX=~/Documents
export lhost=$(ip address show tun0 | grep "inet " | awk '{print $2}' | cut -d "/" -f 1)
export lport=443
export target=10.10.15.187
export name=owasp
mkdir -p $PREFIX/THM/$name
cd $PREFIX/THM/$name
```
