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

### 3.1 gulp.src\(\) {#gulp.src}

在介绍这个API之前我们首先来说一下Grunt.js和Gulp.js工作方式的一个区别。Grunt主要是以文件为媒介来运行它的工作流的，比如在Grunt中执行完一项任务后，会把结果写入到一个临时文件中，然后可以在这个临时文件内容的基础上执行其它任务，执行完成后又把结果写入到临时文件中，然后又以这个为基础继续执行其它任务...就这样反复下去。而在Gulp中，使用的是Nodejs中的[stream](http://nodejs.org/api/stream.html)\(流\)，首先获取到需要的stream，然后可以通过stream的`pipe()`方法把流导入到你想要的地方，比如Gulp的插件中，经过插件处理后的流又可以继续导入到其他插件中，当然也可以把流写入到文件中。所以Gulp是以stream为媒介的，它不需要频繁的生成临时文件，这也是Gulp的速度比Grunt快的一个原因。再回到正题上来，`gulp.src()`方法正是用来获取流的，但要注意这个流里的内容不是原始的文件流，而是一个虚拟文件对象流\([Vinyl files](https://github.com/wearefractal/vinyl-fs)\)，这个虚拟文件对象中存储着原始文件的路径、文件名、内容等信息，这个我们暂时不用去深入理解，你只需简单的理解可以用这个方法来读取你需要操作的文件就行了。其语法为：

`gulp.src(globs[, options])`

**globs**参数是文件匹配模式\(类似正则表达式\)，用来匹配文件路径\(包括文件名\)，当然这里也可以直接指定某个具体的文件路径。当有多个匹配模式时，该参数可以为一个数组。  
**options**为可选参数。通常情况下我们不需要用到。

下面我们重点说说Gulp用到的glob的匹配规则以及一些文件匹配技巧。  
Gulp内部使用了[node-glob](https://github.com/isaacs/node-glob)模块来实现其文件匹配功能。我们可以使用下面这些特殊的字符来匹配我们想要的文件：

* `*`
  匹配文件路径中的0个或多个字符，但不会匹配路径分隔符，除非路径分隔符出现在末尾
* `**`
  匹配路径中的0个或多个目录及其子目录,需要单独出现，即它左右不能有其他东西了。如果出现在末尾，也能匹配文件。
* `?`
  匹配文件路径中的一个字符\(不会匹配路径分隔符\)
* `[...]`
  匹配方括号中出现的字符中的任意一个，当方括号中第一个字符为
  `^`或`!`时，则表示不匹配方括号中出现的其他字符中的任意一个，类似js正则表达式中的用法
* `!(pattern|pattern|pattern)`
  匹配任何与括号中给定的任一模式都不匹配的
* `?(pattern|pattern|pattern)`
  匹配括号中给定的任一模式0次或1次，类似于js正则中的\(pattern\|pattern\|pattern\)?
* `+(pattern|pattern|pattern)`
  匹配括号中给定的任一模式至少1次，类似于js正则中的\(pattern\|pattern\|pattern\)+
* `*(pattern|pattern|pattern)`
  匹配括号中给定的任一模式0次或多次，类似于js正则中的\(pattern\|pattern\|pattern\)\*
* `@(pattern|pattern|pattern)`
  匹配括号中给定的任一模式1次，类似于js正则中的\(pattern\|pattern\|pattern\)

下面以一系列例子来加深理解



























