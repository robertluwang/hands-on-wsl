# How WSL seamless integrating with win10 docker ecosystem
WSL: win10 wsl Ubuntu-18.04

In general the docker is not running inside WSL but it is possible to run docker client in WSL, and seamless integrating with win10 docker ecosystem.

## docker install in WSL
```
oldhorse@dreamcloud:~$ sudo apt-get update -y
oldhorse@dreamcloud:~$ sudo apt-get install -y \
>     apt-transport-https \
>     ca-certificates \
>     curl \
>     software-properties-common

oldhorse@dreamcloud:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK

oldhorse@dreamcloud:~$ sudo apt-key fingerprint 0EBFCD88

oldhorse@dreamcloud:~$ sudo add-apt-repository \
>    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
>    $(lsb_release -cs) \
>    stable"
```
add user to docker group, then can run docker as own user instead of root later.
```
oldhorse@dreamcloud:~$ sudo usermod -aG docker $USER
```
## install docker-compose as local
```
oldhorse@dreamcloud:~$ pip install --user docker-compose
oldhorse@dreamcloud:~$ ll .local/bin/|grep docker
-rwxrwxr-x 1 oldhorse oldhorse   218 Feb  7 08:59 docker-compose*
```
when run docker-compose, cannot find due to local installation not in PATH, can add it to local profile ~/.bashrc
```
PATH=$HOME/.local/bin:$PATH
```
verify installation by
```
oldhorse@dreamcloud:~$ docker-compose --version
docker-compose version 1.23.2, build 1110ad0
```
The docker-compose can be ran as python package locally or as window binary.

## run dockerd in wsl 
We cannot run dockerd in WSL since it is not real Linux, there is not Linux kernel there, let's have a try.
```
oldhorse@dreamcloud:~$ sudo dockerd
WARN[2019-02-02T13:56:26.228797300-05:00] could not use snapshotter btrfs in metadata plugin  error="path /var/lib/docker/containerd/daemon/io.containerd.snapshotter.v1.btrfs must be a btrfs filesystem to be used with the btrfs snapshotter"
WARN[2019-02-02T13:56:26.228813700-05:00] could not use snapshotter aufs in metadata plugin  error="modprobe aufs failed: "modprobe: ERROR: ../libkmod/libkmod.c:586 kmod_search_moddep() could not open moddep file '/lib/modules/4.4.0-43-Microsoft/modules.dep.bin'\nmodprobe: FATAL: Module aufs not found in directory /lib/modules/4.4.0-43-Microsoft\n": exit status 1"
INFO[2019-02-02T13:56:26.254870700-05:00] containerd successfully booted in 0.107059s
WARN[2019-02-02T13:56:26.374695100-05:00] Your kernel does not support cgroup memory limit
Error starting daemon: Devices cgroup isn't mounted
```
so it is impossible to run dockerd daemon in WSL, need to run docker server in docker machine node on win10.

## portable docker on window
There are few options to setup docker running on win10:
- Docker Desktop for Windows(https://docs.docker.com/docker-for-windows/install/), it needs Hyper-V enabled but will disable virtualbox, so not my option
- docker toolbox(https://docs.docker.com/toolbox/overview/), which still use virtualbox
- portable docker toolbox, means simply extract or download docker tools binary and place to one folder.
```
oldhorse@dreamcloud:/mnt/c/portableapps$ ll dockertoolbox/
total 144108
drwxrwxrwx 0 root root      512 Feb  3 10:54 ./
drwxrwxrwx 0 root root      512 Feb  6 19:23 ../
-rwxrwxrwx 1 root root  6960939 Oct  8  2017 docker-compose.exe*
-rwxrwxrwx 1 root root 26885120 Oct  8  2017 docker-machine.exe*
-rwxrwxrwx 1 root root 19532800 Oct  8  2017 docker.exe*
-rwxrwxrwx 1 root root 52721152 Dec 10  2017 kubectl.exe*
-rwxrwxrwx 1 root root 41459200 Dec  9  2017 minikube.exe*
```
The all toolbox written by Go, so they are highly portable, can run on win10 or inside WSL.

## docker server on win10
There is not docker daemon running on win10, will use docker-machine.exe to launch/stop docker machine node, the behind it is fact the tiny vm running in virtualbox.

I used short name for these most frequently operations:
```
wsl
alias dc='/mnt/c/portableapps/dockertoolbox/docker-compose.exe'
alias dm='/mnt/c/portableapps/dockertoolbox/docker-machine.exe'

win msys 
alias dc='/c/portableapps/dockertoolbox/docker-compose.exe'
alias dm='/c/portableapps/dockertoolbox/docker-machine.exe'
```
here is sample to create new docker machine in win10 msys64,
```
$ dm create demo
Running pre-create checks...
(demo) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(demo) Latest release for github.com/boot2docker/boot2docker is v18.09.1
(demo) Downloading C:\portableapps\msys64\home\win10user\.docker\machine\cache\boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.09.1/boot2docker.iso...
(demo) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(demo) Copying C:\portableapps\msys64\home\win10user\.docker\machine\cache\boot2docker.iso to C:\portableapps\msys64\home\win10user\.docker\machine\machines\demo\boot2docker.iso...
(demo) Creating VirtualBox VM...
(demo) Creating SSH key...
(demo) Starting the VM...
(demo) Check network to re-create if needed...
(demo) Windows might ask for the permission to create a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(demo) Found a new host-only adapter: "VirtualBox Host-Only Ethernet Adapter #3"
(demo) Windows might ask for the permission to configure a network adapter. Sometimes, such confirmation window is minimized in the taskbar.
(demo) Windows might ask for the permission to configure a dhcp server. Sometimes, such confirmation window is minimized in the taskbar.
(demo) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: C:\portableapps\dockertoolbox\docker-machine.exe env demo
```
when run docker client, will get error, which means docker env not ready yet.
```
$ docker ps
error during connect: Get http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.31/containers/json: open //./pipe/docker_engine: The system cannot find the file specified. In the default daemon configuration on Windows, the docker client must be run elevated to connect. This error may also indicate that the docker daemon is not running.
```
remedy as below,
```
eval $(dm env demo --shell bash) 
```
then docker is running well, 
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ docker run hello-world

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
f5ffcdd8586a        hello-world         "/hello"            6 seconds ago       Exited (0) 4 seconds ago                       agitated_mclaren

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        4 weeks ago         1.84kB

$ dm status demo
Running
```
Let's ssh to docker machine demo to have deep look, 
```
$ dm ssh demo
   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net

docker@demo:~$

docker@demo:~$ df -h
Filesystem                Size      Used Available Use% Mounted on
tmpfs                   890.4M    229.1M    661.3M  26% /
tmpfs                   494.7M         0    494.7M   0% /dev/shm
/dev/sda1                17.8G     46.0M     16.9G   0% /mnt/sda1
cgroup                  494.7M         0    494.7M   0% /sys/fs/cgroup
/c/Users                476.0G    421.1G     54.9G  88% /c/Users
/dev/sda1                17.8G     46.0M     16.9G   0% /mnt/sda1/var/lib/docker

docker@demo:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ec:a0:19 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feec:a019/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:42:6c:e7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.99.100/24 brd 192.168.99.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe42:6ce7/64 scope link
       valid_lft forever preferred_lft forever
4: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:be:dd:bb:c1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

docker@demo:~$ uname -a
Linux demo 4.14.92-boot2docker #1 SMP Wed Jan 9 22:03:23 UTC 2019 x86_64 GNU/Linux

docker@demo:~$ ping google.ca
PING google.ca (172.217.1.227): 56 data bytes
64 bytes from 172.217.1.227: seq=0 ttl=47 time=43.959 ms
64 bytes from 172.217.1.227: seq=1 ttl=47 time=44.437 ms
♥
--- google.ca ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 43.959/44.198/44.437 ms
docker@demo:~$
docker@demo:~$ ps -ef|grep docker
root      1960     1  0 19:05 ?        00:00:00 /sbin/udhcpc -b -i eth0 -x hostname:boot2docker -p /var/run/udhcpc.eth0.pid
root      1970     1  0 19:05 ?        00:00:00 /sbin/udhcpc -b -i eth1 -x hostname:boot2docker -p /var/run/udhcpc.eth1.pid
root      2252     1  0 19:05 ?        00:00:00 crond -L /var/lib/boot2docker/log/crond.log
docker    2360  2356  0 19:05 tty1     00:00:00 -bash
root      2621     1  0 19:06 ?        00:00:00 dockerd --data-root /var/lib/docker -H unix:// --label provider=virtualbox -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/var/lib/boot2docker/ca.pem --tlskey=/var/lib/boot2docker/server-key.pem --tlscert=/var/lib/boot2docker/server.pem --pidfile /var/run/docker.pid
root      2626  2621  0 19:06 ?        00:00:01 containerd --config /var/run/docker/containerd/containerd.toml --log-level info
root      2769  2304  0 19:07 ?        00:00:00 sshd: docker [priv]
docker    2771  2769  0 19:07 ?        00:00:00 sshd: docker@pts/0
docker    2772  2771  0 19:07 pts/0    00:00:00 -bash
docker    2794  2772  0 19:09 pts/0    00:00:00 ps -ef
docker    2795  2772  0 19:09 pts/0    00:00:00 grep docker
```

## run win docker machine in WSL 
window docker machine is running smoothly in WSL, the default docker home is C:\Users\win10user\.docker, 
```
oldhorse@dreamcloud:~$ dm create new
Creating CA: C:\Users\win10user\.docker\machine\certs\ca.pem
Creating client certificate: C:\Users\win10user\.docker\machine\certs\cert.pem
Running pre-create checks...
(new) Image cache directory does not exist, creating it at C:\Users\win10user\.docker\machine\cache...
(new) No default Boot2Docker ISO found locally, downloading the latest release...
(new) Latest release for github.com/boot2docker/boot2docker is v18.09.1
(new) Downloading C:\Users\win10user\.docker\machine\cache\boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.09.1/boot2docker.iso...
(new) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(new) Copying C:\Users\win10user\.docker\machine\cache\boot2docker.iso to C:\Users\win10user\.docker\machine\machines\new\boot2docker.iso...
(new) Creating VirtualBox VM...
(new) Creating SSH key...
(new) Starting the VM...
(new) Check network to re-create if needed...
(new) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: C:\portableapps\dockertoolbox\docker-machine.exe env new

oldhorse@dreamcloud:~$ dm status new
Running
```
docker env setup is trick on WSL, 
```
oldhorse@dreamcloud:~$ docker-machine.exe env new
You can further specify your shell with either 'cmd' or 'powershell' with the --shell flag.

SET DOCKER_TLS_VERIFY=1
SET DOCKER_HOST=tcp://192.168.99.101:2376
SET DOCKER_CERT_PATH=C:\Users\erobwan\.docker\machine\machines\new
SET DOCKER_MACHINE_NAME=new
SET COMPOSE_CONVERT_WINDOWS_PATHS=true
REM Run this command to configure your shell:
REM     @FOR /f "tokens=*" %i IN ('"C:\portableapps\dockertoolbox\docker-machine.exe" env new') DO @%i
```
the DOCKER_CERT_PATH is wrong for WSL, which should be 
```
SET DOCKER_CERT_PATH=/mnt/c/Users/win10user/.docker/machine/machines/new
```
here is correct remedy,
```
eval $(dm env new --shell bash |sed -e 's|\\|/|g' -e 's|C:/|/mnt/c/|g')
```
## run WSL native docker client 
Keep in the win docker client is not working well on WSL, that is why we installed native docker client before.
```
oldhorse@dreamcloud:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

oldhorse@dreamcloud:~$ dm ssh new
   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net
```

## summary 
- need both docker installed in win and WSL
- docker-machine.exe to create/stop/remove/ssh docker machine node on top of virtualbox and run inside WSL directly
- use WSL native docker cli to handle docker container 

