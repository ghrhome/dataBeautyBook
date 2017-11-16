# webpack 如何分割代码到指定文件夹？ {#questionTitle}

```
const Home = resolve => {
  require.ensure(['./views/Home.vue'], () => {
    resolve(require('./views/Home.vue'));
  }, 'my_home');
};
```

分割后的的文件路径为build/0.my\_home.\[hash\].js

```
const Home = resolve => {
  require.ensure(['./views/Home.vue'], () => {
    resolve(require('./views/Home.vue'));
  }, 'js/my_home');
};
```

分割后的的文件路径为build/0.js/my\_home.\[hash\].js

但是我想要build/js/my\_home.\[hash\].js ,默认的代码分割总是会在我的名字前加上0. 怎么避免？？

```
设chunkFilename:[name].[hash].js 就好了
```

## VueJS + Webpack 代码分割的三种方式

对单页应用实行代码分割，是提高页面加载速度的一种很好的方式。因为用户不必在一次请求里加载完所有的代码，能够更快的看到页面并进行交互，这将会提升用户体验（特别是在移动端）；同时因为 Google 会给加载缓慢的网站降权，代码分割也对 SEO 有好处。

上周我写过一篇关于 用 Webpack 对 Vue.js 应用进行代码分割 的文章。简单来说，在单文件组件里引入的任何东西都能轻松的实行代码分割，因为 Webpack 能在导入模块的时候创建分割点，同时 Vue 能很方便的对一个组件进行异步加载。

我认为，实施代码分割并不难，难在搞清楚在什么时候、什么地方进行。甚至可以说，代码分割需要在程序设计的时候，有良好的应用架构。

在本文中，我将提供 Vue.js 单页应用进行代码分割的三种思路：

按页面分割

使用折叠

按条件分割

![](/assets/u=3829109644,485138480&fm=173&s=07724432118A554D40D148D70100D0B2&w=640&h=469&img.JPG)

注：原文来自 Vue.js 开发者博客 2017/07/08

1. 按页面

按页面来进行代码分割，是最明显的一种方式。比如下面的例子当中，有三个页面：

![](/assets/u=135897549,328337930&fm=173&s=2D2874320B22652008D9ADDA0000C0B2&w=640&h=294&img.JPG)

如果能确保每个单文件组件代表一个页面，如Home.vue,About.vue以及Contact.vue，那么我们就可以使用 Webpack 的 "动态导入" 函数 \(import\) 来将它们分割至单独的构建文件中。之后后，当用户访问一个新页面的时候，Webpack 将异步加载该请求的页面文件。

如果用到了 vue-router，由于页面已经分割成了单独的组件，实施起来会非常方便。

```
const Home = () => import(/* webpackChunkName: "home" */ './Home.vue'); 
const About = () => import(/* webpackChunkName: "about" */ './About.vue'); 
const Contact = () => import(/* webpackChunkName: "contact" */ './Contact.vue'); 
const routes = [ 
{ path: '/', name: 'home', component: Home }, 
{ path: '/about', name: 'about', component: About }, 
{ path: '/contact', name: 'contact', component: Contact } 
];
```

代码编译完成后，通过查看生成的统计数据得知：每个页面都有自己单独的文件，同时有多出来一个名为 build\_main.js 的打包文件。里面包含一些公共的代码以及逻辑，用来异步加载其它文件，因此它需要在用户访问路由之前加载完成。

![](/assets/u=3592158701,2392499394&fm=173&s=E6A8F74B2FA1896A4EE5660B030050C6&w=640&h=222&img.JPG)

现在，通过访问[http://localhost:8080/\#/contact](http://localhost:8080/#/contact). 这个链接，加载了 Contact 页面 。当查看浏览器的“网络”标签时，发现下面这些文件被加载了：

![](/assets/u=214741930,1063211393&fm=173&w=640&h=65&img.JPG)

注意：build\_main.js 是由 \(index\) 触发加载的，这意味着 index.html 如期的请求了这个脚本；但是，build\_1.js 的触发器却是 bootstrap\_a877… 这个文件是 Webpack 负责异步加载文件的脚本，它在你使用 Webpack “动态导入函数” 的时候就被添加进来到构建中了。关键的一点是，build\_1.js 并不会阻塞初始页面的加载。

1. 折叠之下

“折叠” 之下，是指页面初次加载时，视图的不可见部分。用户通常会花费 1~2 秒来浏览可视区域，特别是第一次访问网站的时候（可能更久），之后才开始向下滑动页面。这个时候，你可以异步加载剩余的内容。

![](/assets/u=2636185613,1954447111&fm=173&s=18843C72098A544D0AD1D8C7000080B1&w=640&h=468&img.JPG)

在下面这个应用示例当中，我考虑将折叠线放到报头下方。所以，我们在页面最开始加载的时候引入导航条和报头，之后的代码将在稍后加载。现在我创建了一个名为“BelowFold”的组件，相关标记抽象如下所示：

注意，我给模态框设置了v-if属性，绑定了show这个变量。一方面用来控制模态框是否显示，同时也决定了是否应该渲染模态框组件。当页面加载的时候，它的值为 false，模态框的代码只有当它显示的时候才会被加载。

最酷的是，如果用户永远不打开这个模态框，他就永远不必下载这部分代码！但是也有一点不好，可能会增加很小的用户体验成本：用户点击按钮后，需要等待代码文件下载完成。

我们再重新构建一次，结果如下图所示：

![](/assets/u=4193024985,1781480628&fm=173&s=E6B877CB07B1886B4CF56611030080C6&w=640&h=246&img %281%29.JPG)

大约 5KB 的文件我们不必提前加载。

结论

以上三种，就是进行代码分割的架构设计思路。我确定，还有其它一些你能想到的的实现方式。

