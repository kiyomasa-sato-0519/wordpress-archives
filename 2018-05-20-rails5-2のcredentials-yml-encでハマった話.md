---
title: "Rails5.2のcredentials.yml.encでハマった話"
date: "2018-05-20"
categories: 
  - "未分類"
tags: 
  - "capistrano"
  - "git"
  - "rails5-2"
---

Rails.5.2から_secrets_.ymlがcredentials.yml.encに変わったらしい。

やり方ととしては(rbenv系を入れていると想定して書いている)

```
 EDITOR="vi" bundle exec rails credentials:edit
```

でEDITORは適宜好きなものを指定する。

するとconfig/master.keyが生成される。

ここからが本題だが

capistranoでデプロイ環境を作ったときにどうもこのkeyファイルが原因でcapistranoが途中で落ちる。

shared/config/master.keyに鍵ファイルをscpしてデプロイすればうまくいくはず、と海外サイトには記載があったがどうしてもうまくいかない。

keyを作り直したり変更したりしても変わらず試行錯誤の上、とても単純な理由でうまく行っていないことがわかった。

それはgit上に登録したcredentials.yml.encとローカルのmaster.keyが一致していない、つまり最新版をコミットしていないのが原因だった。

capistranoがgitからリモートサーバーにファイルを作るから当然といえば当然だがdeployするときには必ずコミットしないと思わぬ罠を踏むと知った。

Rails5.2.0は情報が少なくて罠を踏んだ時のユースケースが少ないから現状個人的にはあまりおすすめしない。
