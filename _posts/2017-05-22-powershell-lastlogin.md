---
layout: post
title: "PowershellでLast Login Userを調べる"
date: 2017-05-22 21:18:24
description:
tags:
- powershell
twitter_text:
introduction:
---

最後にログインしたユーザーを取得できます。
何か問題が起きた時に犯人を突き止めちゃいましょう。

```powershell
function Get-LastLogon { 
  [CmdletBinding()] 
  param( 
    [String]$Computer="$env:COMPUTERNAME", 
    [String]$WQLFilter="NOT SID = 'S-1-5-18' AND NOT SID = 'S-1-5-19' AND NOT SID = 'S-1-5-20'" 
  ) 
  $Win32User = Get-WmiObject -Class Win32_UserProfile -Filter $WQLFilter -ComputerName $Computer 
  $LastUser = $Win32User | Sort-Object -Property LastUseTime -Descending | Select-Object -First 1 
  $Loaded = $LastUser.Loaded 
  $Script:Time = ([WMI]'').ConvertToDateTime($LastUser.LastUseTime) 
    
  #Convert SID to Account for friendly display 
  $Script:UserSID = New-Object System.Security.Principal.SecurityIdentifier($LastUser.SID) 
  $User = $Script:UserSID.Translate([System.Security.Principal.NTAccount]) 
    
  #Creating Custom PSObject For Output 
  New-Object -TypeName PSObject -Property @{ 
    Computer = $Computer 
    User = $User 
    SID = $Script:UserSID 
    Time = $Script:Time 
    CurrentlyLoggedOn = $Loaded 
  } | Select-Object Computer, User, SID, Time, CurrentlyLoggedOn      
}
```
