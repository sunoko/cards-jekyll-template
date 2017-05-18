---
layout: post
title: "PowershellでHyperVのVM作成自動化"
date: 2017-05-17 21:27:26
description:
tags:
- powershell
twitter_text:
introduction:
---

## Hyper-VのVM作成自動化とテスト

```powershell
function New-HyperVVirtualMachine {
<#
.SYNOPSIS
Create virtual machine on Hyper-V using template vhdx file.

.EXAMPLE
New-HyperVVirtualMachine -VMNames $VMNames -VHDXSourcePath $VHDXSourcePath

.NOTES
Should be prepared template vhdx file.
Should run local administrator user.
#>    
    [CmdletBinding()]     
    Param
    (
        [Parameter(ValueFromPipeline=$true,Mandatory=$true)]
        [String[]]$VMNames,

        [Parameter(Mandatory=$false)]
        [String]$VHDXDir = (Get-VMHost).VirtualHardDiskPath,

        [Parameter(Mandatory=$true)]
        [String]$VHDXSourcePath,        
        
        [Parameter(Mandatory=$false)]
        [int]$ProcessorCount = 2,

        [Parameter(Mandatory=$false)]
        [int]$Generation = 1,

        [Parameter(Mandatory=$false)]
        [Int64]$ServerMaximumMemoryMB = 4096MB,

        [Parameter(Mandatory=$false)]
        [Int64]$ServerMinimumMemoryMB = 512MB,

        [Parameter(Mandatory=$false)]
        [Int64]$MemoryStartupBytes = 512MB,

        [Parameter(Mandatory=$false)]
        [Int64]$VHDXDiskSizeBytes = 127GB,        

        [Parameter(Mandatory=$false)]
        [String]$SwitchType = "Internal",

        [Parameter(Mandatory=$false)]
        [String]$SwitchName = "InternalSwitch"
    )
    Begin
    {
        Import-Module Hyper-V
        if(!(Test-Path -Path $VHDXDir)) { New-Item $VHDXDir -ItemType Directory | Out-Null }
        if ((Get-VMSwitch).SwitchType -notcontains $SwitchType -and (Get-VMSwitch).Name -notcontains $SwitchName) {
            New-VMSwitch -Name $SwitchName -SwitchType $SwitchType | Out-Null
        }

        $VMSettings = @{
          "ProcessorCount" = $ProcessorCount
          "MemoryMinimumBytes" = $ServerMinimumMemoryMB
          "MemoryMaximumBytes" = $ServerMaximumMemoryMB
          "DynamicMemory" = $true
          "Passthru" = $true
        }
    }
    Process
    {
        foreach($VMName in $VMNames) {
            try {
                $VHDXPath = (New-VHD -Path "$($VHDXDir)\$($VMName).vhdx" `
                                     -ParentPath $VHDXSourcePath `
                                     -SizeBytes $VHDXDiskSizeBytes `
                                     -ErrorAction Stop).Path
            }
            catch {
                Write-Verbose $_
                return 1
            }
            $VMParams = @{
                "Name" = [String]$VMName
                "Generation" = $Generation
                "MemoryStartupBytes" = $MemoryStartupBytes
                "SwitchName" = $SwitchName
                "VHDPath" = $VHDXPath
            }
            Write-Verbose "Creating VM : $($VMName)"
            New-VM @VMParams | Set-VM @VMSettings | Start-VM
            Write-Verbose "$($VMName) created successfully"
        }
        Write-Verbose "VM created successfully"        
    }
    End {}
}
```

```powershell
function Remove-HyperVVirtualMachine {
<#
.SYNOPSIS
Delete virtual machine and vhdx file.

.EXAMPLE
Remove-HyperVVirtualMachine -VMNames $VMNames

.NOTES
Should run local administrator user.
#>       
    [CmdletBinding()]     
    Param
    (
        [Parameter(ValueFromPipeline=$true,Mandatory=$true)]
        [String[]]$VMNames,  

        [Parameter(Mandatory=$false)]
        [String]$VHDXDir = (Get-VMHost).VirtualHardDiskPath        
    )
    Import-Module Hyper-V
    foreach ($VMName in $VMNames) {
        Stop-VM $VMName -Force -ErrorAction SilentlyContinue
        Remove-VM $VMName -Force -ErrorAction SilentlyContinue
        Remove-Item (Join-Path $VHDXDir "$($VMName).vhdx") -ErrorAction SilentlyContinue
    }
}
```

```powershell
$parent = Split-Path -Parent (Split-Path -Parent $MyInvocation.MyCommand.Path)
$sut = (Split-Path -Leaf $MyInvocation.MyCommand.Path) -replace '\.Tests\.', '.'
. "$parent\$sut"
. "$parent\Remove-HyperVVirtualMachine.ps1"
$TestConfig = Get-Content .\testData.json | ConvertFrom-Json
$PSDefaultParameterValues = @{
    "New-HyperVVirtualMachine:VMNames" = $TestConfig.VMNames.VMName
    "New-HyperVVirtualMachine:VHDXSourcePath" = $TestConfig.VHDXSourcePath
    "Remove-HyperVVirtualMachine:VMNames" = $TestConfig.VMNames.VMName
}

Describe "New-HyperVVirtualMachine" {
    Context "Run function nomally" {
        It "shold return null for creating VMs successfully" {
            Remove-HyperVVirtualMachine
            New-HyperVVirtualMachine | Should BeNullOrEmpty
        }
           It "Throw" {
        {throw "error message"} | Should Throw "error message"
    }
        It "should get specific vhdx file" {
            $VHDXDir = (Get-VMHost).VirtualHardDiskPath
            $VHDXFiles = (Get-ChildItem $VHDXDir).Name
            foreach ($VMName in $TestConfig.VMNames.VMName) {
                [Regex]::Match($VHDXFiles, $VMName).Success | Should Be $true   
            }
        }
        It "should get specific virtual machine" {
            foreach ($VMName in $TestConfig.VMNames.VMName) {            
                (Get-VM).Name -contains $VMName | Should Be $true
            }
        }
        It "should return exception value" {
            New-HyperVVirtualMachine | Should Be 1
        }
        Remove-HyperVVirtualMachine
    }
}
```

```json
[
    {
        "VHDXSourcePath": "C:\\somewhere.vhdx",
        "VMNames":
        [
            {
                "VMName": "VM01"
            },
            {
                "VMName": "VM02"
            }
        ]
    }
]
```

## VMをPowersehllから操作するための準備
#### 3. WinRM起動
接続されるPCで実行

```powershell
Enable-PSRemoting -Force
Enter-PSSession -ComputerName localhost
```
#### 4. Firewall無効化
接続されるPCで実行

```powershell
Get-NetFirewallProfile | Set-NetFirewallProfile -Enabled false
```
#### 5. TrustedHosts設定
接続するPCで実行

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value * -Force
Get-Item WSMan:\localhost\Client\TrustedHosts
# Type            Name                           SourceOfValue   Value
# ----            ----                           -------------   -----
# System.String   TrustedHosts                                   *
```

#### 6. PowershellからVMに接続

```powershell
$cred = Get-Credential
Enter-PSSession -ComputerName "IP or HostName" -Credential $cred
```
