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



