---
title: "vagrantのデフォルトSSHポートを変更する"
date: "2018-04-30"
categories: 
  - "未分類"
tags: 
  - "ssh"
  - "vagrant"
---

vagrantはデフォルトで空きポートをsshのポートに使おうとする。

そのためvagrantの起動順番によってはssh-configで定義したポートが一致しないということが起こり得る。

そのためssh-configに出力されるportかつデフォルトのssh用のポートフォワード設定を変更する

例として2727でportを固定する。`id: "ssh"`をつける。 これをつけないとたとえ22にポートフォワードする設定を書いたとしても空きポートで22にポートフォワードしようとしてしまうので無駄にホストマシンのポートを埋めてしまう。

```
config.vm.network :forwarded_port, host: 2727, guest: 22, id: "ssh"
```
