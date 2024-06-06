## js的this指向



在 JavaScript 中，`this` 关键字的指向是在函数被调用时确定的，而不是在函数被定义时确定的。`this` 的指向主要取决于函数的调用方式。以下是一些常见的情况：

1. **全局上下文**：在全局上下文（非函数内部）中，`this` 指向全局对象。在浏览器中，全局对象是 `window`。

```javascript
console.log(this); // window
```

2. **函数调用**：当一个函数被直接调用（不是作为对象的方法调用），`this` 通常指向全局对象。但在严格模式下，`this` 会是 `undefined`。

```javascript
function myFunction() {
  console.log(this);
}

myFunction(); // window in non-strict mode, undefined in strict mode
```

3. **方法调用**：当一个函数作为对象的方法被调用时，`this` 指向调用该方法的对象。

```javascript
const myObject = {
  myMethod() {
    console.log(this);
  }
};

myObject.myMethod(); // myObject
```

4. **构造函数**：当一个函数被作为构造函数使用（通过 `new` 关键字）时，`this` 指向新创建的对象。

```javascript
function MyConstructor() {
  console.log(this);
}

new MyConstructor(); // a new MyConstructor object
```

5. **事件处理器**：在事件处理器中，`this` 通常指向触发事件的元素。

```javascript
button.addEventListener('click', function() {
  console.log(this); // button
});
```

6. **箭头函数**：箭头函数没有自己的 `this`，它的 `this` 是继承自包含它的函数或全局作用域的 `this`。

```javascript
const myObject = {
  myMethod() {
    const arrowFunction = () => {
      console.log(this);
    };
    arrowFunction();
  }
};

myObject.myMethod(); // myObject
```

请注意，你可以使用 `call`，`apply` 或 `bind` 方法改变函数调用时 `this` 的指向。