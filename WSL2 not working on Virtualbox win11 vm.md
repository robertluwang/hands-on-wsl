# WSL2 not working on Virtualbox win11 vm
## enable nested VT on win11 vm
WSL2 is hyper-v based vm on the ground, so nested VT is pre-condition.

I installed VB 6.1.40, found the nested VT option grey out in VB System/Processor tab.

As remedy, enable it from CMD as admin,
```
set PATH=%PATH%;"C:\Program Files\Oracle\VirtualBox"
VBoxManage modifyvm win11 --nested-hw-virt on
```
![](../images/nested%20VT.png)

## WSL2 installation failure
Based on requests, 2 features needed for WSL2:
- Virtual Machine Platform 
- Windows Subsystem for Linux 

Did but still got error, 
![](../images/WSL2%20failure.png)

## verify virtualization for win11 vm
Above error seems mean the nested VT not really working at all for VB VM.

Let's check the status from different perspectives.

systeminfo Hyper-V Requirements is empty, 
```
PS C:\Users\oldhorse> systeminfo

System Model:              VirtualBox
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```
check HyperV status,
```
PS C:\Users\oldhorse> Get-ComputerInfo -property "HyperV*"

HyperVisorPresent                                 : True
HyperVRequirementDataExecutionPreventionAvailable :
HyperVRequirementSecondLevelAddressTranslation    :
HyperVRequirementVirtualizationFirmwareEnabled    :
HyperVRequirementVMMonitorModeExtensions          :
```
Confirmed the nested VT not working.

## conclusion
Looks like nested VT still not supported by Virtualbox 6.1 even assume it will be supported according to Oracle document.
