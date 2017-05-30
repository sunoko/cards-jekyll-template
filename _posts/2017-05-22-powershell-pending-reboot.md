---
layout: post
title: "Powershellで保留されているRebootがあるか確認する"
date: 2017-05-22 08:03:44
description:
tags:
- powershell
twitter_text:
introduction:
---

WindowsUpdateやらソフトウェアのインストール後に再起動が求められる場合がありますが、その再起動待ちがあるのか確認できます。
サーバーだと頻繁に再起動できないので保留にしがちですが、そういえば再起動待ちってあったっけ？という場合に使えるのではないでしょうか。

```powershell
function Get-PendingReboot { 
  [CmdletBinding()] 
  param( 
    [string]$Computer = "$env:COMPUTERNAME",
    [bool]$RenameComputer = $false,
    [bool]$RenameFile = $false,
    [bool]$Pending = $false,
    [bool]$CBSRebootPend = $false
  )        
  # Making registry connection to the local/remote computer 
  $HKLM = [UInt32] "0x80000002" 
  $WMI_Reg = [WMIClass] "\\$Computer\root\default:StdRegProv" 
          
  # CBS
  $RegCBS = $WMI_Reg.EnumKey($HKLM,"SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\") 
  $CBSRebootPend = $RegCBS.sNames -contains "RebootPending"     
            
  # Windows Update
  $RegWUAU = $WMI_Reg.EnumKey($HKLM,"SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\") 
  $WUAURebootReq = $RegWUAU.sNames -contains "RebootRequired" 
          
  # FileRenameOperations
  $RegSessionManager = $WMI_Reg.GetMultiStringValue($HKLM,"SYSTEM\CurrentControlSet\Control\Session Manager\","PendingFileRenameOperations") 
  $RegValuePFRO = $RegSessionManager.sValue 
  If ($RegValuePFRO) { 
    $RenameFile = $true 
  }   

  # ComputerName and ActiveComputerName
  $ActCompNm = $WMI_Reg.GetStringValue($HKLM,"SYSTEM\CurrentControlSet\Control\ComputerName\ActiveComputerName\","ComputerName")       
  $CompNm = $WMI_Reg.GetStringValue($HKLM,"SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName\","ComputerName") 
  If ($ActCompNm -ne $CompNm) { 
    $RenameComputer = $true 
  } 

  New-Object -TypeName PSObject -Property @{ 
    Computer = $Computer
    CBServicing = $CBSRebootPend 
    WindowsUpdate = $WUAURebootReq 
    PendComputerRename = $RenameComputer 
    RenameFile = $RenameFile 
    # PendFileRenVal=$RegValuePFRO 
    RebootPending = ($RenameComputer -or $CBSRebootPend -or $WUAURebootReq -or $RenameFile) 
  } | Select-Object Computer, CBServicing, WindowsUpdate, PendComputerRename, RenameFile, RebootPending
}

Get-PendingReboot
# Computer           : P26564
# CBServicing        : False
# WindowsUpdate      : False
# PendComputerRename : False
# RenameFile         : True
# RebootPending      : True
```