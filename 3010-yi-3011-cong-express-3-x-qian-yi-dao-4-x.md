# 【译】从 Express 3.x 迁移到 4.x {#articleTitle}

Express 3.x 到 4.0 的迁移指南。你可能对这篇文章也有兴趣[4.x版中的新功能](https://github.com/visionmedia/express/wiki/New-features-in-4.x)。

更多的例子和完整的API文档，请参见[Express 4.x 的文档](http://expressjs.com/4x/api.html)。

# 概述 {#articleHeader0}

Express 4 不再依赖 Connect 。这意味着所有捆绑的中间件（除了`static`）都不再能从`express`模块中被调用。这些中间件都可以作为下面提及的模块进行调用。这一变化是为了让这些中间件在获取修复，更新和发布的同时不影响 express 的发布周期，反之亦然。

* bodyParser
  [body-parser](https://github.com/expressjs/body-parser)
* cookieParser
  [cookie-parser](https://github.com/expressjs/cookie-parser)
* favicon
  [static-favicon](https://github.com/expressjs/favicon)
* session
  [express-session](https://github.com/expressjs/session)

其他模块在这里[https://github.com/senchalabs/connect\#middleware](https://github.com/senchalabs/connect#middleware)

# 移除 {#articleHeader1}

## app.configure\(\) {#articleHeader2}

这种方法不再可用。如果你想配置基于环境的不同路由，使用 if 语句或替代模块。

```
app.configure('development', function() {
   // configure stuff here
});
// 现在改为
var env = process.env.NODE_ENV || 'development';
if ('development' == env) {
   // configure stuff here
}
```

## app.router {#articleHeader3}

这个中间件已经全面改版，以此避免`.use`同`.get`之间的混淆（或者是其他HTTP动作），同时，不再需要手动声明的`app.use(app.router)`（已被移除）。查看下面关于新的中间件和路由 API 的一节。

如果你的代码看起来像这样：

```
app.use(cookieParser());
app.use(bodyParser());
/// 其他的中间件，并没有影响
app.use(app.router); // <--- 这行会被移除

// 更多的中间件（在路由之后执行）
app.use(function(req, res, next);
// 处理错误的中间件
app.use(function(err, req, res, next) {});

app.get('/' ...);
app.post(...);
```

`app.router`已被移除，中间件和路由按照它们添加的顺序被执行。在你的代码中，你应该将原本在`app.use(app.router)`之后的向`app.use`的请求移动到其他路由之后（HTTP动作）。

```
app.use(cookieParser());
app.use(bodyParser());
/// 其他的中间件，并没有影响

app.get('/' ...);
app.post(...);

// 更多的中间件（在路由后执行）
app.use(function(req, res, next);
// 处理错误的中间件
app.use(function(err, req, res, next) {});
```

## express.createServer\(\) {#articleHeader4}

长期弃用。现在只要使用\`express\(\)'创建一个新的应用。

## connect 中间件 {#articleHeader5}

除了`express.static`为了便捷性直接封装在 express 中，其他所有 connect 的中间件都被分离到了独立的模块中。由此，每个独立的模块都可以拥有自己的版本控制。

## connect 的补丁 {#articleHeader6}

Connect 对 node 的原型进行了全局的改动。  
这被认为是不正确的，所以在 Connect 3 中已被移除。  
其中一些补丁是：

* `res.on('header')`
* `res.charset`
* `res.headerSent`
  * use node's
    `res.headersSent`
    instead

你不应该在任何 Connect 或 Express 的库中再使用这些。

## res.charset {#articleHeader7}

如果你想快速设置默认的字符集（你确实应该这么做），  
使用`res.set('content-type')`或者`res.type()`来设置 header 。  
当使用`res.setHeader()`时，默认的字符集将不会添加。

# 改动 {#articleHeader8}

## app.use {#articleHeader9}

`app.use`现在可以接受`:params`.

```
app.use('/users/:user_id', function(req, res, next) {
  // req.params.user_id 可以正确获取
});
```

## req.accepted\(\) {#articleHeader10}

使用`req.accepts()`来代替。

* `req.accepts()`
* `req.acceptsEncodings()`
* `req.acceptsCharsets()`
* `req.acceptsLanguages()`

都在内部使用[accepts](https://github.com/expressjs/accepts)。  
请参考`accepts`来提交问题或者查看文档。

请注意，这些属性可能已经从数组改为函数。  
要把它们作为“数组”使用，只要不向它们传递任何参数即可。

例如,`req.acceptsLanguages() // => ['en', 'es', 'fr']`.

## res.location\(\) {#articleHeader11}

不再做相对路径的 URL 解析。浏览器自身将处理相对路径的 URL 。

## app.route -&gt; app.mountpath {#articleHeader12}

当把一个 express 应用安装到另一个 express 应用中时。

## 配置改动 {#articleHeader13}

* `json spaces`
  在开发模式下不再默认启用。

## req.params {#articleHeader14}

现在它是一个对象而不是数组。如果你以前使用`req.params[##]`的形式，这不会破坏你的应用，它可以在不知道参数名的情况下，使用正则来处理路由。

## res.locals {#articleHeader15}

不再是一个函数。现在这是一个普通的JS对象。所以把它当作普通的对象来处理。

## res.headerSent {#articleHeader16}

改为`headersSent`来匹配 node.js 的 ServerResponse 对象。你的应用可能并没有使用到这个，因此它不会是一个问题。

req.is

现在在内部使用[type-is](https://github.com/expressjs/type-is)。  
请参考`type-is`来提交问题或者查看文档。

# 新加入 {#articleHeader18}

## app.route\(path\) {#articleHeader19}

返回一个新的`Route`的实例。当收到和这个路由的`path`相匹配的请求时，这个路由就会被调用。路由可以有自己的中间件和针对 HTTP 动作的方法来处理请求。

请参阅路由和路由线路文档，以便更好地了解如何在 express 中创建路由。

## Router 和 Route 中间件 {#articleHeader20}

Router 已经全面改版，现在它是一个功能完善的中间件路由。Router 是在不牺牲参数匹配和中间件的情况下，将你的路由分离到多个文件或者模块的好方法。请参阅 Routes 及

[路由文档](http://expressjs.com/4x/api.html#router)

