### koa 是什么
Koa 是一个 nodejs 的web应用框架，由 Express 原班人马打造，致力于成为 web 应用和 API 开发领域中的一个更小、更富有表现力、更健壮的基石。 通过利用 async 函数，Koa 帮你丢弃回调函数，并有力地增强错误处理。 Koa 并没有捆绑任何中间件， 而是提供了一套优雅的方法，帮助大家快速搭建web应用。

### koa 的作者
作为一个开源框架，koa有一堆的贡献者，但是社区神人 TJ 大神不能不提，可以感受下大神的力量[tj](https://github.com/tj)。
他是多个项目的核心开发者，随便一个项目都有雨成千上万的star。
鉴于他奇高的效率和代码提交质量，甚至有人怀疑他是某大厂的团队伪装的账号。

自带光环的大神已经成为开源社区的一盏明灯，自从2014年他宣布从nodejs转向go之后，go语言迅速崛起（到底是谁成就了谁）。

### koa 和 express
express 是 nodejs 的第一代 web 应用框架，express 提供了一个大而全的web框架，提供了一个传统web框架应该提供的功能，但是有碍于当时的javascript语言发展，当时express还是通过callback等方式来处理回掉函数
koa 是由express团队的核心成员基于es5打造的，非常轻量的框架，整个koa非常小，核心只包含了4个文件。代码里使用 Generator 方式，代替了 callback，避免了 callback hell 的问题。
koa2 是 koa 的进化版本，和koa的差异是是使用es6的 async / await 代替了原来的 Generator 方式。
总的来看，koa 是一个很轻量web框架，更像一个中间件集成处理器，但是给我们提供的是非常简单的核心功能和非常强大的扩展能力。

[koa的主要特性](https://eggjs.org/zh-cn/intro/egg-and-koa.html#koa)

### 洋葱圈模型
对于一个web框架来说，怎么处理web请求，是最重要的一件事情。不知道是不是大神们切洋葱的时候边流泪边灵光一现。
洋葱圈模型诞生了，利用这个模型，可以对koa的中间件思想一目了然。

[洋葱圈模型](https://eggjs.org/zh-cn/intro/egg-and-koa.html#middleware)

### 看源码的方法
之前听过一位社区大牛的课程，介绍了一些看源码的方法，深以为然，本系列也用这种方式来看源码。
其实本质是一种调试的方法，然后根据调用栈摸清楚代码的来龙去脉

以对koa的代码研究为例，列举步骤如下：
step0：准备nodejs环境（>8）,chrome(>67,最好是最新版本的)

step1：clone koa 源码，在根目录下新建一个 ```x.js``` 文件

step2：把需要运行的例子copy到x.js中，比如
```javascript 1.8
const Koa = require('./lib/application');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(8000);
```

step3：运行调试 ```node --inspect-brk x.js```,如果输出如下，说明成功启动
![inpesct-success](/assets/koa-index-1.png)

step4: chrome地址栏输入：```chrome://inspect/``` 打开 Romate Target 下的 inspect，就可以进行调试了

step5：用step forward 一步一步开始愉快的debug~~

[next:开始](./application.md)

### 源码版本
当前分析的koa源码版本为 2.11.0，如后续有少许变更，请对号入坐~~
