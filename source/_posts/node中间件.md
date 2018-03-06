---
title: node常用库
tags: [node,express,koa]
categories: node
---

## node库
* [superagent](https://visionmedia.github.io/superagent/) : http库，可以发起get，post等请求，常用于爬虫或者代理。
* [cheerio](https://github.com/cheeriojs/cheerio) : 网页处理库，node.js版的jquery，可用类似api实现网页中css selector取数据，常用于爬虫。
<!--more-->
* [eventProxy](https://github.com/JacksonTian/eventproxy) : 事件发布订阅工具，个人感觉有点像轻量级的rxjs，可用于解决回调金字塔（当然async也可以）。
* [supertest](https://github.com/tj/supertest) ： 测试库，同样是tj大神的，用来配合 express（准确来说是所有兼容connect 的web框架）进行集成测试。
* [mongoose](http://mongoosejs.com/docs/guide.html) : mongodb的odm（Object-Document Mapping ，对象文档映射），用于操作mongodb。
* [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) : 反向代理（webpack也采用这个）

## express库
* [cookie-parser](https://github.com/expressjs/cookie-parser) : express 4.x 操作cookie。
* [express-session](https://github.com/expressjs/session) : 操作session。
* [connect-redis](https://github.com/tj/connect-redis) :  操作session时设置redis缓存，来实现进程间共享session。
* [connect-mongo](https://www.npmjs.com/package/connect-mongo) :  操作session时,将 session 存储于 mongodb，需结合 express-session 使用。
* [connect-flash](https://www.npmjs.com/package/connect-flash) : 基于session 实现的用于通知功能的中间件，需结合 express-session 使用。
