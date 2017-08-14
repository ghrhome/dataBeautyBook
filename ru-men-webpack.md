#### 优化插件

webpack提供了一些在发布阶段非常有用的优化插件，它们大多来自于webpack社区，可以通过npm安装，通过以下插件可以完成产品发布阶段所需的功能

* OccurenceOrderPlugin :为组件分配ID，通过这个插件webpack可以分析和优先考虑使用最多的模块，并为它们分配最小的ID
* UglifyJsPlugin：压缩JS代码；
* ExtractTextPlugin：分离CSS和JS文件

我们继续用例子来看看如何添加它们，OccurenceOrder 和 UglifyJS plugins 都是内置插件，你需要做的只是安装它们

```
npm install --save-dev extract-text-webpack-plugin
```

在配置文件的plugins后引用它们

```
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  entry: __dirname + "/app/main.js",
  output: {
    path: __dirname + "/build",
    filename: "bundle.js"
  },

  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel'
      },
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', 'css?modules!postcss')
      }
    ]
  },
  postcss: [
    require('autoprefixer')
  ],

  plugins: [
    new HtmlWebpackPlugin({
      template: __dirname + "/app/index.tmpl.html"
    }),
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.optimize.UglifyJsPlugin(),
    new ExtractTextPlugin("style.css")
  ]
}
```

#### 缓存

缓存无处不在，使用缓存的最好方法是保证你的文件名和文件内容是匹配的（内容改变，名称相应改变）

webpack可以把一个哈希值添加到打包的文件名中，使用方法如下,添加特殊的字符串混合体（\[name\], \[id\] and \[hash\]）到输出文件名前

```
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  entry: __dirname + "/app/main.js",
  output: {
    path: __dirname + "/build",
    filename: "[name]-[hash].js"
  },

  module: {
    loaders: [
      {
        test: /\.json$/,
        loader: "json"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel'
      },
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', 'css?modules!postcss')
      }
    ]
  },
  postcss: [
    require('autoprefixer')
  ],

  plugins: [
    new HtmlWebpackPlugin({
      template: __dirname + "/app/index.tmpl.html"
    }),
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.optimize.UglifyJsPlugin(),
    new ExtractTextPlugin("[name]-[hash].css")
  ]
}
```

现在用户会有合理的缓存了。

### 总结

这是一篇好长的文章，谢谢你的耐心，能仔细看到了这里，大概半个月前我第一次自己一步步配置项目所需的Webpack后就一直想写一篇笔记做总结，几次动笔都不能让自己满意，总觉得写不清楚。直到看到本文的英文版[Webpack for React](http://www.pro-react.com/materials/appendixA/)，真的有多次豁然开朗的感觉，喜欢看原文的点链接就可以看了。其实关于Webpack本文讲述得仍不完全，不过相信你看完后已经进入Webpack的大门，能够更好的探索其它的关于Webpack的知识了。

欢迎大家在文后发表自己的观点讨论。

> 补充一下，现在已经不允许在配置loaders的loader关键字时省略后面的“-loader”后缀了，所以上面的使用json的loader要从json改为json-loader

### 处理图片和其他静态文件

这个和其他一样，也许你也已经会玩了。安装loader，处理文件。诸如图片，字体等等，不过有个神奇的地方它可以根据你的需求将一些图片自动转成base64编码的，为你减轻很多的网络请求。

安装url-loader

```
npm install url-loader --save-dev
```

配置config文件

```
{
        test: /\.(png|jpg)$/,
        loader: 'url?limit=40000'
      }
```

注意后面那个limit的参数，当你图片大小小于这个限制的时候，会自动启用base64编码图片。

下面举个栗子。

新建一个imgs文件夹，往里面添加一张崔叔的照片。在scss文件中添加如下的东西。

```
@import "./variables.scss";

h1 {
  color: $red;
  background: url('./imgs/avatar.jpg');
}
```

pm start, 然后查看图片的url，发现神奇。

## 添加第三方库

有的时候还想来点jquery，moment，undersocre之类的库，webpack可以非常容易的做到这一点，有谣言说Bower即将停止开发了, 作者推荐都使用npm来管理依赖。那么我们现在安装在我们的app中添加jquery和moment的支持。

```
npm install jquery moment --save-dev
```

在js中引用

```
var sub = require('./sub');
var $ = require('jquery');
var moment = require('moment');
var app  = document.createElement('div');
app.innerHTML = '<h1>Hello World it</h1>';
document.body.appendChild(app);
app.appendChild(sub());
$('body').append('<p>look at me! now is ' + moment().format() + '</p>');
```

看看浏览器，成功！ jquery和moment现在都起作用了！





