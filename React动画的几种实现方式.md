## 手动实现 {#articleHeader1}

在很久以前，React还没有`ReactCSStransitionGroup`

（ react.js 0.14 发布）的时候，我也还是一个初学者，曾经手动实现了一段动画，方法比较low，原理很简单，大家看看就好。

### 原理 {#articleHeader2}

显示：直接通过添加类的方式执行动画。

这个方式比较简单，通过props.show控制

`display none`

从而控制DOM元素的显示隐藏，因此，在组件挂载之后，当props.show发生变化后，先通过添加类的方式来执行动画。

```
componentDidUpdate(preProps) {//显示的时候执行动画
    if (this.props.show === true && preProps.show === false) {
      this.inAnimationTime = setTimeout(()=> {
        ReactDOM.findDOMNode(this).getElementsByClassName("modal")[0].classList.add("in");
      }, 50);
    }
  }
```

这里的适当的延时是为了保证动画的有效执行。

* 隐藏：先将`state.inOutAnimation`（动画执行中）设置为`true`，在`componentWillReceiveProps`里监听到nextProps.show设置为false时，先通过添加类的方式执行动画，再将`inOutAnimation`设置为false。  
  我们都知道，`display:none`会导致动画失效。所以只能在动画执行完之后，才能将display设置为none。

只有当`this.props.show`和`this.state.inOutAnimation}`都为false时，`display none`才会生效。

```
{"com-modal": true, "show": this.props.show || this.state.inOutAnimation}
```

```
componentWillReceiveProps(nextProps) {//隐藏的时候执行动画
    if (this.props.show === true && nextProps.show === false) {
      this.setState({
        inOutAnimation: true
      });
      ReactDOM.findDOMNode(this).getElementsByClassName("modal")[0].classList.remove("in");
      this.outAnimationTime = setTimeout(()=> {
        this.setState({
          inOutAnimation: false
        });
      }, 300);
    }
  }
```

## Animation Add-Ons {#articleHeader4}

[React animation官方文档](https://facebook.github.io/react/docs/animation.html)

### ReactCSStransitionGroup {#articleHeader5}

首先感谢React发布了`Animation Add-Ons`，具体包含这才`ReactCSStransitionGroup`和`ReactTransitionGroup`是React写动画的正确姿势嘛。  
先将讲简单的和实用的，那就是ReactCSStransitionGroup 。ReactCSStransitionGroup是在插件类ReactTransitionGroup这个底层API基础上进一步封装的高级API，来简单的实现基本的CSS动画和过渡。

#### 属性

* transitionName:关联CSS类  
  需要自己实现css动画实现的类。如`transitionName="up-to-down"`，那么你需要在css写以下类，分别是进入前后的状态和离开前后的状态：

```
.right-to-left-enter {
  opacity: 0;
  transform: translate(5%, 0);
}

.right-to-left-enter.right-to-left-enter-active {
  opacity: 1;
  transform: translate(0, 0);
  transition: opacity 150ms linear, transform 200ms ease-out;
}

.right-to-left-leave {
  opacity: 1;
  transform: translate(0, 0);
}

.right-to-left-leave.right-to-left-leave-active {
  opacity: 0;
  transform: translate(1%, 0);
  transition: opacity 100ms linear, transform 200ms linear;
}
```

*  transitionEnterTimeout 进入动画执行的时间
* transitionLeaveTimeout 离开的动画执行的时间

* transitionAppear 是否在初次挂载的时候执行动画如果你选择true，那么你还要加上挂载前后端的类

```
.up-to-down-appear {
  opacity: 0;
  transform: translate(0, -45%);
}

.up-to-down-appear.up-to-down-appear-active {
  opacity: 1;
  transform: translate(0, 0);
  transition: opacity 150ms linear, transform 200ms ease-out;
}
```

#### 注意

* 一定要为ReactCSSTransitionGroup的所有子级提供 key属性。即使只渲染一个项目。React靠key来决定哪一个子级进入，离开，或者停留。

* ReactCSSTransitionGroup必须已经挂载到了DOM才能工作。为了使过渡效果应用到子级上，ReactCSSTransitionGroup必须已经挂载到了DOM或者 prop transitionAppear 必须被设置为 true。ReactCSSTransitionGroup 不能随同新项目被挂载，而是新项目应该在它内部被挂载。

* 该动画只对子级元素有效，对孙子级元素无效。

### ReactTransitionGroup {#articleHeader6}

ReactTransitionGroup是底层的API，相对来说会复杂点，后面再补充吧。

## 第三方库 {#articleHeader7}

如果你觉得React提供的`Animation Add-Ons`不好用或者是不够用，那么尝试一下下面的几个React动画相关的开源项目吧。

### [react-motion](https://github.com/chenglou/react-motion) {#articleHeader8}

### [velocity-react](https://www.npmjs.com/package/velocity-react) {#articleHeader9}

### [animation](https://github.com/react-component/animate) {#articleHeader10}



