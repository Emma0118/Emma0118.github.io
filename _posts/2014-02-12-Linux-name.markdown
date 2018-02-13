---
layout: post
title: linux下使用alias设置指令别名并使其永远生效
date: 2014-02-12 05:10:00
---

1. 在用户目录下打开文件

```bash
emma@main:~$ vi .bashrc

```

2. 在文件下面位置写入

待写入指令：

```bash
alias tachyon="/home/emma/files/tachyon-0.7.1-bin/bin/test"

```
插入后：

```bash
# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias tachyon="/home/emma/files/tachyon-0.7.1-bin/bin/test"

```

3.命令使其生效

```bash
emma@main:~$ source .bashrc
```

