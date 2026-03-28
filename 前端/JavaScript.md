 > JavaScript 负责页面的**行为与交互**，与 [[HTML]] 定义结构、[[CSS]] 控制样式共同构成前端三件套。  
> 本笔记涵盖核心语法、DOM 操作、事件、异步编程，以及渗透测试中需关注的前端安全点。

---

# 一、JavaScript 是什么

- 轻量级解释型脚本语言，运行在浏览器或 Node.js 环境中。
- 在浏览器端，JS 可操作 DOM（文档对象模型）、处理用户事件、发送网络请求、存储数据等。
- 安全视角：JS 是 XSS（跨站脚本）攻击的核心载体，也常用于前端验证绕过、信息泄露。

---

# 二、引入方式

|方式|示例|说明|
|---|---|---|
|**外部脚本**|`<script src="script.js" defer></script>`|推荐，分离行为与结构，可缓存，`defer` 延迟执行直到 HTML 解析完成|
|**内部脚本**|`<script> console.log('hello'); </script>`|写在 HTML 中，可在 `<head>` 或 `<body>` 末尾|
|**内联事件**|`<button onclick="alert('hi')">`|可维护性差，但可能在渗透测试中发现测试代码|
**注意**：`<script>` 默认会阻塞 HTML 解析，建议放在 `</body>` 前或使用 `async/defer`。

---

# 三、变量与数据类型

## 1. 变量声明

| 关键字     | 作用域   | 特点                  |
| ------- | ----- | ------------------- |
| `var`   | 函数作用域 | 存在变量提升，可重复声明，不推荐使用  |
| `let`   | 块级作用域 | 推荐，不可重复声明           |
| `const` | 块级作用域 | 声明常量，引用不可变，对象属性仍可修改 |
## 2. 基本数据类型
- `string`, `number`, `boolean`, `null`, `undefined`, `symbol` (ES6), `bigint`
- 类型判断：`typeof` 运算符

### 3. 引用类型
- `object`：普通对象 `{}`、数组 `[]`、函数 `function`
- 判断数组：`Array.isArray(arr)`

### 4. 类型转换
- 隐式转换：`'5' - 3` → 2，`'5' + 3` → '53'（字符串拼接）
- 显式转换：`Number()`、`String()`、`Boolean()`、`parseInt()`、`parseFloat()`

---

# 四、运算符与流程控制

- **算术、比较、逻辑、三元**：`&&`、`||`、`!`、`??`（空值合并）、`?.`（可选链）
- **条件语句**：`if...else`、`switch`
- **循环**：`for`、`while`、`do...while`、`for...of`（遍历可迭代对象）、`for...in`（遍历对象属性）

---

# 五、函数

## 1. 函数声明
```javascript
function add(a, b) {
    return a + b;
}
```
## 2. 函数表达式
```javascript
const add = function(a, b) { return a + b; };
```
### 3. 箭头函数 (ES6)

javascript

复制

下载

const add = (a, b) => a + b;
// 注意：箭头函数没有自己的 this，不绑定 arguments

### 4. 默认参数

javascript

复制

下载

function greet(name = 'Guest') { ... }

### 5. 高阶函数

- 函数作为参数或返回值，常见于 `map`、`filter`、`reduce`、`setTimeout` 等。