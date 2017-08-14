# 入门webpack补遗

## 多个入口起点 {#多个入口起点}

如果配置创建了多个单独的 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用[占位符\(substitutions\)](http://www.css88.com/doc/webpack/configuration/output#output-filename)来确保每个文件具有唯一的名称。

```
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}

// 写入到硬盘：./dist/app.js, ./dist/search.js

```



