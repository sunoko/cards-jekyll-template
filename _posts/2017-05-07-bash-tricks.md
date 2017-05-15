---
layout: post
title: "忘れがちで便利なBashまとめ"
date: 2017-05-07 19:33:48
image: '/assets/img/dica-rapida-2/main.png'
description: '忘れがちで便利なBashまとめ'
tags:
- bash
categories:
twitter_text: '忘れがちで便利なBashまとめ'
introduction: ""
---

## テンプレートが書かれたファイルを複数作成する
下記サンプルではRubyのユニットテスト（Railsではなく）するためのファイルを複数作成します。
test_1.rbというファイル名のファイルが30個作成されます。ファイルの中身はユニットテストをするために必要な基本的なものが書かれています。
ファイルの中身は別として、ファイルを複数作成したい時には早速使えるかと思います。

```bash
initial="
require 'test/unit'\n\n
def methodName\n\n
end\n\n
class TestClassName < Test::Unit::TestCase\n\n
\tdef TestMethodName\n\n
\tend\n\n
end
"

for ((i=1; i < 31; i++)); do
    echo -e $initial > test_$i.rb
done
```
上記スクリプトファイルのあるディレクトリで実行すると同じ階層に下記のようなファイルが30個作成されます。

```rb
require 'test/unit'

def methodName

end

class TestClassName < Test::Unit::TestCase

	def TestMethodName
	
	end
	
end
```

## bash commandエイリアス作成
Docker使ってると想像以上に`docker-compose`をタイプしている気がするので省略してしまいましょう。
なんなら`d`だけでもいいかもしれませんね。

```bash
alias dcm='docker-compose'
```

## .bash_profile再読み込み
普段使いすぎてる感のあるコマンドや流れ作業のようなコマンド群を、エイリアスやら関数にして.bash_profileに記載した際に
すぐ動作確認できます。

```bash
bash -l
```

## bashプロンプト変更
自己満足しか存在しません。でも僕たちはコンピュータではないのでビジュアルは大事にしたいです。

```bash
# 一時的変更
export PS1="\[\e[3;34m\]\w/ \[\e[0m\]$ "

# bash起動時に反映させる
echo "export PS1='\[\e[3;34m\]\w/ \[\e[0m\]$ '" > ~/.bash_rc
echo "export PS1='\[\e[3;34m\]\w/ \[\e[0m\]$ '" > ~/.bash_profile
```
