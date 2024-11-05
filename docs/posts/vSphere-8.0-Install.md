---
title: "VSphere 8.0 Install & Configuration"
date: 2023-11-12T17:36:39-08:00
draft: false
tags:
  - vSphere 8.0
  - VMware
  - ESXi
  - Installation
  - Configuration
  - PowerCLI
  - Virtualization
  - Datastore
  - Networking
  - DNS
  - IPv6
  - Root Password
  - Hardware Compatibility
  - GUI Installation
  - VMware PowerCLI
  - vSphere Installation Guide
categories:
  - Virtualization
  - System Administration
  - VMware
  - Tutorials
  - How-to
  - IT Infrastructure
---

## Prerequisites

- VMware vSphere 8.0 ISO
  - Sign-up on VMWare's site [here](https://customerconnect.vmware.com/en/downloads/details?downloadGroup=ESXI80U1&productId=1345&rPId=112741) to download the evaluation copy
- USB Drive
- Hardware that meets [VMware's Compatibility Guide](https://www.vmware.com/resources/compatibility/search.php)
- VMware's PowerCLI PowerShell module
  - [VMware Docs](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-F02D0C2D-B226-4908-9E5C-2E783D41FE2D.html) for installation
  
<!-- more -->

## GUI Installation


### Welcome To the VMware ESXi 8.0.1 Installation

Press `Enter` at this screen to continue

![Welcome To the VMware ESXi 8.0.1 Installation](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/1-Welcome.png?raw=true)


### End User License Agreement (EULA)

Press `F11` to Accept and Continue to progress

![End User License Agreement (EULA)](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/2-EULA.png?raw=true)

### Select a Disk to Install or Upgrade

Choose which disk you want to install the OS. Here I chose the 278GB drive as the 1.09TB drive drive is going to be used for the datastore.

**Write down the last 4 numbers of the other drive it will be used later.**

![Select a Disk to Install or Upgrade](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/3-Select-Disk.png?raw=true)

### Please select a keyboard layout

Press `Enter` to select the default layout, or you can select a new layout by using the up and down arrows.

![Please select a keyboard layout](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/4-Select-Keyboard.png?raw=true)

### Enter a root password

Create the password for the root account that you will use to manage the hypervisor. [ESXi Password Requirements.](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-DC96FFDB-F5F2-43EC-8C73-05ACDAE6BE43.html)

![Enter a root password](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/5-Create-Root.png?raw=true)

### Error(s)/Warning(s) Found During System Scan

If there are any issues found, they will be listed here. As you can see my 2 Intel EXeon E5-2620 v4's may no longer be supported by VMware in future releases, and that my BIOS is still currently in Legacy Mode, which I will be changing since I was not aware!

Press `Enter` to continue.

![Error(s)/Warning(s) Found During System Scan](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/6-Error-Warnings.png?raw=true)

### Confirm Install

Press `F11` to confirm the installation.

![Confirm Install](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/7-Confirm-Install.png?raw=true)

### Installation Complete

After the installation has completed, remove the installation media. Then press `Enter` to reboot.

![Installation Complete](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/8-Installation-Complete.png?raw=true)

### After First Reboot

After the first reboot has happened, you'll see the following screen. I have DHCP configured so the host was given `10.0.1.107`. We'll be using this in the coming sections.

![After First Reboot](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/9-After-First-Reboot.png?raw=true)

## Configuring with PowerCLI

We're going to configure;

- Static IPv4
- Disable IPv6
- Hostname
- DNS
- Datastore

### Connecting to the host

```powershell
# Provide the credentials to the root account
$ESXiRoot = Get-Credential root
```
Enter the root password you used during the installation into the dialog box.

![Get-Credential Dialog Box](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/10-Get-Credential.png?raw=true)


```powershell
# Command to connect to the host
Connect-VIServer -Server '10.0.1.107' -Credential $ESXiRoot
```
You'll receive a Certificate warning. Press P to Accept the self-signed certificate Permanently.

![Certificate Warning](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/11-Certificate-Warning.png?raw=true)


After a successful connection you'll see the output below.

```shell
Name                           Port  User
----                           ----  ----
10.0.1.107                     443   root
```

### Configuring a Static IP

```powershell
Get-VMHostNetworkAdapter | 
    Select-Object -Property Name,IP,SubnetMask
```

```shell    
Name   IP         SubnetMask
----   --         ----------
vmnic0
vmnic1
vmnic2
vmnic3
vmk0   10.0.1.107 255.255.255.0
```

Take note of the name of the interface you're going to be changing, which for me it's `vmk0`.

```powershell
# Select only the adapter you're altering
$vmk0 = Get-VMHostNetworkAdapter | 
    Where-Object {$_.Name -eq "vmk0"}

# Pipe the network adapter to the set command
$vmk0 | 
    Set-VMHostNetworkAdapter -IP '10.0.1.52' -SubnetMask '255.255.255.0' -IPv6Enabled $false
```

```Shell
Perform operation?
Performing operation 'Configuring VM host network adapter.' on network adapter with IP '10.0.1.107' and device name 'vmk0'.
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):

```
Press `Enter` to continue the operation.

```Shell
WARNING: The underlying connection was closed: A connection that was expected to be kept alive was closed by the server.
Set-VMHostNetworkAdapter : 2023-11-12 10:04:23 PM       Set-VMHostNetworkAdapter                Object reference not set to an instance of an object.
At line:1 char:9
+ $vmk0 | Set-VMHostNetworkAdapter -IP '10.0.1.52' -SubnetMask '255.255 ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Set-VMHostNetworkAdapter], VimException
    + FullyQualifiedErrorId : Core_BaseCmdlet_UnknownError,VMware.VimAutomation.ViCore.Cmdlets.Commands.Host.SetVMHostNetworkAdapter

```
After the connection has been lost, check your management console and see if the IP address has been configured.

![Remote Management Console](https://github.com/stim-ca/blog.timewelltech.ca/blob/master/static/images/vSphere-8.0-Install/13-After-Static.png?raw=true)

You have to disconnect your session with the only IP address `10.0.1.107`.

```powershell
Disconnect-VIServer -Server 10.0.1.107 -Force
```
```shell
Confirm
Are you sure you want to perform this action?
Performing the operation "Disconnect VIServer" on target "User: root, Server: 10.0.1.107, Port: 443".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```
Press `Enter` to disconnect.

Reconnect the PowerShell session to the ESXi host with the new IP `10.0.1.52`.
```powershell
# Provide the credentials to the root account
$ESXiRoot = Get-Credential root

# Command to connect to the host
Connect-VIServer -Server '10.0.1.52' -Credential $ESXiRoot
```
Press `P` to accept the certificate again.

### Setting HostName, DNS, and disabling IPv6

Since we only have a basic network setup, we don't need to specify a VMHostNetwork. 

```powershell
Get-VMHostNetwork | 
    Set-VMHostNetwork -HostName TTE-ESX-02 -DnsFromDhcp $false -DnsAddress 10.0.1.2,10.0.1.3 -IPv6Enabled $false
```
```Shell
WARNING: Disabling IPv6 will take effect after host reboot.
HostName     DomainName   DnsFro ConsoleGateway  ConsoleGatewayD DnsAddress
                          mDhcp                  evice
--------     ----------   ------ --------------  --------------- ----------
TTE-ESX-02   AD.Timewe... False                                  {10.0.1.2, 10.0.1.3}
```
We will reboot the server later for IPv6 to be disabled.

### Creating a New Datastore

Get the device path on which you want to create the new datastore.
```powershell
# Save to variable for later
$VMHostDisk = Get-VMHostDisk
# Output
$VMHostDisk
```
```Shell
DeviceName                                                      TotalSectors
----------                                                      ------------
/vmfs/devices/disks/naa.6d0946600a41e9002ca66ae9f0e32e0b          2339373056
/vmfs/devices/disks/naa.6d0946600a41e9002ca66c7708a11de7           584843264
```
If you wrote down the last 4 numbers from before, you can tell that the one with more sectors is the drive I'm going to be using for the new datastore.

```PowerShell
New-Datastore -Vmfs -Name 'data' -Path $VMHostDisk[0].ScsiLun.CanonicalName -VMHost '10.0.1.52' -FileSystemVersion 6
```
```Shell
Name                               FreeSpaceGB      CapacityGB
----                               -----------      ----------
data                                 1,113.827       1,115.250
```

### Restarting the ESXi Host

```powershell
Get-VMHost | 
    Restart-VMHost -Force
```
```Shell
Confirm
Are you sure you want to perform this action?
Performing the operation "Restart VM host.: Force: True" on target "10.0.1.52".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```
Press `Enter` to restart the host.


VMWare's PowerCLI documentation can be found [here.](https://developer.vmware.com/docs/powercli/latest/products/vmwarevsphereandvsan/)

Next Post I'll be copying ISO's and provisioning VMs for the home-lab.

Thank you for reading!