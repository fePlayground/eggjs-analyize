### context的作用
context 主要是为了koa的application创建上下文对象使用，其中主要实现了一些工具方法，比如```toJSON```和对 [request](./request.md)，[response](./response.md)
对象的代理。

### context分析
context的作用是为了构造请求的运行是上下文，代码很简洁，主要由两个部分组成，proto 对象构造 和 代理构造

第一部分
```javascript 1.8
// proto对象构造
const proto = module.exports = {
  // 首先是一系列方法的声明，这里择要分析
  // 实现toJSON函数，返回一个比较简洁的对象
  toJSON() {
    return {
      request: this.request.toJSON(),
      response: this.response.toJSON(),
      app: this.app.toJSON(),
      originalUrl: this.originalUrl,
      req: '<original node req>',
      res: '<original node res>',
      socket: '<original node socket>'
    };
  },

  // 默认的错误处理函数，会对app抛出的错误做一些处理，同时设置status值之类的操作
  onerror(err) {...},

  // 这里构造了cookies的set/get方法
  get cookies() {
    ...
  },

  set cookies(_cookies) {
    ...
  }
};
```

第二部分，delegate对象，这里以response对象的代理为例子,具体delegate的实现也很简单，可以参看TJ大神作品[delegate](https://github.com/tj/node-delegates)
deleagate方法这里的主要作用就是把部分方法和属性外化到context上，方便使用

```javascript 1.8
delegate(proto, 'response')
  .method('attachment') 
  .access('status')
  .getter('writable');
```


### context在application中的使用
[application](./application.md)的解析中有提到context的使用入口，主要在两个地方

```javascript 1.8
// application.constructor
// 这里创造了一个空的context对象，console可以看出，这个对象里面没有任何东西，只有prototype上挂了一堆东西
constructor(options) {
    ...
    this.context = Object.create(context);
    ...
  }

// application.createContext
// 当每一个请求调用 handleRequest之前都会调用这个函数对ctx对象重新赋值
// 为了方便使用 req 和 res 的部分常用属性和方法 在context.js中，使用了delegate对 resonse 和 request 对象进行代理
  createContext(req, res) {
    // 这里创造了一个空对象, 可以断点一下，此时结果如下
    const context = Object.create(this.context);
    ...
  }
```

![执行到此的结果](/assets/koa-context1.png)
```javascript 1.8
createContext(req, res) {
    ...
    // delegate 及原始对象绑定
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    context.app = request.app = response.app = this;
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.ctx = response.ctx = context;
    request.response = response;
    response.request = request;
    context.originalUrl = request.originalUrl = req.url;
    context.state = {};
    // 执行到此的结果如下
    return context;
  }
```
![执行到此的结果](/assets/koa-context2.png)

如果对 delegate 不是很了解的，可以参看一下源码，下面是对```.access('status')```代理结果的展示
![执行到此的结果](/assets/koa-context3.png)

所以，context的使用基本如上，本质上只是做了一些对req和res对象的部分属性和方法的代理，方便使用而已

[下一篇：request](./requestAndResponse.md)
