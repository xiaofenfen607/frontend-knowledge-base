## 一、DOM（Document Object Model）操作

DOM 是浏览器提供的操作网页内容的 API，是前端基础能力核心之一。

### 查找节点

```js
document.getElementById('id')
document.querySelector('.class') // 支持选择器语法
document.querySelectorAll('.list li')
```

### 节点属性操作

```js
element.innerHTML = '<p>hello</p>'  // 解析为 DOM
element.textContent = '纯文本'     // 不解析标签
element.setAttribute('data-id', '123')
element.getAttribute('data-id')
element.classList.add('active')
```

### 节点结构操作

```js
parent.appendChild(child)
parent.insertBefore(newNode, referenceNode)
element.remove()
```

### 创建节点

```js
const div = document.createElement('div')
div.innerHTML = 'hello'
document.body.appendChild(div)
```

## 二、事件模型基础

### 什么是事件？
事件是用户或浏览器对页面执行的交互动作，比如点击、鼠标移动、键盘按下等。

### addEventListener 的使用

```js
element.addEventListener('click', function (e) {
  console.log('Clicked!')
})
```

可选第三个参数：

- `false`：默认，监听冒泡阶段
- `true`：监听捕获阶段

```js
element.addEventListener('click', fn, true) // 捕获阶段触发
```

## 三、事件流（捕获 → 目标 → 冒泡）

事件从 `window` 开始，按顺序捕获到目标，再从目标冒泡回 `window`：

```plaintext
捕获阶段：window → document → html → body → div
目标阶段：div
冒泡阶段：div → body → html → document → window
```

### 示例：

```html
<div id="outer">
  <button id="inner">Click me</button>
</div>
```

```js
document.getElementById('outer').addEventListener('click', () => {
  console.log('outer clicked')
})

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner clicked')
})
```

点击按钮输出：
```
inner clicked
outer clicked
```

即事件先在目标触发，然后冒泡到父级。

## 四、事件委托（Event Delegation）

### 原理
利用事件冒泡机制，只在**父元素**绑定事件处理器，根据 `event.target` 判断实际触发的子元素。

### 优势
- 减少绑定数量，提升性能
- 支持动态新增元素（无需再次绑定）

### 示例

```html
<ul id="list">
  <li>Item 1</li>
  <li>Item 2</li>
</ul>
```

```js
document.getElementById('list').addEventListener('click', function (e) {
  if (e.target.tagName === 'LI') {
    console.log('Clicked:', e.target.textContent)
  }
})
```

新增项也会生效：

```js
const li = document.createElement('li')
li.textContent = 'Item 3'
document.getElementById('list').appendChild(li)
```

## 五、事件对象常用属性和方法

```js
element.addEventListener('click', function (e) {
  console.log(e.target)      // 实际点击元素
  console.log(e.currentTarget) // 绑定事件的元素
  e.stopPropagation()         // 阻止冒泡
  e.preventDefault()          // 阻止默认行为
})
```

## 六、常见题目与陷阱

### 阻止事件冒泡但不阻止默认行为：

```js
el.addEventListener('click', e => {
  e.stopPropagation()
})
```

### 阻止表单提交默认行为：

```js
form.addEventListener('submit', e => {
  e.preventDefault()
})
```


## 七、拓展：委托事件时如何只触发一次？

```js
element.addEventListener('click', fn, { once: true })
```

或手动解绑：

```js
const handler = e => {
  console.log('Only once')
  element.removeEventListener('click', handler)
}
element.addEventListener('click', handler)
```

---

## 八、面试问答速记

### 1. DOM 操作有哪些性能优化建议？
- 减少回流（reflow）、重绘（repaint）
- 批量插入用 fragment
- 缓存访问频繁的 DOM
- 使用事件委托替代多次绑定

### 2. 事件委托和冒泡的关系？
事件委托依赖事件冒泡机制。通过监听父级，在事件冒泡过程中处理子级触发事件。

