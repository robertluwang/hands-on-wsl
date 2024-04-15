# Build k8s Cluster on WSL2

The main difference between WSL1 and WSL2 that later is hyper-v VM based Microsoft Linux version, it sounds like excited news at least it is running full Linux kernel but there are too many issues if using as devops box.

Here are some outstanding pain points:

- default systemd is not enabled, which needed to install packages 
- default network is NAT, always assign dynamic ip after reboot, will give trouble for app like k8s api server

## Lab env

- win10 host  32GB RAM
- WSL2 Ubuntu 22.04
- Docker 25.0.4 on WSL2
- k8s 1.29.3 on WSL2

## Install WSL2 Ubuntu
Install WSL2 from CMD or PS on win host, following official doc, 

[https://docs.microsoft.com/en-us/windows/wsl/install](https://docs.microsoft.com/en-us/windows/wsl/install)

```
wsl -l -o
The following is a list of valid distributions that can be installed.
Install using 'wsl.exe --install <Distro>'.

NAME                                   FRIENDLY NAME
Ubuntu                                 Ubuntu
Debian                                 Debian GNU/Linux
kali-linux                             Kali Linux Rolling
Ubuntu-18.04                           Ubuntu 18.04 LTS
Ubuntu-20.04                           Ubuntu 20.04 LTS
Ubuntu-22.04                           Ubuntu 22.04 LTS
OracleLinux_7_9                        Oracle Linux 7.9
OracleLinux_8_7                        Oracle Linux 8.7
OracleLinux_9_1                        Oracle Linux 9.1
openSUSE-Leap-15.5                     openSUSE Leap 15.5
SUSE-Linux-Enterprise-Server-15-SP4    SUSE Linux Enterprise Server 15 SP4
SUSE-Linux-Enterprise-15-SP5           SUSE Linux Enterprise 15 SP5
openSUSE-Tumbleweed                    openSUSE Tumbleweed

wsl --install Ubuntu-22.04
```
Launch WSL2 instance from CMD or PS shell,

```wsl -d Ubuntu-22.04```

Launch WSL2 instance from SecureCRT client 
- create wsl2.bat under C:\tools, including below line 

```wsl -d Ubuntu-22.04```

- SecureCRT->Session->Connection/Protocol:'Local Shell', Connection->Local Shell/Shell path: C:\tools\wsl2.bat  

## WSL2 config
There are two kind of config for WSL2:
- global config for all WSL2 instance on win host, `C:\Users\oldhorse\.wslconfig`
```
[wsl2]
memory=16GB
processors=4
swap=0
```
- config per one WSL2 instance inside vm, `/etc/wsl.conf`
```
[boot]
systemd=true
[network]
hostname = wsl2
generateHosts = false
generateResolvConf = false
```
This will enable systemd, setup hostname and don't generate /etc/resolv.conf.

## Remedy for "static" ip
After WSL2 instance launched, will see NAT ip as below,
```
oldhorse@wsl2:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1350 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:ea:e8:22 brd ff:ff:ff:ff:ff:ff
    inet 172.30.73.22/20 brd 172.30.79.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:feea:e822/64 scope link 
       valid_lft forever preferred_lft forever
```
Assume we want to keep static ip 192.168.80.2 for WSL2 Ubuntu, then need to manually assign ip to WSL2 eth0 inside WSL2 VM, and recreate NAT network adaptor for WSL on win host end, it is in fact the network type of Host-only + NAT.

Run below batch script as Administrator on windows host, you can type wsl2ip.bat from CMD, PS or place shortcut icon on desktop. 

`wsl2ip.bat`
```
date /t
time /t
 
wsl -d Ubuntu-22.04 -u root ip addr del $(ip addr show eth0 ^| grep 'inet\b' ^| awk '{print $2}' ^| head -n 1) dev eth0
wsl -d Ubuntu-22.04 -u root ip addr add 192.168.80.2/24 broadcast 192.168.80.255 dev eth0
wsl -d Ubuntu-22.04 -u root ip route add 0.0.0.0/0 via 192.168.80.1 dev eth0
wsl -d Ubuntu-22.04 -u root echo nameserver 8.8.8.8 ^> /etc/resolv.conf
wsl -d Ubuntu-22.04 -u root chattr +i /etc/resolv.conf
 
powershell -c "Get-NetAdapter 'vEthernet (WSL)' | Get-NetIPAddress | Remove-NetIPAddress -Confirm:$False; New-NetIPAddress -IPAddress 192.168.80.1 -PrefixLength 24 -InterfaceAlias 'vEthernet (WSL)'; Get-NetNat | ? Name -Eq WSLNat | Remove-NetNat -Confirm:$False; New-NetNat -Name WSLNat -InternalIPInterfaceAddressPrefix 192.168.80.0/24;"
 
date /t
time /t

pause
```
The default generated /etc/resolv.conf won't work for DNS for Internet access, so that is why we disable generateResolvConf in /etc/wsl.conf, add 8.8.8.8 to it, and changed it to read-only file.

After run above script, you can access from host to WSL2 due to it is host-only static ip now.

```
PS C:\Users\oldhorse> Get-NetAdapter 'vEthernet (WSL)' | Get-NetIPAddress

IPAddress         : 192.168.80.1
InterfaceIndex    : 24
InterfaceAlias    : vEthernet (WSL)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Preferred
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : ActiveStore

PS C:\Users\oldhorse> Get-NetNat 'WSLNat'

Name                             : WSLNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 192.168.80.0/24
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True

PS C:\Users\oldhorse> ipconfig

Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.80.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :

PS C:\Users\oldhorse> ping 192.168.80.2

Pinging 192.168.80.2 with 32 bytes of data:
Reply from 192.168.80.2: bytes=32 time=1ms TTL=64
Reply from 192.168.80.2: bytes=32 time=1ms TTL=64
```

also you can access from WSL2 to host, Internet like google as well, 

```
oldhorse@wsl2:~$ ip addr show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1350 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:ea:e8:22 brd ff:ff:ff:ff:ff:ff
    inet 192.168.80.2/24 brd 192.168.80.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:feea:e822/64 scope link
       valid_lft forever preferred_lft forever

oldhorse@wsl2:~$ ip r
default via 192.168.80.1 dev eth0 
192.168.80.0/24 dev eth0 proto kernel scope link src 192.168.80.2

oldhorse@wsl2:~$ ping google.ca
PING google.ca (142.251.40.99) 56(84) bytes of data.
64 bytes from lga25s79-in-f3.1e100.net (142.251.40.99): icmp_seq=1 ttl=105 time=69.4 ms
64 bytes from lga25s79-in-f3.1e100.net (142.251.40.99): icmp_seq=2 ttl=105 time=60.5 ms
 
oldhorse@wsl2:~$ cat /etc/resolv.conf 
nameserver 8.8.8.8
```
## Common issue for WSL2 NAT 
Since we changed WSL2 NAT manually, sometimes it hanging on wsl cli, as remedy run below reset batch file as admin from win host.
`wslreset.bat`
```
taskkill /f /im wslservice.exe

wsl -l 

pause
```

From now on the WSL2 with static ip and Internet access ready for k8s installation.

## k8s installation automation
It is possible to install k8s installation in one shot using my handy script toolkit.
First of all, download it via git clone, or manually download from github.
```
git clone git@github.com:robertluwang/hands-on-nativecloud.git
cd ./hands-on-nativecloud/src/k8s-cri-dockerd
```
Here is complete step to install a single node k8s cluster on WSL2.
```
bash docker-server.sh

exit and enter to WSL2 instance, to make sure non-root user to access docker.

bash cri-dockerd.sh
bash k8s-install.sh
bash k8s-init.sh 192.168.80.2
```
Next I will explain more details for each step as below.

## Linux native Docker server on WSL2

The systemd is ready so just following the docker doc to install docker on Ubuntu, 

[https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

I make it as handy script, [docker-install.sh](https://github.com/robertluwang/hands-on-nativecloud/blob/main/src/k8s-cri-dockerd/docker-server.sh).

`docker-server.sh`
```
# docker-server.sh
# handy script to install docker on ubuntu 
# run on k8s cluster node (master/worker)
# By Robert Wang @github.com/robertluwang
# Nov 21, 2022

echo === $(date) Provisioning - docker-server.sh by $(whoami) start

sudo apt-get update -y
sudo apt-get install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io 

sudo groupadd docker
sudo usermod -aG docker $USER

# turn off swap
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sleep 30 
sudo systemctl restart docker

echo === $(date) Provisioning - docker-server.sh by $(whoami) end
```
Let's run it,
```
bash docker-server.sh
```
re-login to WSL2, docker working for non-root user, in my case it is `oldhorse`.

```
docker info 
docker ps 
```

verify docker daemon,

```
oldhorse@wsl2:~$ systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2024-04-14 12:07:34 EDT; 1h 10min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 253 (dockerd)
      Tasks: 104
     Memory: 208.5M
     CGroup: /system.slice/docker.service
             ├─253 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

docker bridge docker0 ready, 
```
oldhorse@wsl2:~$ ip addr show docker0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:a7:dc:0d:c9 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```
## cri-dockerd setup for k8s + docker on WSL2
As we known the docker support removed since k8s 1.24+, so need cri-dockerd installed to have cri interface with Docker Engine API for k8s cluster. More details please see https://github.com/Mirantis/cri-dockerd.

You might wonder why we want to still keep docker for k8s setup, because I would like get benefit from pure docker test and k8s test in same local lab.

I also make handy script for setup - [cri-dockerd.sh](https://github.com/robertluwang/hands-on-nativecloud/blob/main/src/k8s-cri-dockerd/cri-dockerd.sh)

`cri-dockerd.sh`
```
# cri-dockerd.sh
# handy script to install cri-dockerd on ubuntu 
# run on k8s cluster node (master/worker)
# By Robert Wang @github.com/robertluwang
# Nov 26, 2022

echo === $(date) Provisioning - cri-dockerd by $(whoami) start

# cri-dockerd 
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
tar xvf cri-dockerd-${VER}.amd64.tgz
sudo mv cri-dockerd/cri-dockerd /usr/local/bin/
sudo chmod +x /usr/local/bin/cri-dockerd

wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket

sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
sudo chown root:root /etc/systemd/system/cri-docker.*
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
sleep 30
sudo systemctl restart cri-docker.service

echo === $(date) Provisioning - cri-dockerd by $(whoami) end
```
Let's run it,
```
bash cri-dockerd.sh
```
Verify it is active running, 
```
oldhorse@wsl2:~$ systemctl status cri-docker
● cri-docker.service - CRI Interface for Docker Application Container Engine
     Loaded: loaded (/etc/systemd/system/cri-docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2024-04-14 12:07:35 EDT; 1h 25min ago
TriggeredBy: ● cri-docker.socket
       Docs: https://docs.mirantis.com
   Main PID: 880 (cri-dockerd)
      Tasks: 17
     Memory: 100.8M
     CGroup: /system.slice/cri-docker.service
             └─880 /usr/local/bin/cri-dockerd --container-runtime-endpoint fd://
```

## k8s packages installation on WSL2
It is time to install k8s following up official k8s doc.

Install k8s packages using below handy script [k8s-install.sh](https://github.com/robertluwang/hands-on-nativecloud/blob/main/src/k8s-cri-dockerd/k8s-install.sh),

[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

`k8s-install.sh`
```
# k8s-install.sh
# handy script to install k8s on ubuntu 
# run on all k8s cluster node (master/worker)
# By Robert Wang @github.com/robertluwang
# Oct 30, 2021

echo === $(date) Provisioning - k8s-install.sh by $(whoami) start

sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# cli completion
sudo apt-get install bash-completion -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
echo "export do='--dry-run=client -o yaml'" >>~/.bashrc

echo === $(date) Provisioning - k8s-install.sh by $(whoami) end
```
Let's run it,
```
bash k8s-install.sh
```
Verify installed package version,

```
oldhorse@wsl2:~/dev$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"29", GitVersion:"v1.29.3", GitCommit:"6813625b7cd706db5bc7388921be03071e1a492d", GitTreeState:"clean", BuildDate:"2024-03-15T00:06:16Z", GoVersion:"
go1.21.8", Compiler:"gc", Platform:"linux/amd64"}
o
ldhorse@wsl2:~/dev$ kubectl version 
Client Version: v1.29.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.29.3

oldhorse@wsl2:~/dev$ kubelet --version
Kubernetes v1.29.3
```

## k8s cluster init on WSL2

Run handy script [k8s-init.sh](https://github.com/robertluwang/hands-on-nativecloud/blob/main/src/k8s-cri-dockerd/k8s-init.sh) for first time setup k8s cluster on WSL2, 

`k8s-init.sh`
```
# k8s-init.sh
# handy script to init k8s cluster on ubuntu 
# run on k8s master node only
# By Robert Wang @github.com/robertluwang
# Nov 21, 2022
# $1 - master/api server ip

echo === $(date) Provisioning - k8s-init.sh $1 by $(whoami) start

if [ -z "$1" ];then
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem --cri-socket unix:///var/run/cri-dockerd.sock | tee /var/tmp/kubeadm.log
else
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=$1 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem --cri-socket unix:///var/run/cri-dockerd.sock | tee /var/tmp/kubeadm.log
fi

# allow normal user to run kubectl
if [ -d $HOME/.kube ]; then
  rm -r $HOME/.kube
fi
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# install calico network addon
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/custom-resources.yaml

# allow run on master
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

echo === $(date) Provisioning - k8s-init.sh $1 by $(whoami) end
```
We need to give eth0 static ip 192.168.80.2 as parameter to k8s-init.sh, let the k8s api server pointing to eth0 ip.
```
bash k8s-init.sh 192.168.80.2
```
## k8s cluster reset on WSL2
Run handy script [k8s-reset.sh](https://github.com/robertluwang/hands-on-nativecloud/blob/main/src/k8s-cri-dockerd/k8s-reset.sh) to quickly rebuild k8s cluster on WSL2 if anything mess up, 

`k8s-reset.sh`
```
# k8s-reset.sh
# handy script to reset k8s cluster on ubuntu 
# run on k8s cluster node (master/worker)
# By Robert Wang @github.com/robertluwang
# Nov 21, 2022

echo === $(date) Provisioning - k8s-reset.sh by $(whoami) start

sudo kubeadm reset -f --cri-socket unix:///var/run/cri-dockerd.sock 

echo === $(date) Provisioning - k8s-reset.sh by $(whoami) end
```
For example,
```
bash k8s-reset.sh
```
then following `k8s-init.sh` with eth0 static ip, 
```
bash k8s-init.sh 192.168.80.2
```
## k8s cluster test on WSL2
```
oldhorse@wsl2:~$ k get node
NAME   STATUS   ROLES           AGE    VERSION
wsl2   Ready    control-plane   7d3h   v1.29.3
oldhorse@wsl2:~$ k get pod -A
NAMESPACE          NAME                                       READY   STATUS    RESTARTS        AGE
calico-apiserver   calico-apiserver-99d8c8bc6-5csh6           1/1     Running   9 (104m ago)    7d3h
calico-apiserver   calico-apiserver-99d8c8bc6-lgdh4           1/1     Running   9 (104m ago)    7d3h
calico-system      calico-kube-controllers-78788579b8-jdhj4   1/1     Running   9 (104m ago)    7d3h
calico-system      calico-node-vzhwj                          1/1     Running   9 (104m ago)    7d3h
calico-system      calico-typha-7647b5ff5b-dkxks              1/1     Running   15 (40m ago)    7d3h
kube-system        coredns-76f75df574-4dg8d                   1/1     Running   9 (104m ago)    7d3h
kube-system        coredns-76f75df574-5cmgf                   1/1     Running   9 (104m ago)    7d3h
kube-system        etcd-wsl2                                  1/1     Running   32 (46m ago)    7d3h
kube-system        kube-apiserver-wsl2                        1/1     Running   30 (46m ago)    7d3h
kube-system        kube-controller-manager-wsl2               1/1     Running   10 (104m ago)   7d3h
kube-system        kube-proxy-x7mxt                           1/1     Running   9 (104m ago)    7d3h
kube-system        kube-scheduler-wsl2                        1/1     Running   10 (104m ago)   7d3h
tigera-operator    tigera-operator-6fbc4f6f8d-d5fgp           1/1     Running   16 (40m ago)    7d3h
```
Do dry run on new pod,
```
oldhorse@wsl2:~$ k run test --image=nginx $do
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test
  name: test
spec:
  containers:
  - image: nginx
    name: test
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
Let's launch test pod if everything looks good,
```
oldhorse@wsl2:~$ k run test --image=nginx 
pod/test created
oldhorse@wsl2:~$ k get pod 
NAME   READY   STATUS              RESTARTS   AGE
test   0/1     ContainerCreating   0          

oldhorse@wsl2:~$ k get pod -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE   NOMINATED NODE   READINESS GATES
test   1/1     Running   0          48s   192.168.9.254   wsl2   <none>           <none>

oldhorse@wsl2:~$ curl 192.168.9.254
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Conclusion 

So far the one single node k8s cluster lab is ready on WSL2:

- k8s cluster has static ip, can be accessed from inside and outside of WSL2 
- it is docker based so same lab env for any docker related test 
- it is ready for local LLM AI test on both docker and k8s 

## About Me

Hey! I am Robert Wang, work@Ericsson, live in Montreal, QC, CA.

More simple and more efficient.

[GitHub: robertluwang](https://github.com/robertluwang)
[Twitter: robertluwang](https://twitter.com/robertluwang)
[LinkedIn: robertluwang](https://www.linkedin.com/in/robertluwang/) 
[Medium: robertluwang](https://medium.com/@robertluwang)
[Dev.to: robertluwang](https://dev.to/robertluwang)
[Web: dreamcloud.artark.ca](https://dreamcloud.artark.ca)

