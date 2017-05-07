---
layout: post
title: "Bash Tricks"
date: 2017-05-07 19:33:48
image: '/assets/img/dica-rapida-2/main.png'
description: 'Aprenda como ser mais social, ter maior relevância nas redes sociais e atrair mais usuários.'
tags:
- bash
categories:
twitter_text: 'Aprenda a usar as meta tags sociais.'
introduction: "Aprenda como ser mais social, ter maior relevância nas redes sociais e atrair mais usuários. Para isso, basta criar as meta tags corretas."
---

# Bash Tricks
![](http://cdn-img.easyicon.net/png/10971/1097179.gif)

## テンプレートが書かれたファイルを複数作成する

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

このようなファイルが30個作成される

```rb:test_1.rb
require 'test/unit'

def methodName

end

class TestClassName < Test::Unit::TestCase

	def TestMethodName
	
	end
	
end
```

## bash commandエイリアス作成

```bash
alias dcm='docker-compose'
```

## .bashrc再読み込み

```bash
bash -l
```

## bashプロンプト変更

```bash
# 一時的変更
export PS1="\[\e[3;34m\]\w/ \[\e[0m\]$ "

# bash起動時に反映させる
echo "export PS1='\[\e[3;34m\]\w/ \[\e[0m\]$ '" > ~/.bash_rc
echo "export PS1='\[\e[3;34m\]\w/ \[\e[0m\]$ '" > ~/.bash_profile
```
