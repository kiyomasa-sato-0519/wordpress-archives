---
title: "ubuntu16 + ansibleでmysql5.6を構築する話"
date: "2018-01-02"
categories: 
  - "未分類"
tags: 
  - "ansible"
  - "aws"
  - "ec2"
  - "mysql"
---

RDSを使ってたけど高い！こんなしょっぱいブログ動かすのに最低マシンでもコスパに見合わない。 そこでインスタンスを立ち上げてmysqlを動かすことにした。 冗長性を捨てることでコスト削減を図ったある意味暴挙であるがこのブログで収益が出るわけでもないので落ちてもいいやの精神で行く。

このエントリは結局失敗に終わった。 成功したエントリはこちら [ubuntu16 + ansibleでmysql5.7を構築する話](https://www.null-engineer.com/2018/01/02/ubuntu16-ansible%e3%81%a7mysql5-7%e3%82%92%e6%a7%8b%e7%af%89%e3%81%99%e3%82%8b%e8%a9%b1/)

ansibleで構築するに当たってぶち当たったのは二点

```
The MySQL-python module is required
```

というエラー。ユーザーの作成などにpythonにmysqlクライアントが必要だと言われている。 pipでMySQL-pythonをインストールする必要があるがubuntu16はpythont3がデフォルトで入っていてMySQL-pythonはpython2なのでpip3でmysqlclientをインストールする。

最終的にこうなった。MYSQL.USERなどの設定は適当にgroup\_varsの下にでも入れてほしい。

```
- name: base module
  apt: name={{ item }} state=installed update_cache=yes
  with_items:
    - python3-pip

- name: add repository
  apt_repository:
    repo: deb http://archive.ubuntu.com/ubuntu trusty universe
    update_cache: yes
    state: present

- name: mysql install
  apt: name={{ item }} state=installed update_cache=yes
  with_items:
    - libmysqlclient-dev
    - mysql-server-5.6
    - mysql-client-5.6

- name: check_mysql
  shell: "pgrep mysql | wc -l"
  register: mysql_check

- name: mysql start
  service:
    name: mysql
    state: restarted
    enabled: yes
  when: mysql_check.stdout != 0

- name: "install"
  become: yes
  shell: pip3 install mysqlclient

- name: Removes anonymous user account for localhost
  mysql_user:
    login_user: root
    name: ''
    host: localhost
    state: absent

- name: Removes all anonymous user accounts
  mysql_user:
    login_user: root
    name: ''
    host_all: yes
    state: absent

- name: create user
  mysql_user:
    login_user: root
    name: "{{ MYSQL.USER }}"
    host: "{{ MYSQL.HOST}}"
    password: "{{ MYSQL.PASS }}"
    priv: "`{{MYSQL.USER}}`.`{{MYSQL.DB}}`:ALL,GRANT"
    state: present
```

とここまでやったがテストで作ったvagrant環境ではうまく行ったものの実際のaws上のubuntu16ではmysql5.7と競合してうまくインストールできなかった。よってこのエントリはほぼ無意味なものとなった。
