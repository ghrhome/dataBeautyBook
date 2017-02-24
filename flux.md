-----From [阮一峰](http://www.ruanyifeng.com/)

React 本身只涉及UI层，如果搭建大型应用，必须搭配一个前端框架。也就是说，你至少要学两样东西，才能基本满足需要：React + 前端框架。

Facebook官方使用的是[Flux 框架](https://facebook.github.io/flux/)。本文就介绍如何在 React 的基础上，使用 Flux 组织代码和安排内部逻辑，使得你的应用更易于开发和维护。

## 一、Flux 是什么？

简单说，Flux 是一种架构思想，专门解决软件的结构问题。它跟[MVC 架构](http://www.ruanyifeng.com/blog/2007/11/mvc.html)是同一类东西，但是更加[简单和清晰](http://www.infoq.com/news/2014/05/facebook-mvc-flux)。

Flux存在多种实现（[至少15种](https://github.com/voronianski/flux-comparison)），本文采用的是[Facebook官方实现](https://github.com/facebook/flux)。

## 二、安装 Demo

为了便于讲解，我写了一个[Demo](https://github.com/ruanyf/extremely-simple-flux-demo)。

请先安装一下。

> ```
> $ git clone https://github.com/ruanyf/extremely-simple-flux-demo.git
> $ cd extremely-simple-flux-demo && npm install
> $ npm start
> ```

然后，访问 [http://127.0.0.1:8080](http://127.0.0.1:8080) 。

![](/assets/bg2016011502.png)

你会看到一个按钮。这就是我们的Demo。

## 三、基本概念

讲解代码之前，你需要知道一些 Flux 的基本概念。

首先，Flux将一个应用分成四个部分。

> **View**:视图层
>
> **Action**（动作）:视图层发出的消息（比如mouseClick）
>
> **Dispatcher**（派发器）:用来接收Actions、执行回调函数
>
> **Store**（数据层）:用来存放应用的状态，一旦发生变动，就提醒Views要更新页面

![](/assets/bg2016011503.png)

Flux 的最大特点，就是数据的"单向流动"。

> 1. 用户访问 View
> 2. View 发出用户的 Action
> 3. Dispatcher 收到 Action，要求 Store 进行相应的更新
> 4. Store 更新后，发出一个"change"事件
> 5. View 收到"change"事件后，更新页面

上面过程中，数据总是"单向流动"，任何相邻的部分都不会发生数据的"双向流动"。这保证了流程的清晰。

读到这里，你可能感到一头雾水，OK，这是正常的。接下来，我会详细讲解每一步。

## 四、View（第一部分）

请打开 Demo 的首页[`index.jsx`](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/index.jsx)，你会看到只加载了一个组件。

```
// index.jsx
var React = require('react');
var ReactDOM = require('react-dom');
var MyButtonController = require('./components/MyButtonController');

ReactDOM.render(
  <MyButtonController/>,
  document.querySelector('#example')
);
```

上面代码中，你可能注意到了，组件的名字不是`MyButton`，而是`MyButtonController`。这是为什么？

这里，我采用的是 React 的[controller view](http://blog.andrewray.me/the-reactjs-controller-view-pattern/)模式。"controller view"组件只用来保存状态，然后将其转发给子组件。`MyButtonController`的[源码](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/components/MyButtonController.jsx)很简单。

```
// components/MyButtonController.jsx
var React = require('react');
var ButtonActions = require('../actions/ButtonActions');
var MyButton = require('./MyButton');

var MyButtonController = React.createClass({
  createNewItem: function (event) {
    ButtonActions.addNewItem('new item');
  },

  render: function() {
    return <MyButton
      onClick={this.createNewItem}
    />;
  }
});

module.exports = MyButtonController;
```

上面代码中，

`MyButtonController`将参数传给子组件`MyButton`。后者的[源码](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/components/MyButton.jsx)甚至更简单。

```
// components/MyButton.jsx
var React = require('react');

var MyButton = function(props) {
  return <div>
    <button onClick={props.onClick}>New Item</button>
  </div>;
};

module.exports = MyButton;
```

上面代码中，你可以看到[`MyButton`](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/components/MyButton.jsx)是一个纯组件（即不含有任何状态），从而方便了测试和复用。这就是"controll view"模式的最大优点。

`MyButton`只有一个逻辑，就是一旦用户点击，就调用[`this.createNewItem`](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/components/MyButtonController.jsx#L27)方法，向Dispatcher发出一个Action。

```
// components/MyButtonController.jsx

  // ...
  createNewItem: function (event) {
    ButtonActions.addNewItem('new item');
  }
```

上面代码中，调用`createNewItem`方法，会触发名为`addNewItem`的Action。

## 五、Action

每个Action都是一个对象，包含一个`actionType`属性（说明动作的类型）和一些其他属性（用来传递数据）。

在这个Demo里面，[`ButtonActions`](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/actions/ButtonActions.js)对象用于存放所有的Action。

```
// actions/ButtonActions.js
var AppDispatcher = require('../dispatcher/AppDispatcher');

var ButtonActions = {
  addNewItem: function (text) {
    AppDispatcher.dispatch({
      actionType: 'ADD_NEW_ITEM',
      text: text
    });
  },
};
```

上面代码中，`ButtonActions.addNewItem`方法使用`AppDispatcher`，把动作`ADD_NEW_ITEM`派发到Store。



## 六、Dispatcher

Dispatcher 的作用是将 Action 派发到 Store、。你可以把它看作一个路由器，负责在 View 和 Store 之间，建立 Action 的正确传递路线。注意，Dispatcher 只能有一个，而且是全局的。

Facebook官方的[Dispatcher 实现](https://github.com/facebook/flux)输出一个类，你要写一个[`AppDispatcher.js`](https://github.com/ruanyf/extremely-simple-flux-demo/blob/master/dispatcher/AppDispatcher.js)，生成 Dispatcher 实例。

```
// dispatcher/AppDispatcher.js
var Dispatcher = require('flux').Dispatcher;
module.exports = new Dispatcher();
```

`AppDispatcher.register()`

方法用来登记各种Action的回调函数。

