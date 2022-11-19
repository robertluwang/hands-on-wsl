# WSL2 docker 
## enable systemd 
By default systemd is not enabled, follow up remedy in [WSL2 systemd](./WSL2%20systemd.md).
verify systemd ready,
```
which systemctl 
systemctl list-units --type=service
```
## install docker 
Just follow up ubuntu docker installation guide [here](https://docs.docker.com/engine/install/ubuntu). 
I made this handy script to install docker on ubuntu in one shot.
[docker-install.sh](https://github.com/robertluwang/miniguide-nativecloud/blob/main/k8s%20script/docker-install.sh)
## verify docker
Restart wsl instance, then check,
```
docker --version 
Docker version 20.10.21, build baeda1f
docker info
docker ps 
```
## docker networking
```
oldhorse@wsl:~$ ip addr show docker0
7: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:d8:fa:e9:51 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
oldhorse@wsl:~$ ip route
default via 172.26.128.1 dev eth0 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.26.128.0/20 dev eth0 proto kernel scope link src 172.26.140.199
```
## docker service failure
Got error to start docker on WSL Ubuntu 22.04, 
```
oldhorse@wsl2:~$ systemctl status docker.service
Failed to dump process list for 'docker.service', ignoring: Input/output error
× docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Sat 2022-11-19 12:59:09 EST; 33s ago
TriggeredBy: × docker.socket
       Docs: https://docs.docker.com
    Process: 476 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
   Main PID: 476 (code=exited, status=1/FAILURE)
      Tasks: 14
     Memory: 157.3M
     CGroup: /system.slice/docker.service
```
Found this [ref post](https://crapts.org/2022/05/15/install-docker-in-wsl2-with-ubuntu-22-04-lts/#:~:text=You%20need%20to%20switch%20to,Docker%20will%20start%20as%20expected!) , 22.04 by default uses iptables-nft instead of iptables-legacy, as remedy switch to iptables-legacy.
```
oldhorse@wsl2:~$ sudo update-alternatives --config iptables
[sudo] password for oldhorse: 
There are 2 choices for the alternative iptables (providing /usr/sbin/iptables).

  Selection    Path                       Priority   Status
------------------------------------------------------------
* 0            /usr/sbin/iptables-nft      20        auto mode
  1            /usr/sbin/iptables-legacy   10        manual mode
  2            /usr/sbin/iptables-nft      20        manual mode
```
after changed to 1, then restart docker smoothly, 
```
systemctl restart docker
```

