---
layout: post
title: ReactNative的组件架构设计
date: 2017-12-15 12:00:00
---
### 整体架构考量

#### 交互维度中需要解决的关键问题及思路
 
组件对外发布：组件对外只允许使用props来暴露功能，不允许使用接口及其它一切方式

父子组件间：组件的子组件通过父组件传递的接口来与父组件通信

兄弟组件间:

方案1：假设a要调用b，参考第一条的话，其实就是a要改变b的props，那么a只要改b的props的来源即可，b的props的来源一般就是根组件的state。那么根组件就要有组织和协调的能力。

方案2：利用事件机制，基本同flux架构。略复杂，且我们并不需要事件的特性，本架构设计不推荐。

#### 实现相马涉及的关键部分及其职责

root-存放state，组织子view组件，组织业务逻辑对象等

子view组件-根据this.props渲染view。

业务逻辑对象-提供业务逻辑方法

### 面向对象的ReactNative组件\页面架构设计

一个独立完整的组件\页面一般由以下元素构成：

root组件，1个，负责初始化state，负责提供对外props列表，负责组合子view组件形成页面效果，负责注册业务逻辑对象提供的业务逻辑方法，负责管理业务逻辑对象

view子组件，0-n个，根据props进行视图的渲染

业务逻辑对象，0-n个，提供业务逻辑方法


#### root组件

root组件由以下元素组成：

props-公有属性

state-RN体系的状态,必须使用Immutable对象

私有属性

业务逻辑对象的引用-在componentWillMount中初始化

私有方法-以下划线开头，内部使用or传递给子组件使用

公有方法【不推荐】，子组件和外部组件都可以用，但不推荐用公有方法来对外发布功能，破坏了面向状态编程，尽可能的使用props来发布功能

注意：定义root组件的state的时候，如果使用es6的方式，要把state的初始化放到componentWillMount中，如果在构造器中this.props为空。

子view组件中包含：

1.props-公有属性

私有属性-如果你不能理解下面的要求，建议没有，统一放在父组件上

绝对不允许和父组件的属性or状态有冗余。无论是显性冗余还是计算结果冗余，除非你能确定结算是性能的瓶颈。

此属性只有自己会用，父组件和兄弟组件不会使用，如果你不确定这点，请把这个组件放到父组件上，方便组件间通信

私有方法-仅作为渲染view的使用，不许有业务逻辑

公有方法【不推荐，理由同root组件】

#### 业务逻辑对象

业务逻辑对象由以下元素组成：

root组件对象引用-this.root

构造器-初始化root对象，初始化私有属性

私有属性

公有方法-对外提供业务逻辑

私有方法-以下划线开头，内部使用

ps1：通用型组件只要求尽量满足上述架构设计

通用型组件一般为不包含任何业务的纯技术组件，具有高复用价值、高定制性、通常不能直接使用需要代码定制等特点。

可以说是一个系统的各个基础零件，比如一个蒙板效果，或者一个模态弹出框。

架构的最终目的是保证系统整体结构良好，代码质量良好，易于维护。一般编写通用型组件的人也是经验较为丰富的工程师，代码质量会有保证。而且，作为零件的通用组件的使用场景和生命周期都和普通组件\页面不同，所以，仅要求通用组件编写尽量满足架构设计即可。

ps2：view子组件复用问题

抛出一个问题，设计的过程中，子组件是否需要复用？子组件是否需要复用会影响到组件设计。

需复用，只暴露props，可以内部自行管理state【尽量避免除非业务需要】

不需复用，只暴露props，内部无state【因为不会单独使用，不需要setState来触发渲染】

其实， 一般按照不需复用的情况设计，除非复用很明确，但这时候应该抽出去，变成独立的组件存在就可以了，所以这个问题是不存在的。

### 如何做监控

因为面向对象的RN架构中去掉了统一的业务逻辑调用facade入口dispatch，那我们如何来做监控呢。

方案1：在需要监控的地方人为加入监控点。

这个方案对业务代码和监控代码的耦合确实有点大，是最差的解决方案了。不推荐。

方案2：在基类BaseLogicObj的构造器中对对象的所有方法进行代理-todo待验证

这个方案对业务代码透明，但是还只是个想法，未进行代码测试和验证。

方案3.....还没有想出别的方案，有没有同学给点思路？

### demo based on Redux

1.list 组件
```bash
'use strict'


let React=require('react-native');
let Immutable = require('immutable');
var BbtRN=require('../../../bbt-react-native');


var {
    BaseLogicObj,
    }=BbtRN;


let {
    AppRegistry,
    Component,
    StyleSheet,
    Text,
    View,
    Navigator,
    TouchableHighlight,
    TouchableOpacity,
    Platform,
    ListView,
    TextInput,
    ScrollView,
    }=React;

//root组件开始-----------------

let  Root =React.createClass({

    //初始化模拟数据，
    data:[{
        name:'aaaaa',
        completed:true,
    },{
        name:'bbbbb',
        completed:false,
    },{
        name:'ccccc',
        completed:false,
    }
    ,{
        name:'ddddd',
        completed:true,
    }],


    componentWillMount(){

        //初始化业务逻辑对象
        this.addTodoObj=new AddTodoObj(this);
        this.todoListObj=new TodoListObj(this);
        this.filterObj=new FilterObj(this);

        //下面可以继续做一些组件初始化动作，比如请求数据等.
        //当然了这些动作最好是业务逻辑对象提供的，这样root组件将非常干净.
        //例如这样：this.todoListObj.queryData();
    },


    //状态初始化
    getInitialState(){
      return {
          data:Immutable.fromJS(this.data),//模拟的初始化数据
          todoName:'',//新任务的text
          curFilter:'all',//过滤条件 all no ok
      }
    },



    //这里组合子view组件 并 注册业务逻辑对象提供的方法到各个子view组件上
    render(){

        return (
            <View style={{marginTop:40,flex:1}}>

                <AddTodo todoName={this.state.todoName}
                        changeText={this.addTodoObj.change.bind(this.addTodoObj)}
                         pressAdd={this.addTodoObj.press.bind(this.addTodoObj)} />

                <TodoList todos={this.state.data}
                          onTodoPress={this.todoListObj.pressTodo.bind(this.todoListObj)} />

                <Footer curFilter={this.state.curFilter}
                    onFilterPress={this.filterObj.filter.bind(this.filterObj)} />

            </View>
        );
    },



});






//业务逻辑对象开始-------------------------可以使用OO的设计方式设计成多个对象

//业务逻辑对象要符合命名规范：以Obj结尾
//BaseLogicObj是架构提供的基类，里面封装了构造器和一些常用取值函数
class AddTodoObj extends BaseLogicObj{

    press(){
        if(!this.getState().todoName)return;
        let list=this.getState().data;
        let todo=Immutable.fromJS({name:this.getState().todoName,completed:false,});
        this.setState({data:list.push(todo),todoName:''});
    }

    change(e){
        this.setState({todoName:e.nativeEvent.text});
    }

}


class TodoListObj extends BaseLogicObj {




    pressTodo(todo){

        let data=this.getState().data;

        let i=data.indexOf(todo);

        let todo2=todo.set('completed',!todo.get('completed'));

        this.setState({data:data.set(i,todo2)});
    }
}


class FilterObj extends BaseLogicObj {


    filter(type){

        let data=this.getState().data.toJS();
        if(type=='all'){
            data.map((todo)=>{
                todo.show=true;
            });
        }else if(type=='no'){
            data.map((todo)=>{
                if(todo.completed)todo.show=false;
                else todo.show=true;
             });
        }else if(type=='ok'){
            data.map((todo)=>{
                if(todo.completed)todo.show=true;
                else todo.show=false;
            });
        }


        this.setState({curFilter:type,data:Immutable.fromJS(data)});
    }



}


//view子组件开始---------------------------


//子view对象中仅仅关注：从this.props转化成view
let Footer=React.createClass({

    render(){

        return (


            <View style={{flexDirection:'row', justifyContent:'flex-end',marginBottom:10,}}>

                <FooterBtn {...this.props} title='全部' name='all'  cur={this.props.curFilter=='all'?true:false} />
                <FooterBtn {...this.props} title='未完成' name='no' cur={this.props.curFilter=='no'?true:false} />
                <FooterBtn {...this.props} title='已完成' name='ok' cur={this.props.curFilter=='ok'?true:false} />

            </View>



        );
    },


});


let FooterBtn=React.createClass({

    render(){

        return (

            <TouchableOpacity onPress={()=>this.props.onFilterPress(this.props.name)}
                              style={[{padding:10,marginRight:10},this.props.cur?{backgroundColor:'green'}:null]} >
                <Text style={[this.props.cur?{color:'fff'}:null]}>
                    {this.props.title}
                </Text>
            </TouchableOpacity>

        );
    },


});


let AddTodo=React.createClass({

    render(){

        return (


            <View style={{flexDirection:'row', alignItems:'center'}}>


                <TextInput value={this.props.todoName}
                    onChange={this.props.changeText}
                    style={{width:200,height:40,borderWidth:1,borderColor:'e5e5e5',margin:10,}}></TextInput>


                <TouchableOpacity onPress={this.props.pressAdd}
                    style={{backgroundColor:'green',padding:10}} >
                    <Text style={{color:'fff'}} >
                        添加任务
                    </Text>
                </TouchableOpacity>

            </View>



        );
    },


});



let Todo=React.createClass({

    render(){
        let todo=this.props.todo;
        return (
            todo.get("show")!=false?
            <TouchableOpacity  onPress={()=>this.props.onTodoPress(todo)}
                style={{padding:10,borderBottomWidth:1,borderBottomColor:'#e5e5e5'}}>
                <Text style={[todo.get('completed')==true?{textDecorationLine:'line-through',color:'#999'}:null]} >
                    {todo.get('completed')==true?'已完成   ':'未完成   '} {todo.get('name')}
                </Text>
            </TouchableOpacity>
             :null
        );
    },


});


let TodoList=React.createClass({
    render(){
        return (
            <ScrollView style={{flex:1}}>
                {this.props.todos.reverse().map((todo, index) => <Todo {...this.props} todo={todo} key={index}  />)}
            </ScrollView>
        );
    },
});




module.exports=Root;
```
2.业务逻辑对象类

```bash
'use strict'

class BaseLogicObj{


    constructor(root){
        if(!root){
            console.error('实例化BaseLogicObj必须传入root组件对象.');
        }
        this.root=root;
    }

    getState(){
        return this.root.state;
    }

    setState(s){
        this.root.setState(s);
    }

    getRefs(){
        return this.root.refs;
    }

    getProps(){
        return this.root.props;
    }
```