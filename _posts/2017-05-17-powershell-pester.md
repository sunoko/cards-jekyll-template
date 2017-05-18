---
layout: post
title: "Powershellでユニットテスト"
date: 2017-05-17 21:26:57
description:
tags:
- powershell
twitter_text:
introduction:
---

## Mockの使い方と使用例
* Out-Hostをテストしようとすると、return valueは空のためテストが難しい
* 同じくGet-Dateのように変動的な結果のもののテストも難しい
* そんな時にMockを使用する

### Sample Code

```powershell
function Write-TimeAndWord ([string] $word) {
	$date_01 = Get-Date
	if($word -eq "TEST_01"){ $date_02 = Get-Date }
	"TEST_" + $word | Out-Host
}
```

```powershell
Describe 'Write-TimeAndWord' {
    Context "First Test" {
        $word = "TEST_01"
        # Get-DateをMockで上書きする
        # Get-Dateを実行すると結果が"2017/03/31 06:00:00"に固定できる
        Mock Get-Date { return [String] "2017/03/31 06:00:00"}
        # Out-HostをMockで上書きする
        # "-parameterFilter"でパラメーターが指定したもの("2017/03/31 06:00:00TEST")かチェックする
        Mock Out-Host -verifiable -parameterFilter { $_ -eq "TEST_TEST_01"}
        It 'return value' {
            # テストしたい関数を実行する
            Write-TimeAndWord $word
        }
        It "Assert-VerifiableMocks" {
            # "-verifiable"があるMockが呼び出されたかチェックする
            # 関数内でOut-Hostが呼び出されていれば問題なし
            Assert-VerifiableMocks
        }
        It "Assert-MockCalled" {
            # 関数内でMockのGet-Dateが2回呼び出されたかチェックする
            Assert-MockCalled -CommandName Get-Date -Times 2
            # 関数内でMockのOut-Hostが1回呼び出されたかチェックする
            Assert-MockCalled -CommandName Out-Host -Times 1
        }
    }
    Context "Second Test" {
        $word = "TEST_02"
        Mock Get-Date { return [String] "2017/03/31 06:00:00"}
        Mock Out-Host -parameterFilter { $_ -eq "TEST_TEST_02"}
        It "All Test" {
            Write-TimeAndWord $word   
            Assert-VerifiableMocks         
            Assert-MockCalled -CommandName Get-Date -Times 1
            Assert-MockCalled -CommandName Out-Host -Times 1
        }
    }
} 
```

実行方法

```powershell
Invoke-Pester .\Write-TimeAndWord.Tests.ps1 
```
### Sample Codeのまとめ
* 引数を固定できる
→ `-parameterFilter`

* コマンドレットや関数を上書きすることができる
→ `Mock Get-Date { return [String] "2017/03/31 06:00:00"}`

* 再現性あるテスト
→ `Get-Date`の結果を固定、`Out-Host`のパラメーターを固定

* 検証が難しい場合でも呼びだしと出力の検証ができる
→ `Assert-VerifiableMocks`、`Assert-MockCalled -CommandName Out-Host -Times 1`

## CodeCoverageの使い方と使用例

```powershell
Invoke-Pester .\Write-TimeAndWord.Tests.ps1 -CodeCoverage .\Write-TimeAndWord.ps1

Describing Write-TimeAndWord
   Context First Test
    [+] return value 69ms
    [+] Assert-VerifiableMocks 15ms
    [+] Assert-MockCalled 20ms
   Context Second Test
    [+] All Test 56ms
Tests completed in 162ms
Passed: 4 Failed: 0 Skipped: 0 Pending: 0 Inconclusiv

Code coverage report:
Covered 0.00% of 5 analyzed commands in 1 file.

Missed commands:

File                  Function          Line Command
----                  --------          ---- -------
Write-TimeAndWord.ps1 Write-TimeAndWord    2 $date_01
Write-TimeAndWord.ps1 Write-TimeAndWord    3 if($word
Write-TimeAndWord.ps1 Write-TimeAndWord    3 $date_02
Write-TimeAndWord.ps1 Write-TimeAndWord    4 "TEST_"
Write-TimeAndWord.ps1 Write-TimeAndWord    4 Out-Host
```

* 変数のCodeCoverageをどうやってCoverするのか不明です。どなたかご教授いただけないでしょうか。

## Test環境初期化スクリプト
例：SharePointモジュールUnitテスト
Powershell Module Folder/

┣ Scripts/
┃    ┗script_01.ps1
┃    ┗script_02.ps1
┃    ┗script_03.ps1
┃    ┣Tests/
┃        ┗test_01.Tests.ps1
┃        ┗test_02.Tests.ps1
┃        ┗test_03.Tests.ps1
┃        ┗TestInitialize.ps1
┃        ┗TestInitialize.json

```powershell
$parent = Split-Path -Parent (Split-Path -Parent $MyInvocation.MyCommand.Path)
$scripts = (Get-ChildItem $parent -Filter "*.ps1").FullName
foreach($script in $scripts){
    Invoke-Expression (Get-Content $script -Raw)
}
$Script:TestConfig = Get-Content "${PSScriptRoot}\TestConfiguration.json" | ConvertFrom-Json
```

```json
[
    {
        "SiteUrl":  "SharePoint Site URL",
        "LoginUserName":  "username",
        "LoginPassword":  "password",
        "ListName":  "Test-Tmp-List",
        "SiteName":  "Test-Tmp-Site",
        "testData":
        [
            {
                "DisplayName":  "Region",
                "Name":  "Region",
                "Type":  "Text",
                "Data": "Japan"
            },
            {
                "DisplayName":  "Priority",
                "Name":  "Priority",
                "Type":  "Choice",
                "Data": "High"
            },
            {
                "DisplayName":  "Approval Date",
                "Name":  "Approval Date",
                "Type":  "DateTime",
                "Data": "2017/03/10"
            },
            {
                "DisplayName":  "Kebin Budget",
                "Name":  "Kebin Budget",
                "Type":  "Currency",
                "Data": "3000"
            }
        ]
    }
]
```


```powershell
. "${PSScriptRoot}\TestInitialize.ps1"
# モジュールのテストコード
```

```powershell
$VerbosePreference = 'SilentlyContinue'
$Tests  = @(Get-ChildItem -Path $PSScriptRoot\*.ps1 -Exclude "*FullTest*" -ErrorAction SilentlyContinue)

foreach ($test in $Tests) {
    try {
        . ($test.FullName)
    } catch {
        Write-Error -Message "Failed to import function $($import.fullname): $_"
    }
}
```
