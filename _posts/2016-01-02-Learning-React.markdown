---
layout: post
title: 初识 React
date: 2016-01-02 05:10:00
---


### 下载 react

$ git clone git@github.com:ruanyf/react-demos.git

### HTML模板

```bash
<!DOCTYPE html>
<html>
  <head>
    <script src="../build/react.js"></script>
    <script src="../build/react-dom.js"></script>
    <script src="../build/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      // ** Our code goes here! **
    </script>
  </body>
</html>
```
上面代码一共用了三个库： react.js 、react-dom.js 和 Browser.js ，它们必须首先加载。其中，react.js 是 React 的核心库，react-dom.js 是提供与 DOM 相关的功能，Browser.js 的作用是将 JSX 语法转为 JavaScript语法，这一步很消耗时间，实际上线的时候，应该将它放到服务器完成。

$ babel src --out-dir build    将 src 子目录的 js 文件进行语法转换，转码后的文件全部放在 build 子目录。

```bash
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('example')
);
```
### JSX语法

HTML语言直接写在 JavaScript语言之中，不加任何引号，这就是 JSX的语法，它允许 HTML与 JavaScript的混写

```bash
var names=['Alice','Emily','Kate'];

ReactDOM.render(
  <div>
  {
    names.map(function(name) {
      return <div>Hello, {name}!</div>
    })
  }
  </div>,
  document.getElementById('example')
);
```

### 组件

React允许将代码封装成组件（component），然后像插入普通 HTML 标签一样，在网页中插入这个组件。React.createClass方法就用于生成一个组件类。

```bash
var HelloMessage= React.createClass({
  render: function() {
    return <h1>Hello{this.props.name}</h1>;
  }
});

ReactDOM.render(
  <HelloMessage name="John"/>,
  document.getElementById('example')
);
```

上面代码中，变量 HelloMessage 就是一个组件类。模板插入 <HelloMessage /> 时，会自动生成 HelloMessage的一个实例（下文的"组件"都指组件类的实例）。所有组件类都必须有自己的 render 方法，用于输出组件。

注意，组件类的第一个字母必须大写，否则会报错，比如HelloMessage不能写成helloMessage。另外，组件类只能包含一个顶层标签，否则也会报错。

### this.props.children

this.props 对象的属性与组件的属性一一对应，但是有一个例外，就是 this.props.children 属性。它表示组件的所有子节点

```bash
var NotesList= React.createClass({
  render: function() {
    return (
      <ol>
      {
        React.Children.map(this.props.children,function(child) {
          return <li>{child}</li>;
        })
      }
      </ol>
    );
  }
});

ReactDOM.render(
  <NotesList>
    <span>hello</span>
    <span>world</span>
  </NotesList>,
  document.body
);
```


### PropTypes

组件的属性可以接受任意值，字符串、对象、函数等等都可以。有时，我们需要一种机制，验证别人使用组件时，提供的参数是否符合要求。

组件类的PropTypes属性，就是用来验证组件实例的属性是否符合要求
```bash
var MyTitle= React.createClass({
  propTypes: {
    title: React.PropTypes.string.isRequired,
  },

render:function() {
     return <h1>{this.props.title}</h1>;
   }
});
```

上面的Mytitle组件有一个title属性。PropTypes 告诉 React，这个 title 属性是必须的，而且它的值必须是字符串。现在，我们设置 title 属性的值是一个数值。

### 获取真实的DOM节点

组件并不是真实的 DOM节点，而是存在于内存之中的一种数据结构，叫做虚拟 DOM（virtual DOM）。只有当它插入文档以后，才会变成真实的 DOM。根据 React的设计，所有的 DOM变动，都先在虚拟 DOM上发生，然后再将实际发生变动的部分，反映在真实 DOM上，这种算法叫做 DOM diff ，它可以极大提高网页的性能表现。

但是，有时需要从组件获取真实 DOM的节点，这时就要用到 ref 属性

### this.state

组件免不了要与用户互动，React的一大创新，就是将组件看成是一个状态机，一开始有一个初始状态，然后用户互动，导致状态变化，从而触发重新渲染 UI, this.state 是会随着用户互动而产生变化的特性

### 表单

用户在表单填入的内容，属于用户跟组件的互动，所以不能用 this.props 读取

```bash
var Input= React.createClass({
  getInitialState: function() {
    return {value:'Hello!'};
  },
  handleChange: function(event) {
    this.setState({value: event.target.value});
  },
  render: function() {
    var value= this.state.value;
    return (
      <div>
        <input type="text" value={value} onChange={this.handleChange}/>
        <p>{value}</p>
      </div>
    );
  }
});

ReactDOM.render(<Input/>, document.body);
```

上面代码中，文本输入框的值，不能用 this.props.value 读取，而要定义一个 onChange 事件的回调函数，通过 event.target.value 读取用户输入的值。textarea 元素、select元素、radio元素都属于这种情况，更多介绍请参考官方文档。


### 组件的生命周期

组件的生命周期分成三个状态：

Mounting：已插入真实 DOM

Updating：正在被重新渲染

Unmounting：已移出真实 DOM

React为每个状态都提供了两种处理函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用，三种状态共计五种处理函数。


componentWillMount()

componentDidMount()

componentWillUpdate(object nextProps, object nextState)

componentDidUpdate(object prevProps, object prevState)
componentWillUnmount()

此外，React还提供两种特殊状态的处理函数。

componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用

shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用

这些方法的详细说明，可以参考官方文档。

demo

```bash
var Hello= React.createClass({
  getInitialState: function () {
    return {
      opacity: 1.0
    };
  },

componentDidMount:function() {
    this.timer=setInterval(function() {
      var opacity= this.state.opacity;
      opacity -= .05;
      if (opacity<0.1) {
        opacity = 1.0;
      }
      this.setState({
        opacity: opacity
      });
    }.bind(this),100);
  },

render:function() {
    return (
      <div style={{opacity: this.state.opacity}}>
        Hello {this.props.name}
      </div>
    );
  }
});

ReactDOM.render(
  <Hello name="world"/>,
  document.body
);

```

上面代码在hello组件加载以后，通过 componentDidMount 方法设置一个定时器，每隔100毫秒，就重新设置组件的透明度，从而引发重新渲染。

### Ajax

组件的数据来源，通常是通过 Ajax请求从服务器获取，可以使用 componentDidMount 方法设置 Ajax 请求，等到请求成功，再用 this.setState 方法重新渲染 UI

```bash
var UserGist= React.createClass({
  getInitialState: function() {
    return {
      username: '',
      lastGistUrl: ''
    };
  },

componentDidMount:function() {
    $.get(this.props.source,function(result) {
      var lastGist= result[0];
      if (this.isMounted()) {
        this.setState({
          username: lastGist.owner.login,
          lastGistUrl: lastGist.html_url
        });
      }
    }.bind(this));
  },

render:function() {
    return (
      <div>
        {this.state.username}'s last gist is
        <a href={this.state.lastGistUrl}>here</a>.
      </div>
    );
  }
});

ReactDOM.render(
  <UserGist source="https://api.github.com/users/octocat/gists"/>,
  document.body
);
```

上面代码使用 jQuery完成 Ajax请求，这是为了便于说明。React本身没有任何依赖，完全可以不用jQuery，而使用其他库。

我们甚至可以把一个Promise对象传入组件,例如

```bash
ReactDOM.render(
  <RepoList
    promise={$.getJSON('https://api.github.com/search/repositories?q=javascript&sort=stars')}
  />,
  document.body
);
```