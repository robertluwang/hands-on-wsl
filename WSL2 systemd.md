# WSL2 systemd
By default systemd not enabled on WSL2, 
```
oldhorse@wsl2$ systemctl
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```
## enable systemd on WSL2
There is WSL preview to allow quickly enable systemd on WSL2, but with condition:
1) preview available for win11
2) WSL install from app store only

the config change by
```
/etc/wsl.conf
[boot]
systemd=true
```
tested it is not working for win10, or maybe my wsl not installed from app store.

Here seems only available remedy for win10.
```
git clone https://github.com/DamionGans/ubuntu-wsl2-systemd-script.git
cd ubuntu-wsl2-systemd-script/
bash ubuntu-wsl2-systemd-script.sh
```
restart wsl2 instance
```
PS > wsl -t Ubuntu
PS > wsl -d Ubuntu
```
got error
```
nsenter: cannot open /proc/26/ns/time: No such file or directory
```
Found one fix https://github.com/DamionGans/ubuntu-wsl2-systemd-script/issues/36 on this issue, need to update nsenter option from "-a" to "-m -p" in enter script, 
```
oldhorse@wsl:~/tools/ubuntu-wsl2-systemd-script$ cat enter-systemd-namespace |grep nsenter
        exec /usr/bin/nsenter -t "$SYSTEMD_PID" -a \
        exec /usr/bin/nsenter -t "$SYSTEMD_PID" -a \
oldhorse@wsl:~/tools/ubuntu-wsl2-systemd-script$ cat enter-systemd-namespace |grep nsenter
        exec /usr/bin/nsenter -t "$SYSTEMD_PID" -m -p \
        exec /usr/bin/nsenter -t "$SYSTEMD_PID" -m -p \
```
re-run remedy script and restart wsl instance, 
```
sudo rm /usr/sbin/*systemd-namespace
bash ubuntu-wsl2-systemd-script.sh
PS > wsl -t Ubuntu
PS > wsl -d Ubuntu
```
Finally made it working now with systemd,
```
oldhorse@wsl$ systemctl list-units --type=service
  UNIT                                 LOAD   ACTIVE SUB     DESCRIPTION
  blk-availability.service             loaded active exited  Availability of block devices
  dbus.service                         loaded active running D-Bus System Message Bus
  finalrd.service                      loaded active exited  Create final runtime dir for shutdown pivot root
  keyboard-setup.service               loaded active exited  Set the console keyboard layout
  setvtrgb.service                     loaded active exited  Set console scheme
  systemd-journal-flush.service        loaded active exited  Flush Journal to Persistent Storage
  systemd-journald.service             loaded active running Journal Service
  systemd-logind.service               loaded active running Login Service
  systemd-networkd-wait-online.service loaded active exited  Wait for Network to be Configured
  systemd-networkd.service             loaded active running Network Service
● systemd-remount-fs.service           loaded failed failed  Remount Root and Kernel File Systems
  systemd-sysctl.service               loaded active exited  Apply Kernel Variables
  systemd-sysusers.service             loaded active exited  Create System Users
  systemd-tmpfiles-setup-dev.service   loaded active exited  Create Static Device Nodes in /dev
  systemd-tmpfiles-setup.service       loaded active exited  Create Volatile Files and Directories
  systemd-udev-settle.service          loaded active exited  udev Wait for Complete Device Initialization
  systemd-udev-trigger.service         loaded active exited  udev Coldplug all Devices
  systemd-udevd.service                loaded active running udev Kernel Device Manager
  systemd-update-utmp.service          loaded active exited  Update UTMP about System Boot/Shutdown
  user-runtime-dir@1000.service        loaded active exited  User Runtime Directory /run/user/1000
  user@1000.service                    loaded active running User Manager for UID 1000

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

21 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```




