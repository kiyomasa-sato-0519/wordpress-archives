---
title: "ubuntu16 + lsyncdで相互同期した話"
date: "2018-01-12"
categories: 
  - "未分類"
tags: 
  - "ansible"
  - "lsyncd"
---

lsyncdでサーバー間の相互同期をはかって色々ハマったのでその件について。 まず構成としてはrsyncd + lsyncdと一般的な同期用のミドルウェアを利用する。

ところが設定してみてもlsyncdが起動せず、しかもログにも何も出ないしstartコマンド投げたら起動OKと出ている割に起動していない。 困り果てていたのだが これを投げ見てみてコメント部分でエラー？が出ていたようなので消したら動くようになった。 具体的にどこが原因だったのかついぞわからなかったがコメント部分に問題があったのか，の位置に問題があったのかそこら辺が原因だと推測している。

```
lsyncd -nodaemon -log scarce /etc/lsyncd/lsyncd.conf.lua
Error: error loading /etc/lsyncd/lsyncd.conf.lua: /etc/lsyncd/lsyncd.conf.lua:11: unexpected symbol near '?'
```

そして最終的には下記のようなansibleでうまく行った。 ちなみにroot同士がsshでつながるように鍵を配置している。 また、awsでport873のインバウンドを許可している (アウトバウンドは全許可しているため割愛)

```
$ tree roles/lsyncd/
roles/lsyncd/
├── files
│   ├── rsync
│   └── rsyncd.conf
├── tasks
│   └── main.yml
└── templates
    └── lsyncd.conf.lua.j2
```

roles/lsyncd/files/rsyncはインストール後のFileからRSYNC\_ENABLEを下記に変更しただけ

```
RSYNC_ENABLE=true
```

これがroles/lsyncd/tasks/main.yml本体

```
- name: sysctrl change
  sysctl:
    name: fs.inotify.max_user_watches
    value: 32768
    state: present
- name: create log directory
  file: path=/var/log/lsyncd state=directory owner=root group=root mode=0755

- name: rsyncd install
  apt: name={{ item }} state=installed update_cache=yes
  with_items:
    - rsync
    - lsyncd

- name: template lsyncd conf
  template: src=templates/lsyncd.conf.lua.j2 dest=/etc/lsyncd/lsyncd.conf.lua
  register: lsyncd

- name: syncd started
  service:
    name: lsyncd
    state: restarted
    enabled: yes
  when: lsyncd.changed

- name: copy rsync conf
  template: src=files/rsyncd.conf dest=/etc/rsyncd.conf
  register: rsyncd_conf

- name: copy rsync conf
  template: src=files/rsync dest=/etc/default/rsync
  register: rsync_conf

- name: syncd started
  service:
    name: rsync
    state: restarted
    enabled: yes
  when: rsyncd_conf.changed or rsync_conf.changed
```

roles/lsyncd/templates/lsyncd.conf.lua.j2

```
# 新規作成
settings{
    logfile = "/var/log/lsyncd/lsyncd.log",
    statusFile = "/tmp/lsyncd.stat",
    statusInterval = 1,
    delay = 0,
    nodaemon  = false
}
sync{
    default.rsync,
    source="/var/www/wordpress",
    target="{{ lsyncd_mirror_host }}::wordpress",
    delete="running",
    init = false,
    rsync = {
      archive = true,
      links = true,
      update = true,
      verbose = false
    }
}
```

さらに同期先のHOSTの設定をansibeで指定している

```
[www]
www01-aws lsyncd_mirror_host={ここにwww02のIP}
www02-aws lsyncd_mirror_host={ここにwww01のIP}
```
