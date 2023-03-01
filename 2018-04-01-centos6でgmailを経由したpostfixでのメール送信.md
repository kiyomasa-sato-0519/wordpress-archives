---
title: "CentOS6でGMailを経由したpostfixでのメール送信"
date: "2018-04-01"
categories: 
  - "未分類"
tags: 
  - "centos6"
  - "gmail"
  - "postfix"
  - "vagrant"
---

vagrant環境で構築したpostfixからメールを送りたかった。 調べたところGmailを経由して送信するのが簡単そうだったので対応した。

メール送信に必要なモジュールのインストール

```
sudo yum -y install cyrus-sasl-md5 cyrus-sasl-plain
```

postfix設定

```
sudo vi /etc/postfix/main.cf
```

```
inet_interfaces = all
inet_protocols = ipv4

relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_tls_CApath = /etc/pki/tls/certs/ca-bundle.crt
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_tls_security_options = noanonymous
smtp_sasl_mechanism_filter = plain
```

リレーの設定 予めGmailのアカウント設定からアプリパスワードを発行して適当な名前をつけてパスワードを記録しておく。

```
sudo vi /etc/postfix/sasl_passwd
```

```
[smtp.gmail.com]:587 example@gmail.com:アプリパスワード
```

```
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap hash:/etc/postfix/sasl_passwd
sudo /etc/init.d/postfix restart
```

mailコマンド実行なければインストール

```
sudo yum install -y mail
```

テストメール送信

```
mail example@gmail.com
Subject: test
test
.
```

最後に.を打つと送信される

maillogを見て

```
status=sent (250 2.0.0 OK 1522571906 j21sm19989236pgn.61 - gsmtp)
```

が確認できればメール送信は成功している。
