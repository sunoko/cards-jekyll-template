---
layout: post
title: "Docker Tricks"
date: 2017-05-07 22:43:23
image: '/assets/img/linter/errors-list.png'
description: 'docker-compose.ymlとDockerfileの設定まとめ'
tags:
- docker
categories:
twitter_text: 'Valide seu código em JS/ES6 em busca de erros e melhore a sua qualidade.'
introduction: 'docker-compose.ymlとDockerfileの設定まとめ'
---

# Docker
![](http://applech2.com/wp-content/uploads/2016/07/Docker-logo-icon.jpg)

## docker commands

```rb
# イメージ取得
docker pull ruby:2.4.1-alpine

# イメージ確認
~/ $ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ruby                2.4.1-alpine        5eadd5d1419a        12 days ago         56.3 MB

# コンテナ確認
docker ps

# コンテナ削除
docker rm container_id

# イメージ削除
docker rmi image_id
```

## docker-compose commands
### Start

```rb
# Runs a one-time command
docker-compose run
# Remove container after run
--rm
# DB周りの初期設定
#rails:db:create, rails db:schema:load, rails db:seed
rails db:setup

# app, rails db:setup実行、
docker-compose run --rm app rails db:setup

# Builds, (re)creates, starts
docker-compose up

# Stops and removes containers
docker-compose down

# Starts existing containers 
docker-compose start

# Stops running containers without removing them
docker-compose stop

# コンテナの確認
docker-compose ls

#コンテナの削除
docker-compose rm
```

### Gemsを更新した場合

```ruby
docker run --rm -v ${PWD}:/home -w /home ruby:2.4 bundle lock (Append --update to also update gems.)

# コンテナを開始前にイメージを構築(--build)
docker-compose up -d --build
```

```ruby
# Get a shell in container
docker-compose exec app bash

# Logging can be configured→default outputs everything on stdout/stderr
docker-compose logs
    
# Run the rails minitest.
docker-compose run --rm app rails test
```

## [Dockerfile](http://docs.docker.jp/engine/reference/builder.html#from)
イメージを作り上げる命令を記述する
`docker build`でイメージを構築

1. FROM: ベース・イメージ を指定  
	`FROM <イメージ>`
2. RUN: イメージ上でコマンドを実行  
	`RUN /bin/bash -c 'source $HOME/.bashrc ; echo $HOME'
`
3. CMD: 構築時には何もしませんが、イメージで実行するコマンドを指定  
		コンテナ実行時のデフォルトを提供  
		`CMD echo "This is a test." | wc -`
4. EXPOSE: portをコンテナが実行時にリッスンすることをDockerに伝える  
	`EXPOSE <port> [<port>...]`
5. COPY: sourceのfileやdirectoryをコンテナ内のファイルシステム上にコピー  
	`COPY ソース 送信先(絶対パスor WORKDIRからの相対パス)` 
6. WORKDIR: RUN,CMD,ENTRYPOINT,COPY,ADD実行時の作業ディレクトリ
	`WORKDIR /a`

### sample

`Dockerfile`

```
FROM nginx:1.11
COPY . /home/config/
```

`Dockerfile`

```
FROM ruby:2.4

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    nodejs \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/src/
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPY . .

EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]
```

`.dockerignore`

```
tmp/
log/
config/nginx/
```

## docker-compose.yml
[networks](http://docs.docker.jp/compose/compose-file.html#network-configuration-reference)、[volumes](http://docs.docker.jp/compose/compose-file.html#volume-configuration-reference)、[services](http://docs.docker.jp/compose/compose-file.html#id14)を定義する

#### build: 構築時に適用するオプション

```
build: ./dir
```

#### enviroment

```
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```

#### volumes: マウント・パス指定

```
volumes:
    - ./cache:/tmp/cache
```

#### depends_on: docker-compose up を実行したら、依存関係のある順番に従ってサービスを起動
    
```
depends_on:
      - db
      - redis
```
#### commnad: デフォルトのコマンドを上書き
    
```
    command: [bundle, exec, thin, -p, 3000]
```
#### ports: 公開用のポート

```
ports:
 - "3000"
 - "8000:8000"
```
#### image: コンテナを実行時に元となるイメージを指定

```
image: redis
image: ubuntu:14.04
```

### sample

`docker-compose.yml`

```

```

