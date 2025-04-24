
## 闭包：
**闭包是函数 + 其定义时的词法作用域的组合。**

通俗点说：  
> 当一个**函数在它的作用域外被调用**，但它**仍然“记住”了当初创建它时的作用域里的变量**，这就是闭包。

## 为什么会有闭包？
因为 JS 中函数是“一等公民”，可以作为值传递、返回、赋值，  
而 JS 使用**词法作用域（Lexical Scope）**，函数定义的时候就“记住”了周围的环境。

## 代码示例（超经典形式）：

```javascript
function createCounter() {
  let count = 0; // 外部变量

  return function() {
    count++;      // 内部函数可以访问外部的 count
    console.log(count);
  };
}

const counter = createCounter();
counter(); // 1
counter(); // 2
counter(); // 3
```

### 解释：

- `createCounter` 执行完后，它的 `count` 理论上应该销毁。
- 但 `return function()` 引用了 `count`，JS 引擎就把 `count` 和它一起“打包”存了下来。
- 所以每次调用 `counter()`，它都能“记得”上一次的 `count` 值，这就是闭包。


## 实际应用场景：

### 1. **做私有变量（封装）**

```javascript
function createPerson(name) {
  return {
    getName: function() {
      return name;  // 通过闭包读取私有变量
    },
    setName: function(newName) {
      name = newName;
    }
  };
}

const p = createPerson("Alice");
console.log(p.getName()); // Alice
p.setName("Bob");
console.log(p.getName()); // Bob
```

### 2. **循环中绑定变量（for 闭包经典问题）**

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);  // 全都是 3，因为 i 被共享
  }, 1000);
}
```

改成闭包形式：

```javascript
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);  // 0, 1, 2
    }, 1000);
  })(i);
}
```
or
```javascript
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
      console.log(j);  // 0, 1, 2 let块级作用域
    }, 1000);
}
```

## 其他闭包demo

### 🌰 模拟实现 `once` 函数 —— 只执行一次的函数

```javascript
function once(fn) {
  // 补全
}

const runOnce = once(() => console.log('Run'));
runOnce(); // Run
runOnce(); // 什么都不输出
```

### 答案：

```javascript
function once(fn) {
  let hasRun = false;

  return function() {
    if (!hasRun) {
      hasRun = true;
      return fn();
    }
  };
}
```

**用途**：加载动画、按钮防止重复点击等。

## 闭包与内存泄漏

### 原理：

闭包会引用其“外部作用域”的变量，如果这些变量永远不会被释放，就可能造成 **内存泄漏**。

### 例子：

```javascript
function leaky() {
  let largeData = new Array(1000000).fill('*'); // 模拟大内存变量

  return function() {
    console.log('I’m still here');
  };
}

const hold = leaky(); // largeData 永远不会释放
```

即使我们不再需要 `largeData`，因为闭包引用了它所在作用域，垃圾回收器无法清理它。

### 避免方式：

- 不滥用闭包
- 不在全局或长生命周期对象中保存闭包
- 用完及时置 `null` 或 `undefined`


## 闭包与垃圾回收

### JS 的垃圾回收机制（GC）

JS 引擎采用**标记清除（mark-and-sweep）**策略：

- 所有从根（如全局对象、作用域链）可达的对象都不会被回收；
- 所有“不可达”的对象，会被 GC 回收。

### 那闭包怎么影响 GC？

闭包中引用的变量，**因为“仍然可达”，不会被 GC 回收**。

### 示例：

```javascript
function outer() {
  let name = 'Alice';

  return function inner() {
    console.log(name);
  };
}

const fn = outer(); // name 无法被回收
```

在这里，`name` 是可达的，因为 `fn` 引用了它所在作用域。


### 实践建议：

- ✅ 小范围使用闭包，不在全局对象或大型组件上挂闭包引用
- ✅ 不保留无用的 DOM 引用（尤其是老浏览器下）
- ✅ 使用 `WeakMap` / `WeakRef` 等工具，减少内存压力（如果你用的是现代浏览器/Node）


## 总结：

| 话题 | 核心点 |
|------|--------|
| 手写闭包题 | 模拟“状态记忆”功能，比如计数器、once、缓存等 |
| 内存泄漏 | 闭包引用外部变量，生命周期过长，导致变量无法释放 |
| 垃圾回收 | 引用链不断，变量仍可达，就不会被回收；闭包常见问题点 |


## 作用域

### 1. 什么是作用域？
- **定义**：作用域是程序中定义变量的区域，决定了变量的**可访问性**。
- **分类**：
  - ✅ 全局作用域
  - ✅ 函数作用域（Function Scope）
  - ✅ 块级作用域（Block Scope） ← ES6 新增


## 作用域类型详解

### 2. 全局作用域
- 在任何函数体外声明的变量、函数、对象都在全局作用域中。
- 全局变量挂载在 `window`（浏览器）或 `global`（Node）对象上。

```js
var a = 1;
console.log(window.a); // 1
```

### 3. 函数作用域
- 函数内部声明的变量只能在函数内部访问。
- 使用 `var` 声明的变量不具备块级作用域。

```js
function foo() {
  var x = 10;
}
console.log(x); // ReferenceError
```

### 4. 块级作用域（ES6）
- 由 `{}` 包裹的代码块，如 `if`、`for`、`while`、函数体内等。
- `let` 和 `const` 具有块级作用域。

```js
{
  let a = 2;
}
console.log(a); // ReferenceError
```

## 作用域链（Scope Chain）

### 5. 原理
- 当变量在当前作用域找不到时，JS 引擎会沿着作用域链向上查找。
- 作用域链是由当前执行上下文的变量对象及其所有父级作用域组成的链表结构。

```js
function outer() {
  let a = 1;
  function inner() {
    console.log(a); // 查找到 outer 中的 a
  }
  inner();
}
```

## 四、变量提升（Hoisting）

### 6. `var` 的变量提升
- `var` 声明的变量会被提升到作用域顶部，但初始化不会提升。
```js
console.log(a); // undefined
var a = 5;
```

### 7. 函数声明 vs 函数表达式
- 函数声明也会被提升，但函数表达式不会。
```js
foo(); // ok
function foo() {}

bar(); // TypeError
var bar = function() {};
```

## 闭包与作用域

### 8. 闭包基础
- 闭包本质：函数+其作用域的组合。
- 函数可以「记住」定义时的作用域，即使在其作用域外被调用。

```js
function outer() {
  let count = 0;
  return function() {
    return ++count;
  }
}
const inc = outer();
inc(); // 1
```


## 变量声明关键字对作用域的影响

| 关键字 | 是否提升 | 块级作用域 | 可重复声明 | 特性 |
|--------|-----------|--------------|--------------|------|
| var    | 是        | 否           | 是           | 函数作用域 |
| let    | 否        | 是           | 否           | 暂时性死区 |
| const  | 否        | 是           | 否           | 不可重新赋值 |


## 🛠 七、作用域常见面试题

### 9. for 循环与闭包结合

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// 输出：3, 3, 3（因为 i 是 var 声明，共享作用域）

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// 输出：0, 1, 2（因为 let 是块级作用域）
```

## 作用域相关易错点与陷阱

### 10. 临时性死区（Temporal Dead Zone）
- 使用 `let` 和 `const` 声明的变量在声明前访问会报错。
```js
console.log(x); // ReferenceError
let x = 10;
```

### 11. 函数参数作用域
```js
function test(x) {
  console.log(x); // 1
  var x = 2;
  console.log(x); // 2
}
test(1);
```

## `this` 

- `this` 是 JS 中的一个**关键字**，表示**当前执行上下文的拥有者**。
- 它的值**不是在定义时决定的**，而是在**运行时根据函数的调用方式来决定的**。

## `this` 的绑定规则（5大规则）

### 1. 默认绑定（普通函数调用）
```js
function show() {
  console.log(this);
}
show(); // 浏览器中：window（严格模式下为 undefined）
```

### 2. 隐式绑定（作为对象的方法调用）
```js
const obj = {
  name: 'JS',
  say() {
    console.log(this.name);
  }
};
obj.say(); // JS（this 指向 obj）
```

📌 注意：**隐式丢失**（常见坑）：
```js
const fn = obj.say;
fn(); // undefined（默认绑定）
```

### 3. 显式绑定（call / apply / bind）
```js
function greet() {
  console.log(this.name);
}
const person = { name: 'Alice' };
greet.call(person); // Alice
greet.apply(person); // Alice
greet.bind(person)(); // Alice
```

### 4. new 绑定（构造函数）
```js
function Person(name) {
  this.name = name;
}
const p = new Person('Tom');
console.log(p.name); // Tom
```

### 5. 箭头函数绑定（词法作用域绑定）
- 箭头函数**不绑定自己的 this**，它的 this 来自**定义时的外层作用域**。

```js
const obj = {
  name: 'Arrow',
  say() {
    const inner = () => console.log(this.name);
    inner();
  }
};
obj.say(); // Arrow
```

## `this` 的优先级（面试重点）

| 优先级 | 调用方式 | `this` 指向 |
|--------|-----------|--------------|
| 1️⃣ new 绑定        | `new Foo()`     | 新创建的对象 |
| 2️⃣ 显式绑定        | `call/apply/bind` | 指定对象 |
| 3️⃣ 隐式绑定        | `obj.fn()`       | 调用对象 |
| 4️⃣ 默认绑定        | `fn()`           | `window` 或 `undefined` |

## 易错

### 1. setTimeout & this
```js
const obj = {
  name: 'timeout',
  show() {
    setTimeout(function () {
      console.log(this.name);
    }, 100);
  }
};
obj.show(); // undefined（默认绑定）

// 修复：
setTimeout(() => console.log(this.name), 100); // 箭头函数继承外层 this
```

### 2. bind 返回的是函数
```js
function say() {
  console.log(this.name);
}
const bound = say.bind({ name: 'Bind' });
bound(); // Bind
```

### 3. 方法丢失的陷阱
```js
const obj = {
  name: 'Oops',
  getName() {
    return this.name;
  }
};

const fn = obj.getName;
fn(); // undefined，丢失 this 绑定
```

## this 相关面试题（经典）

### 题 1：
```js
var name = 'Global';
const obj = {
  name: 'Local',
  foo: function () {
    return function () {
      console.log(this.name);
    }
  }
}
obj.foo()(); // Global（默认绑定）
```

### 题 2：
```js
const obj = {
  name: 'Test',
  foo() {
    setTimeout(function () {
      console.log(this.name);
    }, 0);
  }
}
obj.foo(); // undefined（默认绑定）

// 改写成箭头函数就可以打印 Test
```

## 总结

> `this` 是运行时绑定的，取决于函数的调用方式。优先级依次为：new > 显式 > 隐式 > 默认。箭头函数不绑定自己的 this，而是继承外层作用域。常见陷阱包括方法丢失、定时器回调、事件监听中的 this 处理等。

## 需要掌握的相关方法

- `call`, `apply`, `bind` 的区别
- 箭头函数与普通函数的 `this` 区别
- `this` 在类（class）和事件处理器中的表现

## 执行上下文

> **执行上下文就是 JavaScript 代码被解释执行时的运行环境。**

每当一段 JS 代码执行时，它都会在一个“执行上下文”中运行，这个环境会提供变量、函数、`this` 的值等一整套“上下文信息”。

## 执行上下文的类型

JS 中一共有三种执行上下文：

| 类型 | 举例 | 描述 |
|------|------|------|
| 全局执行上下文 | 整个 script | 程序开始执行时的默认上下文，全局 `this` 就是 `window`（浏览器） |
| 函数执行上下文 | 函数调用时 | 每次函数被调用时就会创建一个新的上下文 |
| `eval()` 执行上下文 | `eval("var a = 1")` | 用得很少，几乎被忽视，不推荐使用 |


## 执行上下文的生命周期（3个阶段）

### 1. 创建阶段（Creation Phase）

做了以下几件事：
- **创建作用域链（Scope Chain）**
- **变量对象（Variable Object）初始化**
  - `var` 声明的变量：**提升为 undefined**
  - `function` 声明：**整体提升**
- **`this` 绑定**

### 2. 执行阶段（Execution Phase）

- 真正执行代码
- 变量赋值、函数调用等在这个阶段发生

## 执行上下文栈（Execution Stack）

- JS 是**单线程**的，所有执行上下文以 **“栈”** 的方式管理。
- 每次执行新代码（如调用函数），会创建一个新的上下文压入栈顶。
- 当前正在运行的上下文永远在**栈顶**。

```js
function a() {
  function b() {
    console.log("B");
  }
  b();
}

a();
```

执行上下文栈变化：
1. 全局上下文（global）入栈
2. 函数 `a` 执行，`a` 上下文入栈
3. `a` 中调用 `b()`，`b` 上下文入栈
4. `b` 执行完毕，出栈
5. `a` 执行完毕，出栈
6. 全局上下文保持直到页面关闭

## 变量对象（Variable Object）详解（面试点）

每个上下文内部维护一个“变量对象”，它包含：
- 函数参数
- `var` 变量（初始化为 undefined）
- 函数声明（整体提升）

## 结合示例讲理解

```js
var a = 1;
function foo() {
  console.log(a); // undefined
  var a = 2;
}
foo();
```

解释流程：
1. 创建上下文 → `var a` 被提升 → 初始化为 `undefined`
2. 执行阶段 → 覆盖为 `2`，但 `console.log(a)` 是在赋值前执行的 → 输出 `undefined`

## 🔥 七、执行上下文 vs 作用域 vs 闭包 的区别

| 概念 | 概述 |
|------|------|
| 执行上下文 | JS 运行代码时的**环境** |
| 作用域 | 定义变量时决定的**可访问范围** |
| 闭包 | 函数+其定义时的作用域（即“记住”上下文） |


## 总结

> 执行上下文是 JS 中代码运行的环境，它包含变量、作用域链、`this` 等信息。JS 会以“执行上下文栈”方式管理多个上下文，支持函数嵌套调用。理解上下文生命周期（创建 → 执行）有助于搞清楚变量提升、作用域链和闭包等核心机制。