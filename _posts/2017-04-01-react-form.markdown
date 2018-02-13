---
layout: post
title: React Form
date: 2016-04-01 23:10:30
---

## 表单事件

React支持所有的HTML事件，这些事件遵循驼峰命名的约定，且会转成合成事件，这些事件是标准化的，提供饿了跨浏览器的一致接口。所有合成事件提供了event.target来访问触发事件的DOM节点。

```bash
handleEvent：funciton(syntheticEvent){

var DOMNode=syntheticEvent.target;

var newValue = DOMNode.value;
}
```
这是访问约束组件的值的最简单方式之一。Lavbel是表单元素中很重要的组件，通过Label可以明确地向用户传达你的要求，提升单选框和复选框的可用性。


### Label

但Label与for属性有一个冲突的地方，因为使用JSX，这个属性会被转换成一个JavaScript对象，且作为第一个参数传递给组件的构造器，但由于for属于JavaScript的一个保留字，所以我们无法把它作为一个对象的属性。

在React中，与class变成了className类似，for也变成了htmlFor。

```bash
//JSX
<labelhtmlFor ="name">Name :</label>
//React.DOM.label({htmlFor:"Name:");
```
渲染后：

<label for = "name">Name<label>

### 文本框和Select

React对<textarea/>和<select/>的接口做了一些修改，提升了一致性，让它们操作起来更容易。

<textarea/>被改的更像<input/>了，允许我们设置value和defaulteValue。

```bash
//非约束的

<textareadefaultValue ="HelloWorld" />



//约束的

<textareavalue={this.state.helloTo } onChange ={this.handleChange} />



<selectdefaultValue="8">

<option value="A">First Option</option>

<option value="B">Second Option</option>

<option value= "C">Third Option</option>

</select>



//约束的

<selectvalue ={this.state.helloTo} onchange={this.handleChange}>

<option value="A">FirstOption </option>

<option value="B">Second Option </option>

<option value="C">Third Option</option>

</select>
```

React支持都选selce他，需要给value的defauletValue传递一个数组，如：defaultValue={["A","B"]}.


### 复选框和单选框

复选框和单选框使用的则是完全不同的控制方式。

在HTML中，类似为checkbox或者radio的<input/>的行为完全不一样，通常，复选框或者单选框的值是不变的，只有checked的状态会变化，要控制复选框或者单选框，就要控制他们的checked属性，你要可以在非约束的复选或者单选框中使用defaultChecked。

###  多表单与change处理器

在使用约束的表单组件时，没有愿意重复地为每一个组件编写change处理器。可以在React中重用一个事件处理器。

示例：通过.bind传递其他参数

```bash

varMyForm =React.createClass({

getIntialState:funciton(){

return{

given_name:"",

family_name:""

};

},

handleChange:funciton(name,event){

var newState={};

newState[name]=event.target.value;

this.setState(newState);

},

submitHandler:funciton(event){

event.preventDefault();

var words=[

"Hi",this.state.given_name,this.state.family_name

];

alert(words.join(" "));

},

render:funciton(){

return(

<form onSubmit ={this.submitHandler}>

<label htmlFor="given_name">givenName:</label>

<br />

<input

type="text"

name ="given_name"

value ={this.state.given_name}

onChange={this.handleChange.bind(this,'given_name')}/>

<br/>

<label htmlFor ="family_name">FamilyName:</label>

<br/>

<input

type="text"

name ="given_name"

value ={this.state.given_name}

onChange={this.handleChange.bind(this,'given_name')}/>

<br/>

</form>

);
}
});
```

### Focus

React实现了autoFocus属性，因此在组件第一次被挂载时，如果没有其他的表单聚焦时React就会把焦点放到这个组件对应的表单域中，例如：

<input type="text" name="given_name"autoFocus=“true”/>

还有一种方法就是调用DOMNode的focus方法，手动设置表单域聚焦。

### 可用性

React虽然提高了开发者的生产力，但是也有不尽如人意的地方，使用React编写的组件常常缺乏可用性，例如表单提交无法通过键盘敲击回车键来实现，而这明明是HTML表单默认的提交方式

