---
title: webpack4常用配置
tags: [webpack]
categories: webpack
---
## webpack4 特性
webpack4 通过一系列默认配置，将 webpack3 常用的 plugin 都默认引入了，相对简化了配置项。实际上，一般项目 webpack4 与 webpack3 在基本配置上差别并不是很大，主要有以下不同 ：
* webpack4 需要配合 webpack-cli 一起使用

<!--more-->
* webpack4 增加了 `mode` 属性，设置为 development / production
    * development
        1. process.env.NODE_ENV 的值设为 development

        2. 默认开启以下插件，充分利用了持久化缓存。参考[基于 webpack 的持久化缓存方案](https://github.com/pigcan/blog/issues/9)
            * `NamedChunksPlugin` ：以名称固化 chunk id

            * `NamedModulesPlugin` ：以名称固化 module id
    * production
        1. process.env.NODE_ENV 的值设为 production

        2. 默认开启以下插件，其中 `SideEffectsFlagPlugin` 和 `UglifyJsPlugin` 用于 tree-shaking
            * `FlagDependencyUsagePlugin` ：编译时标记依赖

            * `FlagIncludedChunksPlugin` ：标记子chunks，防子chunks多次加载

            * `ModuleConcatenationPlugin` ：作用域提升(scope hosting),预编译功能,提升或者预编译所有模块到一个闭包中，提升代码在浏览器中的执行速度

            * `NoEmitOnErrorsPlugin` ：在输出阶段时，遇到编译错误跳过

            * `OccurrenceOrderPlugin` ：给经常使用的ids更短的值

            * `SideEffectsFlagPlugin` ：识别 package.json 或者 module.rules 的 sideEffects 标志（纯的 ES2015 模块)，安全地删除未用到的 export 导出

            * `UglifyJsPlugin` ：删除未引用代码，并压缩

* webpack4 删除了 `CommonsChunkPlugin` 插件，改用 optimization 属性，重点是 `splitChunks` (自定义公用代码提取—vendor) 和 `runtimeChunk` (webpack运行代码提取—manifest)
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
* entry
    ```javascript
        entry: {
            index: './src/index.js',
            // main: './src/main.js'   //多页面设置直接添加即可，同时plugins需要加上一个新的HtmlWebpackPlugin
        }
    ```
* output
    ```javascript
        output: {
            filename: '[name].js',                       //打包后名称
            path: path.resolve(__dirname, '../dist'),    //打包后路径
        }
    ```
* resolve
    ```javascript
        resolve: {
            mainFields: ['jsnext:main', 'browser', 'main'], //配合tree-shaking，优先使用es6模块化入口（import）
            extensions: ['.js', '.json', '.css'],           //可省后缀
            alias: {
                '@': path.resolve(__dirname, '../src')      //别名
            }
        }
    ```
* module
    ```javascript
        module: {
            noParse: /three\.js/, //这些库都是不依赖其它库的库 不需要解析他们可以加快编译速度
            rules: [{
                    test: /\.js$/,
                    use: 'babel-loader?cacheDirectory=true',//babel-loader的cacheDirectory表示缓存转换结果，提高webpack下次编译效率
                    // include: /src/,                      //只转化src目录下js
                    exclude: /node_modules/                 //不转化node_modules目录下js
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
* plugins
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
* externals 该属性同时需要在模板html 里插入 cdn 的 `script` (注 ：本例并没有使用 cdn 来引入 three.js ，这里仅是参考)
    ```javascript
        externals: {
            three:'THREE' //属性是three,即排除 import 'three' 中的 three 模块，'THREE'则用于检索一个全局 THREE 变量
        }
    ```

然后是 `optimization` ，替代了原来的 `CommonsChunkPlugin` 公共代码抽离 :
```javascript
        optimization: {
            splitChunks: {
                chunks: 'all',                              //'all'|'async'|'initial'(全部|按需加载|初始加载)的chunks
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
同样 `manifest, vendor, utils` 都需要在 `HtmlWebpackPlugin` 的 `chunks` 里加上。

#### webpack.dev.conf.js
开发环境下，通过 `webpack-merge` 添加上需要的更多开发配置，示例如下：
```javascript
    const webpack = require('webpack');
    const merge = require('webpack-merge');
    const path = require("path");
    const base = require('./webpack.base.conf');
    // const FriendlyErrorsWebpackPlugin = require("friendly-errors-webpack-plugin"); //更好的错误输出

    module.exports = merge(base, {
        module: {
            rules: [{
                    test: /\.css$/,
                    use: ['style-loader', 'css-loader', 'postcss-loader']
                },
                {
                    test: /\.scss$/,
                    use: ['style-loader', 'css-loader', 'postcss-loader', 'sass-loader']
                }
            ]
        },
        plugins: [
            new webpack.HotModuleReplacementPlugin(),           //热更新，还需在index.js里配置
            // new FriendlyErrorsWebpackPlugin()                //优化 webpack 输出信息

        ],

        devtool: '#source-map',                                 //方便断点调试
        // devtool: '#cheap-module-eval-source-map',            //构建速度快，采用eval执行

        devServer: {
            contentBase: path.resolve(__dirname, '../dist'),    //服务路径，存在于缓存中
            host: 'localhost',                                  // 默认是localhost
            port: 8080,                                         // 端口
            open: true,                                         // 自动打开浏览器
            hot: true,                                          // 开启热更新，只监听js文件，所以css假如被抽取后，就监听不到了
            // inline: true,                                    //inline模式开启服务器(默认开启)
            // proxy: xxx                                       //接口代理配置
            // quiet: true                                      //和friendly-errors-webpack-plugin配合,但webpack自身的错误或警告在控制台不可见。
            clientLogLevel: "none",                             //阻止打印那种搞乱七八糟的控制台信息
        },
        mode: 'development'                                     //开发环境

    })
```
假如启用 `css-modules` 需要额外在 css-loader 的 options 里配置，使用方法可参考[①](https://webpack.docschina.org/loaders/css-loader/#modules)，[②](https://www.cnblogs.com/diligentYe/p/6602010.html)
```javascript
   {
       loader:"css-loader",
       options:{
           modules: true,                                          //使用css-modules
           minimize: true,                                         //压缩css 
           importLoaders: 1,
           localIdentName: "[name]__[local]__[hash:base64:5]"      //指定生成的名称
       }
   }
```

#### webpack.prod.conf.js
生产环境下，同样需要的类似于hash，等配置，一种示例如下：
```javascript
    const webpack = require('webpack');
    const base = require('./webpack.base.conf');
    const merge = require('webpack-merge');
    const path = require("path");
    const CleanWebpackPlugin = require('clean-webpack-plugin');  //每次都清空 dist 文件夹

    // 1. webpack-bundle-analyzer 可视化定位体积大的模块
    // const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

    // 2. happypack 开启多个子进程，加快 webpack 构建/打包速度。由于其对`file-loader`, `url-loader` 支持的不友好，不建议对这两 loader 使用（本例只是示例对css使用，实际可以加上js的构建，代码类似）
    const Happypack = require('happypack'); 
    const os = require('os');
    const happyThreadPool = Happypack.ThreadPool({ size: os.cpus().length }); //cpu 核数

    // 3. extract-text-webpack-plugin 拆分css，会把css文件放到dist目录下的css/[name].[md5:contenthash:hex:20].css，以link的方式引入css
    const ExtractTextWebpackPlugin = require('extract-text-webpack-plugin');
    let styleCss = new ExtractTextWebpackPlugin({
        filename: 'css/[name].[md5:contenthash:hex:20].css',
        allChunks: true
    });

    module.exports = merge(base, {
        output: {
            filename: '[name].[chunkhash].js',                  //chunkhash:根据自身的内容计算而来
        },
        module: {
            rules: [{
                test: /\.(css|scss|sass)$/,
                use: styleCss.extract({
                    fallback: 'style-loader',                   // 样式没有被抽取时，使用style-loader 
                    use: 'happypack/loader?id=css',             // 将css用link的方式引入就不再需要style-loader了，loader采用happypack
                    publicPath: '../'                           //与url-loader里的outputPath对应，这样可以根据相对路径引用图片资源
                })
            }]
        },
        plugins: [
            styleCss,
            new CleanWebpackPlugin('dist', {
                root: path.resolve(__dirname, '../'),
                verbose: true
            }),
            new Happypack({
                id: "css",                                      //id与module.rules里loader里的id一致
                loaders: [                                      //相当于module.rules里loader
                    { loader: 'css-loader', options: { importLoaders: 1, minimize: true } },
                    'postcss-loader',
                    'sass-loader'
                ],
                threadPool: happyThreadPool,
                verbose: true
            }),
            new webpack.HashedModuleIdsPlugin(),                //固化module id

            // new BundleAnalyzerPlugin()                       // 使用默认配置，启动127.0.0.1:8888
        ],
        mode: 'production'
    })
```
当我以为这样就写完了的时候，坑爹的事情来了。可能你已经发现，为何 extract-text-webpack-plugin 使用了 `[md5:contenthash:hex:20]` 而不是 [contenthash]？

原因就是 extract-text-webpack-plugin 即将弃用，bata 版目前只能在 Webpack 4.2.0 以下可用，这也导致了在最新版 webpack 中，假如使用 [contenthash] ，则会报错：
`Error: Path variable [contenthash] not implemented in this context: css/[name].[contenthash].css`

一种过渡方案就是使用 `[md5:contenthash:hex:20]`，另外一种就是使用官方推荐的 `mini-css-extract-plugin`。好了，那 mini-css-extract-plugin 该怎么改写呢？
```javascript
   const MiniCssExtractPlugin = require("mini-css-extract-plugin");
   //...
   module: {
        rules: [{
            test: /\.(css|scss|sass)$/,
            use: [{
                loader: MiniCssExtractPlugin.loader,
                options: {
                    publicPath: '../'                   //同extract-text-webpack-plugin一样,与url-loader里的outputPath对应
                }
            }, {
                loader: 'happypack/loader?id=css'
            }]
        }]
    },
    //...
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].[contenthash].css',
            chunkFilename: 'css/[name].[contenthash].css',
        })
    ],
```
更多配置可以参考[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)，通过配合 `optimization`，可以实现自定义的css压缩；多个 css chunk 合并。目前依然有坑，比如可能会多加 61bytes 的js（我好像没遇到...不过貌似是 webpack 的问题，即将解决了），可以关注其 [issues](https://github.com/webpack-contrib/mini-css-extract-plugin/issues) 。

webpack5 将对 css 的处理直接集成，期待能够解决 css 的痛点。

## 其他
`optimization` 抽离代码非常有用，其中匹配用的 `test` 属性除了正则，还可以用 function ,参数就是每个 module，实际使用还得查 webpack 的 [test](https://github.com/webpack/webpack/tree/master/test) 才能知道这些内置的方法，下面的注释仅是从打印内容和单词含义猜测的，网上没有找到具体解释。
```javascript
    test: module => module.nameForCondition &&
        /\.css$/.test(module.nameForCondition()) &&       //module.nameForCondition() 得到的应该是module的路径
        !/^javascript/.test(module.type)                  //module.type 应该就是实际类型，会有 javascript/auto 还有 mini-css-extract-plugin 注入的


    //如果 module 在 a 或者 b chunk 被引入，并且 module 的路径包含 node\_modules ，那这个 module 就应该被打包到这个 vendor 中
    test: module => {
      for (const chunk of module.chunksIterable) {        //所有chunks的迭代
            if (chunk.name && /(a|b)/.test(chunk.name)) { //chunk的名称 
                if (module.nameForCondition() && /[\\/]node_modules[\\/]/.test(module.nameForCondition())) {
                 return true;
             }
            }
       }
      return false;
    }
```

## 代码
本代码示例 [threejs-water](https://github.com/ihupoo/threejs-water)

## 参考
1. [webpack4-demo](https://github.com/carrot-wu/webpack4-demo)
2. [webpack4-用之初体验，一起敲它十一遍](https://juejin.im/post/5adea0106fb9a07a9d6ff6de)
3. [Webpack(含 4)配置详解——关注细节](https://juejin.im/post/5ae925cff265da0ba76f89b7#heading-13)

