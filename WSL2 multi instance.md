# WSL2 multi instance of Ubuntu
## WSL2 installation limitation
You can install standard instance from available list, 
```
PS > wsl -l -o 
The following is a list of valid distributions that can be installed.
Install using 'wsl --install -d <Distro>'.

NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
Ubuntu-16.04    Ubuntu 16.04 LTS
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS
```
You do install multi instance for different distro on same win10 but you cannot install multi instance for same type of distro like Ubuntu, when you install 2nd Ubuntu instance with different version, only can access to first wsl instance.
```
wsl --install -d <distro_name>
```
## WSL import instance
In this case, you will need to download or build wsl cloud image and import to WSL manually.

Let us test on latest LTS 22.04 as 2nd instance on win10.
```
wsl --import <Distribution Name> <Installation Folder> <Ubuntu WSL2 Image Tarball path>
```
## verify WSL default version
Before go head to import WSL, quick verify default version, 
```
PS > Get-ItemPropertyValue `
>>       -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Lxss `
>>       -Name DefaultVersion
2
```
if it is version 1, change to 2,
```
PS > wsl --set-default-version 2
F o r   i n f o r m a t i o n   o n   k e y   d i f f e r e n c e s   w i t h   W S L   2   p l e a s e   v i s i t   h t t p s : / 
/ a k a . m s / w s l 2 
 
T h e   o p e r a t i o n   c o m p l e t e d   s u c c e s s f u l l y . 
```
## download WSL Ubuntu 22.04 cloud image 
Here is latest download link for 22.04 wsl image, 
https://cloud-images.ubuntu.com/wsl/jammy/current/ubuntu-jammy-wsl-arm64-wsl.rootfs.tar.gz

## install 2nd Ubuntu on WSL
Will create new wsl instance name Ubuntu-22.04, located at folder "C:\tools\wsl\Ubuntu-22.04",
```
PS > dir "C:\tools\wsl\Ubuntu-22.04" // before wsl import

PS > wsl --import Ubuntu-22.04 "C:\tools\wsl\Ubuntu-22.04"  C:\tools\wsl\ubuntu-jammy-wsl-amd64-wsl.rootfs.tar.gz

PS > dir "C:\tools\wsl\Ubuntu-22.04" // after wsl import
    Directory: C:\tools\wsl\Ubuntu-22.04
Mode                 LastWriteTime         Length Name                                                                             
----                 -------------         ------ ----                                                                             
-a----        2022-11-18   7:40 AM     1106247680 ext4.vhdx     
PS > wsl -l -v
  NAME            STATE           VERSION
* Ubuntu          Running         1
  Ubuntu-22.04    Stopped         2
```
verify login,
```
PS > wsl -d Ubuntu-22.04
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.10.102.1-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This message is shown once a day. To disable it please create the
/root/.hushlogin file.
root:/mnt/c/Users/oldhorse#
```
## add sudo user
By default it is root when login using wsl, better to add new sudo user for daily work, 
```
root# useradd -m -G sudo -s /bin/bash oldhorse
root# passwd oldhorse
```
We want to change sudo user as default user to login 22.04 Ubuntu, need to add default user to wsl /etc/wsl.conf.
```
root# tee /etc/wsl.conf << EOF
[user]
default=oldhorse
EOF
root# cat /etc/wsl.conf
[user]
default=oldhorse
```
restart wsl instance 22.04,
```
PS > wsl -t Ubuntu-22.04
PS > wsl -d Ubuntu-22.04
```
## hostname rename 
Here is way to change hostname in WSL2.
```
/etc/wsl.conf
[network]
hostname = wsl2
generateHosts = false
```
update hostname /etc/hosts, 
```
127.0.1.1       wsl2
```
## DNS issue 
The default DNS not working on WSL2, no matter with VPN or without VPN.
```$ ping google.ca
ping: google.ca: Temporary failure in name resolution
```
From DNS test it works for common DNS server 8.8.8.8, 
```
oldhorse@wsl2:$ host -t A google.ca 8.8.8.8
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases:

google.ca has address 172.217.13.163
```
As remedy, disable auto-generated /etc/resolv.conf and only point to 8.8.8.8 or 1.1.1.1 in /etc/resolv.conf.
1) disable auto-generated /etc/resolv.conf
```
/etc/wsl.conf
[network]
hostname = wsl2
generateHosts = false
generateResolvConf = false
```
2) remove soft link /etc/resolv.conf
```
rm /etc/resolv.conf
```
3) add DNS server 
```
/etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
```
4) restart wsl2 instance
```
PS > wsl -t Ubuntu-22.04
PS > wsl -d Ubuntu-22.04
```
## conclusion 
We demo the step by step how to import multi instance of Ubuntu to WSL2, and correct DNS, sudo user, hostname etc issues.







