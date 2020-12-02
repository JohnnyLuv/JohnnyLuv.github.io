---
title: webpack4.x多页面应用(MPA)配置案例
date: 2019-04-25 11:32:05
categories: JavaScript
tags: [webpack, JavaScript]
---
踩坑无数后，记录下`webpack4.x`多页面应用配置的最(mian'qiang)佳(neng'yong)实践。
***write less do more，***让我们开始吧！

<!-- more -->
项目地址： ***[https://github.com/JohnnyLuv/webpack4.x-MPA-demo](https://github.com/JohnnyLuv/webpack4.x-MPA-demo)***


### SPA && MPA 
> **单页面应用（SinglePage Web Application，SPA）**
> 只有一张Web页面的应用，是一种从Web服务器加载的富客户端，单页面跳转仅刷新局部资源 ，公共资源(js、css等)仅需加载一次，常用于PC端官网、购物等网站
>
> **多页面应用（MultiPage Application，MPA）**
> 多页面跳转刷新所有资源，每个公共资源(js、css等)需选择性重新加载，常用于 app 或 客户端等
>
> ***[——《你要懂的单页面应用和多页面应用》](https://juejin.im/post/5a0ea4ec6fb9a0450407725c)***


### Project Structure
+ 引用关系参照 ***[git@github](https://github.com/JohnnyLuv/webpack4.x-MPA-demo)***

```js
project
├── dist
├── node_modules
├── src
│   ├── css
│   │   ├── home.styl
│   │   └── modules.styl
│   ├── img
│   │   └── avatar.jpeg
│   ├── js
│   │   ├── home.js
│   │   ├── index.js
│   │   └── utils.js
│   ├── home.html
│   └── index.html
├── package.json
├── postcss.config.js
└── webpack.config.js
```


### Code View

#### `package.json`
+ 这里需要注意的是引用要不要加`@`，不同版本可能需要的配置项也不同，
  具体以 ***[npmjs.com](https://www.npmjs.com/)*** 或官方文档为准
  
```json
{
  "name": "pages-obj",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "build": "webpack --mode production",
    "dev": "webpack --mode development",
    "serve": "webpack-dev-server --progress --host 0.0.0.0"
  },
  "devDependencies": {
    "@babel/core": "^7.5.5",
    "@babel/preset-env": "^7.5.5",
    "autoprefixer": "^9.6.1",
    "babel-loader": "^8.0.6",
    "clean-webpack-plugin": "^3.0.0",
    "copy-webpack-plugin": "^5.0.3",
    "css-loader": "^3.1.0",
    "file-loader": "^4.1.0",
    "html-webpack-plugin": "^3.2.0",
    "html-withimg-loader": "^0.1.16",
    "mini-css-extract-plugin": "^0.8.0",
    "optimize-css-assets-webpack-plugin": "^5.0.3",
    "postcss-loader": "^3.0.0",
    "style-loader": "^0.23.1",
    "stylus": "^0.54.5",
    "stylus-loader": "^3.0.2",
    "uglifyjs-webpack-plugin": "^2.1.3",
    "url-loader": "^2.1.0",
    "webpack": "^4.36.1",
    "webpack-cli": "^3.3.6",
    "webpack-dev-server": "^3.7.2"
  }
}
```

#### `webpack.config.js`
+ 值得一提的是，如果打包前的项目结构与打包后的结构不同，则静态资源可能引用失败，
  比如`.html`中的文件路径，再比如`.css`中的背景图路径，诸如此类
+ 解决方法：在抽离页面`style`的插件(`mini-css-extract-plugin`)中添加配置项，
  为`test:/\.css$/`配置`loader: MiniCssExtractPlugin.loader`的`options`，
  配置项为`publicPath:'../'`

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin') // 模版生成html
const { CleanWebpackPlugin } = require('clean-webpack-plugin') // 每次打包前进行清理
// const CopyWebpackPlugin = require('copy-webpack-plugin') // 拷贝静态资源
const MiniCssExtractPlugin = require('mini-css-extract-plugin') // 把css从页面中抽离成单独的文件
const OptimizeCss = require('optimize-css-assets-webpack-plugin') // 压缩css
const UglifyJsPlugin = require('uglifyjs-webpack-plugin') // 压缩js

module.exports = {
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 8080,
  },
  devtool: 'source-map', // 源码映射，开发环境调试使用
  mode: 'development', // production / development
  optimization: {
    minimizer: [
      new OptimizeCss(), // 压缩css，仅在 'production' 模式下生效
      new UglifyJsPlugin({
        cache: true,
        parallel: true,
        sourceMap: true,
      })
    ]
  },
  entry: { // 多入口
    index: './src/js/index.js',
    home: './src/js/home.js'
  },
  output: { // 多出口
    // [name] => index, home
    filename: './js/[name].js',
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [
    new CleanWebpackPlugin(), // 打包前清理
    /* new CopyWebpackPlugin([ // 拷贝静态文件
      {
        from: path.resolve(__dirname, 'src/img'),
        to: './img'
      }
    ]), */
    new MiniCssExtractPlugin({ // 抽离css
      // filename: './css/index.css',
      filename: './css/[name].css',
      chunkFilename: './css/[id].css',
    }),

    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: './index.html',
      title: 'index page',
      chunks: ['index'],
      minify: {
        collapseWhitespace: true,
        removeComments: true,
        removeRedundantAttributes: true,
        removeScriptTypeAttributes: true,
        removeStyleLinkTypeAttributes: true,
        useShortDoctype: true
      }
    }),
    new HtmlWebpackPlugin({
      template: './src/home.html',
      filename: './home.html',
      chunks: ['home'],
      minify: {
        collapseWhitespace: true,
        removeComments: true,
        removeRedundantAttributes: true,
        removeScriptTypeAttributes: true,
        removeStyleLinkTypeAttributes: true,
        useShortDoctype: true
      }
    })
  ],
  module: {
    rules: [
      {
        test: /\.html$/, // 打包 html 中的图片
        use: 'html-withimg-loader'
      },
      {
        test: /\.(png|jpg|jpeg|gif)$/, // 打包图片
        use: {
          loader: 'url-loader',
          options: {
            limit: 10000, // 把小于1000B的文件打成Base64的格式，写入JS
            name: 'img/[name]-[hash:6].[ext]',
            // publicPath: "../dist/img/",
            // outputPath: "img/",
          }
        }
      },
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-env' // es6 转 es5
            ]
          }
        }
      },
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader, // 抽离css
            options: {
              publicPath:'../'
            }
          },
          // 'style-loader',
          'css-loader',
          'postcss-loader'
        ]
      },
      {
        test: /\.styl$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader, // 抽离css
            options: {
              publicPath:'../'
            }
          },
          // 'style-loader',
          'css-loader',
          'postcss-loader',
          'stylus-loader'
        ]
      }
    ]
  },
}
```

#### `postcss.config.js`
+ postcss配置文件

```js
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
}
```


### Final
吐槽一下中文文档竟然和原文档差别真的 超级 无敌 <span style="font-size:30px;font-weight:bolder;">大</span>
<div style="background-image: -webkit-linear-gradient(left,#D81159, #E53A40 10%, #FFBC42 20%, #75D701 30%, #30A9DE 40%,#D81159 50%, #E53A40 60%, #FFBC42 70%, #75D701 80%, #30A9DE 90%,#D81159);-webkit-background-clip: text;color:transparent;" title="花里胡哨的一定行(ง •̀_•́)ง">ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ...ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ...ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ...ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ...ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ...ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ...ʕ •ᴥ•ʔ...ᶘ ᵒᴥᵒᶅ</div>
