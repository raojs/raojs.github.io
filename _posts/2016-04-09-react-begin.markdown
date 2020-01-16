---
layout: post
title:  "React 入门"
date:   2016-04-09
categories: react
---

<span style="background-color: red">API 已过时，请看<a href="https://reactjs.org/docs/getting-started.html" target="_blank">官方文档</a></span>

什么是React？

[官方网站](https://facebook.github.io/react/)的第一句话：A JAVASCRIPT LIBRARY FOR BUILDING USER INTERFACES. 用于构建用户界面的Javascript库。

React为应用提供也仅仅提供视图层(UI)的解决方案，通过几个简单示例，我们可以对React有个初步的了解。示例位于代码库[__learning-react__](https://github.com/raojs/learning-react)的[__primer__](https://github.com/raojs/learning-react/tree/master/primer)下。

## 01 你好, 世界

[_在线演示_](http://raojs.github.io/learning-react/primer/01-hello-world.html)

React允许在Javascript代码中嵌入HTML标签，通过`ReactDOM.render`方法，将模板转化为HTML并插入指定的DOM节点。

```js
ReactDOM.render(
  <h1>hello, world</h1>,
  document.body
);
```

注意：
- `<script type="text/babel">`是为了兼容JSX语法（就是将HTML嵌入Javascript这种方式）。
- 引入`browser.min.js`将JSX语法转化为Javascript语法，实际上线应该在服务器完成转换以节省时间。

## 02 组件和JSX语法

[_在线演示_](http://raojs.github.io/learning-react/primer/02-component&JSX.html)

组件即代码和功能的集合，可以像使用HTML标签一样使用组件。通过`React.createClass`方法生成组件类。

下面的代码UI展现了水果组件(Fruits)，水果组件又引入了价格组件(PirceList)。

```js
var Fruits = React.createClass({
  render: function() {
    return <div>
      <h1>Fruit</h1>
      <PriceList />
    </div>;
  }
});
var PriceList = React.createClass({
  render: function() {
    ...
  }
});
ReactDOM.render(
  <Fruits />,
  document.body
)
```

JSX允许HTML与Javascript代码混写，当遇到`<`时解析为HTML，当遇到`{`时解析为Javascript。

注意：
- 组件类的第一个字母必须大写。
- 所有组件必须有render方法。
- 组件return只能包含一个顶层标签。
- 组件return与标签左尖括号必须在同一行，为了代码美观可以用圆括号包一层。
- JSX输出Javascript变量时，如果变量是数组，会自动展开。

## 03 props和state

[_在线演示_](http://raojs.github.io/learning-react/primer/03-props&state.html)

组件的UI展现由两个数据决定，初始化后不变的数据`this.props`和随着用户交互改变的数据`this.state`。`getDefaultProps`方法和`getInitialState`方法分别用来初始化默认的`props`和`state`。

下面的代码中，标语(banner)是组件初始化之后不应该再改变的内容，而水果价格的隐藏与否是随用户的交互而变化的。`setState`方法修改状态值，然后自动调用render方法重新渲染UI。

```js
getDefaultProps: function() {
  return {banner: 'Default banner: Welcome'};
},
getInitialState: function() {
  return {hide: false};
},
onClick: function(e) {
  this.setState({hide: !this.state.hide});
},
render: function() {
  var fruit = {name: 'apple', price: 6888};
  return (
    <div>
      <h1>{this.props.banner}</h1>
      <p>{fruit.name} {this.state.hide ? '--' : '￥' + fruit.price}</p>
      <button onClick={this.onClick}>{this.state.hide ? 'show price' : 'hide price'}</button>
    </div>
  )
}
```

知道组件的state，就知道组件渲染出来的样子，不用再一步步跟踪程序流程，这在团队开发中是非常重要和具有优势的。

注意：
- 直接改变`this.state`并不会重新渲染。(类似这样: `this.state.hide = true`)
- 传入的`props`会覆盖默认的`props`。

## 04 组件的生命周期

[_在线演示_](http://raojs.github.io/learning-react/primer/04-lifecycle.html)

组件的状态分为挂载(Mount)、更新(Update)和移除(Unmount)，相应的有以下五个方法：

__componentWillMount()__

仅调用一次，在初始化渲染之前执行，此时调用`setState`方法并不会引起二次渲染。

__componentDidMount()__

仅调用一次，在初始化渲染之后执行，可以在这个阶段设置定时器或发送AJAX请求。可以通过`this.refs`获取到真实的DOM节点。

__componentWillReceiveProps(object nextProps)__

接收到新的props时调用，此时调用`setState`不会引起二次渲染。

__shouldComponentUpdate(object nextProps, object nextState)__

在接收到新的props或state时执行，决定是否需要更新组件，默认返回true。如果返回false，`componentWillUpdate`、`componentDidUpdate`和`render`都不会执行。初始化渲染和执行`forceUpdate`时，该方法不会调用。

__componentWillUpdate(object nextProps, object nextState)__

当获取到新的props或state的时候执行，初始化渲染的时候不执行。`nextProps`和`nextState`是新获取的数据，此时可以通过`this.props`和`this.state`获取到老的值。

__componentDidUpdate(object prevProps, object prevState)__

更新已经同步到DOM后执行，初始化渲染的时候不执行。

__componentWillUnmount()__

组件从DOM移除时执行，可以在这个阶段执行必要的清理，比如无效的定时器。

```js
var PriceList = React.createClass({
  componentWillMount: function() {
    this.setState({name: 'apple', price: 6888});
  },
  componentDidMount: function() {
    this.timer = setInterval(this.cheap, 1000);
  },
  componentWillUnmount: function() {
    clearInterval(this.timer);
  },
  cheap: function() {
    this.setState({price: this.state.price - 100});
  },
  render: function() {
    return <h3>{this.state.name} {this.state.price}</h3>;
  }
});
ReactDOM.render(
  <PriceList />,
  document.body
);
```

更多生命周期相关方法参见[官方文档](http://facebook.github.io/react/docs/component-specs.html#lifecycle-methods)

注意：
- 不要在`componentWillUpdate`方法中调用`setState`，可能会造成无限循环。

## 05 表单

[_在线演示_](http://raojs.github.io/learning-react/primer/05-forms.html)

在React中，涉及用户交互的表单组件(input、checkbox等)，如果设置了默认值(value、checked等)，那么它是一个受限组件，受限组件将始终渲染它的默认值。

下面代码中input就是一个受限组件，用户输入无法改变它的渲染。

```js
<input type="text" value="can not change me" />
```

有两种方式可以改变这一现状：

1. 使用特有属性

  对于input、select使用`defaultValue`属性，对于checkbox、radio使用`defaultChecked`属性。

  ```js
  <input type="text" defaultValue="change me" />
  ```

1. 使用事件监听

  在表单组件中监听值的改变，然后更新表单当前值。

  ```js
  handleChange: function(event) {
    this.setState({text: event.target.value});
  )

  <input type="text" value={this.state.text} onChange={this.handleChange} />
  ```

详细请查看[表单事件](http://facebook.github.io/react/docs/events.html#form-events)
