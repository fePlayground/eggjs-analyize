## koa application 分析
接上节,我们借助最简单的建立服务器来看一下 koa/lib/application 里面的实现,作为web应用核心的启动模块，我们按照使用场景进行分析

### new Koa() && listen
```javascript 1.8
const Koa = require('./lib/application');
// debugger here
const app = new Koa();
// 调用构造函数，构建了koa对象，其中包含几个重要对象
// this.middleware 中间件数组
// this.context = Object.create(context) 注意，这里的context使用的是 context.js 返回的对象
// this.request = Object.create(request) 注意，这里的request使用的是 request.js 返回的对象
// this.response = Object.create(response) 注意，这里的response使用的是 response.js 返回的对象

// application.use，只是简单的把 fn push 到 this.middleware里面，值得一提的，这里有一个 convert函数，用于兼容 Generator 的写法
// 这是一个核心方法，所有的fn都会被push到 this.middleware 数组里面，在返回前都会调用
app.use(async ctx => {
  ctx.body = 'Hello World';
});
// 执行完以后，可以看到 middleware.length = 1

// application.listen, 直接使用了http.createServer 然后将参数透传进去
// 这里要注意一下，这里传入的 this.callback 函数是实现洋葱圈的核心代码, 【详见】setion2
app.listen(8000);
```
参考资料：[http.createServer](http://nodejs.cn/api/http.html#http_http_createserver_options_requestlistener)

### callback 和 handleRequire
刚才说到，在我们使用 Koa.listen 启动一个web服务器的时候，```application.listen``` 调用了```this.callback```,
接下来，我们看看 this.callback 干了什么
* 这里会涉及到 ```koa-compose、on-finished```这些包
```javascript 1.8
// 从这里开始
const server = http.createServer(this.callback());

// 惰性函数，生成一个handleRequest的回调，传入 createServer 的回调函数
callback() {
    // compose 是引用 koa-compose，这个是中间件洋葱圈的核心，建议阅读理解一下
    // koa-compose 是middleware的调用核心，代码非常简单，本质是根据 ```this.middleware``` 依照array push顺序进行递归调用，这里包装了promise保证异步
    // fn是一个按照push进行middleware注册函数的调用链，每次 dispatch 函数执行时通过 next 来实现俄罗斯套娃，所以这里注册 middleware 的顺序很重要
    const fn = compose(this.middleware);

    // 错误事件处理
    if (!this.listenerCount('error')) this.on('error', this.onerror);

    // 实际 request 处理函数
    const handleRequest = (req, res) => {
      // 每次都根据req和res构建一个新的上下文对象 
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }
  
  // 把上下文对象和 compose 生成的函数传入，此函数是真的
  handleRequest(ctx, fnMiddleware) {
      const res = ctx.res;
      // 先搞成 404 后面响应的时候会重置
      res.statusCode = 404;
      const onerror = err => ctx.onerror(err);
      // 独立的响应函数，【详见 section】
      const handleResponse = () => respond(ctx);
      // 这里用到了 on-finished 包，用来处理错误回掉
      onFinished(res, onerror);
      // 一堆中间件玩玩了以后，才会开始处理 respond
      return fnMiddleware(ctx).then(handleResponse).catch(onerror);
    }
    
    // 每个请求都会构建新的上下文对象
    createContext(req, res) {
        const context = Object.create(this.context);
        const request = context.request = Object.create(this.request);
        const response = context.response = Object.create(this.response);
        // 整个 app 实例在这里
        context.app = request.app = response.app = this;
        context.req = request.req = response.req = req;
        context.res = request.res = response.res = res;
        // 这里就是我们使用的最多的 ctx
        request.ctx = response.ctx = context;
        request.response = response;
        response.request = request;
        context.originalUrl = request.originalUrl = req.url;
        context.state = {};
        return context;
      }
```
至此，在启动webserver的时候，中间件的注册，和调用链已经构筑完成了，接下来一起试试请求一把

### request && response
以上，利用koa.application.listen 启动了web server,接下来看看koa.application是怎么处理请求的
先在 ```application.request``` 第一行打个断点，然后浏览器打开 服务器地址（127.0.0.1：8000）
前面省略_http_server.js中的 ```parserOnIncoming``` 进入的，下面来看看```handleRequest```

```javascript 1.8
// 主要就是这里，section 已经说明，这里就不再赘述了
return fnMiddleware(ctx).then(handleResponse).catch(onerror);

// response 函数，其实就是根据不同的返回类型，来进行处理返回
function respond(ctx) {
  // koa可以传入，如果是false就不搞
  if (false === ctx.respond) return;

  if (!ctx.writable) return;

  const res = ctx.res;
  let body = ctx.body;
  const code = ctx.status;

  ......

  // 各种情况设置 head，body
  if ('HEAD' === ctx.method) {
    if (!res.headersSent && !ctx.response.has('Content-Length')) {
      const { length } = ctx.response;
      if (Number.isInteger(length)) ctx.length = length;
    }
    return res.end();
  }

  // status body
  if (null == body) {
    if (ctx.req.httpVersionMajor >= 2) {
      body = String(code);
    } else {
      body = ctx.message || String(code);
    }
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res);

  // body: json
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}

```

### 总结
application.js 是koa的核心文件之一，主要负责 web server 的启动和事件响应，而各类增加的中间件，会在 ```response``` 之前通过递归和promise串联起来

[next: 洋葱圈的实现](./compose.md)

