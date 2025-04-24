# JavaScript Event Loop

JavaScript 是一种单线程的语言，其异步行为背后的核心机制就是 **Event Loop（事件循环）**。为了写出高性能、无阻塞的前端代码，深入理解 Event Loop 是每个前端开发者的必修课。

---

## 一、为什么需要 Event Loop？

JavaScript 是单线程运行的，这意味着在同一时刻只能执行一段代码。但现代 Web 应用中，需要处理大量异步任务（如网络请求、定时器、事件监听等），如果没有异步机制，就会造成页面卡死、响应迟钝。

于是，JS 引入了异步机制，其中最核心的调度机制就是 **Event Loop**。

---

## 二、任务队列模型概览

JavaScript 的任务被分为两类：

### 宏任务（Macro Task）
宏任务是指 JS 执行时调度到的主任务，每个宏任务都是独立的一个调度单元。

- 🧾 常见宏任务类型：

| 来源 | 说明 |
|------|------|
| `setTimeout(fn, delay)` | 最常见的异步延迟机制 |
| `setInterval(fn, interval)` | 循环定时任务 |
| `setImmediate(fn)` | Node.js 中的宏任务 |
| `MessageChannel` | 浏览器中的任务调度 API |
| 整个 script 脚本 | 页面或模块加载时整个 JS 代码块 |

### 微任务（Micro Task）
-微任务是在当前宏任务执行完成后、下一个宏任务开始前立即执行的任务。用于高优先级、精细控制的异步逻辑。

- 🧾 常见微任务类型：

| 来源 | 说明 |
|------|------|
| `Promise.then()` / `catch()` / `finally()` | Promise 的异步回调 |
| `MutationObserver` | DOM 变化监听 |
| `queueMicrotask(fn)` | 明确插入微任务队列 |
| `process.nextTick()` | Node.js 中优于微任务执行的机制（特殊微任务） |


### 执行顺序：
每一次 Event Loop 的执行步骤如下：
1. 从任务队列中取出一个**宏任务**并执行。
2. 执行过程中如遇微任务，将其加入微任务队列。
3. 当前宏任务执行结束后，立即依次执行所有微任务。
4. 清空微任务后，执行下一轮 Event Loop（下一宏任务）。

---

## 三、代码示例：任务执行顺序

```js
console.log('script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(() => {
  console.log('promise1');
}).then(() => {
  console.log('promise2');
});

console.log('script end');
```

**输出顺序：**
```
script start
script end
promise1
promise2
setTimeout
```

**解析：**
- script 本身是宏任务，立即执行。
- `setTimeout` 注册宏任务，等待下一轮事件循环。
- `Promise.then()` 是微任务，排在当前宏任务完成后执行。

---

## 四、图解 Event Loop 执行过程

```text
┌───────────────────────────────┐
│ call stack（执行栈）           │
└───────────────────────────────┘
              ↓
┌───────────────────────────────┐
│ microtask queue（微任务队列） │
└───────────────────────────────┘
              ↓
┌───────────────────────────────┐
│ macrotask queue（宏任务队列） │
└───────────────────────────────┘
```

每一轮事件循环：
1. 执行一个宏任务（包含整体 script）
2. 清空微任务队列
3. UI 更新（浏览器）
4. 进入下一轮循环

---

## 五、async / await 与微任务

```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
setTimeout(() => {
  console.log('setTimeout');
}, 0);
async1();
new Promise(resolve => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
});
console.log('script end');
```

**输出顺序：**
```
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

**解析：**
- `async1()` 执行到 `await async2()` 被挂起，后面的语句进入微任务队列。
- `Promise.then()` 同样是微任务，按注册顺序执行。

---

## 六、经典陷阱题

```js
console.log(1);
setTimeout(() => console.log(2));
Promise.resolve().then(() => console.log(3));
console.log(4);
```

**输出：**
```
1
4
3
2
```

**原因：**
- script 是宏任务，依次执行 `1 -> 4`
- `Promise.then()` 是微任务，插入当前轮次微任务队列，立即执行 `3`
- `setTimeout` 为宏任务，下一轮循环执行 `2`

---

## 七、浏览器与 Node 的差异

- **浏览器微任务先于渲染**，确保每个宏任务执行前，微任务已清空。
- **Node.js** 使用 libuv 实现事件循环，引入更多任务队列（`nextTickQueue`, `checkQueue` 等）。

Node 执行优先级：
1. `process.nextTick()`（比微任务还快）
2. 微任务（Promise）
3. `setImmediate`
4. `setTimeout` / `setInterval`

---

## 八、总结

- JS 是单线程语言，为支持异步引入 Event Loop
- 每轮事件循环包含一个宏任务 + 执行全部微任务
- `Promise.then()` / `await` 属于微任务
- 牢记：**微任务总是在当前宏任务结束后、下一个宏任务开始前执行**

---

## 九、练习题（可配合输出分析）

```js
console.log('A');
setTimeout(() => console.log('B'));
Promise.resolve().then(() => console.log('C'));
console.log('D');
```

```js
async function fn() {
  console.log('1');
  await fn2();
  console.log('2');
}
async function fn2() {
  console.log('3');
}
fn();
console.log('4');
```
---
# JavaScript Promise 

Promise 是 JavaScript 处理异步编程的一种重要方式，它使得异步操作更加清晰、可维护，广泛用于现代前端开发中。


## 一、Promise 是什么？

Promise 是一种用于表示**未来某个异步操作结果**的对象。

- 状态一旦改变就不可逆（pending -> fulfilled / rejected）
- 支持链式调用
- 是 ES6 引入的原生特性

### 三种状态：
1. **pending（进行中）**
2. **fulfilled（已成功）**
3. **rejected（已失败）**

---

## 二、Promise 基本语法

```js
const promise = new Promise((resolve, reject) => {
  // 异步操作
  if (success) {
    resolve(value);
  } else {
    reject(reason);
  }
});
```

---

## 三、then、catch、finally

### `then()`
处理成功的情况：
```js
promise.then(result => {
  console.log('成功：', result);
});
```

### `catch()`
处理失败的情况（等价于 `then(null, err => {})`）：
```js
promise.catch(error => {
  console.error('失败：', error);
});
```

### `finally()`
无论成功还是失败，都会执行：
```js
promise.finally(() => {
  console.log('总会执行');
});
```

---

## 四、链式调用

```js
Promise.resolve(1)
  .then(res => {
    console.log(res);
    return res + 1;
  })
  .then(res => {
    console.log(res);
  })
  .catch(err => {
    console.error(err);
  });
```

**特点：** 每次 `then()` 都返回一个新的 Promise 对象，可链式操作。

---

## 五、静态方法

### `Promise.resolve(value)`
返回一个状态为 fulfilled 的 Promise。

### `Promise.reject(reason)`
返回一个状态为 rejected 的 Promise。

### `Promise.all([p1, p2, ...])`
- 所有 Promise 成功 -> 返回所有结果数组
- 有一个失败 -> 立即失败

### `Promise.race([p1, p2, ...])`
- 谁先返回（无论成功失败）就返回那个结果

### `Promise.allSettled([p1, p2, ...])`
- 等所有 Promise 都完成（不管成功失败），返回每个结果的状态和值数组

---

## 六、陷阱题

### 示例 1：
```js
console.log('start');

Promise.resolve().then(() => {
  console.log('promise1');
}).then(() => {
  console.log('promise2');
});

console.log('end');
```

输出顺序：
```
start
end
promise1
promise2
```

### 示例 2：链式返回值影响
```js
Promise.resolve()
  .then(() => {
    return Promise.resolve('a');
  })
  .then(res => {
    console.log(res); // 输出 'a'
  });
```

---

## 七、手写 Promise 简化版本

```js
class MyPromise {
  constructor(fn) {
    this.state = 'pending';
    this.value = undefined;
    this.callbacks = [];

    const resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.callbacks.forEach(cb => cb(value));
      }
    };

    fn(resolve);
  }

  then(onFulfilled) {
    if (this.state === 'fulfilled') {
      onFulfilled(this.value);
    } else {
      this.callbacks.push(onFulfilled);
    }
    return this;
  }
}
```

---

## 八、Promise 与 async/await

```js
async function getData() {
  try {
    const result = await fetch('/api');
    const data = await result.json();
    console.log(data);
  } catch (e) {
    console.error('出错了', e);
  }
}
```

`await` 本质上是对 Promise 的语法糖，使得异步代码更像同步代码。

---

## 九、练习题建议

1. 写一个 `sleep(ms)` 函数：
```js
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

2. 实现带并发控制的请求池（进阶）

---

## 十、总结

- Promise 是异步编程的核心基础
- 了解其状态流转、链式调用机制
- 掌握 then/catch/finally 与常见陷阱题
- 深入了解配合 async/await 使用
---

# JavaScript async/await
`async/await` 是基于 Promise 的语法糖，用于编写更清晰、更像同步风格的异步代码。它在 ES2017（ES8）被正式引入，是现代 JS 开发中极其常用的特性。


## 一、async 函数

### 基本语法：
```js
async function foo() {
  return 'hello';
}
```

特点：
- `async` 函数返回一个 `Promise`
- 函数内部可以使用 `await`
- 如果函数显式返回非 Promise，会自动包装成一个 resolved 的 Promise

```js
foo().then(console.log); // 输出 'hello'
```

---

## 二、await 表达式

### 基本用法：
```js
async function getData() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  console.log(data);
}
```

特点：
- `await` 后面跟一个 Promise 表达式，等待其完成并返回结果
- 只能在 `async` 函数中使用

---

## 三、错误处理

```js
async function fetchData() {
  try {
    const res = await fetch('xxx');
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error('出错了：', err);
  }
}
```

等价于：
```js
fetch('xxx')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

---

## 四、并发处理

### 正确方式：
```js
async function parallel() {
  const [res1, res2] = await Promise.all([
    fetch('/api/1'),
    fetch('/api/2')
  ]);
}
```

### 错误方式（顺序等待）:
```js
const res1 = await fetch('/api/1');
const res2 = await fetch('/api/2');
```

---

## 五、与普通 Promise 的对比

### 普通 Promise：
```js
getData()
  .then(res => process(res))
  .then(final => console.log(final))
  .catch(err => console.error(err));
```

### async/await：
```js
try {
  const res = await getData();
  const final = await process(res);
  console.log(final);
} catch (err) {
  console.error(err);
}
```

更加线性、清晰、类似同步代码。

---

## 六、await 后面不是 Promise 会怎样？

```js
async function test() {
  const val = await 42;
  console.log(val); // 输出 42
}
```

解释：如果不是 Promise，会被自动包装成 `Promise.resolve(42)`。

---

## 七、for 循环中的 await

### 错误写法（串行）：
```js
async function wrong() {
  const arr = [1, 2, 3];
  for (let i of arr) {
    await delay(i);
  }
}
```

### 并发写法：
```js
async function correct() {
  await Promise.all(arr.map(i => delay(i)));
}
```

---

## 八、顶层 await（Top-level await）

在模块化环境中（`<script type="module">` 或 ES Module 文件），可以在模块顶层使用 await：

```js
const data = await fetch('/api').then(res => res.json());
console.log(data);
```

注意：必须在模块中，不能在普通 script 中直接使用。

---

## 九、练习题

1. 写一个 async 函数模拟 3 秒倒计时：
```js
async function countdown() {
  for (let i = 3; i > 0; i--) {
    console.log(i);
    await new Promise(res => setTimeout(res, 1000));
  }
  console.log('开始！');
}
```

2. 实现一个带错误重试的 fetch 请求：
```js
async function fetchWithRetry(url, retries = 3) {
  try {
    const res = await fetch(url);
    return await res.json();
  } catch (err) {
    if (retries > 0) return fetchWithRetry(url, retries - 1);
    throw err;
  }
}
```

---

## 十、总结

- async/await 是 Promise 的语法糖
- 使异步代码更接近同步结构，方便阅读和维护
- 需要注意并发、错误处理、循环 await 等细节
- 与 ES Module 配合可以实现顶层 await

建议多写练习，结合真实异步流程深入理解 async/await 的威力




