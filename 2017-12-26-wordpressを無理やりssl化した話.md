---
title: "wordpressを無理やりSSL化した話"
date: "2017-12-26"
categories: 
  - "未分類"
tags: 
  - "aws"
  - "php"
  - "wordpress"
---

テーマに埋まってるhttpの部分もあるため当ブログでは

- wp-includes/functions.php

上記ファイルに無理やりフィルターをかませてhttps化と同時にstatic系ファイルをs3のURLに書き換えている。

具体的には下記（文書に埋まっているURLにも効果が及ぶためあえてスペースを入れて引っかからないようにしている。

```
function replace_content_url() {
 ob_start('replace_static_url');
}
function replace_static_url($html) {
 if(!is_admin()) {
 $html = str_replace('http ://www.null-engineer.com/wp-includes/css', 'https://static.null-engineer.ml/wp-includes/css', $html);
 $html = str_replace('http ://www.null-engineer.com/wp-includes/js', 'https://static.null-engineer.ml/wp-includes/js', $html);
 $html = str_replace('http ://www.null-engineer.com/wp-content/', 'https://static.null-engineer.ml/wp-content/', $html);
 $html = str_replace('http ://www.null-engineer.com', 'https://www.null-engineer.com', $html);
 $html = str_replace('http ://0.gravatar.com', 'https://0.gravatar.com', $html);
 $html = str_replace('http :\/\/', 'https:\/\/', $html);
 }
 return $html;
}
```

wp\_loadedをフックして動的生成じゃない部分のURLをS3のURLに置換し、埋まっている残りのhttpをhttpsに置換している。もっと賢いやり方もありそうだけど面倒なのでこれはこれでありかもしれない。
