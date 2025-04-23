
# JavaScript基础知识

## 变量声明
```javascript
// ES6+
const immutable = '不可变';
let mutable = '可变';

// 旧版
var oldWay = '旧方式';
```

## 函数
```javascript
// 函数声明
function sayHello(name) {
    return `Hello, ${name}!`;
}

// 箭头函数
const add = (a, b) => a + b;
```

## 异步编程
```javascript
// Promise
fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));

// Async/Await
async function getData() {
    try {
        const response = await fetch('/api/data');
        return await response.json();
    } catch (error) {
        console.error(error);
    }
}
