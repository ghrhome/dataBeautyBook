## Express中的app.use与app.get的区别

# app.use {#appuse}

`app.use`的作用是将一个中间件绑定到应用中，参数`path`是一个路径前缀，用于限定中间件的作用范围，所有以该前缀开始的请求路径均是中间件的作用范围，不考虑http的请求方法，例如：  
如果path 设置为’/’,则  
- GET /  
- PUT /foo  
- POST /foo/bar  
均是中间件的作用范围。

# app.get {#appget}

app.get是express中应用路由的一部分，用于匹配并处理一个特定的请求，且请求方法必须是GET。

```

app.use('/',function(req, res,next) {
    res.send('Hello');
    next();
});
```

等同于：

```
app.all(/^\/.*/, function (req, res) {
    res.send('Hello');
});
```

# Demo {#demo}

```
app.use('/', function(req, res, next) {
    res.write(' root middleware');
    next();
});

app.use('/user', function(req, res, next) {
    res.write(' user middleware');
    next();
});

app.get('/', function(req, res) {
    res.end(' /');
});

app.get('/user', function(req, res) {
    res.end(' user');
});
```

上述代码中，如果请求[http://localhost:3000](http://localhost:3000/)对应的输出为

```
root middleware /
```

请求[http://localhost:3000/user](http://localhost:3000/user)对应的输出为

```
https://stackoverflow.com/questions/15601703/difference-between-app-use-and-app-get-in-express-js
```



