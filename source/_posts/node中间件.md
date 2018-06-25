---
title: node常用库
tags: [node,express,koa]
categories: node
---

## node库
* 通用
    * [pm2](http://pm2.keymetrics.io) : node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等。
* 调试/测试
    * [nodemon](https://github.com/remy/nodemon) : node调试工具，自动重启服务，开发时候很好用。
    * [supertest](https://github.com/tj/supertest) ： 测试库，用来配合 express（准确来说是所有兼容connect的web框架）进行集成测试，属于http请求测试库。其他测试有mocha（框架测试），chai（测试结果断言库）
    
<!--more-->
* 代理
    * [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) : 反向代理（webpack也采用这个）。
    * [whistle](https://github.com/avwo/whistle) : 反向代理，据说很好用。
* 数据库
    * [mongoose](http://mongoosejs.com/docs/guide.html) : mongodb的odm（Object-Document Mapping ，对象文档映射），用于操作mongodb。
    * [mysql](https://github.com/mysqljs/mysql) : node操作MySQL的引擎，可以在node.js环境下对MySQL数据库进行建表，增、删、改、查等操作。
* 爬虫
    * [superagent](https://visionmedia.github.io/superagent/) : http库，可以发起get，post等请求，常用于爬虫或者代理。
    * [cheerio](https://github.com/cheeriojs/cheerio) : 网页处理库，node.js版的jquery，可用类似api实现网页中css selector取数据，常用于爬虫。
* 其他工具
    * [decimal.js](https://github.com/MikeMcl/decimal.js) : 浮点计算
    * [eventProxy](https://github.com/JacksonTian/eventproxy) : 事件发布订阅工具，个人感觉有点像轻量级的rxjs，可用于解决回调金字塔（当然async也可以）。
    * [node-schedule](https://github.com/node-schedule/node-schedule) : 定时器/job任务，适合复杂的时间调度，例如每周一每小时的20分钟和50分钟做something，普通的setInterval适合每60秒执行什么任务，较为简单容易。

## express库
* cookie
    * [cookie-parser](https://github.com/expressjs/cookie-parser) : express 4.x 操作cookie。
* session
    * [express-session](https://github.com/expressjs/session) : 操作session。
    * [connect-redis](https://github.com/tj/connect-redis) :  操作session时设置redis缓存，来实现进程间共享session。
    * [connect-mongo](https://www.npmjs.com/package/connect-mongo) :  操作session时,将 session 存储于 mongodb，需结合 express-session 使用。
    * [connect-flash](https://www.npmjs.com/package/connect-flash) : 基于session 实现的用于通知功能的中间件，需结合 express-session 使用。

## koa2库
* 路由
    * [koa-router](https://github.com/alexmingoia/koa-router) : 路由中间件，使用后不管html文件具体在什么位置，跳转html使用路由路径。
* 迁移工具
    * [koa-convert](https://github.com/koajs/convert) : 对generator中间件的转换，例如 app.use(convert(json()))等等。
* 日志
    * [koa-logger](https://github.com/koajs/logger) : koa开发时替换console.log输出。
    * [log4js-node](https://github.com/log4js-node/log4js-node) : 需要按照时间或者按照文件大小，本地输出log文件，可以使用邮件等形式发送日志。
* 请求
    * [koa-static](https://github.com/koajs/static) : 静态资源中间件,和koa-router一起用，假如请求路径重名，优先级高。
    * [koa-bodyparser](https://github.com/koajs/bodyparser) : 把koa2上下文的formData数据解析到ctx.request.body。xml数据可以用koa-xml-body。
    * [busboy](https://github.com/mscdex/busboy) : 解析POST请求，node原生req中的文件流，用于上传文件。
    * [koa-jsonp](https://github.com/queckezz/koa-views) : 直接处理jsonp请求。
* session
    * [koa-session-minimal](https://github.com/longztian/koa-session-minimal)/[koa-session](https://github.com/koajs/session) : session中间件，提供存储介质的读写接口。
    * [koa-mysql-session](https://github.com/tb01923/koa-mysql-session) : koa-session-minimal中间件提供MySQL数据库的session数据读写操作。
* 模板引擎
    * [koa-views](https://github.com/queckezz/koa-views) : 模板引擎，可搭配pug等使用。
* 错误获取
    * [koa-onerror](https://github.com/koajs/onerror) : 错误信息，hack了ctx.onerror，能够在一个地方处理所有的错误。
* token
    * [koa-jwt](https://github.com/haorui/koa-jwt) : koa中间件，验证json web tokens模块。
    * [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) : 生成token，可以和koa-jwt配合使用。
* 压缩
    * [koa-compress](https://github.com/koajs/compress) : 压缩工具，可以采用Gzip等压缩。
