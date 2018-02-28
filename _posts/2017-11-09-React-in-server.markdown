---
layout: post
title: React服务端渲染及实现原理
date: 2017-11-09 07:33:00
---
React 提供了两个方法 renderToString 和 renderToStaticMarkup 用来将组件（Virtual DOM）输出成  HTML 字符串，这是 React服务器端渲染的基础，它移除了服务器端对于浏览器环境的依赖，所以让服务器  端渲染变成了一件有吸引力的事情。

首先，服务器端渲染除了要解决对浏览器环境的依赖，还要解决两个问题：

1.前后端可以共享代码

2.前后端路由可以统一处理

React 生态提供了很多选择方案，这里我们选用 Redux 和 react-router 来做说明。

### 2 分钟了解 Redux 是如何运作的

 1.Store
 
 整个应用只有一个唯一的 Store
 
 Store 对应的状态树（State），由调用一个 reducer 函数（root reducer）生成状态树上的每个字段都可以进一步由不同的 reducer 函数生成
 
 Store 包含了几个方法比如 dispatch, getState 来处理数据流
 
 Store 的状态树只能由 dispatch(action) 来触发更改

 2.  Redux 的数据流：
 action 是一个包含 { type, payload } 的对象
 reducer 函数通过 store.dispatch(action) 触发
 reducer 函数接受 (state, action) 两个参数，返回一个新的 state
 reducer 函数判断 action.type 然后处理对应的 action.payload 数据来更新状态树

  所以对于整个应用来说，一个 Store 就对应一个 UI快照，服务器端渲染就简化成了在服务器端初始化 Store，将      Store 传入应用的根组件，针对根组件调用 renderToString 就将整个应用输出成包含了初始化数据的 HTML。

### react-router

  react-router 通过一种声明式的方式匹配不同路由决定在页面上展示不同的组件，并且通过props 将路由信息传递给组件使用，所以只要路由变更，props 就会变化，触发组件 re-render。

  假设有一个很简单的应用，只有两个页面，一个列表页 /list 和一个详情页 /item/:id，点击列表上的条目进入详情页。

  可以这样定义路由，./routes.js

  ```bash
  import React from 'react';
  import { Route } from 'react-router';
  import { List, Item } from './components';

  //无状态（stateless）组件，一个简单的容器，react-router 会根据 route
  // 规则匹配到的组件作为 `props.children` 传入
  const Container = (props) => {
    return (
      <div>{props.children}</div>
    );
  };

  // route 规则：
  // - `/list` 显示 `List` 组件
  // - `/item/:id` 显示 `Item` 组件
  const routes = (
    <Route path="/"component={Container} >
      <Route path="list"component={List} />
      <Route path="item/:id"component={Item} />
    </Route>
  );

  export default routes;
  ```

 ### Reducer
  ```bash
  import listReducer from './list';
  import itemReducer from './item';

  export default functionrootReducer(state = {}, action) {
    return {
      list: listReducer(state.list,action),
      item: itemReducer(state.item,action)
    };
  }
  ```
rootReducer 的 state 参数就是整个Store 的状态树，状态树下的每个字段对应也可以有自己的

reducer，所以这里引入了 listReducer 和 itemReducer，可以看到这两个 reducer

的 state 参数就只是整个状态树上对应的 list 和 item 字段。


list.js
```bash
const initialState = [];

export default functionlistReducer(state = initialState, action) {
  switch(action.type) {
  case 'FETCH_LIST_SUCCESS': return[...action.payload];
  default: return state;
  }
}
```
list 就是一个包含 items 的简单数组，可能类似这种结构：[{ id: 0, name:'first item'}, {id: 1, name: 'second item'}]，从 'FETCH_LIST_SUCCESS' 的action.payload 获得。

### Action

对应的应该要有两个 action 来获取 list 和 item，触发 reducer 更改Store，这里我们定义 fetchList 和fetchItem 两个 action。

```bash
import fetch from 'isomorphic-fetch';

export functionfetchList() {
  return (dispatch) => {
    return fetch('/api/list')
        .then(res => res.json())
        .then(json => dispatch({ type:'FETCH_LIST_SUCCESS', payload: json }));
  }
}

export functionfetchItem(id) {
  return (dispatch) => {
    if (!id) returnPromise.resolve();
    return fetch(`/api/item/${id}`)
        .then(res =>res.json())
        .then(json => dispatch({ type:'FETCH_ITEM_SUCCESS', payload: json }));
  }
}
```
isomorphic-fetch 是一个前后端通用的Ajax 实现，前后端要共享代码这点很重要。

另外因为涉及到异步请求，这里的 action 用到了 thunk，也就是函数，redux 通过 thunk-middleware 来处理这类 action，把函数当作普通的 action dispatch就好了，比如 dispatch(fetchList())

###Store

我们用一个独立的 ./store.js，配置（比如Apply Middleware）生成 Store
```bash
import { createStore } from 'redux';
import rootReducer from './reducers';

// Apply middlewarehere
// ...

export default functionconfigureStore(initialState) {
  const store = createStore(rootReducer,initialState);
  return store;
}
```



### react-redux

接下来实现 <List>，<Item> 组件，然后把 redux 和 react 组件关联起来
app.js
```bash
import React from 'react';
import { render } from 'react-dom';
import { Router } from 'react-router';
import createBrowserHistory from'history/lib/createBrowserHistory';
import { Provider } from 'react-redux';
import routes from './routes';
import configureStore from './store';

// `__INITIAL_STATE__`来自服务器端渲染，下一部分细说
const initialState = window.__INITIAL_STATE__;
const store = configureStore(initialState);
const Root = (props) => {
  return (
    <div>
      <Providerstore={store}>
        <Routerhistory={createBrowserHistory()}>
          {routes}
        </Router>
      </Provider>
    </div>
  );
}

render(<Root />,document.getElementById('root'));
```
至此，客户端部分结束。

### Server Rendering

接下来的服务器端就比较简单了，获取数据可以调用 action，routes 在服务器端的处理参考 react-router server rendering，在服务器端用一个 match 方法将拿到的request url 匹配到我们之前定义的 routes，解析成和客户端一致的 props 对象传递给组件。

server.js
```bash
import express from 'express';
import React from 'react';
import { renderToString } from 'react-dom/server';
import { RoutingContext, match } from 'react-router';
import { Provider } from 'react-redux';
import routes from './routes';
import configureStore from './store';

const app = express();

functionrenderFullPage(html, initialState) {
  return `
    <!DOCTYPE html>
    <htmllang="en">
    <head>
      <metacharset="UTF-8">
    </head>
    <body>
      <divid="root">
        <div>
          ${html}
        </div>
      </div>
      <script>
        window.__INITIAL_STATE__ =${JSON.stringify(initialState)};
      </script>
      <scriptsrc="/static/bundle.js"></script>
    </body>
    </html>
  `;
}

app.use((req, res) =>{
  match({ routes, location: req.url },(err, redirectLocation, renderProps) => {
    if (err) {
      res.status(500).end(`InternalServer Error ${err}`);
    } else if (redirectLocation){
     res.redirect(redirectLocation.pathname + redirectLocation.search);
    } else if (renderProps) {
      const store =configureStore();
      const state = store.getState();

Promise.all([
       store.dispatch(fetchList()),
       store.dispatch(fetchItem(renderProps.params.id))
      ])
      .then(() => {
        const html =renderToString(
          <Providerstore={store}>
            <RoutingContext{...renderProps} />
          </Provider>
        );
        res.end(renderFullPage(html,store.getState()));
      });
    } else {
      res.status(404).end('Not found');
    }
  });
});
```

服务器端渲染部分可以直接通过共用客户端 store.dispatch(action) 来统一获取 Store 数据。另外注意renderFullPage 生成的页面 HTML 在 React 组件 mount 的部分(<divid="root">)，前后端的HTML结构应该是一致的。然后要把 store 的状态树写入一个全局变量（__INITIAL_STATE__），这样客户端初始化 render 的时候能够校验服务器生成的 HTML结构，并且同步到初始化状态，然后整个页面被客户端接管。