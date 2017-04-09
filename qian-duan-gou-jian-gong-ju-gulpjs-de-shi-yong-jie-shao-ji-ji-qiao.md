## 前端构建工具gulpjs的使用介绍及技巧

[gulpjs](http://gulpjs.com/)是一个前端构建工具，与[gruntjs](http://gruntjs.com/)相比，gulpjs无需写一大堆繁杂的配置参数，API也非常简单，学习起来很容易，而且gulpjs使用的是nodejs中[stream](http://nodejs.org/api/stream.html)来读取和操作数据，其速度更快。如果你还没有使用过前端构建工具，或者觉得gruntjs太难用的话，那就尝试一下gulp吧。

本文导航：

1. gulp的安装
2. 开始使用gulp
3. gulp的API介绍
4. 一些常用的gulp插件

# 1、gulp的安装 {#gulp的安装}

首先确保你已经正确安装了nodejs环境。然后以全局方式安装gulp：

```
npm install -g gulp
```

全局安装gulp后，还需要在每个要使用gulp的项目中都单独安装一次。把目录切换到你的项目文件夹中，然后在命令行中执行：

```
npm install gulp
```

如果想在安装的时候把gulp写进项目package.json文件的依赖中，则可以加上--save-dev：

```
npm install --save-dev gulp
```

这样就完成了gulp的安装。至于为什么在全局安装gulp后，还需要在项目中本地安装一次，有兴趣的可以看下stackoverflow上有人做出的回答：[why-do-we-need-to-install-gulp-globally-and-locally](http://stackoverflow.com/questions/22115400/why-do-we-need-to-install-gulp-globally-and-locally)、[what-is-the-point-of-double-install-in-gulp](http://stackoverflow.com/questions/25713618/what-is-the-point-of-double-install-in-gulp)。大体就是为了版本的灵活性，但如果没理解那也不必太去纠结这个问题，只需要知道通常我们是要这样做就行了。

# 2、开始使用gulp {#开始使用gulp}

### 2.1 建立gulpfile.js文件 {#建立gulpfile.js文件}

就像gruntjs需要一个`Gruntfile.js`文件一样，gulp也需要一个文件作为它的主文件，在gulp中这个文件叫做`gulpfile.js`。新建一个文件名为`gulpfile.js`的文件，然后放到你的项目目录中。之后要做的事情就是在`gulpfile.js`文件中定义我们的任务了。下面是一个最简单的`gulpfile.js`文件内容示例，它定义了一个默认的任务。

```
var gulp = require('gulp');
gulp.task('default',function(){
    console.log('hello world');
});
```

此时我们的目录结构是这样子的：

├── gulpfile.js  
├── node\_modules  
│ └── gulp  
└── package.json

### 2.2 运行gulp任务 {#运行gulp任务}

要运行gulp任务，只需切换到存放`gulpfile.js`文件的目录\(windows平台请使用cmd或者Power Shell等工具\)，然后在命令行中执行`gulp`命令就行了，`gulp`后面可以加上要执行的任务名，例如`gulp task1`，如果没有指定任务名，则会执行任务名为`default`的默认任务。

# 3、gulp的API介绍 {#gulp的api介绍}

使用gulp，仅需知道4个API即可：`gulp.task()`,`gulp.src()`,`gulp.dest()`,`gulp.watch()`，所以很容易就能掌握，但有几个地方需理解透彻才行，我会在下面一一说明。为了避免出现理解偏差，建议先看一遍[官方文档](https://github.com/gulpjs/gulp/blob/master/docs/API.md)。





