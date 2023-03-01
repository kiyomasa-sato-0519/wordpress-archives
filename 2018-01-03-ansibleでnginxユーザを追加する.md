---
title: "ansibleでnginxユーザを追加する"
date: "2018-01-03"
categories: 
  - "未分類"
tags: 
  - "ansible"
  - "nginx"
---

nginxはインストールしたものの、ユーザやグループは自動では作られていなかった。 そのためansibleでユーザとグループを追加することにした。

```
- name: add nginx group
  group:
    name: nginx

- name: add nginx user
  user:
    name: nginx
    group: nginx
```

uidとgidは競合すると面倒なのであえて固定はしていない。
