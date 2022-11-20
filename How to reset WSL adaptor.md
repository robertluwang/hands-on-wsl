I did once to change WSL gateway to have static ip for WSL2, 
```
Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.80.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```
This is default setting before,
```
Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::a58d:462c:523f:6517%77
   IPv4 Address. . . . . . . . . . . : 172.26.128.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . : 
```
How can I restore the default ip for WSL on win10?

## remedy to reset all network on pc
run as CMD or PS as admin, 
```
wsl --shutdown
netsh winsock reset
netsh int ip reset all
netsh winhttp reset proxy
ipconfig /flushdns
```
Here is log, 
```
C:\Windows\System32>netsh winsock reset

Sucessfully reset the Winsock Catalog.
You must restart the computer in order to complete the reset.

C:\Windows\System32>netsh int ip reset all
Resetting Compartment Forwarding, OK!
Resetting Compartment, OK!
Resetting Control Protocol, OK!
Resetting Echo Sequence Request, OK!
Resetting Global, OK!
Resetting Interface, OK!
Resetting Anycast Address, OK!
Resetting Multicast Address, OK!
Resetting Unicast Address, OK!
Resetting Neighbor, OK!
Resetting Path, OK!
Resetting Potential, OK!
Resetting Prefix Policy, OK!
Resetting Proxy Neighbor, OK!
Resetting Route, OK!
Resetting Site Prefix, OK!
Resetting Subinterface, OK!
Resetting Wakeup Pattern, OK!
Resetting Resolve Neighbor, OK!
Resetting , OK!
Resetting , OK!
Resetting , OK!
Resetting , OK!
Resetting , failed.
Access is denied.

Resetting , OK!
Resetting , OK!
Resetting , OK!
Resetting , OK!
Resetting , OK!
Resetting , OK!
Resetting , OK!
Restart the computer to complete this action.

C:\Windows\System32>netsh winhttp reset proxy

Current WinHTTP proxy settings:
    Direct access (no proxy server).

C:\Windows\System32>ipconfig /flushdns

Windows IP Configuration
Successfully flushed the DNS Resolver Cache.
```
Then need to reboot host pc.

WSL gateway ip changed after reset, 
```
Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::13d:9440:d084:b78e%77
   IPv4 Address. . . . . . . . . . . : 172.19.224.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :
```
## reset WSL adaptor only 
You also can reset WSL gateway only if you keep original setting info, 
```
C:\Users\erobwan>powershell -c "Get-NetAdapter 'vEthernet (WSL)' | Get-NetIPAddress | Remove-NetIPAddress -Confirm:$False; New-NetIPAddress -IPAddress 172.26.128.1 -PrefixLength 20 -InterfaceAlias 'vEthernet (WSL)'; Get-NetNat | ? Name -Eq WSLNat | Remove-NetNat -Confirm:$False; New-NetNat -Name WSLNat -InternalIPInterfaceAddressPrefix 172.26.128.0/20;"

IPAddress         : 172.26.128.1
InterfaceIndex    : 77
InterfaceAlias    : vEthernet (WSL)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 20
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Tentative
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : ActiveStore

IPAddress         : 172.26.128.1
InterfaceIndex    : 77
InterfaceAlias    : vEthernet (WSL)
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 20
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Invalid
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : PersistentStore

Caption                          :
Description                      :
ElementName                      :
InstanceID                       : WSLNat;0
Active                           : True
ExternalIPInterfaceAddressPrefix :
IcmpQueryTimeout                 : 30
InternalIPInterfaceAddressPrefix : 172.26.128.0/20
InternalRoutingDomainId          : {00000000-0000-0000-0000-000000000000}
Name                             : WSLNat
Store                            : Local
TcpEstablishedConnectionTimeout  : 1800
TcpFilteringBehavior             : AddressDependentFiltering
TcpTransientConnectionTimeout    : 120
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
PSComputerName                   :
```
let's check WSL ip, 
```
Ethernet adapter vEthernet (WSL):

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 172.26.128.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :
```








