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

## Animation Add-Ons  {#articleHeader4}

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







