---
title: nodejs+koa2+jwt登录验证和鉴权
date: 2020-03-24 10:19:47
categories: JavaScript
tags: [JavaScript,Nodejs,Koa2,Mongodb]
---

### 写在前面
上篇中，我们做了基础的用户增删改查
这一篇我们加入登录验证
项目地址：[https://github.com/JohnnyLuv/nodejs-easy-demo-202003/tree/jwt](https://github.com/JohnnyLuv/nodejs-easy-demo-202003/tree/jwt)


### 交互流程
用户发起登录，通过数据库判断登录结果，成功后签发 `token`
客户端本地保存 `token`，访问需要登录权限的接口时，在请求头中携带 `token`
服务器验证 `token`是否有效，并做相应返回
这种验证机制就是 **`jwt`**🦄
```bash
|--------|                             |---------|
|        |         user login          |         |
|        |---------------------------->|         |
|        |<----------------------------|         |
|        |       response token        |         |
|        |                             |         |
|        |                             |         |
|        |  request with valid token   |         |
| client |---------------------------->| service |
|        |<----------------------------|         |
|        |        response 200         |         |
|        |                             |         |       
|        |                             |         |
|        |   request with bad token    |         |
|        |---------------------------->|         |
|        |<----------------------------|         |
|        |        response 401         |         |
|--------|                             |---------|
```
<!-- more -->


### 什么是jwt
> JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。


### 服务端-生成token
在 `controller` 中暴露 `login` 接口
```js
// controller/userController.js

const Router = require('koa-router'),
  router = new Router()
router.prefix('/api') // 添加请求前缀

const userModule = require('../module/userModule')

// 用户登录
router.post('/login', async ctx => {
  const response = await userModule.login(ctx)
  ctx.body = response
})

module.exports = router
```

安装 `jsonwebtoken` 来进行加密和签名
```bash
$ yarn add jsonwebtoken
```

在 `module` 中添加 `login` 的业务逻辑
```js
// module/userModule.js

const DB = require('./db')
const jwt = require('jsonwebtoken')


/**
 * 用户登录
 * @param {Object} ctx 
 */
const login = async ctx => {
  const query = ctx.request.body
  // console.log(query)

  let response = {}
  switch (true) {
    case !query.username:
      response = { status: 201, msg: 'username 必填' }
      break
    case !query.password:
      response = { status: 201, msg: 'password 必填' }
      break
    default:
      const data = await DB.find('user', { username: query.username })
      if (query.username === data[0].username && query.password === data[0].password) {
        // 用户名和密码匹配，登录成功，生成token
        const token = jwt.sign(
          // 携带自定义参数
          {
            _id: data[0]._id,
            username: data[0].username,
          },
          'johnny_jwt_secret', // 私钥，加密和解密时，要保证一致
          { expiresIn: '1h' } // 过期时间
        )
        response = {
          status: 200,
          data: { ...data[0], token: `Bearer ${token}` },
          msg: '登录成功',
        }
      } else {
        response = { status: 201, msg: '账号或密码错误' }
      }
      break
  }
  return response
}


module.exports = {
  login
}
```


### 客户端-获取token
客户端发起登录请求，登录成功后获取 `token` 并保存在本地
```js
// views/login.html

new Vue({
  el: "#root",
  data: {
    username: "",
    password: "",
  },
  methods: {
    onSubmit() {
      const params = {
        username: this.username,
        password: this.password,
      }
      // console.log(params)
      axios.post('/api/login', params).then(res => {
        alert('登录成功', res.username)
        localStorage.setItem('token', res.token)
        location.href = '/'
      })
    }
  }
})
```

配置axios请求拦截，向请求头注入 `token`
```js
// assets/js/util.js

// 客户端请求拦截器
axios.interceptors.request.use(
  config => {
    // Do something before request is sent
    const token = window.localStorage.getItem('token')
    token && (config.headers.Authorization = token) // 在请求头中设置 Authorization:token
    return config
  },
  error => {
    // Do something with request error
    return Promise.reject(error)
  }
)

// 服务端响应拦截器
axios.interceptors.response.use(
  response => {
    // Any status code that lie within the range of 2xx cause this function to trigger
    // Do something with response data
    switch (response.data.status) {
      case 200:
        return response.data.data
      case 401:
        // 状态码401 token失效需要重新登录重新获取
        alert(response.data.msg)
        location.href = '/login'
        // return Promise.reject(response.data)
      default:
        alert(response.data.msg)
        return Promise.reject(response.data)
    }
  },
  error => {
    // Any status codes that falls outside the range of 2xx cause this function to trigger
    // Do something with response error
    alert('网路异常')
    return Promise.reject(error)
  }
)
```


### 服务端-验证token
安装 `koa-jwt` 进行解密，虽然 `jsonwebtoken` 自带解密方法，但是比较繁琐
```bash
$ yarn add koa-jwt
```

在入口文件添加鉴权验证，需要注意的是中间件的顺序，jwt鉴权模块要放在所有路由模块之前
```js
// app.js

const Koa = require('koa'),
  app = new Koa()

const Router = require('koa-router'),
  router = new Router()

// 模板引擎
const views = require('koa-views')
app.use(views('views'))

// request中间件
const bodyParser = require('koa-bodyparser')
app.use(bodyParser())

// 设置静态文件目录
const static = require('koa-static')
app.use(static(__dirname))


// jwt鉴权
app.use((ctx, next) => {
  return next().catch(err => {
    if (err.status === 401) {
      ctx.status = 200
      ctx.body = {
        status: 401,
        msg: '登录失效'
      }
    } else {
      return next()
    }
  })
})
const jwt = require('koa-jwt')
app.use(jwt({ secret: 'johnny_jwt_secret' }).unless({ path: [ '/', '/login', /^\/user\//, '/api/login'] }))


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


### 文档和攻略
1. 《JSON Web Token 入门教程》阮一峰：[www.ruanyifeng.com/](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
2. jsonwebtoken 文档：[github.com/auth0/node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken#readme)
3. koa-jwt 文档：[https://github.com/koajs/jwt](https://github.com/koajs/jwt#koa-jwt)


### 完结
🎊撒花，以上就是 `jwt` 的核心部分，可以到 `git` 仓库 `jwt` 分支查看全部代码