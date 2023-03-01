---
title: "anyenvインストール済みのmacでbrew doctorを実行したらwarningが出る場合"
date: "2019-02-08"
categories: 
  - "未分類"
tags: 
  - "anyenv"
  - "homebrew"
  - "mac"
---

```
[xxxxx@MacBook-Air:~]$ brew doctor
Please note that these warnings are just used to help the Homebrew maintainers
with debugging if you file an issue. If everything you use Homebrew for is
working fine: please don't worry or file an issue; just ignore this. Thanks!

Warning: "config" scripts exist outside your system or Homebrew directories.
`./configure` scripts often look for *-config scripts to determine if
software packages are installed, and what additional flags to use when
compiling and linking.

Having additional scripts in your path can confuse software installed via
Homebrew if the config script overrides a system or Homebrew provided
script of the same name. We found the following "config" scripts:
/Users/xxxxx/.anyenv/envs/pyenv/shims/python2-config
/Users/xxxxx/.anyenv/envs/pyenv/shims/python3.6m-config
/Users/xxxxx/.anyenv/envs/pyenv/shims/python2.7-config
/Users/xxxxx/.anyenv/envs/pyenv/shims/python-config
/Users/xxxxx/.anyenv/envs/pyenv/shims/python3-config
/Users/xxxxx/.anyenv/envs/pyenv/shims/python3.6-config
```

上記のようなwarningが出た。 原因はbrewが参照するパスにpythonのconfigが複数あるせいとのことでbrewコマンドのaliasを作って余計なパスを見ないようにするのがスマートな解決策だと思う

.bashrcなどに下記を追加してターミナルの再起動かsource .bashrc (.bash\_profileで.bashrcを読み込む設定にしていない人は.bash\_profileに直接書くか.bashrcを読むようにしよう)

```
alias brew="PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin brew"
```
