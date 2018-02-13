---
layout: post
title: Ajax in Jquery
date: 2016-01-09 23:13:00
---

jQuery对Ajax操作进行了封装，在jQuery中$.ajax()方法属于最底层的方法，第二层是load()、$.get()和$.post()方法，第三层是$.getScript()和$.getJSON()方法。


##  load（）方法

1、载入HTML文档

load（）方法是jQuery中最为简单和常用的Ajax方法，能载入远程HTML代码并插入DOM中，它的结构为load（url【，data】【，callback】）

url：参数类型为String，请求HTML页面的URL地址

data（可选）类型是Object，发送至服务器的key/value数据

callback（可选）参数类型是function ，请求完成后的回调函数，无论请求成功或失败

首先构建一个被load（）方法加载并追加到页面中的HTML文件，名字为test.html,HTML代码如下：

```bash
<div class=“comment">

已有评论：

</div>

<divclass="comment">

<h6>张三</h6>

<p class="para">沙发</p>\

</div>

<divclass="comment">

<h6>李四</h6>

<p class="para">板凳</p>

</div>

<divclass="comment">

<h6>王五</h6>

<p class="para">地板</p>

</div>
```
使用一个button来主动触发Ajaxt事件，接下来就可以开始编写jQuery代码了，等DOM元素加载完毕后，通过单机id为send的按钮来调用load（）方法，然后将test.html的内容加载到id为resTest的元素里。

```bash
jQuery代码如下：

function（）{

$("#send").click(fucntion(){

$("#resText").load("test.html");

});

}
```

2、传递方式

load（）方法的传递方式根据参数data来自动指定，如果没有参数传递，则采用GET方式传递，反之自动转换为POST方式：

//无参数，则是GET方式

$("#restText").load("test.php",function(){});

3、回调函数

 对于必须在加载完成后才可以继续的操作，load（）方法提供了回调函数（callback），该函数有三个参数，分别代表请求返回的内容、请求状态和XMLHttpRequest对象，jQuery代码如下：
 $("#resText").load("test.html",function(responseText,textStatus,XMLHttpRequest){});

 ### $.get() VS $.post()

 1. $.get()方法使用GET方式来进行异步请求。它的结构为：
 $.get(url[,data][,callback][,type])

 url,类型是String，请求HTML页的URL地址

 data（可选)类型是Object，发送至服务器的key/value数据会作为QueryString附加到请求URL中

 callback（可选），Function 载入成功时回调函数（只有当response的返回状态是success才调用该方法）自动将请求结果和状态传递该方法

 type（可选），类型是String，服务器返回内容的格式，包括XML、html、script、json、text和_default。





 $.get()的回调函数参数只有两个参数，代码如下：

 function(data,textStatus){

 //data : 返回的内容可以是XML文档、JSON文件、HTML片段等等

 //textStatus：请求状态：sucess、error、notmodified、timout四种

 }

 data参数请求返回的内容，textStatus参数代码请求的状态，而且回到函数只有当数据成功返回（sucess）后才被调用，这点与load（）方法不一样。

## $.post()

它和$.get()方式使用相同，但是存在写区别：

get请求会将参数跟在URL后进行传递，而POST请求则作为HTTP消息的实体内容发送给Web服务端。当然，在Ajax请求中，这种区别对于用户不可见的。

GET请求方式对传输的数据大小限制（通常不能大于2kb），而使用post请求数据量比GEt请求大得多（理论上讲不受限制）。

GET凡是请求数据会被浏览器缓存起来，因此其他人可以从浏览器的浏览记录里面读取这些数据，例如账号和密码等，某些情况下GET请求有严重的安全隐患。

GET和POST方式传递数据在服务器端获取的方式不同。



三、$.getScript()方法和$.getJson()方法

 1、$.getScript()方法

有时候页面初次加载的时候就会取得所需的全部JavaScript文件是完全没有必要的。虽然可以在选哟哪个JavaScript文件时，动态创建<script>标签，如下：

$(document.createElcement("script")).attr("src","test.js").appendTo("head");



但是这种方式并不是很理想，为此jQuery提供了$.getScript()方法来直接加载.js文件，与加载一个HTML片段一样简单方便，并且不需要对JavaScript文件进行处理。

jQuery代码如下：
```bash
$(function(){
$("#send").click(funciton(){
       $.getScript('test.js");
});
})
```

##  $.getJSON()

$.getJSON()方法用于加载JSON文件，与$.getScript()方法的用法相同。


回调函数中的function（data）{ }变量可以遍历相应的数据，也可以使用迭代方式为每个项构建相应的HTML代码，虽然在这里可以使用传统的for循环来实现，同样jQuery提供了一个通用的$.each()方法，可以用来遍历对象和数组。

$.each()函数不同于jQuery对象的each（）方法，它是一个全局函数，不操作jQuery对象，而是一个数组或者对象作为第一个参数，以一个回调函数作为第二个参数，回调函数拥有两个参数：第一个为对象的成员或数组索引，第二个为对应变量或内容。例如：

$.each(data,function(){commentIndex.comment){});

## $.ajax()方法

$.ajax()方法是jQuery最底层的Ajax实现。

它的结构为$.ajax(options)

该方法只有一个参数，但在这个对象里包含了$.ajax()方法所需要的请求设置以及回调函数等信息，参数以key/value的形式存在，所有参数都是可选的。参数解释如下：



url ： String类型，发送请求的地址（默认是当前页地址）

type ： String类型，请求方式（POST或者GET），注意其他HTTP请求方式例如put和DELETE也可以使用，但是仅部分浏览器支持。

timeout： Number ,设置请求的超时时间，此设置将覆盖$.ajaxSetup()方法全局设置

data： Object或者String ，发送到服务器端的数据，如果已经被奴是字符串会自动转换成字符串格式，GET请求中奖附加到URL后，防止这种自动转换，可以查看processData选项，对象必须为key/value格式例如：{“foo1”：“bar”，“foo2”：“bar2”}转换为&foo1=bar1&foo2=bar2。如果是数组jQuery会转换为不同值对应同一名称。例如:[foo:["bar1","bar2"]转换为&foo=bar&foo=bar2;

dataType:String ,预期返回的数据类型，如果不指定，jQueryjiang自动根据HTTP包MIME信息返回responseXML或者responseText,作为回调函数参数传递。

可用的类型如下xml(xml文档）、html（返回HTML）、script（返回js纯文本）、json（返回JSON数据）、jsonp（返回JSONP格式）、text（返回纯文本字符串）

beforeSend:Function(),发送请求先可以修改XMLHttpRequest对象的函数。例如添加自定义HTTP头，如果返回false则取消本次Ajax请求XMLHttpRequest对象是唯一的参数。

complete：Function（），请求完成后调用的回调函数（请求成功个或者失败都会调用）。参数是XMLHttpRequest对象和一个描述成功类型的字符串。

success：Function，请求成功后回调函数，有两个参数。一个是服务器返回的data并根据dataType处理后的数据，二个是描述状态的字符串funciton (data,textStatus)

error : Function(),请求失败时被调用的函数，该函数有三个参数，即XMLHttpRequest对象、错误信息、不做的额错误对象（可选）。

global ：boolean，默认是true，表示是否出发全局Ajax事件设置为false将不会触发全局Ajax事件。



