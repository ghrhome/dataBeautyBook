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

> View:视图层
>
> **Action**（动作）:视图层发出的消息（比如mouseClick）
>
> **Dispatcher**（派发器）:用来接收Actions、执行回调函数



