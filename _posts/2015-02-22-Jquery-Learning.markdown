---
layout: post
title: 常用jQuery动画
date: 2014-02-12 16:10:00
---


一、show（）方法和hide（）方法



1、show()方法和hide()方法

show方法和hide方法是jQuery最基本的动画方法，在HTML文档里，为一个元素调用hide（）方法，会将钙元素的display样式改为“none”。也就是说：

$("elemetn").hide()；与element.css("dispaly","none");

当把元素隐藏后show（）方法会将样式display设置成block或者inline，或者其他除了none之外的值。



2、show方法和hide（）方法让元素动起来

可以为show（）方法至此那个一个速度参数，例如:
$("element").show("slow");参数的关机字还有normal、fast从slow到normal到fast分别为600、400、200毫秒，不仅仅可以使用这三个参数还可以直接指定一个数字。

$("element").show(1000);在100毫秒内展示；



二、fadeIn（）方法和fadeOut（）方法



1、fadeln（）方法和fadeOut（）方法只改变元素的不透明度。

fadeOut（）方法会在指定的一段时间内降低元素的不透明度，知道元素完全消失，fadeIn()则相反。



三、slideUp（）方法和slideDown（）方法

slideDown方法将元素由上至下延伸展示。slideUp（）正好相反，缩短上下展示。



四、自定义动画animate（）



语法结构：animate（params，speed，callback）；

参数说明如下：

（1）params：一个包含样式属性及值得映射，比如{property1：“value1",property2:"value2",...}。

（2）speed：速度参数，可选。

（3）callback：在动画完成时完成的函数，可选。



1、自定义简单动画

如：$(this).animate({left:"500px"},3000);

2、累加、累减动画

$(this).animate({left:"+=500px"},300);//当前位置加500px

3、多重动画

1）同时执行多个动画

$(this).animate({left:"500px",heitht:"200px"},3000);



2)按顺序执行多个动画

$(this).animate({left:"500px"},3000);

$(this).animate({height:"200px"},3000);

因为animate方法都是对同一个对象进行操作可以携程链式：

$(this).animate({left:"500px"},3000)

.anmate({height:"200px"},3000);

像这样动画效果执行具有先后顺序，称为动画队列。



4、综合动画

按照anmate的规则则可以对元素进行各种动画操作，进行各种动画操作的组合：

$(this).anmate({left:"400px",height:"200px",opcity:"1"},3000)

.anmate({top:"200px",width:"200px"},3000)

.fadeOut("slow");



五、动画回调函数

在上例中，如果想在最后一步切换元素的CSS样式，而不是隐藏元素：如果将fadeOut（“slow”）改成css（"border","5px solidblue");将不可以达到预期的效果，预期的是在动画最后一步执行css（）方法，但是在动画开始的时候就执行了，导致这个的原因是因为css（）方法不再动画队列中。可以使用回调函数（callback）对非动画方法实现排队。

例如：

.anmate({top:"200px",width:"200px"},3000，function（）{

//$(this).css("border","5px solid blue");

})



六、停止动画和判断是否处于动画状态



1、停止元素动画

stop（【clearQueue】【，gotoEnd】）；

参数clearQueue和gotoEnd都是可选的参数，为Boolean值（ture或flase）；

stop（）参数为空的时候一旦出发就会从当前触发时停止执行的动画；

当有多个组合动画的队列时，仅仅使用stop（）就不可以方便的执行了，这时候可以把第一个参数（cleanQueue)设置成true，此时程序会把当前元素接下来尚未执行完成的动画队列都清空。



第二个参数（gotoEnd）可以用于让正在执行的动画直接到达结束时刻的状态，通常用于后一个动画需要给予前一个动画的末状态的情况，可以通过stop(false,true)这种方式来让前动画直接到达末状态。



2、判断元素是否处于动画状态

当执行animate（）动画时，判断是否处于动画可以：
if（！$(element).is(":animated")){

//如果没有执行动画，再添加动画或者执行其他操作

}



七、其他动画效果

jQuery专门用于交互的动画方法：

toggle（speed，【callback】）

slideToggle（speed，【callback】）

fadeTo(speed,opacity,[callback])



1、toggle（）方法

可以切换元素的可见状态，如果元素是可见的，则切换为隐藏的如果元素是隐藏的则切换成可见的。

$(this).next("div.content").toggle();



2、slideToggle()方法

通过高度的变化切换匹配元素的可见性，这个效果只调整元素的高度。

相当于.slideUp()和slideDown方法不断的切换组合。



3、fadeTo（）方法

可以把元素的不透明度以渐进的方式调整到指定的值，这个动画只调整元素的不透明度

$(this).next("div.content").fadeTo(600,0.2);



八、动画方法的概括

基本动画方法hide（）和show（），到fadeIn()和fadeOut(),然后到slideUp（）和sildeDown（），再到自定义动画animate方法。
