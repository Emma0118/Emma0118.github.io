React 组件之间交流的方式，可以分为以下 3 种：

1.【父组件】向【子组件】传值；

2.【子组件】向【父组件】传值；

3.没有任何嵌套关系的组件之间传值（PS：比如：兄弟组件之间传值）

## 【父组件】向【子组件】传值
###初步使用
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