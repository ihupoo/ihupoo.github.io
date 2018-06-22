---
title: webpack4基本配置
tags: [webpack]
categories: webpack
---
## webpack4 特性
webpack4 通过一系列默认配置，将 webpack3 常用的 plugin 都默认引入了，相对简化了配置项。实际上，一般项目 webpack4 与 webpack3 在基本配置上差别并不是很大，主要有以下不同 ：
* webpack4 需要配合 webpack-cli 一起使用
* webpack4 增加了 `mode` 属性，设置为 `development` / `production`
<!--more-->
    * `development`
        1. process.env.NODE_ENV 的值设为 `development`
        2. 默认开启以下插件，充分利用了持久化缓存。参考[基于 webpack 的持久化缓存方案](https://github.com/pigcan/blog/issues/9)
            * `NamedChunksPlugin` ：以名称固化 chunk id
            * `NamedModulesPlugin` ：以名称固化 module id
    * `production`
        1. process.env.NODE_ENV 的值设为 `production`
        2. 默认开启以下插件，其中 `SideEffectsFlagPlugin` 和 `UglifyJsPlugin` 用于 `tree-shaking`
            * `FlagDependencyUsagePlugin` ：编译时标记依赖
            * `FlagIncludedChunksPlugin` ：标记子chunks，防子chunks多次加载
            * `ModuleConcatenationPlugin` ：作用域提升(scope hosting),预编译功能,提升或者预编译所有模块到一个闭包中，提升代码在浏览器中的执行速度。
            * `NoEmitOnErrorsPlugin` ：在输出阶段时，遇到编译错误跳过
            * `OccurrenceOrderPlugin` ：给经常使用的ids更短的值
            * `SideEffectsFlagPlugin` ：识别 package.json 或者 module.rules 的 sideEffects 标志（纯的 ES2015 模块)，安全地删除未用到的 export 导出。
            * `UglifyJsPlugin` ：删除未引用代码，并压缩
* webpack4 删除了 `CommonsChunkPlugin` 插件，改用 optimization 属性,重点是 `splitChunks` (自定义公用代码提取—vendor) 和 `runtimeChunk` (webpack运行代码提取—manifest)
* webpack4 增加了 WebAssembly 的支持，可以直接 import/export wasm 模块，也可以通过编写 loaders 直接 import C++/C/Rust

## 项目目录结构
一个项目一般有开发环境和生产环境，所以相应的项目基本结构大致如下 ： 
```
    ├─ config
    │  ├─ webpack.base.conf.js          //webpack基础配置
    │  ├─ webpack.dev.conf.js           //webpack开发配置
    │  └─ webpack.prod.conf.js          //webpack生产配置
    ├─ src
    │  ├─ css                           //css文件
    │  │   └─ common.css 
    │  ├─ js                            //js文件
    │  │   └─ common.js 
    │  ├─ index.html                    //html模板
    │  └─ index.js                      //入口js
    ├─ .babelrc                         //babel配置
    ├─ package.json                     //package.json
    └─ postcss.config.js                //postcss配置
```

## 代码示例
#### webpack.base.conf.js
首先是 webpack4 的基础配置 `webpack.base.conf.js`，集合了开发和生产环境的通用配置，结构如下 ：
```javascript
    module.exports = {
        entry: {},
        output: {},
        resolve: {},
        module: {},
        plugins: {},
        optimization: {}
    }
```
除去 `optimization` 其他的都是很熟悉的webpack3的配置，不一一介绍，示例代码如下 :
* `entry`
    ```javascript
        entry: {
            index: './src/index.js',
            // main: './src/main.js'   //多页面设置直接添加即可，同时plugins需要加上一个新的HtmlWebpackPlugin
        }
    ```
* `output`
    ```javascript
        output: {
            filename: '[name].js',                       //打包后名称
            path: path.resolve(__dirname, '../dist'),    //打包后路径
        }
    ```
* `resolve`
    ```javascript
        resolve: {
            mainFields: ['jsnext:main', 'browser', 'main'], //配合tree-shaking，优先使用es6模块化入口（import）
            extensions: ['.js', '.json', '.css'],           //可省后缀
            alias: {
                '@': path.resolve(__dirname, '../src')      //别名
            }
        }
    ```
* `module`
    ```javascript
        module: {
            noParse: /three\.js/, //这些库都是不依赖其它库的库 不需要解析他们可以加快编译速度
            rules: [{
                    test: /\.js$/,
                    use: 'babel-loader',
                    // include: /src/,                      //只转化src目录下js
                    exclude: /node_modules/                 //不转化node_modules目录下js
                },
                {
                    test: /\.css$/,
                    use: ['style-loader', 'css-loader', 'postcss-loader']
                },
                {
                    test: /\.scss$/,
                    use: ['style-loader', 'css-loader', 'postcss-loader', 'sass-loader']
                },
                {
                    test: /\.(html|htm)$/,
                    use: 'html-withimg-loader'              //html下的img路径
                },
                {
                    test: /\.(eot|ttf|woff|svg|woff2)$/,
                    use: 'file-loader'
                },
                {
                    test: /\.(jpe?g|png|gif)$/,
                    use: [{
                        loader: 'url-loader',
                        options: {
                            limit: 10000,
                            outputPath: 'images/',          //打包目录
                            name: '[name].[hash:7].[ext]'
                        }
                    }]
                }
            ]
        }
    ```
* `plugins`
    ```javascript
        plugins: [
            new HtmlWebpackPlugin({
                filename: 'index.html',                           //目标文件
                template: './src/index.html',                     //模板文件
                chunks: ['manifest', 'vendor', 'utils', 'index']  //对应关系，index.js对应的是index.html
            }),
            new webpack.ProvidePlugin({                           //自动加载模块，而不必到处 import 或 require
                'THREE': 'three'
            })
        ]
    ```
* `externals` 该属性同时需要在模板html 里插入 cdn 的 `script` (注 ：本例并没有使用 cdn 来引入 three.js ，这里仅是参考)
    ```javascript
        externals: {
            three:'THREE' //属性是three,即排除 import 'three' 中的 three 模块，'THREE'则用于检索一个全局 THREE 变量
        }
    ```
然后是 `optimization` ，替代了原来的 `CommonsChunkPlugin` 公共代码抽离 :
```javascript
        optimization: {
            splitChunks: {
                chunks: 'all',
                // maxAsyncRequests: 1,                     // 最大异步请求数， 默认1
                // maxInitialRequests: 1,                   // 最大初始化请求书，默认1
                cacheGroups: {
                    // 抽离第三方插件
                    vendor: {
                        test: /node_modules/,            //指定是node_modules下的第三方包
                        chunks: 'all',
                        name: 'vendor',                  //打包后的文件名，任意命名
                        priority: 10,                    //设置优先级，防止和自定义公共代码提取时被覆盖，不进行打包
                        
                    },
                    // 抽离自己写的公共代码，utils这个名字可以随意起
                    utils: {
                        chunks: 'all',
                        name: 'utils',
                        minSize: 0,                      //只要超出0字节就生成一个新包
                        minChunks: 2,                     //至少两个chucks用到
                        // maxAsyncRequests: 1,             // 最大异步请求数， 默认1
                        maxInitialRequests: 5,           // 最大初始化请求书，默认1
                    }
                }
            },
            //提取webpack运行时的代码
            runtimeChunk: {                              
                name: 'manifest'
            }
        }
```
同样 `manifest`, `vendor`, `utils` 都需要在 `HtmlWebpackPlugin` 的 `chunks` 里加上。






## 一些 webpack 优化
* `webpack-bundle-analyzer` 可视化定位体积大的模块
    ```javascript
        const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
        plugins: [
            new BundleAnalyzerPlugin() // 使用默认配置，启动127.0.0.1:8888
        ],
    ```

* `happypack` 开启多个子进程，加快 webpack 打包速度，webpack4 需要 `happypack@next` 


## 更多
本代码示例 
找到一个很优秀的参考demo [webpack4-demo](https://github.com/carrot-wu/webpack4-demo)

