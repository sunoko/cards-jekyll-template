---
layout: post
title: "Powershellでプロセスを監視する"
date: 2017-05-22 08:03:59
description:
tags:
- powershell
twitter_text:
introduction:
---

```powershell
function Watch-Process {
  param
  (
    [string]$ProcessName,
    [int]$Span
  )
  while ($true) {
    $process = Get-Process -Name $ProcessName -ErrorAction SilentlyContinue
    if (!$process) { Write-Host "not exist $ProcessName"; break }
    $process
    Start-Sleep -Second $Span
  }
}

Watch-Process -ProcessName wuauclt -Span 60

# Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
# -------  ------    -----      -----     ------     --  -- -----------
#      78       6      860       3924       0.02   7760   7 PING
#      78       6      860       3924       0.02   7760   7 PING
#      78       6      860       3924       0.02   7760   7 PING
#      78       6      860       3924       0.02   7760   7 PING
#      78       6      872       3936       0.02   7760   7 PING
# not exist ping
```