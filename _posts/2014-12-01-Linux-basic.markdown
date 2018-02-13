---
layout: post
title: linux系统变量和别名---basic
date: 2014-12-01 15:20:30
---

## 常见变量包

~/.bashrc：bash打开时会读取

~/.zshrc：zsh打开时会读取

/etc/profile:此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.
并从/etc/profile.d目录的配置文件中搜集shell的设置.

/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.
~/.bash_profile:每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该
文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件.

~/.bashrc:该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该
该文件被读取.

~/.bash_logout:当每次退出系统(退出bash shell)时,执行该文件.
另外,/etc/profile中设定的变量(全局)的可以作用于任何用户,而~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量,他们是\”父子\”关系.

## Differences

$PATH和alias不一样，前者会引到一些可执行文件。而后者则是具体调用文件的命令

$PATH:/bin/bin目录下有一堆可执行文件，然后我们可以直接在命令行打开

alias：alias报出来的是命令的别名。

## 常用 Bash

export：export命令将变量输出至任何子shell
PATH：PATH=$PATH:路径1:路径2:...:路径n

alias：新建别名。例如alias jpub='justpublish(){git add --all; git commit -m "$1"; git push origin master;just publish;};justpublish'，其中，alias可以包含函数，可以包含变量，也可以多个命令顺序执行。

使用其他语言执行文件：#!/usr/bin/env node在文件开头写上这么一句，那么即使是如ocms名字（无后缀）的文件也可以用nodejs来执行了