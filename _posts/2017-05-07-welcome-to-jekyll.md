---
layout: post
title: "Welcome to Powershell"
date: 2017-05-07 20:34:26
image: '/assets/img/'
description: 'Put your description here.'
main-class: 'powershell'
color: '#B31917'
tags:
- "powershell"
categories:
twitter_text: 'This is Powershell Tricks.'
introduction: 'This is Powershell Tricks.'
---

# Powershell Tricks
![](https://www.powershellgallery.com/Content/Images/packageDefaultIcon.png)

## クロージャー

```ps1
function Closure {
  $n = 1
  {
    $script:n = $n + 1
    $n
  }.GetNewClosure() 
}

$obj1 = Closure
$obj2 = Closure
& $obj1
# 2
& $obj1
# 3
& $obj2
# 2
& $obj2
# 3
```

## filter

```ps1
# 偶数のみ取り出す
1..10 | ? { $_ % 2 -eq 0 }
```

## map

```ps1
# 全てに2をかける
1..10 | % { $_ * 2 }
```

## portを使ってるプロセス

```ps1
netstat -aon | Select-String ".0.0:4000"
netstat -aon | Select-String -Pattern ".0.0:4000$"
```

## &{}と.{}の違い

```ps1
$hoge = $null
& {$hoge = 4}
# [String]::IsNullOrEmpty($hoge)
# True
# &{}はスクリプト内部で処理を留める

$hoge = $null
. {$hoge = 4}
# [String]::IsNullOrEmpty($hoge)
# False
# .{}は元のスコープに影響する
$hoge
# 4
```

## Firewall設定確認

```ps1
Get-NetFirewallRule | where {$_.DisplayName -eq "Something"}
```

## Test-ComputerSecureChannel コマンドレットの使い方

概要
`Test-ComputerSecureChannel` コマンドレットは、信頼関係の状態をチェックして、ローカルコンピューターとドメインの間のセキュリティで保護されたチャネルが正しく機能しているかどうかを検証します。

```ps1
# ローカルコンピューター上のAdministratorsグループのメンバーで実行する
# 確認
Test-ComputerSecureChannel

# 修復
Test-ComputerSecureChannel -credential administrator -repair
```

## Gulp的なファイルの変更監視スクリプト
下記の場合、`Register-TaskRunner.ps1`と同じフォルダーに置いた拡張子が`ps1`のファイルを監視する。
また、変更したファイルのユニットテストが同じフォルダにある場合は変更の度に実行される。

```ps1:Register-TaskRunner.ps1
function Register-TaskRunner {
<#
.SYNOPSIS
Watch changing file and run any tasks.

.DESCRIPTION
Powershell task runner

.EXAMPLE
$parent = Split-Path -Parent (Split-Path -Parent $MyInvocation.MyCommand.Path)
Register-Watcher -Folder $parent

.NOTES
https://msdn.microsoft.com/ja-jp/library/system.io.filesystemwatcher(v=vs.110).aspx  
https://mcpmag.com/articles/2015/09/24/changes-to-a-folder-using-powershell.aspx?m=1

#>
    [CmdletBinding()]
    Param
    (
      [Parameter(Mandatory=$true)]
      [string]$Folder,
      
      [Parameter(Mandatory=$false)]
      [string]$Filter = "*.ps1"
    )    

    $VerbosePreference = "Continue"
    $events = Get-EventSubscriber
    foreach ($event in $events) {
        if ($event.EventName -eq "Changed" -and $Folder -eq $event.SourceObject.Path) { 
            Write-Verbose "already exist the event."
            return
        }
    }
    
    $Watcher = New-Object IO.FileSystemWatcher -Property @{ 
        Path = $Folder
        Filter = $Filter
        IncludeSubdirectories = $false
        EnableRaisingEvents = $true
    }

    $changeAction = [scriptblock]::Create('
        $path = $Event.SourceEventArgs.FullPath
        $changedFileName = $Event.SourceEventArgs.Name
        $changeType = $Event.SourceEventArgs.ChangeType
        $timeStamp = $Event.TimeGenerated  
        $testScript = $changedFileName -replace ".ps1", ".Tests.ps1"  
        $testFullPath = "$($Event.MessageData)\$testScript"
        Write-Host "The file $changedFileName was $changeType at $timeStamp"
        if (Test-Path $testFullPath) { 
            Write-Host "Running $testScript" -BackgroundColor DarkGray       
            . $testFullPath
        } else {
            Write-Host "not exist $testScript" -BackgroundColor DarkGray       
        }
    ')

    Register-ObjectEvent -InputObject $Watcher `
                         -EventName "Changed" `
                         -Action $changeAction `
                         -MessageData $Folder
}
$here = Split-Path -Parent $MyInvocation.MyCommand.Path
Register-TaskRunner -Folder $here
```

```ps1:Unregister-TaskRunner.ps1
function Unregister-TaskRunner {
    Get-EventSubscriber | Unregister-Event
}
```

## Githubのトレンドを取得する
このように使用する
`Get-GitHubTrend`
`Get-GitHubTrend -target javascript`
`Get-GitHubTrend -target ruby -length 10`

```ps1:Get-GitHubTrend.ps1
function Get-GitHubTrend {
    Param
    (
       [Parameter(Mandatory=$false)]
       [string]$target = "powershell",

       [Parameter(Mandatory=$false)]
       [string]$baseUrl = "https://github.com",

       [Parameter(Mandatory=$false)]
       [string]$regex = '\"(.*)\"',

       [Parameter(Mandatory=$false)]
       [int]$length = 5,

       [Parameter(Mandatory=$false)]
       [switch]$week,

       [Parameter(Mandatory=$false)]
       [switch]$month
    )
    if ($week) {
        $url = "$baseUrl/trending/$($target)?since=weekly"
    } elseif ($month) {
        $url = "$baseUrl/trending/$($target)?since=monthly"
    } else {
        $url = "$baseUrl/trending/$target"
    }
    
    $geturl = Invoke-WebRequest -Uri $url
    $comment = $geturl.ParsedHtml.body.getElementsByTagName('div') | 
        Where {$_.getAttributeNode('class').Value -eq 'py-1'}
    $hrefs = $geturl.ParsedHtml.body.getElementsByTagName('div') | 
        Where {$_.getAttributeNode('class').Value -eq 'd-inline-block col-9 mb-1'}


    foreach ($html in $hrefs.innerHTML) {
        $result = (([regex]::Matches($html, $regex)).Groups[1].Value)
        $href += @("$baseUrl$result")
    }

    for ($i = 0; $i -lt $length; $i++) {
        if ($href[$i] -eq $null) { return }
        Write-Host $hrefs[$i].innerText  -BackgroundColor DarkGreen
        Write-Host $comment[$i].innerText
        Write-Host $href[$i] `n
    }
}
```

## Hashを配列のように扱う

```ps1
$myHash = @{}
$myHash["a"] = 1
$myHash["b"] = 2
$myHash["c"] = 3

foreach($key in $($myHash.keys)){
    $myHash[$key] = 5
}
```
