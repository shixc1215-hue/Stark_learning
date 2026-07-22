# Python Day 46 — FastAPI 入门

> **学习目标**：了解 Web 框架的概念，学会用 FastAPI 搭建第一个 REST API 服务，掌握路径参数、查询参数、请求体、响应模型等核心机制。

---

## 一、为什么学 Web 框架？

### 1.1 什么是 Web 框架？

在前面的学习中，我们编写的 Python 程序都是"在本地跑"的——读取文件、处理数据、打印结果。但如果想让**其他人通过浏览器或手机 App 来使用你的程序**，就需要 Web 框架。

Web 框架的作用：
- 接收来自客户端（浏览器/App）的**HTTP 请求**
- 处理业务逻辑
- 返回**HTTP 响应**（JSON 数据、HTML 页面等）

```
用户浏览器  →  HTTP 请求  →  FastAPI 服务  →  处理逻辑  →  HTTP 响应  →  用户浏览器
```

### 1.2 为什么选 FastAPI？

Python 有多个 Web 框架，FastAPI 是目前最推荐新手学习的，原因如下：

| 特性 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| 学习难度 | ⭐⭐ 中等 | ⭐ 简单 | ⭐⭐⭐ 较难 |
| 性能 | 极快（异步） | 较慢 | 中等 |
| 自动文档 | ✅ 自带 Swagger | ❌ 需插件 | ❌ 需插件 |
| 类型提示支持 | ✅ 原生支持 | ⚠️ 有限 | ⚠️ 有限 |
| 异步支持 | ✅ 原生 | ⚠️ 需扩展 | ✅ 3.1+ |
| 适用场景 | API 服务 | 小型项目 | 大型全栈 |

FastAPI 的三大卖点：
1. **快**：基于 Starlette 和 Pydantic，性能接近 Node.js 和 Go
2. **自带文档**：启动服务后直接访问 `/docs` 就能看到交互式 API 文档
3. **类型安全**：利用 Python 类型注解自动校验请求数据

### 1.3 安装 FastAPI

```python
# 安装 FastAPI 和 ASGI 服务器 uvicorn
# 在终端中执行（不是 Python 代码）：
# pip install fastapi uvicorn
```

安装完成后验证：

```python
import fastapi
print(fastapi.__version__)  # 输出版本号即安装成功
```

---

## 二、第一个 FastAPI 应用

### 2.1 Hello World

创建文件 `main.py`：

```python
from fastapi import FastAPI

# 创建 FastAPI 应用实例
app = FastAPI(title="我的第一个API", version="1.0")


# 使用装饰器定义路由，get 表示 HTTP GET 方法
# "/" 是路径（URL 的一部分）
@app.get("/")
def read_root():
    """根路径，返回欢迎信息"""
    return {"message": "你好，FastAPI！", "version": "1.0"}


@app.get("/hello/{name}")
def say_hello(name: str):
    """向指定用户打招呼"""
    return {"message": f"你好，{name}！欢迎学习 FastAPI"}
```

### 2.2 启动服务

```bash
# 在终端中执行：
# uvicorn main:app --reload
```

命令含义解析：
- `main:app` → `main.py` 文件中的 `app` 对象
- `--reload` → 代码修改后自动重启（开发时必开）

启动后终端会显示：

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [xxxxx]
INFO:     Started server process [xxxxx]
INFO:     Application startup complete.
```

### 2.3 访问你的 API

打开浏览器或使用 Python 测试：

```python
import requests

# 访问根路径
response = requests.get("http://127.0.0.1:8000/")
print(response.json())
# 输出: {'message': '你好，FastAPI！', 'version': '1.0'}

# 访问带路径参数的路径
response = requests.get("http://127.0.0.1:8000/hello/小明")
print(response.json())
# 输出: {'message': '你好，小明！欢迎学习 FastAPI'}
```

### 2.4 自动生成的交互式文档 🎉

这是 FastAPI 最酷的功能！启动服务后访问：

| 地址 | 说明 |
|------|------|
| `http://127.0.0.1:8000/docs` | Swagger UI 文档（可视化操作） |
| `http://127.0.0.1:8000/redoc` | ReDoc 文档（更简洁） |

在 Swagger UI 中你可以：
- 看到所有 API 接口及其参数说明
- 直接在页面上点击 "Try it out" 测试接口
- 查看请求和响应的数据结构

---

## 三、请求参数的三种类型

FastAPI 会根据参数的位置和类型自动区分参数，这是它的核心魔法。

### 3.1 路径参数（Path Parameters）

路径参数是 URL 的一部分，用 `{}` 包裹，在函数参数中直接声明类型即可：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
def get_item(item_id: int):
    """根据 ID 获取物品信息"""
    return {"item_id": item_id, "name": f"物品-{item_id}"}


@app.get("/users/{user_id}/posts/{post_id}")
def get_user_post(user_id: int, post_id: int):
    """多个路径参数"""
    return {"user_id": user_id, "post_id": post_id}
```

访问示例：
```
GET /items/42       → {"item_id": 42, "name": "物品-42"}
GET /items/abc      → 报错！FastAPI 自动校验，abc 不是 int
```

> **关键点**：FastAPI 会自动进行**类型转换和校验**。如果 `item_id` 声明为 `int`，传入字符串会自动转为整数；传入无法转换的值会返回 422 错误。

### 3.2 查询参数（Query Parameters）

查询参数出现在 URL 的 `?` 之后，用 `key=value` 格式，多个参数用 `&` 分隔。函数参数中**非路径参数**的参数会被自动识别为查询参数：

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()


@app.get("/items")
def list_items(skip: int = 0, limit: int = 10, category: Optional[str] = None):
    """
    获取物品列表
    - skip: 跳过前几条
    - limit: 最多返回几条
    - category: 可选的分类筛选
    """
    items = [{"id": i, "name": f"物品-{i}"} for i in range(100)]

    # 筛选分类
    if category:
        items = [item for item in items if category in item["name"]]

    # 分页
    return {"items": items[skip : skip + limit], "total": len(items)}


@app.get("/search")
def search(q: str, page: int = 1, size: int = 20):
    """搜索接口，q 是必填的查询参数"""
    return {
        "query": q,
        "page": page,
        "size": size,
        "results": [f"结果{i}匹配'{q}'" for i in range(3)]
    }
```

访问示例：
```
GET /items                      → skip=0, limit=10（使用默认值）
GET /items?skip=20&limit=5      → skip=20, limit=5
GET /items?category=水果        → 筛选分类为"水果"
GET /search?q=Python&page=2     → q 是必填，不传会报错
```

### 3.3 请求体（Request Body）

当需要接收复杂的数据（如创建或更新资源），通常使用 POST 方法并传递 JSON 请求体。FastAPI 使用 **Pydantic 模型** 来定义和校验请求体：

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()


# 定义请求体模型
class Item(BaseModel):
    name: str           # 必填
    price: float         # 必填，自动校验是否为数字
    description: Optional[str] = None  # 可选，默认 None
    tax: Optional[float] = None        # 可选
    in_stock: bool = True             # 可选，有默认值


@app.post("/items")
def create_item(item: Item):
    """创建一个新物品"""
    # FastAPI 自动将 JSON 请求体解析为 Item 对象
    item_dict = item.model_dump()  # Pydantic v2 用法（v1 用 dict()）
    item_dict["id"] = 42  # 模拟生成 ID
    return item_dict


@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    """更新指定 ID 的物品"""
    return {"item_id": item_id, **item.model_dump()}
```

使用 Python 请求测试：

```python
import requests

url = "http://127.0.0.1:8000/items"

# 请求体是 JSON 格式
data = {
    "name": "笔记本电脑",
    "price": 5999.0,
    "description": "高性能办公本",
    "tax": 300.0,
    "in_stock": True
}

response = requests.post(url, json=data)
print(response.json())
# 输出: {'name': '笔记本电脑', 'price': 5999.0, 'description': '高性能办公本',
#        'tax': 300.0, 'in_stock': True, 'id': 42}
```

### 3.4 参数类型对比总结

| 参数类型 | 位置 | 示例 | 适用场景 |
|----------|------|------|----------|
| 路径参数 | URL 路径中 | `/items/{id}` | 标识具体资源 |
| 查询参数 | URL `?` 后面 | `/items?limit=10` | 筛选、排序、分页 |
| 请求体 | HTTP Body 中 | `{"name": "xxx"}` | 提交/修改复杂数据 |

---

## 四、响应模型与状态码

### 4.1 控制响应状态码

```python
from fastapi import FastAPI, status

app = FastAPI()


@app.post("/items", status_code=status.HTTP_201_CREATED)
def create_item():
    """创建资源通常返回 201 而非 200"""
    return {"message": "创建成功"}


@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int):
    """删除成功通常返回 204，无响应体"""
    return None
```

常用状态码：

| 状态码 | 含义 | 用途 |
|--------|------|------|
| 200 | OK | 请求成功 |
| 201 | Created | 资源创建成功 |
| 204 | No Content | 删除成功（无响应体） |
| 400 | Bad Request | 请求参数错误 |
| 404 | Not Found | 资源不存在 |
| 422 | Unprocessable Entity | 数据校验失败 |

### 4.2 定义响应模型

用 `response_model` 指定响应的数据结构，FastAPI 会自动过滤字段：

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()


# 数据库中的完整模型
class UserInDB(BaseModel):
    username: str
    email: str
    hashed_password: str  # 密码哈希值
    age: Optional[int] = None


# 返回给前端的模型（不包含密码）
class UserOut(BaseModel):
    username: str
    email: str
    age: Optional[int] = None


# 模拟数据库
fake_db = {
    "alice": UserInDB(username="alice", email="alice@example.com",
                       hashed_password="hashed_xxx", age=25)
}


@app.get("/users/{username}", response_model=UserOut)
def get_user(username: str):
    """
    response_model=UserOut 会自动过滤掉 hashed_password 字段
    即使函数返回了完整数据，响应中也不会包含敏感信息
    """
    user = fake_db.get(username)
    if not user:
        return None  # 返回 404（需配合 HTTPException，后面会学）
    return user
```

---

## 五、表单数据与文件上传

### 5.1 接收表单数据

```python
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login")
def login(username: str = Form(...), password: str = Form(...)):
    """
    接收表单提交的数据（不是 JSON）
    Form(...) 中的 ... 表示必填
    """
    if username == "admin" and password == "123456":
        return {"message": "登录成功", "username": username}
    return {"message": "用户名或密码错误"}
```

测试表单提交：

```python
import requests

# 注意：表单数据用 data= 而非 json=
response = requests.post(
    "http://127.0.0.1:8000/login",
    data={"username": "admin", "password": "123456"}
)
print(response.json())
```

### 5.2 文件上传

```python
from fastapi import FastAPI, UploadFile, File

app = FastAPI()


@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    """上传文件接口"""
    content = await file.read()  # 读取文件内容
    size = len(content)

    # 保存到本地
    with open(f"uploads/{file.filename}", "wb") as f:
        f.write(content)

    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": size
    }


@app.post("/upload/multiple")
async def upload_files(files: list[UploadFile] = File(...)):
    """批量上传文件"""
    results = []
    for file in files:
        content = await file.read()
        results.append({
            "filename": file.filename,
            "size": len(content)
        })
    return {"uploaded": results, "count": len(results)}
```

> **注意**：文件上传操作中使用了 `async def` 而非 `def`。FastAPI 对异步函数有更好的性能支持，特别是涉及 I/O 操作时。后续课程会详细讲解异步编程。

---

## 六、综合实战：待办事项 API

让我们把今天学到的知识整合起来，构建一个完整的待办事项（Todo）REST API。

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

app = FastAPI(title="待办事项 API", version="1.0")


# ===== 数据模型 =====

class TodoCreate(BaseModel):
    """创建待办的请求体"""
    title: str
    description: Optional[str] = None
    priority: int = 1  # 1-3，1 最高


class TodoUpdate(BaseModel):
    """更新待办的请求体（所有字段可选）"""
    title: Optional[str] = None
    description: Optional[str] = None
    priority: Optional[int] = None
    completed: Optional[bool] = None


class TodoResponse(BaseModel):
    """返回给前端的响应模型"""
    id: int
    title: str
    description: Optional[str] = None
    priority: int
    completed: bool
    created_at: str


# ===== 模拟数据库 =====

todos_db: list[dict] = []
next_id = 1


# ===== API 接口 =====

@app.get("/todos", response_model=list[TodoResponse])
def list_todos(
    completed: Optional[bool] = None,
    priority: Optional[int] = None,
    skip: int = 0,
    limit: int = 20
):
    """
    获取待办列表
    - completed: 按完成状态筛选（true/false）
    - priority: 按优先级筛选（1/2/3）
    - skip/limit: 分页参数
    """
    result = todos_db

    # 按条件筛选
    if completed is not None:
        result = [t for t in result if t["completed"] == completed]
    if priority is not None:
        result = [t for t in result if t["priority"] == priority]

    # 分页
    return result[skip : skip + limit]


@app.post("/todos", response_model=TodoResponse, status_code=201)
def create_todo(todo: TodoCreate):
    """创建新的待办事项"""
    global next_id

    new_todo = {
        "id": next_id,
        "title": todo.title,
        "description": todo.description,
        "priority": todo.priority,
        "completed": False,
        "created_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }
    todos_db.append(new_todo)
    next_id += 1
    return new_todo


@app.get("/todos/{todo_id}", response_model=TodoResponse)
def get_todo(todo_id: int):
    """获取单个待办事项"""
    for todo in todos_db:
        if todo["id"] == todo_id:
            return todo
    raise HTTPException(status_code=404, detail=f"待办 {todo_id} 不存在")


@app.put("/todos/{todo_id}", response_model=TodoResponse)
def update_todo(todo_id: int, todo: TodoUpdate):
    """更新待办事项"""
    for existing in todos_db:
        if existing["id"] == todo_id:
            # 只更新传入的字段
            update_data = todo.model_dump(exclude_unset=True)
            existing.update(update_data)
            return existing
    raise HTTPException(status_code=404, detail=f"待办 {todo_id} 不存在")


@app.delete("/todos/{todo_id}")
def delete_todo(todo_id: int):
    """删除待办事项"""
    for i, todo in enumerate(todos_db):
        if todo["id"] == todo_id:
            todos_db.pop(i)
            return {"message": f"待办 {todo_id} 已删除"}
    raise HTTPException(status_code=404, detail=f"待办 {todo_id} 不存在")


@app.get("/todos/stats/summary")
def get_stats():
    """获取待办统计信息"""
    total = len(todos_db)
    completed = sum(1 for t in todos_db if t["completed"])
    pending = total - completed
    return {
        "total": total,
        "completed": completed,
        "pending": pending,
        "completion_rate": f"{completed / total * 100:.1f}%" if total > 0 else "0%"
    }
```

测试完整流程：

```python
import requests

BASE = "http://127.0.0.1:8000"

# 1. 创建待办
r = requests.post(BASE + "/todos", json={
    "title": "学习 FastAPI",
    "description": "完成 Day 46 的学习",
    "priority": 1
})
print("创建:", r.json())

r = requests.post(BASE + "/todos", json={
    "title": "写作业",
    "priority": 2
})
print("创建:", r.json())

# 2. 查询列表
r = requests.get(BASE + "/todos")
print("列表:", r.json())

# 3. 查询未完成的高优先级任务
r = requests.get(BASE + "/todos?completed=false&priority=1")
print("筛选:", r.json())

# 4. 更新任务状态
r = requests.put(BASE + "/todos/1", json={"completed": True})
print("更新:", r.json())

# 5. 查看统计
r = requests.get(BASE + "/todos/stats/summary")
print("统计:", r.json())

# 6. 删除任务
r = requests.delete(BASE + "/todos/2")
print("删除:", r.json())
```

---

## 七、HTTPException 异常处理

在上面的实战中我们用到了 `HTTPException`，这是 FastAPI 处理错误的推荐方式：

```python
from fastapi import HTTPException

# 当资源不存在时抛出异常
raise HTTPException(
    status_code=404,
    detail="你要找的资源不存在",
    headers={"X-Error": "Not Found"}  # 可选：自定义响应头
)
```

客户端会收到：
```json
{
    "detail": "你要找的资源不存在"
}
```

---

## 八、常见问题

### Q1：uvicorn 和 FastAPI 是什么关系？

FastAPI 是 Web 框架（负责路由、参数解析、文档生成等），但它本身不能直接运行——需要一个 **ASGI 服务器**来接收 HTTP 请求并传给 FastAPI 处理。`uvicorn` 就是最常用的 ASGI 服务器，类似于 Flask 的 `werkzeug`。

关系类比：FastAPI = 餐厅菜单和厨师，uvicorn = 餐厅大门和服务员。

### Q2：`def` 和 `async def` 该用哪个？

简单规则：
- 如果路由内部**没有异步操作**（没有 `await`、没有 I/O），用 `def` 即可
- 如果涉及**文件读写、数据库、网络请求**等 I/O 操作，用 `async def` 性能更好
- FastAPI 会自动处理两种情况，用错了也不会报错，只是性能差异

```python
# 简单计算 → 用 def
@app.get("/add")
def add(a: int, b: int):
    return {"result": a + b}

# 涉及 I/O → 用 async def
@app.get("/external")
async def fetch_external():
    async with httpx.AsyncClient() as client:
        resp = await client.get("https://api.example.com/data")
    return resp.json()
```

### Q3：路径参数和查询参数怎么区分？

看函数签名：
- 函数参数出现在**路径中**（如 `/items/{item_id}`）→ 路径参数
- 函数参数**不在路径中**且有默认值或 `Optional` → 查询参数
- 函数参数的类型是 Pydantic `BaseModel` → 请求体

```python
@app.get("/items/{item_id}")  # item_id 在路径中 → 路径参数
def read(item_id: int, q: Optional[str] = None):
    # q 不在路径中 → 查询参数
    pass
```

### Q4：启动服务后修改代码不生效？

确保启动时加了 `--reload` 参数：
```bash
uvicorn main:app --reload
```
这样每次保存文件，uvicorn 会自动重启服务加载新代码。

### Q5：Pydantic 的 `model_dump()` 和 `dict()` 区别？

这是版本差异：
- **Pydantic V2**（推荐，`pip install pydantic` 默认安装）：用 `model_dump()`
- **Pydantic V1**（旧版）：用 `dict()`

安装 FastAPI 时会自动安装 Pydantic V2，所以统一使用 `model_dump()`。

---

## 九、练习题

### 练习一：书籍管理 API（基础）

创建一个书籍管理 API，包含以下接口：

| 方法 | 路径 | 功能 |
|------|------|------|
| GET | `/books` | 获取所有书籍，支持按 `author` 筛选 |
| GET | `/books/{id}` | 根据 ID 获取一本书 |
| POST | `/books` | 添加新书 |
| PUT | `/books/{id}` | 更新书籍信息 |
| DELETE | `/books/{id}` | 删除书籍 |

书籍模型字段：`title`（书名）、`author`（作者）、`price`（价格）、`isbn`（ISBN 编号，可选）。

### 练习二：BMI 计算器 API（进阶）

创建一个 API，接收身高（米）和体重（公斤），计算 BMI 指数并返回健康评估：

```python
# BMI = 体重 / 身高^2
# < 18.5: 偏瘦
# 18.5-24: 正常
# 24-28: 偏胖
# >= 28: 肥胖
```

要求：使用 Pydantic 模型校验输入（身高 0.5-3.0 米，体重 20-300 公斤）。

### 练习三：记事本 API（挑战）

扩展今天的待办事项 API，增加以下功能：
- 按创建时间倒序排列
- 模糊搜索（标题中包含关键词）
- 批量操作接口：`POST /todos/batch` 一次创建多个待办
- 统计接口增加：按优先级分组统计数量

---

## 十、今日总结

今天我们学到了：

| 知识点 | 关键内容 |
|--------|----------|
| FastAPI 简介 | 现代、高性能、自动文档的 Python Web 框架 |
| 创建应用 | `FastAPI()` 实例 + `@app.get/post` 装饰器 |
| 路径参数 | `/items/{id}`，自动类型转换 |
| 查询参数 | `?skip=0&limit=10`，支持默认值和可选 |
| 请求体 | Pydantic `BaseModel`，自动 JSON 解析和校验 |
| 响应模型 | `response_model` 过滤敏感字段 |
| 表单与文件 | `Form()` 和 `UploadFile` |
| 异常处理 | `HTTPException` |
| 自动文档 | `/docs`（Swagger）和 `/redoc` |

下一节（Day 47）我们将学习 **FastAPI 路由与模型进阶**，包括路由分组、数据校验器、嵌套模型等更强大的功能。

---

## 十一、推荐学习资源

- [FastAPI 官方文档（中文）](https://fastapi.tiangolo.com/zh/) — 非常详尽，强烈推荐作为主要参考
- [FastAPI 官方教程](https://fastapi.tiangolo.com/tutorial/) — 跟着一步步做
- [Pydantic V2 文档](https://docs.pydantic.dev/latest/) — 理解数据模型的进阶用法
- [Uvicorn 文档](https://www.uvicorn.org/) — 了解 ASGI 服务器配置
- [廖雪峰 - 异步编程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017954763369216) — 为 Day 47+ 的异步内容做准备
