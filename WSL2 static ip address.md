# WSL2 static ip address
## why need WSL2 static ip?
Few reasons to keep static ip for wsl instance:
- by default wsl2 eth0 ip changed after wsl instance restart 
- some app not works for dynamic ip like k8s cluster
- it is same ip for multi wsl instance 

## how to make static ip for WSL2
Here is how to change WSL ip to static.

current WSL on host pc,
```
ipconfig

Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::a58d:462c:523f:6517%77
   IPv4 Address. . . . . . . . . . . : 172.26.128.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . : 
```
Would like to change WSL ip to 192.168.80.1, and wsl instance1 to 192.168.80.2, wsl instance2 to 192.168.80.3.

## wsl2ip script 
I made this handy [batch job](https://github.com/robertluwang/wslnote/blob/master/src/wsl2ip.bat), run CMD as admin, then run it.
```
date /t
time /t
 
wsl -d Ubuntu -u root ip addr del $(ip addr show eth0 ^| grep 'inet\b' ^| awk '{print $2}' ^| head -n 1) dev eth0
wsl -d Ubuntu -u root ip addr add 192.168.80.10/24 broadcast 192.168.80.255 dev eth0
wsl -d Ubuntu -u root ip route add 0.0.0.0/0 via 192.168.80.1 dev eth0
wsl -d Ubuntu -u root echo nameserver 8.8.8.8 ^> /etc/resolv.conf

powershell -c "Get-NetAdapter 'vEthernet (WSL)' | Get-NetIPAddress | Remove-NetIPAddress -Confirm:$False; New-NetIPAddress -IPAddress 192.168.80.1 -PrefixLength 24 -InterfaceAlias 'vEthernet (WSL)'; Get-NetNat | ? Name -Eq WSLNat | Remove-NetNat -Confirm:$False; New-NetNat -Name WSLNat -InternalIPInterfaceAddressPrefix 192.168.80.0/24;"
 
date /t
time /t
```
this will update WSL NAT adaptor on host pc and NAT NIC eth0 static ip, also please to make sure wsl instance is running, o/w cannot change NIC ip.
```
C:\tools\wsl>wsl2ip.bat
```
## wsl2ip script result 
We can see WSL on PC changed, 
```
C:\tools\wsl>ipconfig
Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.80.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```
Let's have a look at wsl instance1,
```
oldhorse@wsl:~$ ip addr show eth0 
4: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:6f:8c:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.80.10/24 brd 192.168.80.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe6f:8c94/64 scope link
       valid_lft forever preferred_lft forever
oldhorse@wsl:~$ cat /etc/resolv.conf 
nameserver 8.8.8.8
oldhorse@wsl:~$ ping google.ca
PING google.ca (172.217.13.163) 56(84) bytes of data.
64 bytes from yul03s04-in-f3.1e100.net (172.217.13.163): icmp_seq=1 ttl=105 time=29.6 ms
```
wsl instance2,
```
oldhorse@wsl2:~$ ip addr show eth0 
4: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:6f:8c:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.80.10/24 brd 192.168.80.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe6f:8c94/64 scope link
       valid_lft forever preferred_lft forever
oldhorse@wsl2:~$ cat /etc/resolv.conf
nameserver 8.8.8.8
oldhorse@wsl2:~$ ping google.ca
PING google.ca (172.217.13.163) 56(84) bytes of data.
64 bytes from yul03s04-in-f3.1e100.net (172.217.13.163): icmp_seq=1 ttl=105 time=31.5 ms
```
## WSL2 network limitation 
both are working and access to Internet, however they have same ip 192.168.80.10 for eth0.

The reason is not just WSL adaptor is NAT adaptor, but also in fact that all WSL instances running in one hyper-v based WSL VM, they share same network namespace, NIC, there seems not way to have own static ip for each WSL instance.



