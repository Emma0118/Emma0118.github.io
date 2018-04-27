---
layout: post
title: 获取localStorage大小另外的一种实现
date: 2016-08-11 12:05:00
---



一、一般浏览器的localStorage的大小为 1024 * 1024 * 5
在这里我们通过使用字符串的大小来衡量获得一个相对来说更精确的值。

二、分析过程如下，
首先我们获得某个字符串或其他可以长度累加的数据格式，通过循环累加达到localStorage的容量的最大。当超出容量时，我们获取错误信息，并记录这时的所有字符串的长度即可。

三、代码实现如下


``bash
(function() {
    if(!window.localStorage) {
    console.log('当前浏览器不支持localStorage!')
    }    
    var test = '0123456789';
    var add = function(num) {
      num += num;
      if(num.length == 10240) {
        test = num;
        return;
      }
      add(num);
    }
    add(test);
    var sum = test;
    var show = setInterval(function(){
       sum += test;
       try {
        window.localStorage.removeItem('test');
        window.localStorage.setItem('test', sum);
        console.log(sum.length / 1024 + 'KB');
       } catch(e) {
        console.log(sum.length / 1024 + 'KB超出最大限制');
        clearInterval(show);
       }
    }, 0.1)
  })()
``

