---
title: "docker & rails6で開発を始める"
date: "2020-02-02"
categories: 
  - "未分類"
tags: 
  - "docker"
  - "docker-compose"
  - "rails"
---

表題の通り。０から始める手順。

今までの集大成、というほど大げさなものではないが現時点での最新手順。

```
$ mkdir rails
$ vim docker-compose.yml
# 中身 docker-compose.yml
$ vim Dockerfile
# 中身 Dockerfile
$ docker-compose up
# この時点ではrailsが入っていないのでwebがエラーで終了する

$ vim Gemfile
# gem "rails"の#を外し有効化

$ docker-compose run web bash
# ここからDocker内
$ bundle install
$ bundle exec rails new . -d mysql --skip-turbolinks --skip-test --webpack --skip-sprockets
# なにか問われたらすべてYes
```

## docker-compose.yml

```
version: '3'
services:
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/app
      - bundle-data:/usr/local/bundle
    ports:
      - 3000:3000
    depends_on:
      - db
    tty: true
    stdin_open: true
  db:
    image: mysql:5.7
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
volumes:
  db-data:
    driver: local
  bundle-data:
    driver: local
```

## Dockerfile

```
FROM ruby:2.7.0

ENV APP_DIR "/app"

RUN curl -sL https://deb.nodesource.com/setup_12.x | bash

RUN apt-get install -y \
  nodejs \
  inotify-tools

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -

RUN npm i -g yarn

RUN mkdir $APP_DIR
WORKDIR $APP_DIR
COPY . $APP_DIR

RUN gem update --system
RUN gem update
```
