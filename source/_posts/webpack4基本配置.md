---
title: webpack4基本配置
tags: [webpack]
categories: webpack
---
## webpack4 特性
webpack4 与 webpack3 在基本配置上差别并不是很大，主要有以下不同 ：
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
        2. 默认开启以下插件，
            * `FlagDependencyUsagePlugin` ：编译时标记依赖
            * `FlagIncludedChunksPlugin` ：标记子chunks，防子chunks多次加载
            * `ModuleConcatenationPlugin` ：作用域提升(scope hosting),预编译功能,提升或者预编译所有模块到一个闭包中，提升代码在浏览器中的执行速度。
            * `NoEmitOnErrorsPlugin` ：在输出阶段时，遇到编译错误跳过
            * `OccurrenceOrderPlugin` ：给经常使用的ids更短的值
            * `SideEffectsFlagPlugin` ：识别 package.json 或者 module.rules 的 sideEffects 标志，安全地删除未用到的 export 导出。
            * `UglifyJsPlugin` ：压缩 js 代码
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


