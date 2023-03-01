---
title: "rails5.2とDockerとwebpackerとbootstrap4【インストール編】"
date: "2019-03-03"
categories: 
  - "未分類"
tags: 
  - "docker"
  - "rails"
  - "webpacker"
---

タイトルの通り、この記事ではDocker上で動くrails+webpackerのちょっとした経験を記述しようと思う。  
探してみるとwebpackerをrails上で動かす記事はまぁまぁある。  
だがしかし、webpackerがcssをjsにbundleする関係で単純にそのまま使うとFOUC問題が起きる。

何故かそれに触れているサイトはほぼ皆無。

なので自分でなんとかしてみた、というのがこの記事の趣旨である。  
ちなみにFOUCとは別サイトでは下記のように説明されている。

**FOUC(Flash of Unstyled Content)とは、Webページへアクセスした直後、CSSによるデザインが有効でないページが一瞬だけ表示される現象のことです。**

結論から言うと、webpackerのjsにbundleされるcssを分離してstaticファイルとして読み込めるようにするのが肝である

まずDocker上でrails + webpackerが動く状態を作らなければならない。  
準備として下記のGemfileを用意しておく。  
この記事を書いたときのrailsの最新バージョンは5.2.2だった。

**Gemfile**

```none
# frozen_string_literal: true
source "https://rubygems.org"
git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem "rails"
```

次にDockerfileを用意する。

rubyの2.6で最新安定バージョンは存在するがparserがwarningを吐くので2.5系最新安定バージョンの2.5.3を利用する。

**Dockerfile**

```none
FROM ruby:2.5.3

ENV APP_DIR "/app"

RUN curl -sL https://deb.nodesource.com/setup_10.x | bash

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

webpackerでnode系のインストールが必要なのでrubyイメージにnodeをインストールしている。  
最小構成のためカスタマイズは構成にあったものを用意して欲しい。  
次にdocker-compose用のファイルを用意する。[docker-composeのサンプル](https://www.null-engineer.com/2019/03/03/docker-compose%e3%81%ae%e3%82%b5%e3%83%b3%e3%83%97%e3%83%ab/)

dbはmysqlを選択する。 するとこうなるはず。

```none
$ tree .
.
├── Dockerfile
├── Gemfile
└── docker-compose.yml
```

次にdocker内でrailsの準備をしていく。  
`docker-compose build --no-cache` などしてまずimageを作成する。  
次に `docker-compose run web bash` でdocker内のwebコンテナに接続する。

そのため、以降はコンテナ内で作業しているものと思って欲しい。

ファイルの編集のみホストマシンから行っている。  
このあたりの命名はdocker-compose.ymlに従って適宜読み替えて欲しい。  
ちなみに筆者はmysql:5.7イメージを利用した。

閑話休題。

dockerコンテナにbashで接続したらrailsをセットアップする。  
まず、 `bundle install` でrailsをインストールする。

```none
$ bundle exec rails new . -d mysql --skip-turbolinks --skip-test --webpack --skip-sprockets
```

ここでGemfileを上書きしても問題ないか聞かれると思うが筆者は上書きを選択した。  
また、cssやjsの管理はwebpacker上で行うのでsprocketsは必要ない。  
turbolinksもあまり良い噂を聞かないので最初からインストールしない。  
次にwebpacker4をインストールする。  
Gemfileのwebpackerでバージョンを指定する

```none
gem 'webpacker', '>= 4.0.x'
```

```none
# webpacker4系を使うために@nextで指定
$ yarn add bootstrap@4 jquery@3 popper.js
$ yarn add -D webpack-dev-server
$ bundle exec rails webpacker:install
```

長くなったので記事を別ける。  
[次項へ続く](https://www.null-engineer.com/2019/03/03/rails5-2%e3%81%a8docker%e3%81%a8webpacker%e3%81%a8bootstrap4%e3%80%90%e8%a8%ad%e5%ae%9a%e7%b7%a8%e3%80%91/)
