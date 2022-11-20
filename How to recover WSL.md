# How to recover WSL
## use case
I enabled WSL2 systemd, cannot access to it after switch back to WSL version1, stuck in dead loop, 
```
Sleeping for 1 second to let systemd settle
Sleeping for 1 second to let systemd settle
```
the reason is clear that source line still in bash.bashrc, 
```
cat /etc/bash.bashrc|grep source
source /usr/sbin/start-systemd-namespace
```
so how to comment out when WSL accessible?
## option1 - force login WSL without profile and rc file
Lucky you can always to access WSL instance via wsl from host pc, 
```
C:\>wsl -d Ubuntu -e bash --noprofile --norc
bash-5.0$ cat /etc/bash.bashrc |grep source
source /usr/sbin/start-systemd-namespace
```
## option2 - update file via wsl command line 
```
C:\>wsl -d Ubuntu -u root -e /bin/bash -c "sed -i '/^source/s//#source/' /etc/bash.bashrc"
C:\>wsl -d Ubuntu -u root -e /bin/bash -c "cat /etc/bash.bashrc|grep source"
#source /usr/sbin/start-systemd-namespace
```
## option3 - directly access WSL roofs 
find out below folder. update bash.bashrc using notepad, this works for default WSL instance, not for imported wsl instance, 
```
C:\Users\oldhorse\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\etc
```
 
