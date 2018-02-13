---
layout: post
title: Windows版本编译运行React Native官方实例UIExplorer项目
date: 2016-12-01 05:10:00
---

### React Native项目源码下载

git clone https://github.com/facebook/react-native.git

2.2 Android环境要求如下

①.Android Sdk版本23(在build.gradle中的compileSdkVersion)

②.SDK build tools version 23.0.1(build.gradle中buildToolsVersion)

③.Android Support Repository>=17

④·Andoid NDK需要安装以及配置


### 下载安装cygwin软件

根据官方提供的文档我们需要执行类似于./packager/packager.sh这样的shell脚本，普通的Windows系统是无法执行这样的脚本的。所以我们的Windows系统可以下载安装cygwin之后就可以执行shell脚本

下载完成开始安装:



选择从网络(Internet)进行安装，点击下一步:



安装路径尽量采用英文(不要中文),然后默认选择下一步就行了.

下面我们在系统环境变量PATH中添加我们cygwin的bin目录这样我们就可以通过命令行界面直接使用bash进入cygwin环境啦~配置完成之后，重启命令行终端，然后敲入bash命令进入如下界面，就代表OK了

### 下载安装NDK然后安装以及配置

因为官方的实例是需要进行安装配置NDK的，所以大家需要去官方网站进行下载 [](http://developer.android.com/ndk/dowloads/index.html)

### 添加Node依赖模块:该命令行需要切到react-native项目中,主要运行如下命令

cd react-native

npm install


### 开始编译官方实例UIExploerer项目

打开之前安装的cygwin终端，切换到当前react-native项目中。注意切换路径方法以实际项目路径为准

输入命令

./gradlew
:Example.UIExplore:andriod:app:isntallDebug

漫长的等待... .... ....

./packager/packager.sh


最后就可以在设备中打开 APP 查看效果


