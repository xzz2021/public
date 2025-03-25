### setState 执行机制

思考: setState 到底是异步还是同步? 答案: 都是!

> 在 React 的生命周期和合成事件处理器之内: 异步! 为什么要这样设计?

1. setState 会将状态更新推入一个队列,多个调用会被合并成一次 render,提升性能;
2. 保持 state 和 props 的一致性; -- 如果是同步的,当 state 中有数据传递给子组件,state 变化,立即 render,子组件接收的 state 还是上一次的

> 什么时候是同步执行? React 的生命周期和合成事件处理器之外!

1. 使用 setTimeout 或 setInterval 函数

```js
setTimeout(() => {
  this.setState({ aa: 55 });
  console.log("🚀 ~ xzz: aa", this.state.aa);
}, 0);
```

2. 原生事件处理器中调用 setState 时，会同步更新状态; (dom 添加 addEventListener)

```js
 componentDidMount() {
    document.getElementById('myButton').addEventListener('click', this.handleClick);
  }
  handleClick = () => {
    this.setState({ aa: 2 });
    console.log(this.state.aa); // 输出: 2
  };
```

3. Promise 回调函数中调用, 或者异步函数中的 await 之后调用

```js
 componentDidMount() {
    Promise.resolve().then(() => {
      this.setState({ aa: 2 });
      console.log(this.state.aa); // 输出: 2
    });
  }

 async componentDidMount() {
    await new Promise(resolve => setTimeout(resolve, 0));
    this.setState({ aa: 2 });
    console.log(this.state.aa); // 输出: 2
  }
```

4. 在一些外部库（例如 jQuery、RxJS 等）的回调函数中调用

```js
componentDidMount() {
    $(document).on('click', '#myButton', () => {
      this.setState({ aa: 2 });
      console.log(this.state.aa); // 输出: 2
    });
  }
```

示例

```js
   componentDidMount() {
    this.setState({ aa: 1 }, () => {
      console.log(this.state.aa); // 输出: 1, 因为这是在 setState 的回调中
    });
    console.log(this.state.aa); // 输出: 0, 因为 setState 是异步的
    setTimeout(() => {
      console.log(this.state.aa); // 输出: 1, 因为此时 state 已经被更新
      this.setState({ aa: 2 }, () => {
        console.log(this.state.aa); // 输出: 2, 因为这是在 setState 的回调中
      });
    }, 0);
  }
  // 为什么最后log输出2?
  // 在 setTimeout 回调函数内调用 setState 会立即生效，
  // 这是因为此时 setState 不是在 React 的生命周期方法或合成事件中调用的，
  // 而是处于原生事件中。
```

tips: 合成事件处理器: react 中 onClick 绑定的事件不是 dom 原生的,而是经过对原生点击事件加工合成的
