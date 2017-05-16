---
layout: post
title: "忘れがちで便利なPowershellまとめ"
date: 2017-05-07 21:31:05
description: ""
tags:
- powershell
twitter_text: ""
introduction: ""
---

## クロージャー
クロージャーってなんかカッコいい。コードレビューで、なんでクロージャー使ってんの？の質問に答えられないならオススメしません。  
理にかなったクロージャーはカッコいい。

```powershell
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
`?`は`Where-Object`コマンドと同じですね。

```powershell
# 偶数のみ取り出す
1..10 | ? { $_ % 2 -eq 0 }
```

## map
`%`は`Foreach-Object`コマンドと同じですね。

```powershell
# 全てに2をかける
1..10 | % { $_ * 2 }
```

## portを使ってるプロセス
`Select-String`は正規表現も使用できるので柔軟性が増しますね。

```powershell
netstat -aon | Select-String ".0.0:4000"
netstat -aon | Select-String -Pattern ".0.0:4000$"
```

## &{}と.{}の違い
`&{}`はスクリプト内部で処理を留める  
`.{}`は元のスコープに影響する

```powershell
$hoge = $null
& {$hoge = 4}
# [String]::IsNullOrEmpty($hoge)
# True

$hoge = $null
. {$hoge = 4}
# [String]::IsNullOrEmpty($hoge)
# False

$hoge
# 4
```

## Firewall設定確認

```powershell
Get-NetFirewallRule | where {$_.DisplayName -eq "Something"}
```

## Test-ComputerSecureChannel コマンドレットの使い方
`Test-ComputerSecureChannel` コマンドレットは、信頼関係の状態をチェックして、ローカルコンピューターとドメインの間のセキュリティで保護されたチャネルが正しく機能しているかどうかを検証します。  
会社でOutlookが更新されないなぁって時に確認するといいかもしれません。  
コンパネでネットワークアダプターを無効にして、再度有効にすると調子がようくなる場合もありました。

```powershell
# ローカルコンピューター上のAdministratorsグループのメンバーで実行する
# 確認
Test-ComputerSecureChannel

# 修復
Test-ComputerSecureChannel -credential administrator -repair
```

## Gulp的なファイルの変更監視スクリプト
下記の場合、`Register-TaskRunner.ps1`と同じフォルダーに置いた拡張子が`ps1`のファイルを監視する。
また、変更したファイルのユニットテストが同じフォルダにある場合は変更の度に実行される。  
どこのファイルを監視するかやユニットテストを実行するかは調整できます。  
また、変更を検知した際に実行されるコマンドも調整可能です。

```powershell
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

変更監視をやめたい時には`Unregister-TaskRunner`で。

```powershell
function Unregister-TaskRunner {
    Get-EventSubscriber | Unregister-Event
}
```

## Githubのトレンドを取得する
このように使用する  
`Get-GitHubTrend`  
`Get-GitHubTrend -target javascript`  
`Get-GitHubTrend -target ruby -length 10`  

```powershell
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
Hashをループで回したい時に使えるかと。

```powershell
$myHash = @{}
$myHash["a"] = 1
$myHash["b"] = 2
$myHash["c"] = 3

foreach($key in $($myHash.keys)){
    $myHash[$key] = 5
}
```
