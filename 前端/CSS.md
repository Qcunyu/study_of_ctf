# 一、CSS 是什么

- **层叠样式表**（Cascading Style Sheets），用于控制 HTML 元素的呈现。
- 核心思想：**样式与内容分离**，通过选择器匹配元素，统一设置外观。

---

# 二、引入方式

|方式|示例|说明|
|---|---|---|
|**外部样式表**|`<link rel="stylesheet" href="style.css">`|推荐，分离结构与样式，可复用、可缓存|
|**内部样式表**|`<style> body { margin: 0; } </style>`|写在 HTML 的 `<head>` 中，适用于单页面|
|**内联样式**|`<div style="color: red;">`|优先级最高，但难以维护，渗透测试中可能发现临时调试代码|

---

# 三、选择器（核心）
选择器用于指定哪些元素应用样式，也是 [[JavaScript]] 操作 DOM 的重要依据。

|选择器|示例|说明|
|---|---|---|
|**元素选择器**|`p { }`|所有 `<p>` 标签|
|**类选择器**|`.box { }`|所有 `class="box"` 的元素|
|**ID 选择器**|`#header { }`|`id="header"` 的唯一元素|
|**属性选择器**|`[type="text"] { }`|匹配特定属性值的元素|
|**后代选择器**|`div p { }`|`<div>` 内部的 `<p>`（所有层级）|
|**子选择器**|`div > p { }`|直接子元素 `<p>`|
|**伪类**|`a:hover { }`|元素状态，如悬浮、焦点|
|**伪元素**|`p::before { }`|插入虚拟内容|

**安全视角**：
- 属性选择器可用于检测是否存在特定元素（如 `input[type="hidden"]` 可被 [[JavaScript]] 修改）
- 伪类 `:hover`、`:active` 可能被用来隐藏钓鱼按钮（平时不可见，鼠标移动才显示）

---

# 四、盒模型（布局基础）

每个 HTML 元素都是一个矩形盒子，包含以下部分：
```text
+----------------------------------+
|         margin (外边距)           |
|  +----------------------------+  |
|  |         border (边框)       |  |
|  |  +----------------------+  |  |
|  |  |      padding (内边距) |  |  |
|  |  |  +----------------+  |  |  |
|  |  |  |   content      |  |  |  |
|  |  |  +----------------+  |  |  |
|  |  +----------------------+  |  |
|  +----------------------------+  |
+----------------------------------+
```

|属性|说明|
|---|---|
|`width` / `height`|默认只设置内容区域宽高（可通过 `box-sizing` 调整）|
|`padding`|内边距，背景色会延伸到此处|
|`border`|边框，可设置颜色、粗细、样式|
|`margin`|外边距，元素之间的间距|
|`box-sizing: border-box`|使宽高包含 border 和 padding，布局更友好|

---

# 五、布局方式

## 1. 常规流（块级 / 行内）

- 块级元素独占一行，行内元素同行排列。
- 通过 `display` 改变行为：`display: block`、`inline`、`inline-block`、`none`（隐藏且不占位）、`visibility: hidden`（隐藏但仍占位）
## 2. Flexbox（一维布局）
```css
.container {
    display: flex;
    justify-content: center;  /* 主轴对齐 */
    align-items: center;      /* 交叉轴对齐 */
    flex-direction: row;      /* 主轴方向 */
}
```
- 适用于导航栏、卡片列表、垂直居中。

### 3. Grid（二维布局）

css

复制

下载

.container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;  /* 三列等宽 */
    gap: 10px;
}

- 适用于整体页面布局、复杂网格。
    

### 4. 定位（`position`）

|值|说明|
|---|---|
|`static`|默认，按常规流排列|
|`relative`|相对自身原本位置偏移|
|`absolute`|相对最近的非 static 父元素定位，脱离文档流|
|`fixed`|相对视口定位，不随滚动|
|`sticky`|粘性定位，滚动到阈值时表现类似 fixed|

**安全视角**：

- `position: absolute` + `left: -9999px` 或 `visibility: hidden` 可隐藏恶意元素，用于钓鱼或水坑攻击。
    
- `position: fixed` 可制作覆盖全屏的钓鱼浮层。