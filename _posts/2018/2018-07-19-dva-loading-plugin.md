---
layout: single
author_profile: true
title: "dva-loading用法"
date: 2018-07-19 14:24:53
# toc: true
tags:
  - dva
  - dva-loading
categories:
  - dva
  - 前端技术
---

dva-loading是dvajs的一个插件，封装了对loading状态的处理。它提供了对当前异步加载方法的状态（异步加载中状态为 true，异步加载完成状态为 false）的监听和追踪, 可以用来设定ant design中的组件的loading属性

### 传统做法
比如请求一个用户页面，刚进去的时候由于要去服务器请求数据需要花费时间，这段时间页面应该是不能点击的状态。

如果不使用这个组件，传统做法是写个Model层，提供两个方法 start() 和 end()，当进行 ajax 请求开始时调用 start() 方法给整个页面加上一层蒙版，此时不能进行操作，在请求结束也就是 ajax 的 success 回调函数中调用 end() 方法关闭蒙版，表明数据已经请求到了，可以操作页面。

### dva-loading做法

dva-loading就这个插件就是帮你做了这部分工作，你只需要focus在核心逻辑上就可以了。  
该组件能够监听我们在model里定义的effacts操作的异步加载状态，这从它的调用方式就可以看出来``` const isLoading = loading.effects['user/query']```，其中 user/query 是 model 中的异步请求方法。

loading 在异步请求发出那一刻会持续监听该异步请求方法的状态，在异步请求结束之前 isLoading 的值一直是 true，当此次异步请求结束时 isLoading 的值变成 false，同时 loading 对象停止监听。

用法示例:

dva项目的index.js文件

```
import createLoading from 'dva-loading';

const app = dva();

app.use(createLoading());
```

只要我们进行了上面的配置，我们在任何一个dva项目的route组件中都会比原来多一个loading对象。举例说明:  
没有配置dva-loading的时候，connet是这样的

```
export default connect(({ app }) => ({ app }))(someComponent);
```

配置了dva-loading以后，connect是这样的, 比上面多了一个loading对象

```
export default connect(({ app, loading }) => ({ app, loading }))(someComponent);
```

打印一下loading， 可以看到内容如下:
```
loading: {
  global: false,
  models: {app: false},  //已经注册过的model都会在这里有
  effects: {app: false}  //这里会有我们注册过的model的effect的状态信息，表明是否加载状态
}
```

比如 loading.effects['user/query'] 为监听<code>'user/query'</code>这个异步请求状态，当页面处于异步加载状态时该值为 true，当页面加载完成时，自动监听该值为 false。

如果同时发出若干个异步请求，需求是当所有异步请求都响应才做下一步操作，可以使用 loading.global() 方法，该方法监听所有异步请求的状态。


代码示例:
```
// src > models >user.js
export default {
  namespace: 'user',
  state: {
    userList: [],      // 存放用户列表
  },
  effects: {
    * query ({ payload = {} }, { call, put }) {
      // 获取用户列表，赋值给 userList
      // 使用 axios 或者 ajax 请求后台返回数据
      const result = axios.request('xxx/xxx');
      // 调用 reducers 中的 updateState 方法改变 state 中 userList 的值
      yield put({ type: 'updateState', payload: { userList: result.data });
    }
  },
  reducers: {
    updateState (state, { payload }) {
      return { ...state, ...payload };
    },
  },
}

```

```
// src > routes > user.js
import React from 'react';
import { connect } from 'dva';
import { Table } from 'antd';

const User = ({ dispatch, user, loading }) => {
  /** 
    根据 loading.effects 对象判断当前异步加载是否完成，并将该值传递给 Table 组件的 loading 属性，
    Table 组件会自己控制加载样式。dva-loading 在这里的作用只是提供异步加载的状态，
    具体加载样式由对应组件自己提供。
  */
  const isLoading = loading.effects['user/query']
  const { userList } = user

  return (
    <Table
      dataSource={userList}
      loading={isLoading}
      rowKey={record => record.id}
    />
  );
}

export default connect(({ user, loading }) => ({ user, loading }))(User);

```
