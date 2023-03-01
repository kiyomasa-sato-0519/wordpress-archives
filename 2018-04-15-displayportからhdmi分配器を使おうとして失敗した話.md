---
title: "DisplayPortからHDMI分配器を使おうとして失敗した話"
date: "2018-04-15"
categories: 
  - "未分類"
tags: 
  - "amazon"
  - "displayport"
  - "hdmi"
  - "ケーブル"
  - "モニタ"
  - "周辺機器"
---

現在我が家には二台モニタがあり、片方がPCと接続、もう片方がコンシューマーゲーム用という扱いになっている。

PC用のモニタにはHDMIがついておらずDisplayPort、DVI、VGAのみ端子があり、ゲーム用のはDisplayPortの代わりにHDMIが付いている。

モニタを一台にまとめたいと考え分配器の購入を決意した。

一般的にゲーム機はHDMIが多く、PCのグラフィックボードにもHDMI出力端子がついているのでHDMIの分配器を購入してモニタに接続を試した結果を載せておく。

まず最初に試したのはこちら

<iframe style="width: 120px; height: 240px;" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=null-engineer-blog-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=as_ss_li_til&amp;asins=B0767Q7VTG&amp;linkId=9adc83732f56a99e2b1ce73b056c760e" width="300" height="150" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

これに加え、

<iframe style="width: 120px; height: 240px;" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=null-engineer-blog-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=as_ss_li_til&amp;asins=B01FQXC1GG&amp;linkId=5ee4092106fcba3871fd8b42319393da" width="300" height="150" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

上記でDisplayPort To HDMI変換をして分配器を接続した。 結果、分配器はうんともすんとも言わなかった。直接HDMIディスプレイへの接続で動作は確認出来たもののDisplayPortを挟むと動かないようだった。

相性の問題かと思い次に試したのは下記

<iframe style="width: 120px; height: 240px;" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=null-engineer-blog-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=as_ss_li_til&amp;asins=B00P0SD4BE&amp;linkId=2e97d02b08d00681b6479875c9599ee4" width="300" height="150" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

しかしこちらも同様に動作しなかった。

最終的に、変換アダプタを介したことによって動作しなかったのかと推測し、外部給電可能なタイプを購入した。

<iframe style="width: 120px; height: 240px;" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=null-engineer-blog-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=as_ss_li_til&amp;asins=B071JTQW19&amp;linkId=35b83e3ea0f08eb7f34395fce4fd2bab" width="300" height="150" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

しかしこちらでも何故かDisplayPortでは動作せず。 結局何をしてもDisplayPortのモニタにHDMIの機器をぶら下げることは出来ないという結論に落ち着いた。 無念。
