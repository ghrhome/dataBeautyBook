## Node.js开发框架Express4.x

[从零开始nodejs系列文章](http://blog.fens.me/series-nodejs/)

，将介绍如何利Javascript做为服务端脚本，通过Nodejs框架web开发。Nodejs框架是基于V8的引擎，是目前速度最快的Javascript引擎。chrome浏览器就基于V8，同时打开20-30个网页都很流畅。Nodejs标准的web开发框架Express，可以帮助我们迅速建立web站点，比起PHP的开发效率更高，而且学习曲线更低。非常适合小型网站，个性化网站，我们自己的Geek网站！！

**前言**

Nodejs是一个年轻的编程框架，充满了活力和无限激情，一直都在保持着快速更新。基于Nodejs的官方Web开发库Express也在同步发展着，每年升级一个大版本，甚至对框架底层都做了大手术。在Express4时，替换掉中件间库connect，而改用多个更细粒度的库来取代。带来的好处是明显地，这些中间件能更自由的更新和发布，不会受到Express发布周期的影响；但问题也是很的棘手，不兼容于之前的版本，升级就意味着要修改代码。

之前写过一篇文章“[Nodejs开发框架Express3.0开发手记–从零开始](http://blog.fens.me/nodejs-express3/)”，很多新学Node的朋友都在参考，但由于Express已经升级，文章中有部分的代码已经不能使用，所以就有了这篇介绍Express4.x的文章。

**目录**

1. 建立工程
2. 目录结构
3. package.json项目配置
4. app.js核心文件
5. Bootstrap界面框架
6. 路由功能
7. 程序代码
8. Express3.x和Express4.x的改动列表

## 1. 建立项目

让我们从头开始Express4.x的安装和使用吧，安装Node和NPM在本文就不多说了。Linux环境安装请参考文章，[准备Nodejs开发环境Ubuntu](http://blog.fens.me/nodejs-enviroment/)，Window环境安装直接下载Node的安装文件，双击安装就行了。

我的系统环境

* Win7 64bit
* Nodejs:v0.10.31
* Npm:1.4.23

首先，我们需要安装express库。在Express3.6.x之前的版本，Express需要全局安装的，项目构建器模块是合并在Express项目中的，后来这个构建器被拆分出来，独立成为了一个项目express-generator，现在我们只需要全局安装express-generator项目就行了。

```
~ npm install -g express-generator@4  #全局安装-g
C:\Users\Administrator\AppData\Roaming\npm\express -> C:\Users\Administrator\AppData\Roaming\npm\node_modules\express-ge
nerator\bin\express

express-generator@4.11.2 C:\Users\Administrator\AppData\Roaming\npm\node_modules\express-generator
├── sorted-object@1.0.0
├── commander@2.6.0
└── mkdirp@0.5.0 (minimist@0.0.8)
```

安装好express-generator包后，我们在命令行就可以使用express命令了。

```
~ express -V # 检查express的版本
4.11.2

~ express -h  # 检查看express的帮助命令
  Usage: express [options] [dir]
  Options:
    -h, --help          output usage information
    -V, --version       output the version number
    -e, --ejs           add ejs engine support (defaults to jade)
        --hbs           add handlebars engine support
    -H, --hogan         add hogan.js engine support
    -c, --css   add stylesheet  support (less|stylus|compass) (defaults to plain css)
        --git           add .gitignore
    -f, --force         force on non-empty directory
```

接下来，我们使用express的命令，来创建项目了。

```
~ cd D:\workspace\javascript  # 进入工作目录

~ D:\workspace\javascript>express -e nodejs-demo  # 创建项目
   create : nodejs-demo
   create : nodejs-demo/package.json
   create : nodejs-demo/app.js
   create : nodejs-demo/public/javascripts
   create : nodejs-demo/public/images
   create : nodejs-demo/public
   create : nodejs-demo/public/stylesheets
   create : nodejs-demo/public/stylesheets/style.css
   create : nodejs-demo/views
   create : nodejs-demo/views/index.ejs
   create : nodejs-demo/views/error.ejs
   create : nodejs-demo/routes
   create : nodejs-demo/routes/index.js
   create : nodejs-demo/routes/users.js
   create : nodejs-demo/bin
   create : nodejs-demo/bin/www

   install dependencies:
     $ cd nodejs-demo && npm install
   run the app:
     $ DEBUG=nodejs-demo:* ./bin/www
```

进入项目目录，下载依赖库，构建项目。

```
~ D:\workspace\javascript>cd nodejs-demo && npm install
```

启动项目。

```
~ D:\workspace\javascript\nodejs-demo>npm start

> express4-demo@0.0.0 start D:\workspace\javascript\nodejs-demo
> node ./bin/www

module.js:338
    throw err;
          ^
Error: Cannot find module './routes/users'
    at Function.Module._resolveFilename (module.js:336:15)
    at Function.Module._load (module.js:278:25)
    at Module.require (module.js:365:17)
    at require (module.js:384:17)
    at Object. (D:\workspace\javascript\nodejs-demo\app.js:9:13)
    at Module._compile (module.js:460:26)
    at Object.Module._extensions..js (module.js:478:10)
    at Module.load (module.js:355:32)
    at Function.Module._load (module.js:310:12)
    at Module.require (module.js:365:17)
```

第一次启动发生了错误，可能是express-generator和express不匹配造成的，找到问题在app.js文件中，注释第9行和第26行。

```
..
//var users = require('./routes/users');
..
//app.use('/users', users);       
..
```

再次启动项目。

```
D:\workspace\javascript\nodejs-demo>npm start
> express4-demo@0.0.0 start D:\workspace\javascript\nodejs-demo
> node ./bin/www
```

项目启动成功，打开浏览器 [http://localhost:3000，就可以看到显示的页面了。](http://localhost:3000，就可以看到显示的页面了。)

[![](http://blog.fens.me/wp-content/uploads/2015/02/express_1.png "express\_1")](http://blog.fens.me/wp-content/uploads/2015/02/express_1.png)

这样非常简单地，我们就把一个最基本的Web应用做好了，就是几条命令而已。

## 2. 目录结构

接下来，我们详细看一下Express4项目的结构、配置和使用。

* bin, 存放启动项目的脚本文件
* node\_modules, 存放所有的项目依赖库。
* public，静态文件\(css,js,img\)
* routes，路由文件\(MVC中的C,controller\)
* views，页面文件\(Ejs模板\)
* package.json，项目依赖配置及开发者信息
* app.js，应用核心配置文件

[![](http://blog.fens.me/wp-content/uploads/2015/02/express_2.png "express\_2")](http://blog.fens.me/wp-content/uploads/2015/02/express_2.png)

## 3. package.json项目配置

package.json用于项目依赖配置及开发者信息，scripts属性是用于定义操作命令的，可以非常方便的增加启动命令，比如默认的start，用npm start代表执行node ./bin/www命令。

查看package.json文件。

```
{
  "name": "express4-demo",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.10.2",
    "cookie-parser": "~1.3.3",
    "debug": "~2.1.1",
    "ejs": "~2.2.3",
    "express": "~4.11.1",
    "morgan": "~1.5.1",
    "serve-favicon": "~2.2.0"
  }
}
```

## 4. app.js核心文件

从Express3.x升级到Express4.x，主要的变化就在app.js文件中。查看app.js文件，我已经增加注释说明。

```
// 加载依赖库，原来这个类库都封装在connect中，现在需地注单独加载
var express = require('express'); 
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

// 加载路由控制
var routes = require('./routes/index');
//var users = require('./routes/users');

// 创建项目实例
var app = express();

// 定义EJS模板引擎和模板文件位置，也可以使用jade或其他模型引擎
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

// 定义icon图标
app.use(favicon(__dirname + '/public/favicon.ico'));
// 定义日志和输出级别
app.use(logger('dev'));
// 定义数据解析器
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
// 定义cookie解析器
app.use(cookieParser());
// 定义静态文件目录
app.use(express.static(path.join(__dirname, 'public')));

// 匹配路径和路由
app.use('/', routes);
//app.use('/users', users);

// 404错误处理
app.use(function(req, res, next) {
    var err = new Error('Not Found');
    err.status = 404;
    next(err);
});

// 开发环境，500错误处理和错误堆栈跟踪
if (app.get('env') === 'development') {
    app.use(function(err, req, res, next) {
        res.status(err.status || 500);
        res.render('error', {
            message: err.message,
            error: err
        });
    });
}

// 生产环境，500错误处理
app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
        message: err.message,
        error: {}
    });
});

// 输出模型app
module.exports = app;
```

我们看到在app.js中，原来调用connect库的部分都被其他的库所代替，serve-favicon、morgan、cookie-parser、body-parser，默认项目中，只用到了最基本的几个库，还没有其他需要替换的库，在本文最后有详细列出。

另外，原来用于项目启动代码也被移到./bin/www的文件，www文件也是一个node的脚本，用于分离配置和启动程序。

查看./bin/www文件。

```
#!/usr/bin/env node   

/**
 * 依赖加载
 */
var app = require('../app');
var debug = require('debug')('nodejs-demo:server');
var http = require('http');

/**
 * 定义启动端口
 */
var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

/**
 * 创建HTTP服务器实例
 */
var server = http.createServer(app);

/**
 * 启动网络服务监听端口
 */
server.listen(port);
server.on('error', onError);
server.on('listening', onListening);

/**
 * 端口标准化函数
 */
function normalizePort(val) {
  var port = parseInt(val, 10);
  if (isNaN(port)) {
    return val;
  }
  if (port >= 0) {
    return port;
  }
  return false;
}

/**
 * HTTP异常事件处理函数
 */
function onError(error) {
  if (error.syscall !== 'listen') {
    throw error;
  }

  var bind = typeof port === 'string'
    ? 'Pipe ' + port
    : 'Port ' + port

  // handle specific listen errors with friendly messages
  switch (error.code) {
    case 'EACCES':
      console.error(bind + ' requires elevated privileges');
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error(bind + ' is already in use');
      process.exit(1);
      break;
    default:
      throw error;
  }
}

/**
 * 事件绑定函数
 */
function onListening() {
  var addr = server.address();
  var bind = typeof addr === 'string'
    ? 'pipe ' + addr
    : 'port ' + addr.port;
  debug('Listening on ' + bind);
}
```

## 5. Bootstrap界面框架

创建Bootstrap界面框架，直接在index.ejs文件上面做修改。可以手动下载Bootstrap库放到项目中对应的位置引用，也可以通过bower来管理前端的Javascript库，参考文章[bower解决js的依赖管理](http://blog.fens.me/nodejs-bower-intro/)。另外还可以直接使用免费的CDN源加载Bootstrap的css和js文件。下面我就直接使用Bootcss社区提供的CDN源加载Bootstrap。

编辑views/index.ejs文件

```
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <title><%= title %></title>
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.2/css/bootstrap.min.css">
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <div class="well jumbotron">
      <h1><%= title %></h1>
      <p>This is a simple hero unit, a simple jumbotron-style component for calling extra attention to featured content or information.</p>
      <p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
    </div>
    <script src="http://cdn.bootcss.com/jquery/1.11.2/jquery.min.js"></script>
    <script src="http://cdn.bootcss.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>
  </body>
</html>
```

效果如下，已经加入了bootstrap的样式了。

[![](http://blog.fens.me/wp-content/uploads/2015/02/express_3.png "express\_3")](http://blog.fens.me/wp-content/uploads/2015/02/express_3.png)

接下来，我们把index.ejs页面切分成3个部分：header.ejs, index.ejs, footer.ejs，用于网站页面的模块化。

* header.ejs, 为页面的头部区域
* index.ejs, 为内容显示区域
* footer.ejs，为页面底部区域

编辑header.ejs。

```
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <title><%= title %></title>
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.2/css/bootstrap.min.css">
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
```

编辑footer.ejs。

```
  <script src="http://cdn.bootcss.com/jquery/1.11.2/jquery.min.js"></script>
    <script src="http://cdn.bootcss.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>
  </body>
</html>
```

编辑index.ejs。

```
<% include header.ejs %>

<div class="well jumbotron">
<h1><%= title %></h1>
<p>This is a simple hero unit, a simple jumbotron-style component for calling extra attention to featured content or information.</p>
<p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
</div>

<% include footer.ejs %>
```

把页表和页底的代码分离后，让index.ejs页面的核心代码更少，更容易维护。

## 6. 路由功能

路由功能，是Express4以后全面改版的功能。在应用程序加载隐含路由中间件，不用担心在中间件被装载相对于路由器中间件的顺序。定义路由的方式是不变的，路由系统中增加2个新的功能。

* app.route\(\)函数，创建可链接的途径处理程序的路由路径。
* express.Router类，创建模块化安装路径的处理程序。

app.route方法会返回一个Route实例，它可以继续使用所有的HTTP方法，包括get,post,all,put,delete,head等。

```
app.route('/users')
  .get(function(req, res, next) {})
  .post(function(req, res, next) {})
```

express.Router类，则可以帮助我们更好的组织代码结构。在app.js文件中，定义了app.use\(‘/’, routes\); routes是指向了routes目录下的index.js文件，./routes/index.js文件中，express.Router被定义使用，路径/\*处理都会由routes/index.js文件里的Router来处理。如果我们要管理不同的路径，那么可以直接配置为多个不同的Router

## 7. 程序代码

对于刚接触Express4.x的朋友，可以直接从Github上面下载本文项目中的源代码，按照片文章中的介绍学习Express4，下载地址：[https://github.com/bsspirit/nodejs-demo/tree/express4](https://github.com/bsspirit/nodejs-demo/tree/express4)

也可以直接用github命令行来下载：



对于刚接触Express4.x的朋友，可以直接从Github上面下载本文项目中的源代码，按照片文章中的介绍学习Express4，下载地址：[https://github.com/bsspirit/nodejs-demo/tree/express4](https://github.com/bsspirit/nodejs-demo/tree/express4)

也可以直接用github命令行来下载：

```
~ git clone git@github.com:bsspirit/nodejs-demo.git   # 下载github项目
~ cd nodejs-demo                                      # 进入下载目录
~ git checkout express4                               # 切换到express4的分支
~ npm install                                         # 下载依赖库
~ npm start                                           # 启动服务器
```

注：Github上本项目有3分支，express3和master分支都是express3的例子，express4分支是本文的例子。

当然，本文对express4的介绍其实还远远不够，除了文中说到的express改动的部分，其他的部分都express3类似，所以更详细的使用说明，还请大家同时参考文章，[Nodejs开发框架Express3.0开发手记–从零开始](http://blog.fens.me/nodejs-express3/)来一起学习。

## 8. Express3.x和Express4.x的改动列表

最后，列举一下3.x的内置模块和4.x模块的对照表，我们可以发现大部分都是connect部分的替换。关于connect库的详细介绍，可以参考文章，[Nodejs基础中间件Connect](http://blog.fens.me/nodejs-connect/)

| Express 3 | Express 4 |
| :--- | :--- |
| express.bodyParser | [body-parser](https://github.com/expressjs/body-parser)+ [multer](https://github.com/expressjs/multer) |
| express.compress | [compression](https://github.com/expressjs/compression) |
| express.cookieSession | [cookie-session](https://github.com/expressjs/cookie-session) |
| express.cookieParser | [cookie-parser](https://github.com/expressjs/cookie-parser) |
| express.logger | [morgan](https://github.com/expressjs/morgan) |
| express.session | [express-session](https://github.com/expressjs/session) |
| express.favicon | [serve-favicon](https://github.com/expressjs/serve-favicon) |
| express.responseTime | [response-time](https://github.com/expressjs/response-time) |
| express.errorHandler | [errorhandler](https://github.com/expressjs/errorhandler) |
| express.methodOverride | [method-override](https://github.com/expressjs/method-override) |
| express.timeout | [connect-timeout](https://github.com/expressjs/timeout) |
| express.vhost | [vhost](https://github.com/expressjs/vhost) |
| express.csrf | [csurf](https://github.com/expressjs/csurf) |
| express.directory | [serve-index](https://github.com/expressjs/serve-index) |
| express.static | [serve-static](https://github.com/expressjs/serve-static) |

希望能给入门Nodejs的朋友一些帮助，同时我公司也在招Nodejs/Javascript全端工程师，欢迎有实力、愿意创业的朋友与我联系

