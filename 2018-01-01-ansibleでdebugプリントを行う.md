---
title: "ansibleでdebugプリントを行う"
date: "2018-01-01"
categories: 
  - "未分類"
---

ansible実行時にregisterやらに何が入っているか確認する方法 デバッグしたいregisterの値を指定してansibleを実行する

# 例:

apacheプロセスが存在するかチェックする

```
- name: check apache
  shell: "pgrep apache | wc -l"
  register: check_apache

- name: debug result var
  debug: var=check_apache
```

## 出力内容

```
TASK [sample : check apache] *************************************************
changed: [default]

TASK [sample : debug result var] *********************************************
ok: [default] => {
    "check_apache": {
        "changed": true, 
        "cmd": "pgrep apache | wc -l", 
        "delta": "0:00:00.006549", 
        "end": "2018-01-01 20:25:51.373271", 
        "failed": false, 
        "rc": 0, 
        "start": "2018-01-01 20:25:51.366722", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "0", 
        "stdout_lines": [
            "0"
        ]
    }
}
```

stdoutが0、つまりapacheは起動していないと判断できる。
