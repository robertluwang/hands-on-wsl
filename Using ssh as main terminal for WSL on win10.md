# Using ssh as main terminal for WSL on win10
WSL: win10 wsl Ubuntu-18.04

few benefits to setup ssh to WSL:
- ssh client is fast and lightweight terminal to get control WSL, comparing with cmder/MobaXterm
- can run mixed cli with linux and window, even launch window app GUI directly 
 
## install openssh on wsl 

oldhorse@dreamcloud:~$ sudo apt install openssh-server

I only enabled password access in /etc/ssh/sshd_config, left listen to default 22, you can change to any own port,  
```
#PasswordAuthentication no
PasswordAuthentication yes

#PermitRootLogin prohibit-password
PermitRootLogin no
```
then start it as root, 
oldhorse@dreamcloud:~$ sudo service ssh start
 * Starting OpenBSD Secure Shell server sshd                                   

I tested ssh client in putty and SecureCRT, both are working very well.

## window env in ssh 
when ssh to WSL, the original window env variables not carried forward, this is one difference than running msys/MobaXterm, just need to add window path to WSL login profile.
```
export PATH=/mnt/c/Windows/System32:/mnt/c/Windows/System32/WindowsPowerShell/v1.0:$PATH
```

## WSL ssh start script 
We cannot run sshd as start service due to WSL is not real Linux, but easy to add below start script in login profile.
```
function startwslssh(){
    echo check wsl sshd status
    service ssh status
    if [ "$?" != "0" ]; then
        sudo service ssh start
    fi
}

startwslssh
```
it will start sshd if not start yet, and don't need to start again if already running.

## WSL vim theme  
the default vim color scheme is terrible, prefer to change to desert in WSL.

change local vimrc in home folder, 
```
oldhorse@dreamcloud:~$ cat .vimrc
colorscheme desert
filetype plugin indent on
" show existing tab with 4 spaces width
set tabstop=4
" when indenting with '>', use 4 spaces width
set shiftwidth=4
" On pressing tab, insert 4 spaces
set expandtab
```

So far I used putty as favourite terminal for all daily work on win10. 




