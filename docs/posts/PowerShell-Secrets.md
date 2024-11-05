---
title: "PowerShell Secrets"
date: 2023-11-11T13:19:31-08:00
draft: false
---

## PowerShell Secrets Installation

[Microsoft's Installtion Guide](https://learn.microsoft.com/en-us/powershell/utility-modules/secretmanagement/get-started/using-secretstore?view=ps-modules)


## Creating a Vault and adding a secret

### Register SecretVault

```powershell
Read-Host -AsSecureString | Export-Clixml -Path C:\Temp\TTELAB.xml -Force

$secureVaultPasswordPath = 'C:\Temp\TTELAB.xml'

Register-SecretVault -Name TTELAB -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault

$vaultUnlockPassword = Import-CliXml -Path $secureVaultPasswordPath

Unlock-SecretStore -Password $vaultUnlockPassword

$storeConfiguration = @{
    Authentication = 'Password'
    PasswordTimeout = 3600 # 1 hour
    Interaction = 'None'
    Confirm = $false
}
Set-SecretStoreConfiguration @storeConfiguration

```

<!-- more -->

### Create a Secret

```powershell
$TTEESX01 = Get-Credential root

Set-Secret -Name 'TTE-ESX-01' -Secret $TTEESX01
```

### Use a secret in a script

Provisoning a new VM

```powershell
# Secret Preamble
$secureVaultPasswordPath = 'C:\Temp\TTELAB.xml'
$vaultUnlockPassword = Import-CliXml -Path $secureVaultPasswordPath
Unlock-SecretStore -Password $vaultUnlockPassword
$esxiPassword = Get-Secret -Name 'TTE-ESX-01'

# Connect to ESXi Host
Connect-VIServer -Server '10.0.1.50' -Credential $esxiPassword

# First Time only. Can be commented out after
$copySource = 'C:\Temp\ISO\Windows Server 2022.iso'
$copyDestination = 'vmstores:\10.0.1.50@443\ha-datacenter\data\isos\Windows Server 2022.iso'

Copy-DatastoreItem $copySource $copyDestination -Force

$newVMArguments = @{
    CD             = $true
    CoresPerSocket = 4  
    Datastore      = 'data'
    DiskGB         = 50
    GuestId        = 'windows2019srv_64Guest'
    MemoryGB       = 16
    NetworkName    = 'VM Network'
    NumCpu         = 4
    Name           = 'TTE-DC-01'
}

New-VM @newVMArguments

# Connect ISO to VM's CD Drive
Get-CDDrive -VM $newVMArguments.Name | 
    Set-CDDrive -IsoPath '[data] isos/Windows Server 2022.iso' -StartConnected $true

# Start VM

Start-VM -VM $newVMArguments.Name

# Open Console Window
# Note: VMRC must be installed on local machine
# Otherwise head to the web console

Open-VMConsoleWindow -VM $newVMArguments.Name
```
