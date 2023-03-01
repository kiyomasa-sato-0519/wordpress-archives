---
title: "ubuntu16 + nginx + ansible環境構築"
date: "2018-01-01"
categories: 
  - "未分類"
tags: 
  - "nginx"
  - "php"
  - "wordpress"
---

apache使ってるからか表示が遅い。 ということでnginxに乗り換えることにした。 phpをnginxに対応させるのとnginx自体の設定を書く。

#### ansibleのyml

**roles/wp-nginx.yml**

```
- name: install nginx packages
  apt: name={{ item }} state=installed update_cache=yes
  update_cache: yes
  with_items:
    - nginx
  notify: [ 'Restart nginx' ]

- name: delete default settings
  file:
    path: "{{ item }}"
    state: absent
  with_items:
    - /etc/nginx/sites-enabled/default

- name: copy rails conf
  copy: src=files/rails.conf dest=/etc/nginx/sites-enabled/rails.conf
  notify: [ 'Restart nginx' ]

- name: config test
  service:
    name: nginx
    state: restarted
    enabled: yes
```

**roles/php/tasks/main.yml**

```
- name: install apache packages
  apt: name={{ item }} state=installed
  with_items:
    - php
    - php-mysql
    - php-fpm
    - php7.0-mbstring
    - php7.0-xml
    - php7.0-xmlrpc

- name: config restart
  service: name=php7.0-fpm state=restarted enabled=yes
```

#### nginxのconfはこんな感じ。

server\_tokens offにしてOSバージョン等を隠すのはapacheの時と同様なので細かい説明は割愛。 パーマリンクに対応するために少々長い設定が必要 **roles/wp-nginx/files/wordpress.conf**

```
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=zone1:4m inactive=7d max_size=50m;
proxy_temp_path /data/nginx/tmp;
server_tokens off;

server {
    listen 80;
    server_name www.null-engineer.ml;
    index index.php index.html index.htm;
    set_real_ip_from 10.0.0.0/16;
    real_ip_header  X-Forwarded-For;
    charset utf-8;

    keepalive_timeout 5;
    root /var/www/wordpress ;

    location / {
        try_files $uri $uri/ @wordpress;
    }

    location ~* /wp-config.php {
         deny all;
    }

    location /wp-admin/ {
      allow {許可するIP};
      deny all;
    }

    location ~ \.php$ {
         try_files $uri @wordpress;
         fastcgi_index  index.php;
         fastcgi_split_path_info ^(.+\.php)(.*)$;
         fastcgi_pass   unix:/run/php/php7.0-fpm.sock;
         fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
         include        fastcgi_params;
    }

    location @wordpress {
        fastcgi_index index.php;
        fastcgi_split_path_info ^(.+\.php)(.*)$;
        fastcgi_pass  unix:/run/php/php7.0-fpm.sock;
        fastcgi_param SCRIPT_FILENAME  $document_root/index.php;
        include       fastcgi_params;
    }
    error_page 404 /404.html;
      location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
      location = /50x.html {
    }
}
```

#### 再起動用のhandler

**roles/wp-nginx/handlers/main.yml**

```
---

- name: Restart nginx
  service:
    name: 'nginx'
    state: 'restarted'
```
