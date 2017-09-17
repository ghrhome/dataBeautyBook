# node.js express 4.x Router学习心得

express是基于node.js开发的一款MVC模式的Web框架，该框架轻量、支持MVC模式、支持很多常用的中间件（如 body-parser:用于解析客户端请求的body中的内容,，express-session：session解析，cookie-parser：cookie解析）,个人感觉最好地放就是支持路由。我们开发的时候，经常会用到路由。像其他语言，比如说java，个人理解java对路由的处理是通过filter、或者listener来处理的，node.js是诞生不久，生态圈的完善还有一段很长的路要走。

下面简单说下express 路由的使用。  


# 1、安装express npm install express

# 2、创建路由

//加载express框架

var express = require\('express'\);

//创建一个express实例

var app = express\(\);

//创建express的路由功能，可以根据需要创建多个路由，需要多少，创建多少。

var router = express.Router\(\);

router.use\(function\(req, res, next\) {

    //路由，类似于java中的拦截器功能，在请求到达后台之前，先在这里处理。

    //  some logic here ..

    req.query\["name"\] = "tom";

    console.info\('进入路由，添加一个参数name=tom'\);

    //next的作用是将请求转发，这个必须有，如果没有，请求到这就挂起了。

    next\(\);

}\);

//get\('/login'\) 截取Get请求方式的url中含有/login的请求

router.get\('/login', function\(req, res, next\) {

    console.log\('进入路由，添加一个参数age=28'\);

    req.query\["age"\] = "28";

    next\(\); //请求转发

}\);

  


  


//加载路由，这里要放在下面原始监听/login的上面

  


app.get\('/login', router\);

  


app.get\('/login', function\(req, res\) {

  


    console.log\('打印参数', req.query\);

  


    res.end\('ok'\);

  


}\);

  


  


app.listen\(3000\); //指定端口并启动express web服务

  


  


结果：

![](http://doc.okbase.net/picture/addon/2015/01/29/A090924044-126298.png)  


# 3、router.all\(path, \[callback...\], callback\)

对于一个路由，all方法可以添加多个逻辑方法，logic1，logic2，请求按照顺序转发，即logic1完了进入logic2，等价于

  


router.all\('\*', logic1\)

  


router.all\('\*', logic2\);

  


  


示例：

  


var express = require\('express'\);

  


var app = express\(\);

  


var router = express.Router\(\);

  


  


router.use\(function\(req, res, next\) {

  


    req.query\["name"\] = "tom";

  


    console.info\('进入路由，添加一个参数name=tom'\);

  


    next\(\);

  


}\);

  


  


router.get\('/login', function\(req, res, next\) {

  


    console.log\('进入路由，添加一个参数age=28'\);

  


    req.query\["age"\] = "28";

  


    next\(\); //请求转发

  


}\);

  


  


app.get\('/login', router\);

  


app.get\('/login', function\(req, res\) {

  


    console.log\('打印参数', req.query\);

  


    res.end\('ok'\);

  


}\);

  


  


router.all\('\*', logic1, logic2\);

  


  


function logic1\(req, res, next\) {

  


    req.query\["logic1"\] = "logic1";

  


    next\(\);

  


}

  


function logic2\(req, res, next\) {

  


    req.query\["logic2"\] = "logic2";

  


    next\(\);

  


}

app.listen\(3000\); //指定端口并启动express web服务

结果：

![](http://doc.okbase.net/picture/addon/2015/01/29/A090926122-126298.png)  


# 4、支持正则表达式匹配URL示例： 

var express = require\('express'\);

  


var app = express\(\);

  


var router = express.Router\(\);

  


router.get\(/^\/login\/result1/, function\(req, res\) {

  


    console.log\('打印参数result1 ', req.query\);

  


    res.end\('ok'\);

  


}\)

  


router.get\(/^\/login\/result2/, function\(req, res\) {

  


    console.log\('打印参数 result2 ', req.query\);

  


    res.end\('ok'\);

  


}\)

  


app.get\('/login/\*', router\);

  


app.listen\(3000\);

  


  


测试：http://127.0.0.1:3000/login/result1?var=tom

  


结果

![](http://doc.okbase.net/picture/addon/2015/01/29/A090928200-126298.png)  


测试：http://127.0.0.1:3000/login/result2?var=tom2

结果：

![](http://doc.okbase.net/picture/addon/2015/01/29/A090930278-126298.png)  


# 4、router.param\(\[name\], callback\)

http://127.0.0.1:3000/user/:id方式，router对象的param方法用于路径参数的处理

  


  


var express = require\('express'\);

  


var app = express\(\);

  


var router = express.Router\(\);

  


router.param\('id', function \(req, res, next, id\) {

  


  console.log\('print id1 '+id\);

  


  next\(\);

  


}\)

  


  


router.get\('/user/:id', function \(req, res, next\) {

  


  console.log\('print id2 '+req.params.id\);

  


  next\(\);

  


}\);

  


  


router.get\('/user/:id', function \(req, res\) {

  


  console.log\('print id3 '+req.params.id\);

  


  res.end\(\);

  


}\);

  


app.get\('/\*', router\);

  


app.listen\(3000\);

  


  


测试：http://127.0.0.1:3000/user/tom

  


结果

![](http://doc.okbase.net/picture/addon/2015/01/29/A090932325-126298.png)  


# 5、 router.route\(path\)

路由以链式的方式依次处理get put post delete 等http请求

  


  


var express = require\('express'\);

  


var app = express\(\);

  


var router = express.Router\(\);

  


  


router.param\('user\_id', function\(req, res, next, id\) {

  


  // sample user, would actually fetch from DB, etc...

  


  req.user = {

  


    id: id,

  


    name: 'TJ'

  


  };

  


  next\(\);

  


}\);

  


  


router.route\('/users/:user\_id'\)

  


.all\(function\(req, res, next\) {

  


  // runs for all HTTP verbs first

  


  // think of it as route specific middleware!

  


  next\(\);

  


}\)

  


.get\(function\(req, res, next\) {

  


  res.json\(req.user\);

  


}\)

  


.put\(function\(req, res, next\) {

  


  // just an example of maybe updating the user

  


  req.user.name = req.params.name;

  


  // save user ... etc

  


  res.json\(req.user\);

  


}\)

  


.post\(function\(req, res, next\) {

  


  next\(new Error\('not implemented'\)\);

  


}\)

  


.delete\(function\(req, res, next\) {

  


  next\(new Error\('not implemented'\)\);

  


}\)

this all，后续还会补充完善。



