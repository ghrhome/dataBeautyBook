# 入门Webpack，看这篇就够了

> 本文写作时所用的webpack版本为1._版，与webpack2._在部分语法上存在不同。请见谅

> 阅读本文之前，先看下面这个webpack的配置文件，如果每一项你都懂，那本文能带给你的收获也许就比较有限，你可以快速浏览或直接跳过；如果你和十天前的我一样，对很多选项存在着疑惑，那花一段时间慢慢阅读本文，你的疑惑一定一个一个都会消失；如果你以前没怎么接触过Webpack，而你又你对webpack感兴趣，那么动手跟着本文中那个贯穿始终的例子写一次，写完以后你会发现你已明明白白的走进了Webpack的大门。

> 作者：zhangwang

> 链接：http://www.jianshu.com/p/42e11515c10f

> 來源：简书

> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

```
//一个常见的Webpack配置文件
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



