# Python Day 53 - 前端基础 HTML/CSS/JS

> 前两天我们学了 Streamlit，它能让我们用纯 Python 快速搭建 Web 界面。但真实的项目中，你经常需要自己编写**前端页面**——比如给 FastAPI 接口配一个漂亮的 HTML 表单，或者定制 Streamlit 的样式。今天我们来学习 Web 前端的三大基石：**HTML（结构）、CSS（样式）、JavaScript（交互）**。作为 Python 开发者，你不需要成为前端专家，但要能读懂和修改前端代码。

---

## 一、HTML —— 网页的骨架

HTML（HyperText Markup Language）定义了网页的**结构和内容**。它用"标签"（tag）来告诉浏览器：这里是一个标题、那里是一张图片、这里是一个按钮。

### 1.1 第一个 HTML 页面

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的第一个网页</title>
</head>
<body>
    <h1>你好，世界！</h1>
    <p>这是我的第一个网页。</p>
</body>
</html>
```

**逐行解析：**

| 标签 | 作用 |
|------|------|
| `<!DOCTYPE html>` | 声明文档类型为 HTML5 |
| `<html lang="zh-CN">` | 根元素，`lang` 声明语言为中文 |
| `<head>` | 头部区域，放元信息（不可见） |
| `<meta charset="UTF-8">` | 字符编码为 UTF-8（支持中文） |
| `<title>` | 浏览器标签页上显示的标题 |
| `<body>` | 主体区域，所有可见内容放这里 |

> **Python 类比**：HTML 就像 Python 的数据结构（字典、列表），负责"存储"信息，不负责"展示"。

### 1.2 常用标签速查

```html
<!-- 标题：h1 最大，h6 最小 -->
<h1>一级标题</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>

<!-- 段落 -->
<p>这是一个段落。</p>

<!-- 链接 -->
<a href="https://www.baidu.com" target="_blank">点击打开百度</a>

<!-- 图片 -->
<img src="photo.jpg" alt="风景照片" width="300">

<!-- 无序列表 -->
<ul>
    <li>苹果</li>
    <li>香蕉</li>
    <li>橘子</li>
</ul>

<!-- 有序列表 -->
<ol>
    <li>第一步：打开冰箱</li>
    <li>第二步：放入大象</li>
    <li>第三步：关上冰箱</li>
</ol>

<!-- 表格 -->
<table border="1">
    <tr>
        <th>姓名</th>
        <th>年龄</th>
    </tr>
    <tr>
        <td>张三</td>
        <td>25</td>
    </tr>
</table>

<!-- 表单（重要！用于用户输入） -->
<form action="/submit" method="POST">
    <label for="name">姓名：</label>
    <input type="text" id="name" name="name" placeholder="请输入姓名">

    <label for="email">邮箱：</label>
    <input type="email" id="email" name="email">

    <button type="submit">提交</button>
</form>

<!-- div：通用容器，最常用的布局元素 -->
<div class="container">
    <p>这是一个容器里的内容</p>
</div>

<!-- span：行内容器，不会换行 -->
<p>价格：<span class="price">¥99</span></p>
```

### 1.3 表单 input 的常见类型

作为后端开发者，你经常需要处理表单提交的数据。了解 input 类型很有帮助：

```html
<input type="text">       <!-- 单行文本 -->
<input type="password">   <!-- 密码（隐藏显示） -->
<input type="email">      <!-- 邮箱（自动验证格式） -->
<input type="number">     <!-- 数字 -->
<input type="date">       <!-- 日期选择器 -->
<input type="file">       <!-- 文件上传 -->
<input type="checkbox">    <!-- 复选框 -->
<input type="radio">       <!-- 单选按钮 -->
<input type="hidden">     <!-- 隐藏字段（传数据但不显示） -->
<textarea></textarea>      <!-- 多行文本 -->
<select>
    <option value="bj">北京</option>
    <option value="sh">上海</option>
</select>                 <!-- 下拉选择 -->
```

> **提示**：HTML 注释用 `<!-- -->` 包裹，和 Python 的 `#` 不一样。

---

## 二、CSS —— 网页的衣服

CSS（Cascading Style Sheets）控制网页的**外观和布局**——字体大小、颜色、间距、对齐方式等。

### 2.1 三种引入方式

```html
<!-- 方式一：行内样式（直接写在标签上，不推荐，只做调试用） -->
<p style="color: red; font-size: 18px;">红色大字</p>

<!-- 方式二：内部样式表（写在 <head> 的 <style> 标签中） -->
<head>
    <style>
        h1 { color: blue; }
        p  { font-size: 16px; }
    </style>
</head>

<!-- 方式三：外部样式表（推荐！独立 .css 文件） -->
<head>
    <link rel="stylesheet" href="style.css">
</head>
```

> **Python 类比**：CSS 的引入方式就像 Python 的代码组织——行内样式相当于"把代码写在命令行里"，内部样式相当于"写在脚本文件里"，外部样式表相当于"import 一个模块"。

### 2.2 选择器：定位要美化的元素

```css
/* 标签选择器：选中所有 p 标签 */
p {
    color: #333;           /* 文字颜色 */
    font-size: 16px;       /* 字体大小 */
    line-height: 1.8;      /* 行高 */
}

/* 类选择器：用 . 前缀，选中 class="highlight" 的元素 */
.highlight {
    background-color: yellow;
    padding: 5px 10px;     /* 上下的内边距5px，左右的10px */
    border-radius: 4px;    /* 圆角 */
}

/* ID 选择器：用 # 前缀，选中 id="header" 的元素 */
#header {
    background-color: #4a90d9;
    color: white;
    text-align: center;
    padding: 20px;
}

/* 后代选择器：选中 .card 内部的 .title */
.card .title {
    font-size: 20px;
    font-weight: bold;
}

/* 伪类选择器：鼠标悬停时变色 */
button:hover {
    background-color: #2c7be5;
    cursor: pointer;       /* 显示手指光标 */
}
```

**HTML 中使用类和 ID：**

```html
<div id="header">页面头部</div>
<p class="highlight">这段文字有黄色背景</p>
<div class="card">
    <p class="title">卡片标题</p>
    <p>卡片内容</p>
</div>
```

### 2.3 盒模型：理解布局的核心

每个 HTML 元素都是一个"盒子"，由四层组成：

```
┌─────────────────────────────────────┐
│           margin（外边距）            │
│  ┌───────────────────────────────┐  │
│  │       border（边框）           │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │     padding（内边距）    │  │  │
│  │  │  ┌─────────────────┐  │  │  │
│  │  │  │   content（内容） │  │  │  │
│  │  │  └─────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

```css
.box {
    width: 200px;              /* 内容宽度 */
    height: 100px;             /* 内容高度 */
    padding: 15px;            /* 内边距：内容到边框的距离 */
    border: 2px solid #ccc;    /* 边框：2px宽、实线、灰色 */
    margin: 20px;              /* 外边距：此盒子到其他元素的距离 */

    /* 推荐使用 box-sizing 让 width 包含 padding 和 border */
    box-sizing: border-box;    /* 这样 width=200px 就是总宽度 */
}
```

### 2.4 Flexbox 布局：最实用的布局方式

Flexbox 让我们轻松实现**水平居中、等分布局、对齐**等常见需求：

```css
/* 容器设置 display: flex 启用 Flexbox */
.container {
    display: flex;
    justify-content: center;    /* 水平居中 */
    align-items: center;        /* 垂直居中 */
    gap: 20px;                  /* 子元素之间的间距 */
    height: 100vh;              /* 高度占满整个视口 */
}

/* 水平等分布局 */
.navbar {
    display: flex;
    justify-content: space-between; /* 两端对齐，中间等分 */
    padding: 10px 20px;
    background-color: #333;
}

/* 多行换行 */
.card-list {
    display: flex;
    flex-wrap: wrap;            /* 允许换行 */
    gap: 15px;
}
```

```html
<!-- 简单的 Flex 布局示例 -->
<div class="navbar">
    <span class="logo">我的网站</span>
    <div class="nav-links">
        <a href="#">首页</a>
        <a href="#">关于</a>
        <a href="#">联系</a>
    </div>
</div>
```

### 2.5 常用 CSS 属性速查

```css
/* 文字样式 */
font-family: "Microsoft YaHei", sans-serif;  /* 字体 */
font-weight: bold;       /* 粗体（或用数字 100-900） */
text-align: center;      /* 对齐：left / center / right */
text-decoration: underline;  /* 下划线 / none 去除 */

/* 颜色 */
color: red;              /* 文字颜色 */
background-color: #f0f0f0; /* 背景色 */
opacity: 0.8;            /* 透明度（0-1） */

/* 边框与圆角 */
border: 1px solid #ddd;
border-radius: 8px;      /* 圆角，50% 变圆形 */

/* 阴影 */
box-shadow: 0 2px 8px rgba(0,0,0,0.1);  /* 盒子阴影 */
text-shadow: 1px 1px 2px rgba(0,0,0,0.3); /* 文字阴影 */

/* 显示与隐藏 */
display: none;           /* 完全隐藏，不占空间 */
display: block;          /* 块级元素，独占一行 */
display: inline;         /* 行内元素，不换行 */
display: flex;           /* Flex 布局 */

/* 定位 */
position: relative;       /* 相对定位（相对自己原位置） */
position: absolute;       /* 绝对定位（相对最近的定位父元素） */
position: fixed;         /* 固定定位（相对浏览器窗口，不随滚动） */
```

---

## 三、JavaScript —— 网页的大脑

JavaScript 让网页从"静态文档"变成"动态应用"——点击按钮可以弹窗、实时验证表单、无需刷新页面就能更新内容。

### 3.1 第一个 JavaScript

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JavaScript 入门</title>
</head>
<body>
    <h1 id="greeting">你好</h1>
    <button onclick="sayHello()">点我</button>

    <script>
        // 这是一个 JavaScript 函数
        function sayHello() {
            // 修改网页上的文字
            document.getElementById("greeting").innerText = "你好，JavaScript！";
            // 弹出一个提示框
            alert("按钮被点击了！");
        }
    </script>
</body>
</html>
```

### 3.2 变量与数据类型

JavaScript 的变量声明和 Python 类似但不完全相同：

```javascript
// 声明变量（三种方式）
var name = "张三";       // 旧写法，尽量不用
let age = 25;            // 可变变量（类似 Python 普通变量）
const PI = 3.14;         // 常量，声明后不能修改

// 基本数据类型
let str = "Hello";       // 字符串
let num = 42;            // 数字（不区分 int 和 float）
let isTrue = true;       // 布尔值
let empty = null;         // 空值
let nothing = undefined; // 未定义
let arr = [1, 2, 3];     // 数组（类似 Python 的 list）
let obj = {              // 对象（类似 Python 的 dict）
    name: "张三",
    age: 25
};

// 模板字符串（类似 Python 的 f-string）
let msg = `我叫${obj.name}，今年${age}岁`;
console.log(msg);  // 输出到浏览器控制台（F12 查看）

// 数组常用方法
arr.push(4);          // 添加元素（类似 Python 的 append）
arr.pop();            // 删除最后一个
arr.length;           // 长度（类似 Python 的 len）
arr.includes(2);     // 是否包含
arr.map(x => x * 2); // 映射（类似 Python 的列表推导式）
arr.filter(x => x > 1); // 过滤
```

### 3.3 条件判断与循环

```javascript
// 条件判断（和 Python 类似，注意括号和大括号）
let score = 85;

if (score >= 90) {
    console.log("优秀");
} else if (score >= 60) {
    console.log("及格");
} else {
    console.log("不及格");
}

// for 循环
for (let i = 0; i < 5; i++) {
    console.log(i);  // 输出 0, 1, 2, 3, 4
}

// for...of（类似 Python 的 for item in list）
let colors = ["红", "绿", "蓝"];
for (let color of colors) {
    console.log(color);
}

// 遍历对象（类似 Python 遍历字典）
let person = { name: "张三", age: 25 };
for (let key in person) {
    console.log(`${key}: ${person[key]}`);
}
```

### 3.4 函数

```javascript
// 普通函数
function add(a, b) {
    return a + b;
}

// 箭头函数（Lambda 表达式的 JS 版本）
const multiply = (a, b) => a * b;

// 对象中的方法
const calculator = {
    add: (a, b) => a + b,
    subtract: (a, b) => a - b
};

console.log(add(3, 5));           // 8
console.log(multiply(3, 5));      // 15
console.log(calculator.add(3, 5)); // 8
```

### 3.5 DOM 操作：用 JS 修改网页

DOM（Document Object Model）是 JavaScript 操作网页的接口。这是**最核心的前端技能**：

```javascript
// 获取元素（四种方式）
document.getElementById("myId");           // 通过 ID 获取
document.getElementsByClassName("myClass"); // 通过类名获取
document.getElementsByTagName("p");         // 通过标签名获取
document.querySelector(".myClass");        // CSS 选择器获取第一个
document.querySelectorAll(".myClass");      // CSS 选择器获取所有

// 修改内容和样式
let el = document.getElementById("title");
el.innerText = "新标题";              // 修改文字内容
el.innerHTML = "<strong>加粗标题</strong>"; // 修改 HTML 内容
el.style.color = "red";              // 修改行内样式
el.classList.add("active");           // 添加 CSS 类
el.classList.remove("hidden");        // 移除 CSS 类
el.classList.toggle("show");         // 切换 CSS 类（有则删，无则加）

// 创建和删除元素
let newP = document.createElement("p");  // 创建元素
newP.innerText = "新建的段落";
document.body.appendChild(newP);         // 添加到页面
newP.remove();                           // 从页面删除

// 事件监听（推荐写法，代替 onclick 属性）
document.getElementById("btn").addEventListener("click", function() {
    console.log("按钮被点击了！");
});
```

### 3.6 Fetch API：前端发请求（连接后端）

这是 Python 后端开发者**最需要的 JS 知识**——前端如何调用你的 FastAPI 接口：

```javascript
// GET 请求
fetch("http://localhost:8000/api/users")
    .then(response => response.json())  // 将响应转为 JSON
    .then(data => {
        console.log("获取到的用户：", data);
        // 在页面上显示数据
        document.getElementById("result").innerText =
            `用户名：${data.name}，年龄：${data.age}`;
    })
    .catch(error => {
        console.error("请求失败：", error);
    });

// POST 请求（提交数据）
fetch("http://localhost:8000/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
        name: "李四",
        age: 30
    })
})
    .then(response => response.json())
    .then(data => console.log("创建成功：", data));

// 用 async/await 写法（更清晰，类似 Python 的异步）
async function getUser() {
    try {
        let response = await fetch("http://localhost:8000/api/users/1");
        let data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("出错了：", error);
    }
}
```

> **Python 类比**：`fetch` 相当于 Python 的 `requests.get()`，`async/await` 和 Python 3.5+ 的 `async/await` 几乎一模一样。

---

## 四、综合实战：Python 后端 + HTML 前端

下面用一个完整的例子，展示 Python（FastAPI）后端如何和 HTML/CSS/JS 前端配合工作。

### 4.1 后端：FastAPI

```python
# app.py
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# 挂载静态文件目录（CSS、JS 文件放这里）
app.mount("/static", StaticFiles(directory="static"), name="static")

# 返回 HTML 页面
@app.get("/", response_class=HTMLResponse)
async def index():
    with open("templates/index.html", "r", encoding="utf-8") as f:
        return f.read()

# API 接口：获取所有待办事项
@app.get("/api/todos")
async def get_todos():
    return [
        {"id": 1, "title": "学习 Python", "done": True},
        {"id": 2, "title": "学习前端", "done": False},
        {"id": 3, "title": "做项目", "done": False},
    ]

# API 接口：添加待办事项
@app.post("/api/todos")
async def add_todo(request: Request):
    data = await request.json()
    return {"id": 4, "title": data["title"], "done": False}
```

### 4.2 前端：HTML + CSS + JS

```html
<!-- templates/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>待办事项</title>
    <style>
        /* CSS 样式 */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: "Microsoft YaHei", sans-serif;
            background-color: #f5f5f5;
            padding: 20px;
        }

        .container {
            max-width: 500px;
            margin: 0 auto;
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 20px;
        }

        /* 输入区域 */
        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }

        .input-group input {
            flex: 1;
            padding: 10px 15px;
            border: 1px solid #ddd;
            border-radius: 6px;
            font-size: 14px;
        }

        .input-group input:focus {
            outline: none;
            border-color: #4a90d9;
        }

        .btn {
            padding: 10px 20px;
            background-color: #4a90d9;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
        }

        .btn:hover {
            background-color: #357abd;
        }

        /* 待办列表 */
        .todo-item {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 12px 15px;
            background: white;
            border-radius: 6px;
            margin-bottom: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }

        .todo-item.done span {
            text-decoration: line-through;
            color: #999;
        }

        .todo-item span {
            font-size: 14px;
        }

        .loading {
            text-align: center;
            color: #666;
            padding: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📝 待办事项</h1>

        <div class="input-group">
            <input type="text" id="todoInput" placeholder="输入新的待办事项...">
            <button class="btn" onclick="addTodo()">添加</button>
        </div>

        <div id="todoList">
            <div class="loading">加载中...</div>
        </div>
    </div>

    <script>
        // 页面加载完成后获取待办列表
        document.addEventListener("DOMContentLoaded", loadTodos);

        // 回车键也可以添加
        document.getElementById("todoInput").addEventListener("keypress",
            function(event) {
                if (event.key === "Enter") {
                    addTodo();
                }
            }
        );

        // 获取待办列表
        async function loadTodos() {
            try {
                const response = await fetch("/api/todos");
                const todos = await response.json();
                renderTodos(todos);
            } catch (error) {
                document.getElementById("todoList").innerHTML =
                    `<div class="loading">加载失败：${error.message}</div>`;
            }
        }

        // 渲染待办列表到页面
        function renderTodos(todos) {
            const container = document.getElementById("todoList");
            if (todos.length === 0) {
                container.innerHTML = '<div class="loading">暂无待办事项</div>';
                return;
            }
            container.innerHTML = todos.map(todo => `
                <div class="todo-item ${todo.done ? 'done' : ''}">
                    <span>${todo.title}</span>
                    <span>${todo.done ? '✅' : '⬜'}</span>
                </div>
            `).join("");
        }

        // 添加新待办
        async function addTodo() {
            const input = document.getElementById("todoInput");
            const title = input.value.trim();

            if (!title) {
                alert("请输入待办事项内容！");
                return;
            }

            try {
                const response = await fetch("/api/todos", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ title: title })
                });
                const newTodo = await response.json();
                input.value = "";  // 清空输入框
                await loadTodos(); // 重新加载列表
            } catch (error) {
                alert("添加失败：" + error.message);
            }
        }
    </script>
</body>
</html>
```

### 4.3 运行方式

```bash
# 安装 FastAPI
pip install fastapi uvicorn

# 项目结构
# project/
# ├── app.py              # Python 后端
# ├── static/             # 静态文件（CSS、JS）
# └── templates/           # HTML 模板
#     └── index.html

# 启动服务
uvicorn app:app --reload --port 8000

# 浏览器打开 http://localhost:8000 即可看到页面
```

---

## 五、Python vs JavaScript 语法对比表

| 功能 | Python | JavaScript |
|------|--------|------------|
| 变量声明 | `x = 10` | `let x = 10;` |
| 常量 | 无原生支持 | `const x = 10;` |
| 字符串拼接 | `f"Hello {name}"` | `` `Hello ${name}` `` |
| 列表/数组 | `[1, 2, 3]` | `[1, 2, 3]` |
| 字典/对象 | `{"a": 1}` | `{a: 1}` |
| 条件判断 | `if x > 0:` | `if (x > 0) {` |
| for 循环 | `for i in range(5):` | `for (let i = 0; i < 5; i++) {` |
| 函数 | `def add(a, b):` | `function add(a, b) {` |
| Lambda | `lambda x: x * 2` | `(x) => x * 2` |
| 输出 | `print(x)` | `console.log(x)` |
| 真值判断 | `None, 0, "", []` 为假 | `null, 0, "", []` 为假 |
| 注释 | `# 注释` | `// 注释` |

---

## 六、练习题

### 练习一：个人介绍页面

创建一个 HTML 文件，包含：
- 标题（h1）显示你的名字
- 段落（p）介绍你自己
- 一个无序列表展示你的兴趣爱好
- 一个链接指向你喜欢的网站
- 使用 CSS 让页面居中显示，背景色为浅蓝色

### 练习二：JavaScript 计算器

在 HTML 页面上放两个数字输入框和一个运算符选择框（下拉），一个"计算"按钮。点击按钮后用 JavaScript 计算结果并显示在页面上。

```html
<!-- 提示：关键代码结构 -->
<input type="number" id="num1">
<select id="operator">
    <option value="+">+</option>
    <option value="-">-</option>
    <option value="*">×</option>
    <option value="/">÷</option>
</select>
<input type="number" id="num2">
<button onclick="calculate()">计算</button>
<p id="result"></p>

<script>
function calculate() {
    // 获取值 → 转为数字 → 判断运算符 → 计算结果 → 显示
}
</script>
```

### 练习三：调用 FastAPI 的天气页面

假设你有一个 FastAPI 接口 `GET /api/weather?city=北京` 返回 `{"city": "北京", "temp": 25, "desc": "晴"}`。编写一个 HTML 页面，包含城市输入框和"查询"按钮，点击后用 `fetch` 调用接口并展示天气信息。

---

## 七、常见问题（FAQ）

**Q1：HTML、CSS、JS 分别负责什么？能不能混在一起写？**
A：技术上可以混写，但**强烈建议分开**——HTML 放结构、CSS 放样式（`.css` 文件）、JS 放逻辑（`.js` 文件）。这样代码清晰易维护，团队协作也更方便。

**Q2：我作为 Python 后端开发者，前端需要学多深？**
A：掌握今天的内容就够用了——能读懂 HTML 结构、会写基本 CSS 样式、能用 JS 的 `fetch` 调后端 API。如果需要复杂的前端界面，考虑用 Streamlit（纯 Python）或找前端同事配合。

**Q3：JavaScript 和 Python 有什么本质区别？**
A：最大的区别是 JavaScript 运行在**浏览器**里，Python 运行在**服务器**上。JavaScript 是异步驱动的（因为要处理用户交互），Python 则更适合数据处理和后端逻辑。两者的语法相似，但细节差异很多。

**Q4：什么是 Vue/React？和今天学的有什么关系？**
A：Vue、React 是**前端框架**，帮你更高效地管理复杂的用户界面。它们底层用的就是 HTML + CSS + JS。学好今天的基础，以后学框架会事半功倍。

**Q5：CSS 的 Flexbox 和 Grid 有什么区别？**
A：Flexbox 是**一维布局**（要么横排要么竖排），Grid 是**二维布局**（同时控制行和列）。对于大多数简单场景，Flexbox 就够用了。

---

## 八、免费学习资源

1. **MDN Web Docs**（最权威的 Web 技术文档）：https://developer.mozilla.org/zh-CN/
2. **菜鸟教程 - HTML/CSS/JavaScript**：https://www.runoob.com/html/html-tutorial.html
3. **廖雪峰 - JavaScript 教程**：https://www.liaoxuefeng.com/wiki/1022910821219416
4. **freeCodeCamp**（交互式学习平台）：https://chinese.freecodecamp.org/
5. **Can I Use**（查浏览器兼容性）：https://caniuse.com/

---

**Day 53 小结**：今天我们学习了 Web 前端的三大基石——HTML 搭建页面结构、CSS 美化外观样式、JavaScript 实现动态交互。最关键的技能是理解 DOM 操作和 `fetch` API，这让你的 Python 后端能与前端页面无缝通信。明天我们将进入**项目实战——Web 应用（1）**，把学过的 FastAPI + 前端知识整合成一个完整项目。
