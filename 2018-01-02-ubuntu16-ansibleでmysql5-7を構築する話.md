---
title: "ubuntu16 + ansibleでmysql5.7を構築する話"
date: "2018-01-02"
categories: 
  - "未分類"
tags: 
  - "ansible"
  - "aws"
  - "ec2"
  - "mysql"
---

前回失敗に終わったmysql5.6のインストール。 どうやってもmysql5.7との競合が解決できなかったので開き直ってmysql5.7でインストールすることで解決できた。

[ubuntu16 + ansibleでmysql5.6を構築する話](https://www.null-engineer.com/2018/01/02/ubuntu16-ansible%e3%81%a7mysql5-6%e3%82%92%e6%a7%8b%e7%af%89%e3%81%99%e3%82%8b%e8%a9%b1/)

早速だがその時のansibleがこちら

```
- name: disable app-armor
  service:
    name: apparmor
    state: stopped
    enabled: no

- name: install mysql5.7
  apt: name={{ item }} state=latest update_cache=yes
  with_items:
    - libmysqlclient-dev
    - libmysqlclient20:amd64
    - mysql-client
    - mysql-server
    - mysql-common

- name: check_mysql
  shell: "pgrep mysql | wc -l"
  register: mysql_check

- name: copy conf
  copy: src=files/mysqld.cnf dest=/etc/mysql/mysql.conf.d/mysqld.cnf

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
    name: ''
    host: localhost
    state: absent

- name: Removes all anonymous user accounts
  mysql_user:
    name: ''
    host_all: yes
    state: absent

- name: create user
  mysql_user:
    name: "{{ MYSQL.USER }}"
    host: "{{ MYSQL.HOST}}"
    password: "{{ MYSQL.PASS }}"
    priv: "{{ MYSQL.DB }}.*:ALL,GRANT"
    state: present
```

一体何にハマったかというとインストールされたmysqlのrootユーザのパスワードがわからないから初期化出来ない、となったため。 だがどうやらデフォルトでOSのrootユーザなら入れるようになっているようでmysql\_userでrootの指定を外したらユーザーの作成等、通った。 (become: yes前提)

もう一つハマりポイントがあってデフォルトのconfigだとlocalhostからの接続しか許可していない。 なので**mysqld.cnf**からbind-addressをコメントアウトする。 セキュリティ周りはAWSのセキュリティグループで制限しているのでこれで問題ない。 他環境で構築する人はそのあたりは厳密にした方がいいと思われる。

```

#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
# 
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

[mysqld_safe]
socket        = /var/run/mysqld/mysqld.sock
nice        = 0

[mysqld]
#
# * Basic Settings
#
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket        = /var/run/mysqld/mysqld.sock
port        = 3306
basedir        = /usr
datadir        = /var/lib/mysql
tmpdir        = /tmp
lc-messages-dir    = /usr/share/mysql
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address        = 127.0.0.1
#
# * Fine Tuning
#
key_buffer_size        = 16M
max_allowed_packet    = 16M
thread_stack        = 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover-options  = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit    = 1M
query_cache_size        = 16M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries    = /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id        = 1
#log_bin            = /var/log/mysql/mysql-bin.log
expire_logs_days    = 10
max_binlog_size   = 100M
#binlog_do_db        = include_database_name
#binlog_ignore_db    = include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem
```
