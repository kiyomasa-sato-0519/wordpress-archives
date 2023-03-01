---
title: "WordPressをインストールした後に真っ先に行ったこと"
date: "2017-12-23"
categories: 
  - "未分類"
tags: 
  - "php"
  - "wordpress"
  - "セキュリティ"
---

内容に関しては掲題のとおり。 WordPressをインストールして真っ先にinstall系のファイルを消した。 理由はこれを悪用して不正なアクセスによりサイトを改ざんされるのを防ぐため。

```
$ sudo rm wp-admin/install.php wp-admin/install-helper.php
```

また、/wp-adminへのアクセス制限もかけた。 ただしawsで構築しelbを挟んでる関係で通常の書き方ではリモートIPを取ることが出来ない。 そのため例として下記のようにenvを介して制限をかけた

```
SetEnvIf X-Forwarded-For "^000\.000\.000\.000$" allowed_access
<Location "/wp-admin">
order deny,allow
deny from all
allow from env=allowed_access
</Location>
```

ただしこの方法、一つ問題があって悪意ある攻撃者がX-Forwarded-Forに我が家のIPを入れて送ってくると管理画面が表示できる。 が、そもそも我が家のIPを知る方法がないだろうから問題ないと思われる。 企業が公開しているサービスなんかだと企業のIPで試行して攻撃できると思うので過信は禁物。
