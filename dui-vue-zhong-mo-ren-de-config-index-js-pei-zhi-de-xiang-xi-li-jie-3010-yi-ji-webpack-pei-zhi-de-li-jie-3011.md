# 对vue中 默认的 config/index.js:配置的详细理解 -【以及webpack配置的理解】

当我们需要和后台分离部署的时候，必须配置config/index.[js](http://lib.csdn.net/base/javascript):

用vue-cli 自动构建的目录里面  （环境变量及其基本变量的配置）

```
var path = require('path')
 
module.exports = {
  build: {
    index: path.resolve(__dirname, 'dist/index.html'),
    assetsRoot: path.resolve(__dirname, 'dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    productionSourceMap: true
  },
  dev: {
    port: 8080,
    proxyTable: {}
  }
}
```

在'build'部分，我们有以下选项：

### `build.index` {#buildindex}

> 必须是本地文件系统上的绝对路径。

`index.html` \(带着插入的资源路径\) 会被生成。

如果你在后台框架中使用此模板，你可以编辑`index.html`路径指定到你的后台程序生成的文件。例如Rails程序，可以是`app/views/layouts/application.html.erb`，或者Laravel程序，可以是`resources/views/index.blade.`[`PHP`](http://lib.csdn.net/base/php)。

### `build.assetsRoot` {#buildassetsroot}

> 必须是本地文件系统上的绝对路径。

应该指向包含应用程序的所有静态资产的根目录。`public/` 对应Rails/Laravel。

### `build.assetsSubDirectory` {#buildassetssubdirectory}

被webpack编译处理过的资源文件都会在这个`build.assetsRoot`目录下，所以它不可以混有其它可能在`build.assetsRoot`里面有的文件。例如，假如`build.assetsRoot`参数是`/path/to/dist`，`build.assetsSubDirectory` 参数是 `static`, 那么所以webpack资源会被编译到`path/to/dist/static`目录。

每次编译前，这个目录会被清空，所以这个只能放编译出来的资源文件。

`static/`目录的文件会直接被在构建过程中，直接拷贝到这个目录。这意味着是如果你改变这个规则，所有你依赖于`static/`中文件的绝对地址，都需要改变。

### `build.assetsPublicPath【资源的根目录】` {#buildassetspublicpath}

这个是通过http服务器运行的url路径。在大多数情况下，这个是根目录\(`/`\)。如果你的后台框架对静态资源url前缀要求，你仅需要改变这个参数。在内部，这个是被webpack当做`output.publicPath`来处理的。

后台有要求的话一般要加上./ 或者根据具体目录添加，不然引用不到静态资源

### `build.productionSourceMap` {#buildproductionsourcemap}

在构建生产环境版本时是否开启source map。

### `dev.port` {#devport}

开发服务器监听的特定端口

### `dev.proxyTable` {#devproxytable}

定义开发服务器的代理规则。

 项目中配置的config/index.js，有dev和production两种环境的配置 以下介绍的是production环境下的webpack配置的理解



# 一、文件结构 {#一文件结构}

本文主要分析开发（dev）和构建（build）两个过程涉及到的文件，故下面文件结构仅列出相应的内容。

```
├─build
│   ├─build.js
│   ├─check-versions.js
│   ├─dev-client.js
│   ├─dev-server.js
│   ├─utils.js
│   ├─vue-loader.conf.js
│   ├─webpack.base.conf.js
│   ├─webpack.dev.conf.js
│   ├─webpack.prod.conf.js
│   └─webpack.test.conf.js
├─config
│   ├─dev.env.js
│   ├─index.js
│   ├─prod.env.js
│   └─test.env.js
├─...
└─package.json
```

# 二、指令分析 {#二指令分析}

首先看package.json里面的scripts字段，

```
"scripts": {
  "dev": "node build/dev-server.js",
  "build": "node build/build.js",
  "unit": "cross-env BABEL_ENV=test karma start test/unit/karma.conf.js --single-run",
  "e2e": "node test/e2e/runner.js",
  "test": "npm run unit && npm run e2e",
  "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs"
 }
```

[测试](http://lib.csdn.net/base/softwaretest)的东西先不看，直接看”dev”和”build”。运行”npm run dev”的时候执行的是build/dev-server.

[js](http://lib.csdn.net/base/javascript)文件，运行”npm run build”的时候执行的是build/build.js文件，我们可以从这两个文件开始进行代码阅读分析。

# 三、build文件夹分析 {#三build文件夹分析}

## build/dev-server.js {#builddev-serverjs}

首先来看执行”npm run dev”时候最先执行的build/dev-server.js文件。该文件主要完成下面几件事情：

1. 检查node和npm的版本
2. 引入相关插件和配置
3. 创建express服务器和webpack编译器
4. 配置开发中间件（webpack-dev-middleware）和热重载中间件（webpack-hot-middleware）
5. 挂载代理服务和中间件
6. 配置静态资源
7. 启动服务器监听特定端口（8080）
8. 自动打开浏览器并打开特定网址（localhost:8080）

**说明：**express服务器提供静态文件服务，不过它还使用了http-proxy-middleware，一个http请求代理的中间件。[前端开发](http://lib.csdn.net/base/jquery)过程中需要使用到后台的API的话，可以通过配置proxyTable来将相应的后台请求代理到专用的API服务器。

详情请看代码注释：

```
// 检查NodeJS和npm的版本
require('./check-versions')()

// 获取配置
var config = require('../config')
// 如果Node的环境变量中没有设置当前的环境（NODE_ENV），则使用config中的配置作为当前的环境
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}

// 一个可以调用默认软件打开网址、图片、文件等内容的插件
// 这里用它来调用默认浏览器打开dev-server监听的端口，例如：localhost:8080
var opn = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')

// 一个express中间件，用于将http请求代理到其他服务器
// 例：localhost:8080/api/xxx  -->  localhost:3000/api/xxx
// 这里使用该插件可以将前端开发中涉及到的请求代理到API服务器上，方便与服务器对接
var proxyMiddleware = require('http-proxy-middleware')

// 根据 Node 环境来引入相应的 webpack 配置
var webpackConfig = process.env.NODE_ENV === 'testing'
  ? require('./webpack.prod.conf')
  : require('./webpack.dev.conf')

// dev-server 监听的端口，默认为config.dev.port设置的端口，即8080
var port = process.env.PORT || config.dev.port

// 用于判断是否要自动打开浏览器的布尔变量，当配置文件中没有设置自动打开浏览器的时候其值为 false
var autoOpenBrowser = !!config.dev.autoOpenBrowser

// 定义 HTTP 代理表，代理到 API 服务器
var proxyTable = config.dev.proxyTable

// 创建1个 express 实例
var app = express()

// 根据webpack配置文件创建Compiler对象
var compiler = webpack(webpackConfig)

// webpack-dev-middleware使用compiler对象来对相应的文件进行编译和绑定
// 编译绑定后将得到的产物存放在内存中而没有写进磁盘
// 将这个中间件交给express使用之后即可访问这些编译后的产品文件
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})

// webpack-hot-middleware，用于实现热重载功能的中间件
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {}
})

// 当html-webpack-plugin提交之后通过热重载中间件发布重载动作使得页面重载
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

// 将 proxyTable 中的代理请求配置挂在到express服务器上
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  // 格式化options，例如将'www.example.com'变成{ target: 'www.example.com' }
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
// 重定向不存在的URL，常用于SPA
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
// 使用webpack开发中间件
// 即将webpack编译后输出到内存中的文件资源挂到express服务器上
app.use(devMiddleware)

// enable hot-reload and state-preserving
// compilation error display
// 将热重载中间件挂在到express服务器上
app.use(hotMiddleware)

// serve pure static assets
// 静态资源的路径
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)

// 将静态资源挂到express服务器上
app.use(staticPath, express.static('./static'))

// 应用的地址信息，例如：http://localhost:8080
var uri = 'http://localhost:' + port

// webpack开发中间件合法（valid）之后输出提示语到控制台，表明服务器已启动
devMiddleware.waitUntilValid(function () {
  console.log('> Listening at ' + uri + '\n')
})

// 启动express服务器并监听相应的端口（8080）
module.exports = app.listen(port, function (err) {
  if (err) {
    console.log(err)
    return
  }

  // when env is testing, don't need open it
  // 如果符合自动打开浏览器的条件，则通过opn插件调用系统默认浏览器打开对应的地址uri
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
})
```























