### 洋葱圈模型的实现
[上篇](./application.md)讲到洋葱圈模型，感觉这里还是要展开讲一下，有一个官方的洋葱圈模型gif图很形象

![middleware](https://raw.githubusercontent.com/koajs/koa/a7b6ed0529a58112bac4171e4729b8760a34ab8b/docs/middleware.gif)

上篇中有解释，其实中间件注册就是通过 use 函数 push一个到 ```app.middleWare``` 数组里面，在request的时候先获得执行，下面结合gif中的例子来讲一下
下面介绍一下实现精巧的 compose

```javascript 1.8
  // compose 核心代码
  return function (context, next) {
    // last called middleware #
    let index = -1
    // 从数组第一个启动
    return dispatch(0)
    // 核心函数，用于递归
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        // 绑起来执行
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
```

### gif解析
* step1：按顺序注到```this.middleWare里面```
```javascript 1.8 
this.middleWare = [responseTime, logger, contentLength, body]
```

* step2: application.callback 调用 compose 解开函数，形成递归, 形成的伪代码如下

responseTime(ctx).then(logger(ctx).then(contentLength(ctx).then(body(ctx))))

对的，很像俄罗斯套娃，写成这样，会不会好理解一点？
因为 Generator 的特性，每次碰到 yield 就醒后执行到里面，形成了实例里面的套娃洋葱圈效果

