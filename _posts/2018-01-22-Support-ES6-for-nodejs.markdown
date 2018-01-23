---
layout: post
title: 如何使Node.js支持 ES6
date: 2018-01-23 18:33:00
---



#### 检测当前 Node.js版本对ES6的支持情况
1. 全局安装检测工具
```bash
npm install -g es-checker
```
1. 使用es-checker检测Node.js对ES6的支持情况
```bash
es-checker
```
可以查看当前版本的Node.js对ES6的支持情况
![](/assets/images/es6-1.png)
![](/assets/images/es6-2.png)
![](/assets/images/es6-3.png)

#### 添加ES6支持

1. npm init 初始化工程目录，生成package.json文件
2. 全局安装 Babel-cli npm install babel-cli -g
3. 安装babel-preset-es2015来支持ES6的语法
```bash
npm install babel-preset-es2015 --save
```
4. 增加 .babelrc配置文件
```bash
{
    "presets": [
        "es2015"
    ],
    "plugins": []
}
```


