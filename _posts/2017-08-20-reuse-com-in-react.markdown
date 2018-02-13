---
layout: post
title: 在React实现可复用组件
date: 2017-08-20 11:15:10
---



1、将通用的设计元素（按钮，表单框，布局组件等）拆成接口良好定义的可复用的组件，这样下次开发界面程序时候可以写更少的代码，也意义着更高的开发效率，更少的Bug和更少的程序体积。

2、Prop验证

随着应用不断变大，保证组件被正确使用变得非常有用。因此引入propTypes。React.PropTypes提供很多的验证器（validator）来验证传入数据的有效性，当props传入无效数据时，JavaScript控制台会抛出警告

Ps：为了性能考虑，只在开发环境验证propTypes。

```bash
React.createClass({

  propTypes: {

    // 可以声明 prop 为指定的 JS 基本类型。默认

    // 情况下，这些 prop 都是可传可不传的。

    optionalArray: React.PropTypes.array,

    optionalBool: React.PropTypes.bool,

    optionalFunc: React.PropTypes.func,

    optionalNumber: React.PropTypes.number,

    optionalObject: React.PropTypes.object,

    optionalString: React.PropTypes.string,



    // 所有可以被渲染的对象：数字，

    // 字符串，DOM 元素或包含这些类型的数组。

    optionalNode: React.PropTypes.node,



    // React 元素

    optionalElement: React.PropTypes.element,



    // 用 JS 的 instanceof 操作符声明 prop 为类的实例。

    optionalMessage:React.PropTypes.instanceOf(Message),



    // 用 enum 来限制 prop 只接受指定的值。

    optionalEnum:React.PropTypes.oneOf(['News', 'Photos']),



    // 指定的多个对象类型中的一个

    optionalUnion: React.PropTypes.oneOfType([

      React.PropTypes.string,

      React.PropTypes.number,

      React.PropTypes.instanceOf(Message)

    ]),



    // 指定类型组成的数组

    optionalArrayOf:React.PropTypes.arrayOf(React.PropTypes.number),



    // 指定类型的属性构成的对象

    optionalObjectOf:React.PropTypes.objectOf(React.PropTypes.number),



    // 特定形状参数的对象

    optionalObjectWithShape:React.PropTypes.shape({

      color: React.PropTypes.string,

      fontSize: React.PropTypes.number

    }),



    // 以后任意类型加上 `isRequired` 来使 prop 不可空。

    requiredFunc:React.PropTypes.func.isRequired,



    // 不可空的任意类型

    requiredAny:React.PropTypes.any.isRequired,



    // 自定义验证器。如果验证失败需要返回一个 Error 对象。不要直接

    // 使用 `console.warn` 或抛异常，因为这样 `oneOfType` 会失效。

    customProp: function(props, propName,componentName) {

      if (!/matchme/.test(props[propName])) {

        return new Error('Validation failed!');

      }

    }

  },

  /* ... */

});

```




3、默认Prop值

React支持以声明式的方式来定义props默认值

```bash
var ComponentWithDefaultProps=

 React.createClass({

 getDefaultProps: function() {

 return {

value: 'defaultvalue'

};

 }

/* ... */

});
```



当父级传入props时，getDefaultProps（）可以保证this.props.value有默认值，注意getDefaultProps的结果会被缓存。得益于此，你可以直接使用props，而不必手动编写一些无意义的代码。



4、传递Props：小技巧

一些常用的React组件只是对HTML做简单的扩展，通常，你想少写代码来把传入组件的props复制到对应的HTML元素上，这时JSX的spread语法会帮到你：

```bash
var CheckLink =React.createClass({

render:function(){

//这样会把CheckList所有的props复制到<a>

return <a{...this.props}>{‘ hhhhh'}{this.props.children}</a>

}

});



React.render(

<CheckLink href="/checked.html">

Click here!

</CheckLink>,

document.getElementById('example')

);
```







5、单个子级

React.Proptypes.element可以限定只能有一个子级传入。

```bash
var MyComponent = React.createClass({

propTypes:{

chileren:React.PropTypes.element.isRequired

},

},

render:function(){

renturn(



<div>

{this.props.children}//有且仅有一个元素，否则会抛异常。

</div>

}

});
```





6、Mixins

组件是React里服用代码最佳方式，但是有时一些复杂的组件间也需要公用一些功能。React使用mixins来解决这类问题。

一个通用的场景是：一个组建需要定期更ixn，用setInterval（）做很容易，担当不需要它的时候取消定时器来节省内存是非常重要的。React提供生命周期方法来公知组件创建或销毁的时间。

```bash
var SetIntervalMixin= {

  componentWillMount: function() {

    this.intervals = [];

  },

  setInterval: function() {

    this.intervals.push(setInterval.apply(null,arguments));

  },

  componentWillUnmount: function() {

    this.intervals.map(clearInterval);

  }

};



var TickTock =React.createClass({

  mixins: [SetIntervalMixin],// 引用 mixin

  getInitialState: function() {

    return {seconds: 0};

  },

  componentDidMount: function() {

   this.setInterval(this.tick, 1000); // 调用 mixin 的方法

  },

  tick: function() {

    this.setState({seconds: this.state.seconds+ 1});

  },

  render: function() {

    return (

      <p>

        React has been running for{this.state.seconds} seconds.

      </p>

    );

  }

});



React.render(

  <TickTock />,

  document.getElementById('example')

);
```

关于mixin一个优点是，如果一个组件使用了多个mixin，并且有多个mixin，并且有多个mixin定义了同样的生命周期方法（如：多个mixin多需要在组件销毁时做资源清理操作），左右这些生命周期方法都保证会被执行到，方法的顺序是：首先按照mixin引入顺序执行mixin里的方法，最后执行组建内定义的方法。
