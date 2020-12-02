---
title: nodejs+koa2+mongoDB搭建基础增删改查案例
date: 2020-03-20 15:13:43
categories: JavaScript
tags: [JavaScript,Nodejs,Koa2,Mongodb]
---

### 写在前面
记录一下 `nodejs` 实现最最最基础的增删改查
项目尽可能遵从 `mvc` 设计思想构建，适合初探 `nodejs` 的前端小伙伴作为参考
项目地址：[https://github.com/JohnnyLuv/nodejs-easy-demo-202003/tree/base](https://github.com/JohnnyLuv/nodejs-easy-demo-202003/tree/base)


### 技术栈
**服务端**：`nodejs` + `koa2` + `mongodb`
**客户端**：`vuejs` + `axios`


### 设想
+ 服务端 只负责数据库交互 和 输出接口
+ 客户端 只负责页面展示 和 接口调用
+ 分离要彻底，前后端 只通过接口交互
+ 客户端渲染，不做模板引擎
+ 实现最基础的用户管理（用户列表/添加用户/编辑用户/删除用户）操作
<!-- more -->

### 项目结构
```js
src                      // 根目录
├─ assets                // 客户端静态资源
│  ├─ css
│  ├─ img
│  └─ js
│     ├─ axios.min.js
│     ├─ util.js
│     └─ vue.min.js
│
├─ controller            // 控制器 用于解析用户输入，处理后返回相应的结果
│  └─ userController.js
│
├─ module                // 模型 用于定义数据模型
│  ├─ config.js          // 服务端配置
│  ├─ db.js              // 数据库请求封装
│  └─ userModule.js
│
├─ views                 // 视图 返回客户端视图
│  ├─ addUser.html
│  ├─ editUser.html
│  └─ index.html
│
├─ app.js                // 入口文件
│
├─ package.json          // yarn/npm 配置文件
│
└─ router.js             // 客户端视图路由
```


### 服务端模块
```js
// package.json

{
  "name": "koa-obj",
  "version": "1.0.0",
  "main": "app.js",
  "license": "MIT",
  "dependencies": {
    "koa": "^2.11.0",
    "koa-bodyparser": "^4.2.1",   // 解析客户端请求 request.body 
    "koa-router": "^8.0.8",       // 服务端路由分发
    "koa-static": "^5.0.0",       // 静态文件目录指向
    "koa-views": "^6.2.1",        // 渲染模板使用，本项目没有使用模板引擎，只用来渲染html文件
    "mongodb": "^3.5.5",          // mongodb 数据库模块
    "node": "^13.10.1",
    "nodemon": "^2.0.2"           // 监听服务端变动，热重载 node 服务
  },
  "scripts": {
    "serve": "nodemon app.js"
  }
}
```


### App入口文件
实际开发中前后端分离应该是两个项目
也就是 `app.js` 中的 **页面路由注册** 是没有的
```js
// app.js

const Koa = require('koa'),
  app = new Koa()

const Router = require('koa-router'),
  router = new Router()

// 指定渲染目录
const views = require('koa-views')
app.use(views('views'))

// request中间件
const bodyParser = require('koa-bodyparser')
app.use(bodyParser())

// 设置静态文件目录
const static = require('koa-static')
app.use(static(__dirname))



// 页面路由注册
const viewRouter = require('./router')
app.use(viewRouter.routes()).use(router.allowedMethods())

// 接口路由注册
const userApi = require('./controller/userController')
app.use(userApi.routes()).use(router.allowedMethods())


app.listen(3000, () => {
  console.log('service running at http://localhost:3000/')
})
```


### Controller模块
负责控制 `Api` 的路由分发和业务逻辑指向
```js
// controller/userController.js

const Router = require('koa-router'),
  router = new Router()

const { login, userList, userInfo, addUser, editUser, removeUser } = require('../module/userModule')


// 用户列表
router.get('/userList', async ctx => {
  const response = await userList(ctx)
  ctx.body = response
})

// 查询用户详情
router.get('/userInfo/:_id', async ctx => {
  const response = await userInfo(ctx)
  ctx.body = response
})

// 添加用户
router.post('/addUser', async ctx => {
  const response = await addUser(ctx)
  ctx.body = response
})

// 修改用户
router.post('/editUser', async ctx => {
  const response = await editUser(ctx)
  ctx.body = response
})

// 删除用户
router.del('/removeUser/:_id', async ctx => {
  const response = await removeUser(ctx)
  ctx.body = response
})

module.exports = router
```


### Module模块
负责处理 `Api` 请求的业务逻辑
```js
// module/userModule.js

const DB = require('./db')


/**
 * 用户列表
 * @param {Object} ctx 
 */
const userList = async ctx => {
  let response = {}
  const data = await DB.find('user', {})
  response = {
    status: 200,
    data,
    msg: 'success',
  }
  return response
}

/**
 * 用户详情
 * @param {Object} ctx 
 */
const userInfo = async ctx => {
  const _id = ctx.params._id
  // console.log(_id)
  const result = await DB.find('user', { _id: DB.ObjectId(_id) })
  const response = {
    status: 200,
    data: result[0],
    msg: 'success',
  }
  return response
}

/**
 * 添加用户
 * @param {Object} ctx 
 */
const addUser = async ctx => {
  const query = ctx.request.body
  // console.log(query)
  let response = {}
  switch (true) {
    case query.username === '':
      response = {
        status: 201,
        msg: 'username 必填',
      }
      break
    case query.password === '':
      response = {
        status: 201,
        msg: 'password 必填',
      }
      break
    default:
      const data = await DB.insert('user', query)
      response = {
        status: 200,
        data: data.ops[0],
        msg: '添加成功',
      }
      break
  }
  return response
}

/**
 * 修改用户
 * @param {Object} ctx 
 */
const editUser = async ctx => {
  const query = ctx.request.body
  // console.log(query)
  await DB.update('user', { _id: DB.ObjectId(query._id) }, { username: query.username, password: query.password })
  const response = {
    status: 200,
    data: { _id: query._id },
    msg: 'success',
  }
  return response
}

/**
 * 删除用户
 * @param {Object} ctx 
 */
const removeUser = async ctx => {
  const _id = ctx.params._id
  await DB.delete('user', { _id: DB.ObjectId(_id) })
  const response = {
    status: 200,
    data: { _id },
    msg: 'success',
  }
  return response
}

module.exports = {
  login,
  userList,
  userInfo,
  addUser,
  editUser,
  removeUser,
}
```


### DB库封装
这里需要注意的是，如果通过 `_id` 来查询，需要提前暴露 `mongodb` 模块的 `ObjectId` 方法
```js
// module/db.js

const { MongoClient, ObjectId } = require('mongodb')
const Config = require('./config.js')


class Db {
  static getInstance() { // 单例模式解决多次实例化问题
    if (!Db.instance) {
      Db.instance = new Db()
    }
    return Db.instance
  }

  constructor() {
    this.dbClient = ''
    this.ObjectId = ObjectId
    this.connect() // 实例化时自动连接数据库，提高查询效率
  }

  connect() {
    return new Promise((resolve, reject) => {
      if (!this.dbClient) { // 解决数据库多次链接问题
        MongoClient.connect(Config.dbUrl, (err, client) => {
          if (err) {
            reject(err)
          }
          let db = client.db(Config.dbName)
          this.dbClient = db
          resolve(this.dbClient)
          // db.close() 注释成为长连接
        })
      } else {
        resolve(this.dbClient)
      }
    })
  }

  find(collectionName, json) {
    return new Promise((resolve, reject) => {
      this.connect().then(db => {
        let result = db.collection(collectionName).find(json)
        result.toArray((err, docs) => {
          if (err) {
            reject(err)
          }
          resolve(docs)
        })
      })
    })
  }

  insert(collectionName, json) {
    return new Promise((resolve, reject) => {
      this.connect().then(db => {
        db.collection(collectionName).insertOne(json, (err, res) => {
          if (err) {
            reject(err)
          } else {
            resolve(res)
          }
        })
      })
    })
  }

  update(collectionName, json1, json2) {
    return new Promise((resolve, reject) => {
      this.connect().then(db => {
        db.collection(collectionName).updateOne(json1, { $set: json2 }, (err, res) => {
          if (err) {
            reject(err)
          } else {
            resolve(res)
          }
        })
      })
    })
  }
  
  delete(collectionName, json) {
    return new Promise((resolve, reject) => {
      this.connect().then(db => {
        db.collection(collectionName).removeOne(json, (err, res) => {
          if (err) {
            reject(err)
          } else {
            resolve(res)
          }
        })
      })
    })
  }
}
module.exports = Db.getInstance()
```


### 文档和攻略
1. mongodb 配置使用：[http://blog.starpoetry.cn/](http://blog.starpoetry.cn/2020/03/13/mongodb-in-macos10/)
2. koa2 文档：[https://koa.bootcss.com](https://koa.bootcss.com)
3. koa-router 服务端路由文档：[https://github.com/koajs/router/](https://github.com/koajs/router/blob/master/API.md)
4. axios 客户端路由文档：[https://github.com/axios/](https://github.com/axios/axios)


### 最后
<center>**完结撒花 🥳**</center>