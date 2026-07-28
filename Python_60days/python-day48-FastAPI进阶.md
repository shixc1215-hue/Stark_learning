# Python Day 48 — FastAPI 进阶

> **学习目标**：掌握 FastAPI 的身份认证（OAuth2 + JWT）、后台任务、中间件深入用法、静态文件服务、WebSocket 基础、自定义响应与文件上传进阶，能够构建生产级 API 应用。

---

## 一、身份认证 — 保护你的 API

### 1.1 为什么需要认证？

上一章我们做了一个待办事项 API，但任何人都能直接访问所有接口。在生产环境中，我们通常需要：

- **身份认证**：确认"你是谁"
- **权限控制**：确认"你能做什么"

FastAPI 内置了对 OAuth2 和 JWT（JSON Web Token）的完整支持。

### 1.2 OAuth2 密码流简介

OAuth2 是一套授权框架，其中"密码流"（Password Flow）适合自有客户端（如自己的前端页面）。流程如下：

```
用户输入账号密码 → 发送到 /token 接口 → 服务器返回 JWT 令牌
→ 后续请求携带令牌 → 服务器验证令牌 → 返回数据
```

### 1.3 JWT 是什么？

JWT（JSON Web Token）是一种紧凑的、自包含的令牌格式，由三部分组成：

```
xxxxx.yyyyy.zzzzz
  │       │       │
  头部   载荷   签名
```

- **头部**：声明加密算法（如 HS256）
- **载荷**：存放用户信息（如 username、过期时间）
- **签名**：用密钥对前两部分签名，防止篡改

### 1.4 完整认证示例

先安装依赖：

```bash
pip install python-jose[cryptography] passlib[bcrypt]
```

```python
# main.py
from datetime import datetime, timedelta
from typing import Optional

from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# ========== 配置 ==========

# 密钥（生产环境请使用环境变量，不要硬编码！）
SECRET_KEY = "your-secret-key-change-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# ========== 密码工具 ==========

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# 模拟用户数据库
fake_users_db = {
    "alice": {
        "username": "alice",
        "hashed_password": pwd_context.hash("123456"),  # 密码：123456
        "role": "admin"
    },
    "bob": {
        "username": "bob",
        "hashed_password": pwd_context.hash("abcdef"),
        "role": "user"
    }
}

# ========== 模型 ==========

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

class User(BaseModel):
    username: str
    role: str

# ========== 认证核心 ==========

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def create_access_token(data: dict):
    """创建 JWT 令牌"""
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: str = Depends(oauth2_scheme)):
    """从令牌中获取当前用户（依赖注入）"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="无效的认证凭据",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = fake_users_db.get(username)
    if user is None:
        raise credentials_exception
    return User(username=user["username"], role=user["role"])

# ========== 应用 ==========

app = FastAPI(title="带认证的 API")

@app.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """登录接口：验证密码并返回令牌"""
    user = fake_users_db.get(form_data.username)
    if not user or not pwd_context.verify(form_data.password, user["hashed_password"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误"
        )
    access_token = create_access_token(data={"sub": user["username"]})
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/me")
async def read_me(current_user: User = Depends(get_current_user)):
    """获取当前用户信息（需要认证）"""
    return current_user

@app.get("/admin/dashboard")
async def admin_dashboard(current_user: User = Depends(get_current_user)):
    """管理员仪表盘（需要 admin 角色）"""
    if current_user.role != "admin":
        raise HTTPException(status_code=403, detail="权限不足")
    return {"message": "欢迎管理员", "data": {"users": 2, "orders": 15}}

@app.get("/public")
async def public_info():
    """公开接口，无需认证"""
    return {"message": "这是公开信息，任何人都能访问"}
```

### 1.5 如何测试认证？

1. 启动服务：`uvicorn main:app --reload`
2. 打开 http://127.0.0.1:8000/docs
3. 点击右上角 **Authorize** 按钮，输入用户名 `alice` 和密码 `123456`
4. 点击登录后，再调用 `/me` 或 `/admin/dashboard` 接口即可

```
# 或者用 curl 测试
# 1. 获取令牌
curl -X POST http://127.0.0.1:8000/token \
  -d "username=alice&password=123456"
# 返回：{"access_token": "eyJhbG...", "token_type": "bearer"}

# 2. 使用令牌访问受保护接口
curl http://127.0.0.1:8000/me \
  -H "Authorization: Bearer eyJhbG..."
```

---

## 二、后台任务（Background Tasks）

### 2.1 什么是后台任务？

有些操作不需要在响应之前完成，比如：

- 发送欢迎邮件（用户注册后）
- 生成报表（用户下单后）
- 记录日志或统计数据

这些操作可以放到"后台"异步执行，让 API 立即返回响应，提升用户体验。

### 2.2 基本用法

```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

def send_email(to: str, subject: str, body: str):
    """模拟发送邮件的耗时操作"""
    time.sleep(3)  # 模拟耗时 3 秒
    print(f"[邮件已发送] 收件人: {to}, 主题: {subject}")

@app.post("/register")
async def register(username: str, email: str, background_tasks: BackgroundTasks):
    """用户注册：立即返回，邮件在后台发送"""
    background_tasks.add_task(send_email, email, "欢迎注册", f"{username} 你好！")
    return {"message": "注册成功！欢迎邮件正在发送中..."}
```

用户调用 `/register` 接口会**立即收到响应**，邮件发送在后台 3 秒后才完成。

### 2.3 后台任务 vs 普通写法的对比

```python
# ❌ 不使用后台任务：用户要等 3 秒才能收到响应
@app.post("/register-slow")
async def register_slow(username: str, email: str):
    send_email(email, "欢迎注册", f"{username} 你好！")  # 阻塞！
    return {"message": "注册成功"}

# ✅ 使用后台任务：用户立即收到响应
@app.post("/register-fast")
async def register_fast(username: str, email: str, bg: BackgroundTasks):
    bg.add_task(send_email, email, "欢迎注册", f"{username} 你好！")
    return {"message": "注册成功"}
```

### 2.4 注意事项

```python
from fastapi import BackgroundTasks

# 1. 任务函数按添加顺序执行（串行）
bg.add_task(task_a)
bg.add_task(task_b)  # 等 task_a 完成后才执行 task_b

# 2. 任务函数可以是普通函数（def）或异步函数（async def）
def sync_task():
    print("同步任务")

async def async_task():
    print("异步任务")

bg.add_task(sync_task)
bg.add_task(async_task)

# 3. 任务函数可以接受参数
def cleanup(file_path: str, delay: int):
    time.sleep(delay)
    print(f"清理文件: {file_path}")

bg.add_task(cleanup, "/tmp/data.json", delay=5)
```

---

## 三、中间件（Middleware）深入

### 3.1 什么是中间件？

中间件是在**每个请求到达路由处理函数之前**和**响应返回给客户端之后**执行的代码。可以理解为请求的"收费站"：

```
请求 → [中间件1] → [中间件2] → [路由处理] → [中间件2] → [中间件1] → 响应
```

### 3.2 自定义中间件

```python
from fastapi import FastAPI, Request
import time

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    """记录每个请求的处理时间"""
    start_time = time.time()

    # 调用下一个处理程序（路由或其他中间件）
    response = await call_next(request)

    # 计算耗时并添加到响应头
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = f"{process_time:.3f}s"
    return response

# 也可以记录请求日志
@app.middleware("http")
async def log_requests(request: Request, call_next):
    """记录所有请求的访问日志"""
    start = time.time()
    print(f"→ {request.method} {request.url.path}")

    response = await call_next(request)

    print(f"← {response.status_code} ({time.time() - start:.3f}s)")
    return response
```

### 3.3 CORS 跨域配置详解

当你的前端（如 Vue/React）和后端（FastAPI）在不同端口或域名运行时，需要配置 CORS：

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# 添加 CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=[                          # 允许的源
        "http://localhost:3000",              # 前端开发服务器
        "http://localhost:5173",
        "https://myapp.example.com",          # 生产域名
    ],
    allow_credentials=True,                   # 允许携带 Cookie
    allow_methods=["*"],                       # 允许所有 HTTP 方法
    allow_headers=["*"],                       # 允许所有请求头
)
```

> **安全提醒**：生产环境中 `allow_origins` 不要使用 `["*"]`，应明确列出允许的域名。

### 3.4 中间件执行顺序

中间件按**注册顺序的逆序**执行：

```python
app.add_middleware(Middleware_A)  # 后执行
app.add_middleware(Middleware_B)  # 先执行

# 实际执行顺序：
# 请求 → Middleware_B → Middleware_A → 路由
# 响应 → Middleware_A → Middleware_B → 客户端
```

---

## 四、静态文件服务

### 4.1 为什么需要静态文件服务？

API 经常需要提供静态资源，比如：

- 前端页面（HTML/CSS/JS）
- 图片、文件下载
- API 文档的 Logo

### 4.2 挂载静态文件目录

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# 挂载静态文件目录
# 访问 http://localhost:8000/static/logo.png → 读取 ./static/logo.png
app.mount("/static", StaticFiles(directory="static"), name="static")

# 挂载前端构建产物（SPA 应用）
app.mount("/", StaticFiles(directory="dist", html=True), name="spa")
```

目录结构：

```
project/
├── main.py
├── static/           ← 静态资源目录
│   ├── logo.png
│   └── style.css
└── dist/             ← 前端构建输出
    ├── index.html
    └── app.js
```

> **注意**：`mount` 的路由放在最后，因为它是"兜底"路由，会匹配所有子路径。

---

## 五、自定义响应类型

### 5.1 FastAPI 默认返回 JSON

之前所有接口都自动返回 JSON 格式。但有时你需要返回 HTML 页面、纯文本或文件。

### 5.2 常用响应类型

```python
from fastapi import FastAPI
from fastapi.responses import (
    JSONResponse,
    HTMLResponse,
    PlainTextResponse,
    FileResponse,
    StreamingResponse,
)
from fastapi.encoders import jsonable_encoder

app = FastAPI()

@app.get("/json")
async def return_json():
    """默认就是 JSON 响应，这个写法等价于直接返回 dict"""
    return JSONResponse(content={"message": "hello"})

@app.get("/html", response_class=HTMLResponse)
async def return_html():
    """返回 HTML 页面"""
    return """
    <!DOCTYPE html>
    <html>
    <head><title>FastAPI</title></head>
    <body><h1>Hello from FastAPI!</h1></body>
    </html>
    """

@app.get("/text", response_class=PlainTextResponse)
async def return_text():
    """返回纯文本"""
    return "这是纯文本内容，不是 JSON"

@app.get("/download")
async def download_file():
    """下载文件"""
    return FileResponse(
        path="static/report.pdf",
        filename="月度报告.pdf",      # 用户看到的文件名
        media_type="application/pdf",
    )
```

### 5.3 流式响应

```python
import asyncio

@app.get("/stream")
async def stream_data():
    """流式返回数据（适合大文件或实时数据）"""
    async def generate():
        for i in range(10):
            yield f"第 {i+1} 条数据\n"
            await asyncio.sleep(0.5)

    return StreamingResponse(
        generate(),
        media_type="text/plain",
    )
```

### 5.4 设置响应状态码和头

```python
@app.post("/items", status_code=201)  # 创建成功返回 201
async def create_item(name: str):
    return {"name": name, "id": 1}

@app.delete("/items/{item_id}", status_code=204)  # 删除成功返回 204（无内容）
async def delete_item(item_id: int):
    return None

from fastapi import Response

@app.get("/custom-headers")
async def custom_headers(response: Response):
    response.headers["X-Custom-Header"] = "my-value"
    response.set_cookie("session_id", "abc123", max_age=3600)
    return {"message": "已设置自定义头和 Cookie"}
```

---

## 六、文件上传进阶

### 6.1 单文件上传

```python
from fastapi import FastAPI, UploadFile, File, HTTPException
import os

app = FastAPI()

UPLOAD_DIR = "uploads"
os.makedirs(UPLOAD_DIR, exist_ok=True)

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    """上传单个文件"""
    # 验证文件类型
    allowed_types = {"image/jpeg", "image/png", "image/gif"}
    if file.content_type not in allowed_types:
        raise HTTPException(400, f"不支持的文件类型：{file.content_type}")

    # 验证文件大小（1MB 限制）
    contents = await file.read()
    if len(contents) > 1 * 1024 * 1024:
        raise HTTPException(400, "文件大小不能超过 1MB")

    # 保存文件
    save_path = os.path.join(UPLOAD_DIR, file.filename)
    with open(save_path, "wb") as f:
        f.write(contents)

    return {
        "filename": file.filename,
        "size": len(contents),
        "content_type": file.content_type
    }
```

### 6.2 多文件上传

```python
from typing import List

@app.post("/upload-many")
async def upload_many(files: List[UploadFile] = File(...)):
    """批量上传多个文件"""
    results = []
    for file in files:
        contents = await file.read()
        save_path = os.path.join(UPLOAD_DIR, file.filename)
        with open(save_path, "wb") as f:
            f.write(contents)
        results.append({"filename": file.filename, "size": len(contents)})

    return {"uploaded": len(results), "files": results}
```

### 6.3 文件上传 + 表单数据

```python
from fastapi import Form

@app.post("/upload-with-info")
async def upload_with_info(
    title: str = Form(...),
    description: str = Form(default=""),
    file: UploadFile = File(...),
):
    """同时上传文件和表单数据"""
    return {
        "title": title,
        "description": description,
        "filename": file.filename,
        "size": len(await file.read())
    }
```

---

## 七、WebSocket 基础

### 7.1 什么是 WebSocket？

HTTP 是"请求-响应"模式，每次都要客户端主动请求。WebSocket 则是**全双工通信**，建立连接后双方可以随时发送消息，非常适合：

- 聊天应用
- 实时通知
- 协同编辑
- 在线游戏

### 7.2 FastAPI WebSocket 示例

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List

app = FastAPI()

# 存储所有活跃连接
active_connections: List[WebSocket] = []

@app.websocket("/ws/chat")
async def websocket_chat(websocket: WebSocket):
    """WebSocket 聊天室"""
    await websocket.accept()       # 接受连接
    active_connections.append(websocket)

    try:
        while True:
            # 等待客户端发来的消息
            data = await websocket.receive_text()

            # 广播给所有其他连接
            for connection in active_connections:
                if connection != websocket:
                    await connection.send_text(f"他人: {data}")
    except WebSocketDisconnect:
        active_connections.remove(websocket)
        await broadcast(f"一位用户离开了聊天室（当前 {len(active_connections)} 人）")

async def broadcast(message: str):
    """向所有连接广播消息"""
    for connection in active_connections:
        await connection.send_text(message)
```

### 7.3 测试 WebSocket

```python
# test_ws.html — 在浏览器中打开
<!DOCTYPE html>
<html>
<body>
    <h1>WebSocket 聊天测试</h1>
    <input id="msg" placeholder="输入消息...">
    <button onclick="send()">发送</button>
    <div id="log" style="height:300px;overflow:auto;border:1px solid #ccc;padding:10px;"></div>

    <script>
        const ws = new WebSocket("ws://127.0.0.1:8000/ws/chat");
        const log = document.getElementById("log");

        ws.onmessage = (event) => {
            log.innerHTML += "<p>" + event.data + "</p>";
        };

        function send() {
            const msg = document.getElementById("msg").value;
            ws.send(msg);
            log.innerHTML += "<p>我: " + msg + "</p>";
        }
    </script>
</body>
</html>
```

---

## 八、测试 FastAPI 应用

### 8.1 使用 TestClient

FastAPI 提供了 `TestClient`，可以在不启动服务器的情况下测试 API：

```python
# test_api.py
from fastapi.testclient import TestClient
from main import app          # 导入你的 FastAPI 实例

client = TestClient(app)

def test_public():
    """测试公开接口"""
    response = client.get("/public")
    assert response.status_code == 200
    assert response.json()["message"] == "这是公开信息"

def test_login():
    """测试登录"""
    response = client.post("/token", data={"username": "alice", "password": "123456"})
    assert response.status_code == 200
    token = response.json()["access_token"]
    assert token  # 令牌不为空

def test_me_with_token():
    """测试带认证的接口"""
    # 先登录获取令牌
    login_resp = client.post("/token", data={"username": "alice", "password": "123456"})
    token = login_resp.json()["access_token"]

    # 携带令牌访问受保护接口
    response = client.get("/me", headers={"Authorization": f"Bearer {token}"})
    assert response.status_code == 200
    assert response.json()["username"] == "alice"

def test_me_without_token():
    """测试未认证时返回 401"""
    response = client.get("/me")
    assert response.status_code == 401
```

运行测试：

```bash
pip install httpx pytest
pytest test_api.py -v
```

---

## 九、综合实战 — 带认证的博客系统 API

```python
# blog_api.py
from datetime import datetime, timedelta
from typing import Optional, List
from fastapi import FastAPI, Depends, HTTPException, status, BackgroundTasks
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel, Field

app = FastAPI(title="博客系统 API", version="1.0")

# ========== 配置 ==========
SECRET_KEY = "blog-secret-key-change-me"
ALGORITHM = "HS256"

# ========== 密码工具 ==========
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# ========== 模型 ==========
class UserCreate(BaseModel):
    username: str = Field(min_length=3, max_length=20)
    password: str = Field(min_length=6, max_length=50)
    email: str = Field(pattern=r"^[\w.-]+@[\w.-]+\.\w+$")

class ArticleCreate(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    content: str = Field(min_length=10)
    tags: List[str] = Field(default_factory=list)

class ArticleOut(BaseModel):
    id: int
    title: str
    content: str
    tags: List[str]
    author: str
    created_at: str

# ========== 模拟数据库 ==========
users_db = {}
articles_db = {}
next_article_id = 1

# ========== 工具函数 ==========
def create_token(username: str):
    expire = datetime.utcnow() + timedelta(hours=24)
    return jwt.encode({"sub": username, "exp": expire}, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
    except JWTError:
        raise HTTPException(status_code=401, detail="无效令牌")
    if username not in users_db:
        raise HTTPException(status_code=401, detail="用户不存在")
    return username

# ========== 接口 ==========

@app.post("/register")
async def register(user: UserCreate):
    if user.username in users_db:
        raise HTTPException(400, "用户名已存在")
    users_db[user.username] = {
        "username": user.username,
        "hashed_password": pwd_context.hash(user.password),
        "email": user.email,
    }
    return {"message": "注册成功", "username": user.username}

@app.post("/token")
async def login(form_data: OAuth2PasswordBearer = Depends()):
    raise HTTPException(501, "请使用 /token 端点的表单登录")

# 简化登录
from fastapi.security import OAuth2PasswordRequestForm

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = users_db.get(form_data.username)
    if not user or not pwd_context.verify(form_data.password, user["hashed_password"]):
        raise HTTPException(401, "用户名或密码错误")
    token = create_token(user["username"])
    return {"access_token": token, "token_type": "bearer"}

@app.post("/articles", response_model=ArticleOut)
async def create_article(article: ArticleCreate, username: str = Depends(get_current_user)):
    """发布文章（需登录）"""
    global next_article_id
    aid = next_article_id
    next_article_id += 1
    articles_db[aid] = {
        "id": aid,
        "title": article.title,
        "content": article.content,
        "tags": article.tags,
        "author": username,
        "created_at": datetime.now().isoformat(),
    }
    return articles_db[aid]

@app.get("/articles", response_model=List[ArticleOut])
async def list_articles(tag: Optional[str] = None):
    """获取文章列表（支持按标签筛选）"""
    result = list(articles_db.values())
    if tag:
        result = [a for a in result if tag in a["tags"]]
    return result

@app.get("/articles/{article_id}", response_model=ArticleOut)
async def get_article(article_id: int):
    """获取单篇文章"""
    if article_id not in articles_db:
        raise HTTPException(404, "文章不存在")
    return articles_db[article_id]

@app.delete("/articles/{article_id}")
async def delete_article(article_id: int, username: str = Depends(get_current_user)):
    """删除文章（仅作者本人可删除）"""
    article = articles_db.get(article_id)
    if not article:
        raise HTTPException(404, "文章不存在")
    if article["author"] != username:
        raise HTTPException(403, "只能删除自己的文章")
    del articles_db[article_id]
    return {"message": "删除成功"}
```

---

## 十、练习题

### 练习 1：带令牌刷新的认证系统

在上述认证示例中增加一个 `/refresh` 接口，用户传入旧令牌后返回新令牌（延长有效期）。

```python
@app.post("/refresh")
async def refresh_token(token: str = Depends(oauth2_scheme)):
    # 提示：先验证旧令牌，然后创建新令牌返回
    pass
```

### 练习 2：文件转换服务

创建一个 API，接受文本文件上传，返回文件的字数统计（总字数、行数、最常见的前 5 个词）。

### 练习 3：WebSocket 实时通知系统

扩展聊天示例，加入"系统通知"功能：新用户加入时广播欢迎消息，实现用户上下线通知。

---

## 十一、常见问题（FAQ）

**Q1：JWT 令牌过期了怎么办？**

客户端收到 401 状态码后，应提示用户重新登录。生产项目中通常配合 Refresh Token 实现"静默续期"。

**Q2：后台任务失败了怎么办？**

FastAPI 的 BackgroundTasks 执行失败不会影响已发送的响应。如果需要可靠性保证，应使用消息队列（如 Celery + Redis）。

**Q3：中间件可以访问请求体吗？**

不建议。在中间件中读取请求体后，路由处理函数就无法再次读取。如需修改请求体，请使用依赖注入。

**Q4：生产环境应该怎么部署 FastAPI？**

推荐使用 Gunicorn + Uvicorn worker：

```bash
pip install gunicorn
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker
```

**Q5：FastAPI 和 Flask 怎么选？**

| 对比项 | FastAPI | Flask |
|--------|---------|-------|
| 性能 | 高（异步） | 中等（同步） |
| 类型提示 | 原生支持 | 需要扩展 |
| 自动文档 | 内置 Swagger | 需要插件 |
| 学习曲线 | 稍陡 | 平缓 |
| 生态成熟度 | 发展中 | 非常成熟 |

新项目推荐 FastAPI；已有 Flask 项目或简单场景可以选择 Flask。

---

## 十二、免费学习资源

- **FastAPI 官方文档**（强烈推荐）：https://fastapi.tiangolo.com/zh/
- **Real Python — FastAPI Tutorial**：https://realpython.com/fastapi-python-web-frames/
- **FastAPI 中文教程**：https://fastapi.tiangolo.com/zh/
- **OAuth2 简介图解**：https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html
- **JWT 入门教程**：https://www.ruanyifeng.com/blog/2018/07/json_web_token_tutorial.html

---

> **下一期预告**：Day 49 — 数据库操作 SQLite，学习使用 Python 内置的 SQLite 数据库进行数据的增删改查。
