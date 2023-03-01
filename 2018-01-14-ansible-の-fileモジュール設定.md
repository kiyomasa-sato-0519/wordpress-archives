---
title: "ansible の fileモジュール設定"
date: "2018-01-14"
categories: 
  - "未分類"
tags: 
  - "ansible"
---

すぐ忘れてしまうのでメモしておく。

たとえばあるファイルのパーミッションを600にする場合

```
- file: path={ファイル名} mode=0600
```
