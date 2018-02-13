---
layout: post
title: React组件之间传值
date: 2017-10-21 23:15:30
---


React 组件之间交流的方式，可以分为以下 3 种：

1.【父组件】向【子组件】传值；

2.【子组件】向【父组件】传值；

3.没有任何嵌套关系的组件之间传值（PS：比如：兄弟组件之间传值）

## 【父组件】向【子组件】传值

### 初步使用

这个是相当容易的，在使用 React 开发的过程中经常会使用到，主要是利用 props 来进行交流。例子如下：
```bash
// 父组件
var MyContainer = React.createClass({
  getInitialState: function () {
    return {
      checked: true
    };
  },
  render: function() {
    return (
      <ToggleButton text="Toggle me" checked={this.state.checked} />
    );
  }
});

// 子组件
var ToggleButton = React.createClass({
  render: function () {
    // 从【父组件】获取的值
    var checked = this.props.checked,
        text = this.props.text;

    return (
        <label>{text}: <input type="checkbox" checked={checked} /></label>
    );
  }
});
```
### 进一步讨论

如果组件嵌套层次太深，那么从外到内组件的交流成本就变得很高，通过 props 传递值的优势就不那么明显了。
```bash
// 父组件
var MyContainer = React.createClass({
  render: function() {
    return (
      <Intermediate text="where is my son?" />
    );
  }
});

// 子组件1：中间嵌套的组件
var Intermediate = React.createClass({
  render: function () {
    return (
      <Child text={this.props.text} />
    );
  }
});

// 子组件2：子组件1的子组件
var Child = React.createClass({
  render: function () {
    return (
      <span>{this.props.text}</span>
    );
  }
});
```

## 【子组件】向【父组件】传值

接下来，我们介绍【子组件】控制自己的 state 然后告诉【父组件】的点击状态，然后在【父组件】中展示出来。因此，我们添加一个 change 事件来做交互。
```bash
// 父组件
var MyContainer = React.createClass({
  getInitialState: function () {
    return {
      checked: false
    };
  },
  onChildChanged: function (newState) {
    this.setState({
      checked: newState
    });
  },
  render: function() {
    var isChecked = this.state.checked ? 'yes' : 'no';
    return (
      <div>
        <div>Are you checked: {isChecked}</div>
        <ToggleButton text="Toggle me"
          initialChecked={this.state.checked}
          callbackParent={this.onChildChanged}
          />
      </div>
    );
  }
});

// 子组件
var ToggleButton = React.createClass({
  getInitialState: function () {
    return {
      checked: this.props.initialChecked
    };
  },
  onTextChange: function () {
    var newState = !this.state.checked;
    this.setState({
      checked: newState
    });
    // 这里要注意：setState 是一个异步方法，所以需要操作缓存的当前值
    this.props.callbackParent(newState);
  },
  render: function () {
    // 从【父组件】获取的值
    var text = this.props.text;
    // 组件自身的状态数据
    var checked = this.state.checked;

    return (
        <label>{text}: <input type="checkbox" checked={checked}                 onChange={this.onTextChange} /></label>
    );
  }
});
```

## 没有任何嵌套关系的组件之间传值

如果组件之间没有任何关系，组件嵌套层次比较深（个人认为 2 层以上已经算深了），或者你为了一些组件能够订阅、写入一些信号，不想让组件之间插入一个组件，让两个组件处于独立的关系。对于事件系统，这里有 2个基本操作步骤：订阅（subscribe）/监听（listen）一个事件通知，并发送（send）/触发（trigger）/发布（publish）/发送（dispatch）一个事件通知那些想要的组件。

下面讲介绍 3 种模式来处理事件。

(1) Event Emitter/Target/Dispatcher: 需要一个指定的订阅源

```bash
// to subscribe
otherObject.addEventListener(‘click’, function() { alert(‘click!’); });
// to dispatch
this.dispatchEvent(‘click’);
```
(2) Publish / Subscribe

触发事件的时候，你不需要指定一个特定的源，因为它是使用一个全局对象来处理事件（其实就是一个全局广播的方式来处理事件）

```bash
// to subscribe
globalBroadcaster.subscribe(‘click’, function() { alert(‘click!’); });
// to dispatch
globalBroadcaster.publish(‘click’);
```
(3) Signals

与Event Emitter/Target/Dispatcher相似，但是你不要使用随机的字符串作为事件触发的引用。触发事件的每一个对象都需要一个确切的名字（就是类似硬编码类的去写事件名字），并且在触发的时候，也必须要指定确切的事件。

```bash
// to subscribe
otherObject.clicked.add(function() { alert(‘click’); });
// to dispatch
this.clicked.dispatch();
```
简单实现的demo

```bash
// 简单实现了一下 subscribe 和 dispatch
var EventEmitter = {
    _events: {},
    dispatch: function (event, data) {
        if (!this._events[event]) { // 没有监听事件
          return;
        }
        for (var i = 0; i < this._events[event].length; i++) {
            this._events[event][i](data);
        }
    },
    subscribe: function (event, callback) {
      // 创建一个新事件数组
      if (!this._events[event]) {
        this._events[event] = [];
      }
      this._events[event].push(callback);
    }
};

otherObject.subscribe('namechanged', function(data) { alert(data.name); });
this.dispatch('namechanged', { name: 'Emma' });
```





