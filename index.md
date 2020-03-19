## 概述

## 写在前面
最近和团队大佬聊天的时候，发现大家都对eggjs很是推崇，但是作为一个有追求的程序员，很想知道这个很棒的企业级框架是怎么跑起来的，为什么会这样设计。

抱着这种追根究底的态度，准备挖一个很大的坑，开始分析egg的源码，希望有兴趣的同学一起加入我们，共同往下探索

### 面向读者

* 有兴趣向大前端转向的前端开发者
* 对eggjs有兴趣的nodejs开发者
* 想深入了解eggjs和koa实现原理的开发者

### 你最好知道
技术书籍，有一定的门槛，会尽量写得简单易懂一些，这里把需要的一些知识罗列一下，方便大家在有准备的前提下更好的阅读。

so，在阅读之前，你最好具备如下知识：

* JavaScript的基本知识 （必须）
* nodejs的基本知识 （了解）
* es6，7基础知识 （了解）

如果你不太了解以上知识，可能会在源码解析部分有些卡壳。不过不要紧，通过对知识点的单点突破，基本可以突破障碍，流畅阅读。

## 大纲

### koa2源码解析

- [概述](./docs/koa2/index.md)
- [application](./docs/koa2/application.md)
- context
- require
- response

### egg-core 解析
- Loader
- Middleware
- Plugin
- Config
- Extend
- Router
- Controller
- Service
- schedule
- httpclient
- Error

### egg-cluster模块解析

### egg-bin模块解析

### egg-script模块解析


### 常用插件解析
- egg-loger
- egg-logrotator
- egg-passport
- egg-view-assets
- egg-mysql
- egg-sequelize
- egg-view-nunjucks
- egg-redis
- .....
    
### 常用中间件解析
