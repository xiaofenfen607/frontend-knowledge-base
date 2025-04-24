
## 什么是原型（prototype）

### 1. 每个 JS 对象都有一个内部属性 `[[Prototype]]`  
- 可以通过 `. __proto__` 访问（非标准，但浏览器支持）
- 通过 `Object.getPrototypeOf(obj)` 获取
- 构造函数的实例对象会自动拥有原型对象上的属性和方法

---

### 2. 构造函数的原型（`Function.prototype`）

```js
function Person() {}
console.log(Person.prototype); // => 构造函数的原型对象
```

---

## 原型链（Prototype Chain）

> 原型链是 JS 实现**继承机制**的一种方式。

- 每个对象都可以通过其 `__proto__` 指向其构造函数的 `prototype`
- 如果访问一个对象的属性，它会先在自身查找，如果没有，就会沿着原型链往上查找，直到 `null`

🔗 典型链条图解：

```txt
实例对象 --> 构造函数.prototype --> Object.prototype --> null
```

```js
function Person(name) {
  this.name = name;
}
const p = new Person("Tom");

console.log(p.__proto__ === Person.prototype); // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null
```

---

## 构造函数、原型对象、实例三者关系

| 实体 | 含义 | 作用 |
|------|------|------|
| 构造函数（Function） | 创建对象的模板 | 拥有 `prototype` 属性 |
| 原型对象（prototype） | 实例共享属性存储位置 | 被所有实例通过 `__proto__` 访问 |
| 实例对象 | 通过 `new` 构造出的对象 | 访问属性时先查自己，查不到就沿原型链走 |

---

## 常见原型链结构图

```js
function A() {}
const a = new A();
```

🔗 结构如下：

```txt
a
↓ __proto__
A.prototype
↓ __proto__
Object.prototype
↓ __proto__
null
```

---

## 原型继承的几种方式（重点）

### 1. 原型链继承

```js
function Parent() {
  this.name = "Parent";
}
function Child() {}
Child.prototype = new Parent();
```

缺点：
- 引用类型会被多个子实例共享
- 无法向父构造函数传参

---

### 2. 构造函数继承（借用构造函数）

```js
function Parent(name) {
  this.name = name;
}
function Child(name) {
  Parent.call(this, name);
}
```

缺点：
- 只能继承实例属性/方法，不能继承原型上的方法

---

### 3. 组合继承（经典）

```js
function Parent(name) {
  this.name = name;
}
Parent.prototype.sayHi = function () {}

function Child(name) {
  Parent.call(this, name); // 第一次调用
}
Child.prototype = new Parent(); // 第二次调用
Child.prototype.constructor = Child;
```

---

### 4. 寄生组合继承（最推荐）

```js
function Parent(name) {
  this.name = name;
}
Parent.prototype.sayHi = function () {}

function Child(name) {
  Parent.call(this, name);
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
```

---

## Object.create() 的作用

```js
const parent = { name: 'parent' };
const child = Object.create(parent);
console.log(child.name); // parent
```

👉 `Object.create(proto)` 创建一个新对象，`__proto__` 指向传入的对象。

---

## 常见问题

### 1. instanceof 的原理？
> 判断构造函数的 prototype 是否在对象的原型链上

```js
function instanceOf(obj, Fn) {
  let proto = Object.getPrototypeOf(obj);
  while (proto) {
    if (proto === Fn.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

---

### 2. 手写 new 操作符

```js
function myNew(constructor, ...args) {
  const obj = Object.create(constructor.prototype);
  const result = constructor.apply(obj, args);
  return typeof result === 'object' && result !== null ? result : obj;
}
```

---

## 总结

> JS 是通过原型链实现继承的。每个对象都有 `__proto__`，指向其构造函数的 `prototype`。通过原型链查找属性/方法。推荐使用寄生组合继承避免多次调用父构造函数。

---

## 为什么需要继承？

在 JavaScript 中，继承用于：
- **代码复用**
- **子类对象可以访问父类对象的方法和属性**
- **面向对象思想中的“扩展”能力**

---

## JS 中的继承基础：原型链继承

### 原理：
子类的 `prototype` 指向父类的实例，从而可以访问父类原型中的属性和方法。

### 示例：

```js
function Parent() {
  this.name = 'Parent';
  this.colors = ['red', 'green'];
}

Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child() {}

Child.prototype = new Parent();

const child1 = new Child();
child1.colors.push('blue');
console.log(child1.colors); // ['red', 'green', 'blue']

const child2 = new Child();
console.log(child2.colors); // ['red', 'green', 'blue'] 引用类型共享问题
```

### ⚠️ 缺点：
- 引用类型被所有实例共享
- 无法向父类构造函数传参

---

## 借用构造函数继承（也叫伪经典继承）

### 原理：
在子类构造函数中调用父类构造函数。

### 示例：

```js
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'green'];
}

function Child(name) {
  Parent.call(this, name); // 继承属性
}

const c1 = new Child('Tom');
const c2 = new Child('Jerry');

c1.colors.push('blue');
console.log(c1.colors); // ['red', 'green', 'blue']
console.log(c2.colors); // ['red', 'green']
```

### 优点：
- 解决了引用类型共享问题
- 可以传参

### ⚠️ 缺点：
- 无法继承父类原型上的方法

---

## 组合继承（经典继承）

### 原理：
原型链 + 借用构造函数结合使用

### 示例：

```js
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'green'];
}
Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child(name, age) {
  Parent.call(this, name); // 第一次调用
  this.age = age;
}
Child.prototype = new Parent(); // 第二次调用
Child.prototype.constructor = Child;

const c = new Child('Tom', 18);
c.sayName(); // 'Tom'
```

### ⚠️ 缺点：
- `Parent` 构造函数被调用了两次，浪费资源

---

## 寄生组合继承（最佳实践）

### 原理：
使用 `Object.create()` 实现原型继承，避免多次调用父构造函数。

### 示例：

```js
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'green'];
}
Parent.prototype.sayName = function () {
  console.log(this.name);
};

function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}

// 核心继承方式
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

const c1 = new Child('Tom', 18);
c1.sayName(); // Tom
```

### 优点：
- 父类构造函数只调用一次
- 原型链继承 + 构造函数继承
- 是 ES5 中推荐的继承方式

---

## ES6 的 class 继承（现代写法）

```js
class Parent {
  constructor(name) {
    this.name = name;
  }
  sayName() {
    console.log(this.name);
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name); // 调用父类构造器
    this.age = age;
  }
}

const c = new Child('Tom', 18);
c.sayName(); // Tom
```

### 优点：
- 更接近传统 OOP 写法
- 更清晰简洁
- `super()` 必须先调用，否则 `this` 无法使用

---

## 原型链与继承的关系总结

| 继承方式 | 是否共享引用属性 | 是否继承原型方法 | 可否传参 | 性能 | 推荐 |
|----------|------------------|------------------|----------|-------|--------|
| 原型链继承 | 是 | 是 | 否 | 一般 | ❌ |
| 构造函数继承 | 否 | 否 | 是 | 好 | ❌ |
| 组合继承 | 否 | 是 | 是 | 有冗余 | ✅ |
| 寄生组合继承 | 否 | 是 | 是 | 最优 | ✅✅ |
| ES6 class 继承 | 否 | 是 | 是 | 最佳实践 | ✅✅✅ |

---

## 拓展内容

### 自定义实现 `Object.create()`

```js
function create(proto) {
  function F() {}
  F.prototype = proto;
  return new F();
}
```

### 手动实现 `class` 继承本质

```js
function inheritPrototype(Child, Parent) {
  Child.prototype = Object.create(Parent.prototype);
  Child.prototype.constructor = Child;
}
```

---

## 总结

> JS 继承有多种方式，推荐使用 **寄生组合继承** 或 ES6 的 `class extends`。理解原型链、`Object.create`、`constructor` 指向、构造函数调用时机是核心。继承的本质是设置对象的原型链，从而实现属性与方法的复用。

---

##  基本类型 vs 引用类型

### ✅ 基本类型（值类型）：
- `undefined`、`null`、`boolean`、`number`、`string`、`bigint`、`symbol`

### ✅ 引用类型（对象）：
- `Object`、`Array`、`Function`、`Date`、`RegExp`、`Map`、`Set`、自定义对象等

---

## 常见类型判断方法对比

| 方法 | 能判断哪些 | 准确性 | 场景 | 示例 |
|------|------------|--------|------|------|
| `typeof` | 基本类型 | ❌ null/array/function有误 | 快速判断基础类型 | `typeof 123` → `'number'` |
| `instanceof` | 引用类型 | ❌ 跨 iframe 失效 | 判断构造函数来源 | `[] instanceof Array` → `true` |
| `Object.prototype.toString.call()` | ✅ 所有类型 | ✅ 精准 | 最推荐 | `toString.call(null)` → `[object Null]` |
| `constructor` | 判断构造函数 | ❌ constructor 可被重写 | 一般不推荐 | `arr.constructor === Array` |

---

## 类型判断推荐模板（封装函数）

```js
function getType(val) {
  return Object.prototype.toString.call(val).slice(8, -1);
}

getType(null); // "Null"
getType([]); // "Array"
getType(new Date()); // "Date"
```

---

## 类型判断最佳实践总结：

| 类型 | 推荐方式 |
|------|----------|
| `null` | `val === null` |
| `array` | `Array.isArray(val)` |
| `function` | `typeof val === 'function'` |
| `object` | `getType(val) === 'Object'` |
| `NaN` | `Number.isNaN(val)` |
| `Promise` | `val instanceof Promise` or `typeof val?.then === 'function'` |

---

# JavaScript 深拷贝

---

## 浅拷贝与深拷贝的区别

- **浅拷贝**：只复制对象的第一层，引用类型仍然指向原地址
- **深拷贝**：完全复制一个新的值，包括嵌套对象/数组/函数等

---

## 浅拷贝的实现方式（不复制嵌套）

```js
const shallow = Object.assign({}, obj); // 只能一层
const clone = { ...obj };
```

---

## 深拷贝的常见方式

### 方法 1：`JSON.parse(JSON.stringify(obj))`

```js
const copy = JSON.parse(JSON.stringify(original));
```

#### 缺点：
- 无法处理 `function`、`undefined`、`symbol`、RegExp、Date、Map、Set、循环引用

---

### 方法 2：递归实现（手写深拷贝）

```js
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj);

  const clone = Array.isArray(obj) ? [] : {};
  hash.set(obj, clone);

  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key], hash); // 递归处理
    }
  }

  return clone;
}
```

- ✅ 可处理循环引用
- ✅ 保留对象结构

---

### 方法 3：使用库（推荐）

- [`lodash`](https://lodash.com/): `_.cloneDeep(obj)`
- ✅ 强大、稳定，支持各种边界情况

```bash
npm install lodash
```

```js
import cloneDeep from 'lodash/cloneDeep';
const copy = cloneDeep(obj);
```

---

## 深拷贝陷阱和难点

| 类型 | 处理难点 |
|------|-----------|
| `Date` | 需转为 `new Date(obj)` |
| `RegExp` | `new RegExp(obj.source, obj.flags)` |
| `Map/Set` | 需遍历并递归克隆 |
| `函数` | 无法深拷贝函数逻辑（可考虑传引用） |
| `循环引用` | 需 WeakMap 记录已遍历对象 |
| `Symbol` | 无法 JSON 序列化，需要单独处理 |

---

## 拓展：使用结构化克隆（浏览器原生）

```js
const clone = structuredClone(obj);
```

- ✅ 支持循环引用、Map、Set、Date、Blob 等
- ❗ 仅在现代浏览器/Node 17+ 支持

---

# 🧠 三、总结

> JavaScript 的类型判断推荐使用 `Object.prototype.toString.call`，能精确识别各种对象类型。深拷贝推荐使用 `lodash` 或结构化克隆，手写时需特别处理函数、循环引用、Map/Set 等特殊结构。

---

## 🛠️ Bonus：手写简易 cloneDeep 支持常见类型


