# axios基本用法

vue更新到2.0之后，作者就宣告不再对vue-resource更新，而是推荐的axios,前一段时间用了一下，现在说一下它的基本用法。

首先就是引入axios,如果你使用es6，只需要安装axios模块之后

```
import axios from 'axios';
//安装方法
npm install axios
//或
bower install axios
```

当然也可以用script引入

```
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

axios提供了一下几种请求方式

```
axios.request(config)

axios.get(url[, config])

axios.delete(url[, config])

axios.head(url[, config])

axios.post(url[, data[, config]])

axios.put(url[, data[, config]])

axios.patch(url[, data[, config]])
```

这里的config是对一些基本信息的配置，比如请求头，baseURL，当然这里提供了一些比较方便配置项

```
//config
import Qs from 'qs'
{
  //请求的接口，在请求的时候，如axios.get(url,config);这里的url会覆盖掉config中的url
  url: '/user',

  // 请求方法同上
  method: 'get', // default
  // 基础url前缀
  baseURL: 'https://some-domain.com/api/',
　　
　　　　
  transformRequest: [function (data) {
    // 这里可以在发送请求之前对请求数据做处理，比如form-data格式化等，这里可以使用开头引入的Qs（这个模块在安装axios的时候就已经安装了，不需要另外安装）
　　data = Qs.stringify({});
    return data;
  }],

  transformResponse: [function (data) {
    // 这里提前处理返回的数据

    return data;
  }],

  // 请求头信息
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  //parameter参数
  params: {
    ID: 12345
  },

  //post参数，使用axios.post(url,{},config);如果没有额外的也必须要用一个空对象，否则会报错
  data: {
    firstName: 'Fred'
  },

  //设置超时时间
  timeout: 1000,
  //返回数据类型
  responseType: 'json', // default

 
}
```

有了配置文件，我们就可以减少很多额外的处理代码也更优美，直接使用

```
axios.post(url,{},config)
    .then(function(res){
        console.log(res);
    })
    .catch(function(err){
        console.log(err);
    })
//axios请求返回的也是一个promise,跟踪错误只需要在最后加一个catch就可以了。
//下面是关于同时发起多个请求时的处理

axios.all([get1(), get2()])
  .then(axios.spread(function (res1, res2) {
    // 只有两个请求都完成才会成功，否则会被catch捕获
  }));
```

最后还是说一下配置项，上面讲的是额外配置，如果你不想另外写也可以直接配置全局

```
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';

//当然还可以这么配置
var instance = axios.create({
  baseURL: 'https://api.example.com'
});

```

本文只是介绍基本的用法，详细看官方文档[https://github.com/axios](https://github.com/axios)

我写的两个例子：

使用vue2.0+mintUI+axios+vue-router:[https://github.com/Stevenzwzhai/vue-mobile-application](https://github.com/Stevenzwzhai/vue-mobile-application)

使用vue2.0+elementUI+axios+vue-router:[https://github.com/Stevenzwzhai/vue2.0-elementUI-axios-vueRouter](https://github.com/Stevenzwzhai/vue2.0-elementUI-axios-vueRouter),在这里例子中pwa文件简单配置了一些东西，可以用于pwa应用开发的参考，文章参考[http://www.cnblogs.com/Upton/p/6894589.html](http://www.cnblogs.com/Upton/p/6894589.html)

  


