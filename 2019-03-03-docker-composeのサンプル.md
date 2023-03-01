---
title: "docker-composeのサンプル"
date: "2019-03-03"
categories: 
  - "未分類"
tags: 
  - "docker"
  - "docker-compose"
---

docker-composeのサンプルポイントは以下の点

- bundle installしたディレクトリを永続化する じゃないとコンテナ立ち上げるたびにbundle installが必要になる
- depends\_onでdbを先に起動させる
- dbのdataディレクトリも永続化させる

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
