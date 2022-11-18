# WSL2 update failure
## Ubuntu 21.04 hirsute
WSL2 Ubuntu 21.04 hirsute update failure as below

```
oldhorse@wsl2:~$ sudo apt update
Ign:1 http://archive.ubuntu.com/ubuntu hirsute InRelease
Ign:2 http://archive.ubuntu.com/ubuntu hirsute-updates InRelease
Ign:3 http://security.ubuntu.com/ubuntu hirsute-security InRelease
Ign:4 http://archive.ubuntu.com/ubuntu hirsute-backports InRelease
Err:5 http://security.ubuntu.com/ubuntu hirsute-security Release
  404  Not Found [IP: 91.189.91.38 80]
Err:6 http://archive.ubuntu.com/ubuntu hirsute Release
  404  Not Found [IP: 91.189.91.38 80]
Err:7 http://archive.ubuntu.com/ubuntu hirsute-updates Release
  404  Not Found [IP: 91.189.91.38 80]
Err:8 http://archive.ubuntu.com/ubuntu hirsute-backports Release
  404  Not Found [IP: 91.189.91.38 80]
Reading package lists... Done
E: The repository 'http://security.ubuntu.com/ubuntu hirsute-security Release' no longer has a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: The repository 'http://archive.ubuntu.com/ubuntu hirsute Release' no longer has a Release file.
```

when browser url http://91.189.91.38/ubuntu/dists/, cannot find hirsute in list due to 21.04 is interim release instead of LTS release.

## LTS and interim 
check out here what is difference between LTS and interim,
https://ubuntu.com/about/release-cycle
- LTS : LTS or ‘Long Term Support’ releases are published every two years in April. 
- interim - Every six months between LTS versions, Canonical publishes an interim release of Ubuntu, supported for 9 months.

![](./images/ubuntu%20release.png)

## remove old imported WSL 
In my case, 21.04 is imported WSL instance, cannot uninstall from win10, need to unregister it manually, 
```
PS > wsl -l -v
  NAME            STATE           VERSION
* Ubuntu          Running         1      
  Ubuntu-21.04    Stopped         2
PS > wsl --unregister Ubuntu-21.04
Unregistering...
PS > wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Running         1
```
then manually remove folder for 21.04, 
```
C:<path>\Ubuntu-21.04
```
## always test on LTS 
If you want to test on stable release of Ubuntu, get longer support like update, always go for LTS.

The latest LTS is 22.04.
