---
title: "jQuery.Deferredな話"
date: "2018-01-01"
categories: 
  - "未分類"
tags: 
  - "javascript"
  - "jquery"
---

ある非同期処理を一時的に同期させたい場合、 つまり他の非同期処理の完了を待って何かしらの処理をしたい場合、jQuery.Deferredが非常に便利だ。

具体的なコード例をあげて紹介しようと思う。

さて、次のコードを見ていただきたい。

```
// deferredを格納する配列
var arr = [];
// テキストを入れる要素
var body = $('body');

for(var i = 0; i < 5; i++) {
  // deferredオブジェクトの作成
  var d = $.Deferred();
  var ms = ( i + 1 ) * 1000;

  // 非同期処理
  // 一定時間経過後にbodyにテキストを挿入する
  setTimeout(function(millisec, deferred){
    // divを生成してtextを設定し、bodyに挿入する
    $('<div />').text(millisec + "ms").appendTo(body);
    // 上記が完了した時点でresolveとする
    deferred.resolve();
  },ms, ms, d);

  // 配列にpromiseを入れる
  arr.push(d.promise());
}

// すべてのdeferredが完了ステータスになったらbodyにfinish!と挿入する
// また、配列を引数として渡すためにapplyを利用する
$.when.apply($, arr).done(function(){
  var div = $('<div />').text("finish!");
  body.append(div);
});
var message = $('<div />').text("script end");
body.append(message);
```

このコードの出力結果は

```
script end
1000ms
2000ms
3000ms
4000ms
5000ms
finish!
```

となる。ポイントとしては非同期処理であるsetTimeoutの完了を待ってから**finish!**が表示されているところである。 また、setTimeoutが非同期なため、1+2+3+4+5秒とは待たずにこのコードは即時終了し、**script end**が真っ先に表示される。 そしてほぼ同時に全てのsetTimeoutが実行され、指定された秒数待つので合計で約５秒後には**finish!**が表示される。

端的に言うと全てのsetTimeoutが完了するまでfinish!の表示を待っているのである。 これを可能とするのがjQuery.Deferredである。 使い方自体は至ってシンプルで、まずDeferredオブジェクトを作成する。

```
var d = $.Deferred();
```

そして

```
d.promise()
```

でステータスを持ったpromiseと言うものを返す。 これは処理中や処理完了、失敗などのステータスを持ったオブジェクトと考えるとわかりやすいだろうか。

最終的に処理が完了したときに

```
deferred.resolve();
```

をコールしてステータスを完了状態とする。 ちなみにresolveの代わりにrejectをコールすると失敗としてステータスを更新する。

また、本来whenは引数を複数受け取り、新たなpromiseとして登録されるが、 今回配列の引数を渡すためにapplyでjquery($の部分)オブジェクトをthisとして配列を分解した引数を渡す。

```
$.when.apply($, arr)
```

注意点としてはwhenは渡された全てのdeferredが完了状態になるとdoneやalwaysでコールバックを実行するが、 途中でrejectが実行されると失敗として以降のdeferredのステータスを無視してfailとなる。(その場合でもalwaysは呼ばれる) 余談だがdoneを呼ぶ前にすでにresolveがコールされていた場合は即座にdoneが呼ばれる。

主なメソッドとしては

```
when - 複数のpromiseをまとめて一つのpromiseを返す。（そのためメソッドチェーンでdoneなどが実行できる)
done - resolveがコールされたdeferredオブジェクトに対して実行する
fail - rejectがコールされたdeferredオブジェクトに対して実行する
then - doneとfailを両方設定する。この中でdeferredオブジェクトを返すと次のメソッドに引き継がれる。
```

他のメソッド等に関しては下記のドキュメントを参照してほしい。

[https://api.jquery.com/category/deferred-object/](https://api.jquery.com/category/deferred-object/)
