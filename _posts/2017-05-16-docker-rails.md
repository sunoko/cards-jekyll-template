---
layout: post
title: "Docker上でRails開発環境を構築する"
date: 2017-05-16 18:48:33
description: "rails newして、ブラウザでアクセスするまで"
tags: 
- docker
- rails
twitter_text:
introduction: "rails newして、ブラウザでアクセスするまで"
---

## Dockerfile
* ベースイメージはruby:2.4を使用します。
* ベースイメージの更新を行います。
* ENTRYKITを使用します。ENTRYKITとは、

```
FROM ruby:2.4
RUN apt-get update -qq && \
    apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /app
WORKDIR /app
ADD Gemfile /app/Gemfile
ADD Gemfile.lock /app/Gemfile.lock
ADD . /app

RUN bundle config build.nokogiri --use-system-libraries
EXPOSE 5000

ENV ENTRYKIT_VERSION 0.4.0
RUN wget https://github.com/progrium/entrykit/releases/download/v${ENTRYKIT_VERSION}/entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz \
  && tar -xvzf entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz \
  && rm entrykit_${ENTRYKIT_VERSION}_Linux_x86_64.tgz \
  && mv entrykit /bin/entrykit \
  && chmod +x /bin/entrykit \
  && entrykit --symlink
ENTRYPOINT [ \
  "prehook", "ruby -v", "--", \
  "prehook", "bundle install -j3 --quiet", "--"]
```

## docker-compose.yml
* app, spring, db, dataコンテナー4つを作成します。
* appはrailsが実行されるコンテナーです。
* springはなくてもいいです。rails開発を高速化したい場合は使用しましょう。
* dbはpostgresが実行されるコンテナーです。appコンテナーから参照されます。
* dataはデータを格納するコンテナーです。app, dbコンテナーから参照されます。app, dbコンテナーにデータを保存する場合は不要です。
* `db:/var/lib/postgresql/data`はpostgresのデータを格納します。
* `bundle:/usr/local/bundle`は`bundle install`でインストールされるデータを格納します。

```yml
version: '2'
services:
  app: &app_base
    build:
      context: .
    command: "bundle exec rails s -p 3000 -b '0.0.0.0'"
    environment:
      - "DATABASE_HOST=db"
      - "DATABASE_PORT=5432"
      - "DATABASE_USER=postgres"
      - "DATABASE_PASSWORD=mysecretpassword1234"
    volumes:
      - ".:/app"
    volumes_from:
      - data
    ports:
      - "3000:3000"
    depends_on:
      - db
    tty: true
    stdin_open: true

  spring:
    <<: *app_base
    command: "bundle exec spring server"
    ports: []
    tty: false
    stdin_open: false

  db:
    image: "postgres:9.5"
    environment:
      - "POSTGRES_USER=postgres"
      - "POSTGRES_PASSWORD=mysecretpassword1234"
    volumes_from:
      - data

  data:
    image: "busybox"
    volumes:
      - "db:/var/lib/postgresql/data"
      - "bundle:/usr/local/bundle"

volumes:
  db:
    driver: local
  bundle:
    driver: local 
```

## Gemfile
中身はこれだけ書いておきましょう。

```rb
source 'https://rubygems.org'
gem 'rails', '~> 5.0.1'
```

## Gemfile.lock
中身は何も書かなくていいです。

## 使い方
Dockerfile, docker-compose.yml, Gemfile, Gemfile.lockを同じ階層に準備して、その階層に移動したら下記実行しましょう。

```bash
# 1. Railsプロジェクト作成, イメージ作成
docker-compose run --rm app rails new . --force --database=postgresql --skip-bundle

# 2. Gemfile.lock更新
docker-compose run --rm app bundle install

# 3. Gemfileの更新結果をイメージに反映
docker-compose build

# DB設定
# 4. config/docker-compose.yml更新

# 5. DB作成
docker-compose run --rm app rails db:create

# 6. 起動
docker-compose up
```

## config/database.yml
DB設定の時にこちらをコピーします。

```yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV.fetch('DATABASE_USER') { 'root' } %>
  password: <%= ENV.fetch('DATABASE_PASSWORD') { 'password' } %>
  host: <%= ENV.fetch('DATABASE_HOST') { 'localhost' } %>
  port: <%= ENV.fetch('DATABASE_PORT') { 5432 } %>

development:
  <<: *default
  database: app_development

test:
  <<: *default
  database: app_test

production:
  <<: *default
  database: app_production
```