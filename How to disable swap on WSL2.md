
# How to disable swap on WSL2?
## cannot disable swap from Linux 
I disable swap on WSL2 manually, 
```
sudo swapoff -a
```
However the swap still there, 
```
oldhorse@wsl2:~$ sudo swapoff -a
[sudo] password for oldhorse: 
oldhorse@wsl2:~$ echo $?
32
oldhorse@wsl2$ free -m
               total        used        free      shared  buff/cache   available
Mem:           25401         130       25151           0         119       24995
Swap:           7168           0        7168
oldhorse@wsl2$ swapon -s
Filename                                Type            Size            Used            Priority
/swap/file                              file            7340032         0               -2      
```
## WSL way to disable swap
This is correct way to change or disable swap on WSL, 

C:\Users\oldhorse\.wslconfig, 
```
[wsl2]
swap=0
memory=4GB 
processors=2 
```
restart wsl, 
```
> wsl --shutdown
```
then swap disabled for WSL2,
```
oldhorse@wsl2:~$ free -m
               total        used        free      shared  buff/cache   available
Mem:            3924          80        3690           0         153        3653
Swap:              0           0           0
oldhorse@wsl2:~$ swapon -s
```



