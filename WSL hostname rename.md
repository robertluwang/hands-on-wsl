# WSL hostname rename

WSL hostname controlled by WSL, it is useless to manually change hostname by 
1) update /etc/hostname
2) update /etc/hosts

The correct way to update WSL hostname like below:
```
/etc/wsl.conf
[network]
hostname = wsl2
generateHosts = false
```

after WSL restart
```
PS > wsl -t Ubuntu-22.04
PS > wsl -d Ubuntu-22.04
```
will see /etc/hostname and prompt updated to wsl2, 
```
oldhorse@wsl2:~$ cat /etc/hostname
wsl2
```
However you still cannot ping host alias name wsl2,
you need to manually update in /etc/hosts
```
oldhorse@wsl2:~$ cat /etc/hosts|grep 127.0.1.1
127.0.1.1       ABC.localdomain        ABC
```
to 
```
127.0.1.1      wsl2
```
it works now, 
```
oldhorse@wsl2:~$ ping wsl2
PING wsl2 (127.0.1.1) 56(84) bytes of data.
64 bytes from wsl2 (127.0.1.1): icmp_seq=1 ttl=64 time=0.025 ms
```

