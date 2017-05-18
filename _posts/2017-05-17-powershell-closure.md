---
layout: post
title: "PowershellでClosureを使ってみた"
date: 2017-05-17 21:26:38
description:
tags:
- powershell
twitter_text:
introduction:
---

## Powershellでクロージャー使用例

非常に長いパラメータセットになりがちなので、データとコードをオブジェクトにラップする　
メッセージ作成スクリプトの再利用単純化

```powershell
function Get-MessageObject
{
   param
   (
      [String]$MsgType,
      [String]$Message,
      [String]$CurrentDay,
      [String]$CurrentTime,
      [String]$DayOfWeek
   )
   $Properties = @{
                    'MessageType' = $MsgType
                    'Message'    = $Message
                    'CurrentDay'    = $CurrentDay
                    'CurrentTime'    = $CurrentTime
                    'DayOfWeek'  = $DayOfWeek
                  }
   New-Object -TypeName psobject -Property $Properties
}

function New-MessageGenerator
{
   param
   (
      [String]$MsgType
   )
   $Day = (Get-Date).ToLongDateString()   
   $Time = (Get-Date).ToLongTimeString()
   $DayOfWeek = (Get-Date).DayOfWeek
   {
      param
      (
         [String]$Message
      )
      Get-MessageObject -MsgType $MsgType -Message $Message -CurrentDay $Day -CurrentTime $Time -DayOfWeek $DayOfWeek
   }.GetNewClosure()
}

$function:global:getInfoMsg = New-MessageGenerator -MsgType 'Information'
$function:global:getWarnMsg = New-MessageGenerator -MsgType 'Warning'
$function:global:getErrorMsg = New-MessageGenerator -MsgType 'Error'
$function:global:getExceptionMsg = New-MessageGenerator -MsgType 'Exception'

getInfoMsg -Message 'This is a Information.'
getWarnMsg -Message 'This is a Warning.'
getErrorMsg -Message 'This is a Error.'
getExceptionMsg -Message 'This is a Exception.'
```
