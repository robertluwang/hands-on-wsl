
# vagrant for WSL
vagrant: 2.2.3
WSL: win10 wsl Ubuntu-18.04

vagrant for WSL, basic idea is you need to install vagrant inside WSL, since the window vagrant.exe not working well in WSL, main conflict seems from WSL unix path and window path format.
https://www.vagrantup.com/docs/other/wsl.html

This post is about all details when you deal with vagrant for WSL.

## vagrant package installation
Let's download latest deb 64bits from https://www.vagrantup.com/downloads.html, and install it in WSL, 
```
oldhorse@dreamcloud:~$ sudo dpkg -i vagrant_2.2.3_x86_64.deb
[sudo] password for oldhorse:
Selecting previously unselected package vagrant.
(Reading database ... 29292 files and directories currently installed.)
Preparing to unpack vagrant_2.2.3_x86_64.deb ...
Unpacking vagrant (1:2.2.3) ...
Setting up vagrant (1:2.2.3) ...
```
```
oldhorse@dreamcloud:~$ which vagrant
/usr/bin/vagrant
```
## win virtualbox 
WSL is not real Linux, so there is not hypervisior, need to work with virtualbox from window, add below to let vagrant can talk with virtualbox,
```
export VBOX_MSI_INSTALL_PATH=/mnt/c/Program\ Files/Oracle/VirtualBox/
export PATH=$VBOX_MSI_INSTALL_PATH:$PATH
alias VBoxManage=VBoxManage.exe
```
## enable vagrant access to window
By default, WSL vagrant cannot access to window file system, you will get this warning, 
```
oldhorse@dreamcloud:~/vagrant/centos7test$ vagrant up
Vagrant is unable to use the VirtualBox provider from the Windows Subsystem for
Linux without access to the Windows environment. 
```
need to enable vagrant access to window by
```
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
```

### sync folder location 
normally project Vagrantfile is located at home folder which is inside WSL file system, 

C:\Users\win10user\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs\home\

will get this error,
```
* The host path of the shared folder is not supported from WSL. Host
path of the shared folder must be located on a file system with
DrvFs type. Host path: .
```

The WSL /, /root, /home are VolFs, /mnt/c is DrvFs, shared folder has to be located in DrvFs like /mnt/c.

so I moved vagrant project folder to /mnt/c/vagrant, for example /mnt/c/vagrant/centos7dev, this will be sync folder when launch vagrant guest.

## new vagrant home in WSL
The vagrant home by default is user home folder, the downloaded vagrant box will be in this location, 

drwxrwxrwx 0 oldhorse oldhorse   512 Feb  4 18:07 .vagrant.d/
```
oldhorse@dreamcloud:~$ ll .vagrant.d/
total 4
drwxrwxrwx 0 oldhorse oldhorse  512 Feb  4 18:07 ./
drwxr-xr-x 0 oldhorse oldhorse  512 Feb  6 16:55 ../
drwxrwxrwx 0 oldhorse oldhorse  512 Feb  3 14:32 boxes/
drwxrwxrwx 0 oldhorse oldhorse  512 Feb  3 14:32 data/
drwxrwxrwx 0 oldhorse oldhorse  512 Feb  3 14:46 gems/
-rw------- 1 oldhorse oldhorse 1675 Feb  3 14:32 insecure_private_key
drwxrwxrwx 0 oldhorse oldhorse  512 Feb  3 14:32 rgloader/
-rw-rw-rw- 1 oldhorse oldhorse    3 Feb  3 14:32 setup_version
drwxrwxrwx 0 oldhorse oldhorse  512 Feb  3 14:32 tmp/
```

however it is not necessary always be user home folder like /home/oldhorse/.vagrant.d/ in WSL,

There is new variable used to change to another place,
```
export VAGRANT_WSL_WINDOWS_ACCESS_USER_HOME_PATH="/mnt/c/vagrant"
```
after run vagrant next time, it moved to new location,
```
oldhorse@dreamcloud:/mnt/c/vagrant$ ll
total 0
drwxrwxrwx 0 root root 512 Feb  3 18:57 ./
drwxrwxrwx 0 root root 512 Feb  6 16:29 ../
drwxrwxrwx 0 root root 512 Feb  4 18:07 .vagrant.d/
```

### re-launch in new home 

vagrant up
vagrant ssh
```
[vagrant@centos7dev ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:b4:a5:ff brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86328sec preferred_lft 86328sec
    inet6 fe80::a00:27ff:feb4:a5ff/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:63:73:e0 brd ff:ff:ff:ff:ff:ff
    inet 10.120.0.15/24 brd 10.120.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe63:73e0/64 scope link
       valid_lft forever preferred_lft forever

[vagrant@centos7dev ~]$ ping google.ca
PING google.ca (172.217.1.227) 56(84) bytes of data.
64 bytes from lax17s02-in-f3.1e100.net (172.217.1.227): icmp_seq=1 ttl=47 time=45.3 ms
64 bytes from lax17s02-in-f3.1e100.net (172.217.1.227): icmp_seq=2 ttl=47 time=45.3 ms
^C
--- google.ca ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 45.329/45.336/45.343/0.007 ms

[vagrant@centos7dev ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search ca.am.ericsson.se
nameserver 10.0.2.3

[vagrant@centos7dev ~]$ ip r
default via 10.0.2.2 dev enp0s3 proto static metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.120.0.0/24 dev enp0s8 proto kernel scope link src 10.120.0.15 metric 100
```

so far the vagrant in WSL is working perfect as expected.

## run both win vagrant and wsl vagrant
If I want to run vagrant window version at same time on win10, they can share vagrant home with wsl vagrant, it is useful to share same vagrant box for both vagrant.

as above, the vagrant home in WSL:
```
/mnt/c/vagrant/.vagrant.d

oldhorse@dreamcloud:~$ vagrant box list
dreamcloud/centos7   (virtualbox, 2018.01.26)
dreamcloud/centos7.5 (virtualbox, 2018.10.26)
```
I run window vagrant using cmder/msys64, so default VAGRANT_HOME is ~/.vagrant.d, changed it to same folder used by WSL vagrant, 
```
export VAGRANT_HOME=/c/vagrant/.vagrant.d

win10user@dreamcloud MSYS ~/.vagrant.d/boxes
$ vagrant box list
dreamcloud/centos7   (virtualbox, 2018.01.26)
dreamcloud/centos7.5 (virtualbox, 2018.10.26)
```
Even they shared same vagrant home, but they cannot see vagrant vm each other due to user is different.

If want both vagrant shares same vagrant home and see same vm from both vagrant, the user should be same in win and WSL.

The last but not least, the vagrant version should be same in Window and WSL if really want both vagrant working on same vagrant guest.






