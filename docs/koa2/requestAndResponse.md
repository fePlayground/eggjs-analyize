### request 和 response 的作用
根据官方api文档的说明，这两个对象其实只是对nodejs普通对象的一些抽象，同时增加了一些可以方便日常http开发的附加函数，实时上这两个js代码文件里面也是这么做的

### 实现方式
通篇看下来，```request.js``` 和 ```response.js``` 主要以实现了对象的get/set方法为主，所以看起来并不难理解，在官方的api文档里面，可以看到完整的api文档，
这里就不展开说了。

值得一提的是，在context的结尾，我们看到这样一段代码，request和response对象就是这样被**委托**到ctx对象上的
```javascript 1.8
/**
 * Response delegation.
 */

delegate(proto, 'response')
  .method('attachment')
  ......
  .access('status')
  ......

/**
 * Request delegation.
 */

delegate(proto, 'request')
  .method('acceptsLanguages')
  ......
  .access('querystring')
  ......

```

所以，总的来说，这两个文件没有太多的阅读难度，理解起来比较容易，当然，如果研究里面具体函数的实现，还是很有意思。
鉴于本文主要目的还是希望从结构的角度上去看整个koa，就不把各函数的实现一一罗列出来了，有兴趣的同学推荐直接看源码。

[下一篇: eggjs go](../egg-core/index.md)


