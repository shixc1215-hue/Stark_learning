# Python Day 47 — FastAPI 路由与模型进阶

> **学习目标**：掌握 FastAPI 的路由组织（APIRouter）、Pydantic 模型进阶（校验器、嵌套模型、字段约束）、响应模型高级用法，学会编写结构清晰的大型 API 项目。

---

## 一、为什么需要路由组织？

### 1.1 单文件的问题

在 Day 46 中，我们把所有接口都写在一个文件里。当项目只有几个接口时没问题，但真实项目通常有几十上百个接口，全部堆在一个文件会导致：

- **代码混乱**：找某个接口要在几百行代码中翻找
- **协作困难**：多人同时修改一个文件容易冲突
- **难以维护**：功能边界不清晰

### 1.2 APIRouter — 路由分组方案

FastAPI 提供了 `APIRouter` 来解决这个问题。它的思路和 Python 的"模块化"一致——把相关功能的路由放在同一个模块中，最后在主应用中"注册"。

```
项目结构：
main.py              ← 主入口，组装路由
├── routers/
│   ├── users.py     ← 用户相关路由
│   ├── items.py     ← 物品相关路由
│   └── orders.py    ← 订单相关路由
├── models.py        ← 数据模型
└── database.py      ← 数据库连接
```

---

## 二、APIRouter 基础用法

### 2.1 创建路由模块

```python
# routers/users.py
from fastapi import APIRouter

# 创建路由器实例
router = APIRouter()

# 使用 router.xxx 代替 app.xxx
@router.get("/users")
def list_users():
    """获取所有用户"""
    return [{"id": 1, "name": "小明"}, {"id": 2, "name": "小红"}]


@router.get("/users/{user_id}")
def get_user(user_id: int):
    """根据 ID 获取用户"""
    return {"id": user_id, "name": f"用户-{user_id}"}


@router.post("/users")
def create_user(name: str, age: int = 18):
    """创建用户"""
    return {"id": 999, "name": name, "age": age}
```

### 2.2 在主应用中注册路由

```python
# main.py
from fastapi import FastAPI
from routers import users  # 导入路由模块

app = FastAPI(title="我的API", version="1.0")

# 注册路由（include_router）
app.include_router(users.router)
```

### 2.3 路由前缀和标签

注册路由时可以指定**公共前缀**和**标签**，避免每个接口重复写路径：

```python
# routers/users.py — 路由路径去掉公共前缀
from fastapi import APIRouter

router = APIRouter(prefix="/users", tags=["用户管理"])


@router.get("/")            # 实际路径: GET /users
def list_users():
    return [{"id": 1, "name": "小明"}]


@router.get("/{user_id}")   # 实际路径: GET /users/{user_id}
def get_user(user_id: int):
    return {"id": user_id, "name": f"用户-{user_id}"}


@router.post("/")           # 实际路径: POST /users
def create_user(name: str):
    return {"id": 999, "name": name}
```

```python
# main.py — 注册时无需再加前缀
from fastapi import FastAPI
from routers import users

app = FastAPI()
app.include_router(users.router)
```

### 2.4 多模块注册示例

```python
# main.py
from fastapi import FastAPI
from routers import users, items, orders

app = FastAPI(title="电商API", version="2.0")

# 批量注册，每个模块自带前缀和标签
app.include_router(users.router)    # /users/*   → 标签: 用户管理
app.include_router(items.router)    # /items/*   → 标签: 物品管理
app.include_router(orders.router)   # /orders/*  → 标签: 订单管理


@app.get("/")
def root():
    return {"message": "欢迎使用电商 API", "docs": "/docs"}
```

在 `/docs` 页面中，接口会按标签自动分组，非常清晰。

### 2.5 APIRouter 的响应和状态码

路由器上也可以统一设置默认状态码：

```python
from fastapi import APIRouter

# 创建路由器时设置默认响应
router = APIRouter(
    prefix="/items",
    tags=["物品管理"],
    responses={404: {"description": "未找到资源"}}  # 统一 404 响应格式
)
```

---

## 三、Pydantic 模型进阶

### 3.1 字段约束 — Field()

Pydantic 的 `Field()` 可以对字段添加丰富的约束条件，比单纯的类型注解强大得多：

```python
from pydantic import BaseModel, Field

class UserCreate(BaseModel):
    """用户创建模型"""
    username: str = Field(
        min_length=3,
        max_length=20,
        pattern=r"^[a-zA-Z0-9_]+$",  # 只允许字母、数字、下划线
        description="用户名，3-20个字符，仅支持字母数字下划线"
    )
    email: str = Field(
        ...,
        description="邮箱地址",
        pattern=r"^[\w.-]+@[\w.-]+\.\w+$"  # 简单邮箱正则
    )
    age: int = Field(
        default=18,
        ge=1,        # greater than or equal
        le=150,      # less than or equal
        description="年龄，1-150"
    )
    score: float = Field(
        default=0.0,
        ge=0.0,
        le=100.0,
        description="分数，0-100"
    )
    bio: str | None = Field(
        default=None,
        max_length=500,
        description="个人简介，最多500字"
    )
```

常用约束参数速查：

| 参数 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `...` | — | 必填标记 | `name: str = Field(...)` |
| `default` | any | 默认值 | `age: int = Field(default=18)` |
| `min_length` | int | 字符串最小长度 | `min_length=6` |
| `max_length` | int | 字符串最大长度 | `max_length=100` |
| `ge` | int/float | 大于等于 | `ge=0` |
| `gt` | int/float | 大于 | `gt=0` |
| `le` | int/float | 小于等于 | `le=100` |
| `lt` | int/float | 小于 | `lt=100` |
| `pattern` | str | 正则匹配 | `pattern=r"^\d+$"` |
| `description` | str | 字段描述（出现在文档中） | `description="用户名"` |
| `example` | any | 示例值 | `example="张三"` |
| `title` | str | 字段标题 | `title="Username"` |

### 3.2 数据校验 — validator

当简单的字段约束不够用时，可以用 `field_validator` 编写自定义校验逻辑：

```python
from pydantic import BaseModel, field_validator, model_validator

class UserRegister(BaseModel):
    username: str = Field(min_length=3, max_length=20)
    password: str = Field(min_length=6, description="密码，至少6位")
    confirm_password: str = Field(description="确认密码")
    email: str

    @field_validator("username")
    @classmethod
    def username_must_not_contain_space(cls, v: str) -> str:
        """用户名不能包含空格"""
        if " " in v:
            raise ValueError("用户名不能包含空格")
        return v.lower()  # 自动转为小写

    @field_validator("email")
    @classmethod
    def email_must_be_valid(cls, v: str) -> str:
        """邮箱必须包含 @"""
        if "@" not in v or "." not in v.split("@")[-1]:
            raise ValueError("请输入有效的邮箱地址")
        return v

    @model_validator(mode="after")
    def passwords_must_match(self) -> "UserRegister":
        """密码和确认密码必须一致"""
        if self.password != self.confirm_password:
            raise ValueError("两次输入的密码不一致")
        return self
```

测试效果：

```python
# 合法数据
user = UserRegister(
    username="Alice",
    password="123456",
    confirm_password="123456",
    email="alice@example.com"
)
print(user.username)  # "alice"（自动转小写）

# 非法数据 — 会抛出 ValidationError
try:
    UserRegister(
        username="A B",           # 含空格
        password="123",          # 太短
        confirm_password="456",  # 不一致
        email="invalid"          # 无效邮箱
    )
except Exception as e:
    print(e)
    # 会列出所有校验错误，非常清晰
```

### 3.3 嵌套模型

在实际项目中，数据往往是嵌套的。比如一个订单包含多个商品：

```python
from pydantic import BaseModel, Field
from typing import Optional

class Address(BaseModel):
    """地址模型"""
    province: str = Field(..., description="省份")
    city: str = Field(..., description="城市")
    district: str = Field(..., description="区县")
    detail: str = Field(..., description="详细地址")
    zip_code: Optional[str] = Field(None, pattern=r"\d{6}")


class ProductItem(BaseModel):
    """商品条目"""
    product_id: int
    name: str
    price: float = Field(ge=0)
    quantity: int = Field(ge=1, description="购买数量")


class OrderCreate(BaseModel):
    """创建订单 — 嵌套了 Address 和 ProductItem"""
    user_id: int
    address: Address          # 嵌套模型
    items: list[ProductItem]  # 嵌套模型列表
    remark: Optional[str] = None
```

在 FastAPI 中直接使用嵌套模型，文档会自动展示层级结构：

```python
from fastapi import FastAPI, APIRouter

router = APIRouter(prefix="/orders", tags=["订单"])


@router.post("/", status_code=201)
def create_order(order: OrderCreate):
    """创建订单"""
    total = sum(item.price * item.quantity for item in order.items)
    return {
        "order_id": 1001,
        "total_price": total,
        "item_count": len(order.items),
        "address": order.address.model_dump()
    }
```

客户端发送的 JSON：

```json
{
    "user_id": 1,
    "address": {
        "province": "浙江省",
        "city": "杭州市",
        "district": "西湖区",
        "detail": "文三路 100 号",
        "zip_code": "310000"
    },
    "items": [
        {"product_id": 1, "name": "Python书", "price": 79.9, "quantity": 2},
        {"product_id": 2, "name": "键盘", "price": 299.0, "quantity": 1}
    ],
    "remark": "请尽快发货"
}
```

### 3.4 模型继承

多个模型有共同字段时，用继承避免重复：

```python
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

# ===== 基类 — 公共字段 =====

class ItemBase(BaseModel):
    """物品公共字段"""
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., ge=0)
    description: Optional[str] = Field(None, max_length=1000)


# ===== 输入模型 =====

class ItemCreate(ItemBase):
    """创建物品 — 继承基类，可增加创建时的专属字段"""
    stock: int = Field(default=0, ge=0, description="库存数量")


class ItemUpdate(BaseModel):
    """更新物品 — 所有字段可选"""
    name: Optional[str] = Field(None, min_length=1, max_length=100)
    price: Optional[float] = Field(None, ge=0)
    description: Optional[str] = Field(None, max_length=1000)
    stock: Optional[int] = Field(None, ge=0)


# ===== 输出模型 =====

class ItemResponse(ItemBase):
    """返回给前端的模型 — 包含数据库生成的字段"""
    id: int
    stock: int
    created_at: datetime
```

使用模型继承后，代码层次清晰，修改公共字段只需改一处。

### 3.5 配置类 — model_config

通过 `model_config` 可以控制模型的全局行为：

```python
from pydantic import BaseModel, Field, ConfigDict

class StrictItem(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,  # 自动去除字符串首尾空格
        str_to_lower=True,           # 字符串自动转小写
        frozen=False,                # 是否可修改（True=不可变）
        populate_by_name=True,      # 允许通过字段名或别名赋值
        extra="forbid",              # 禁止额外字段（默认 ignore）
    )

    name: str = Field(alias="itemName")  # 别名：JSON 中用 itemName，代码中用 name
    price: float
```

测试效果：

```python
# 使用别名传参
item = StrictItem(itemName="  LAPTOP  ", price=5999.0)
print(item.name)    # "laptop"（去空格+转小写）
print(item.price)   # 5999.0

# 禁止额外字段
try:
    StrictItem(itemName="手机", price=999, color="红色")  # color 是额外字段
except Exception as e:
    print("拒绝额外字段:", e)
```

---

## 四、响应模型高级用法

### 4.1 不同状态码返回不同模型

同一个接口可能在不同情况下返回不同的数据结构：

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Union

app = FastAPI()


class SuccessResponse(BaseModel):
    """成功响应"""
    code: int = 0
    message: str = "success"
    data: dict


class ErrorResponse(BaseModel):
    """错误响应"""
    code: int
    message: str
    detail: str | None = None


@app.get(
    "/items/{item_id}",
    responses={
        200: {"model": SuccessResponse, "description": "成功"},
        404: {"model": ErrorResponse, "description": "未找到"}
    }
)
def get_item(item_id: int):
    if item_id > 10:
        return ErrorResponse(code=404, message="not found", detail="超出范围")
    return SuccessResponse(code=0, message="ok", data={"id": item_id})
```

### 4.2 response_model_exclude_unset

默认情况下 `response_model` 会返回模型的所有字段（包括默认值）。使用 `exclude_unset` 只返回实际设置了值的字段：

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()


class ItemUpdate(BaseModel):
    name: Optional[str] = None
    price: Optional[float] = None
    stock: Optional[int] = None


@app.patch("/items/{item_id}", response_model=ItemUpdate)
def partial_update(item_id: int, item: ItemUpdate):
    """
    PATCH 部分更新 — 只返回用户实际修改的字段
    response_model_exclude_unset=True 只序列化设置了值的字段
    """
    updated_data = item.model_dump(exclude_unset=True)
    return ItemUpdate(**updated_data)  # 只包含有值的字段
```

---

## 五、依赖注入（Dependency Injection）

### 5.1 什么是依赖注入？

依赖注入是 FastAPI 的核心设计模式之一。简单说：**一个接口需要什么数据/服务，通过参数声明，FastAPI 自动提供**。

这在实际场景中非常有用，比如：多个接口都需要验证用户登录 → 把登录验证做成一个"依赖"，各接口直接使用。

### 5.2 Depends() 基础用法

```python
from fastapi import FastAPI, Depends, HTTPException, Header

app = FastAPI()


# ===== 定义依赖函数 =====

def verify_token(authorization: str = Header(...)):
    """
    验证 Token 的依赖
    Header(...) 从请求头中提取 Authorization 字段
    """
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="无效的认证格式")
    token = authorization.replace("Bearer ", "")
    if token != "my-secret-token":
        raise HTTPException(status_code=403, detail="Token 无效")
    return {"user": "admin", "token": token}


def pagination(skip: int = 0, limit: int = 20):
    """分页参数依赖"""
    return {"skip": skip, "limit": min(limit, 100)}  # 限制最大值


# ===== 使用依赖 =====

@app.get("/protected")
def protected_route(auth: dict = Depends(verify_token)):
    """
    通过 Depends 注入 verify_token
    FastAPI 会先执行 verify_token，把返回值传给 auth
    如果 verify_token 抛出异常，直接返回错误，不执行路由逻辑
    """
    return {"message": f"欢迎回来，{auth['user']}"}


@app.get("/items")
def list_items(p: dict = Depends(pagination)):
    """使用分页依赖"""
    return {"items": ["a", "b", "c"][p["skip"]:p["skip"]+p["limit"]]}
```

### 5.3 使用类作为依赖

当依赖逻辑较复杂时，推荐用类（更易维护和测试）：

```python
from fastapi import FastAPI, Depends, HTTPException

app = FastAPI()


class AuthService:
    """认证服务类"""

    def __init__(self, api_key: str = ""):
        self.api_key = api_key

    def verify(self) -> dict:
        valid_keys = {"key-001": "admin", "key-002": "editor"}
        if self.api_key not in valid_keys:
            raise HTTPException(status_code=403, detail="API Key 无效")
        return {"role": valid_keys[self.api_key]}


# 把 AuthService 变成可注入的依赖
def get_auth_service(api_key: str = "") -> AuthService:
    return AuthService(api_key)


@app.get("/dashboard")
def dashboard(auth: AuthService = Depends(get_auth_service)):
    """类依赖注入"""
    role_info = auth.verify()
    return {"message": f"你的角色是: {role_info['role']}"}
```

---

## 六、中间件（Middleware）

### 6.1 什么是中间件？

中间件是在**每个请求到达路由之前和之后**执行的代码。典型用途：

- 记录请求日志
- 添加 CORS 跨域支持
- 请求计时
- 全局异常处理

### 6.2 添加中间件

```python
from fastapi import FastAPI, Request
import time
import logging

app = FastAPI()

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


# 请求计时中间件
@app.middleware("http")
async def log_requests(request: Request, call_next):
    """
    每个请求经过这个函数两次：
    1. call_next 之前 → 请求到达路由前
    2. call_next 之后 → 路由处理完后
    """
    start_time = time.time()

    # 记录请求信息
    logger.info(f"→ {request.method} {request.url.path}")

    # 执行路由处理
    response = await call_next(request)

    # 计算耗时
    duration = (time.time() - start_time) * 1000
    logger.info(f"← {request.method} {request.url.path} — {duration:.1f}ms [{response.status_code}]")

    # 在响应头中添加耗时
    response.headers["X-Process-Time"] = f"{duration:.1f}ms"
    return response


@app.get("/slow")
def slow_endpoint():
    """模拟耗时接口"""
    time.sleep(1)
    return {"message": "这个接口很慢"}
```

### 6.3 CORS 跨域配置

前后端分离项目中，前端（如 Vue/React）和后端（FastAPI）通常在不同端口运行，需要配置 CORS：

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# 添加 CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "http://localhost:5173"],  # 允许的前端地址
    allow_credentials=True,
    allow_methods=["*"],     # 允许所有 HTTP 方法
    allow_headers=["*"],     # 允许所有请求头
)
```

> **安全提示**：生产环境中 `allow_origins` 不要使用 `["*"]`，应明确指定前端域名。

---

## 七、综合实战：博客系统 API

把今天所有知识点串联起来，构建一个模块化的博客系统 API。

### 7.1 项目结构

```
blog_project/
├── main.py
├── models.py
├── database.py
└── routers/
    ├── __init__.py
    ├── posts.py
    └── comments.py
```

### 7.2 数据模型

```python
# models.py
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime


# ===== 地址/嵌套示例 =====

class AuthorInfo(BaseModel):
    """作者信息（嵌套模型）"""
    author_id: int
    name: str
    avatar: Optional[str] = None


# ===== 基类 + 继承 =====

class PostBase(BaseModel):
    """文章公共字段"""
    title: str = Field(..., min_length=1, max_length=200, description="文章标题")
    content: str = Field(..., min_length=10, description="文章内容")
    tags: list[str] = Field(default_factory=list, description="标签列表")


class PostCreate(PostBase):
    """创建文章"""
    author_id: int = Field(..., description="作者ID")


class PostUpdate(BaseModel):
    """更新文章（所有字段可选）"""
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    content: Optional[str] = Field(None, min_length=10)
    tags: Optional[list[str]] = None


class PostResponse(PostBase):
    """文章响应（包含数据库生成的字段）"""
    id: int
    author: AuthorInfo
    views: int = 0
    created_at: datetime
    updated_at: Optional[datetime] = None


# ===== 评论模型 =====

class CommentCreate(BaseModel):
    """创建评论"""
    post_id: int = Field(..., description="文章ID")
    content: str = Field(..., min_length=1, max_length=1000, description="评论内容")


class CommentResponse(BaseModel):
    """评论响应"""
    id: int
    post_id: int
    content: str
    created_at: datetime
```

### 7.3 文章路由

```python
# routers/posts.py
from fastapi import APIRouter, HTTPException, Depends, Query
from typing import Optional

router = APIRouter(prefix="/posts", tags=["文章管理"])

# 模拟数据库
posts_db: list[dict] = []
authors_db = {1: {"author_id": 1, "name": "小明", "avatar": "avatar1.jpg"},
              2: {"author_id": 2, "name": "小红", "avatar": "avatar2.jpg"}}
next_post_id = 1


def get_post_or_404(post_id: int) -> dict:
    """复用的查询依赖：查找文章，不存在则 404"""
    for post in posts_db:
        if post["id"] == post_id:
            return post
    raise HTTPException(status_code=404, detail=f"文章 {post_id} 不存在")


@router.get("/")
def list_posts(
    tag: Optional[str] = Query(None, description="按标签筛选"),
    author_id: Optional[int] = Query(None, description="按作者筛选"),
    skip: int = Query(0, ge=0),
    limit: int = Query(20, ge=1, le=100)
):
    """获取文章列表"""
    result = posts_db
    if tag:
        result = [p for p in result if tag in p["tags"]]
    if author_id:
        result = [p for p in result if p["author"]["author_id"] == author_id]

    total = len(result)
    return {
        "posts": result[skip:skip + limit],
        "total": total,
        "page": skip // limit + 1
    }


@router.post("/", status_code=201)
def create_post(post: dict):  # 实际使用 PostCreate 模型
    """创建文章"""
    global next_post_id
    # ... 创建逻辑
    return {"id": next_post_id, "message": "创建成功"}


@router.get("/{post_id}")
def get_post(post: dict = Depends(get_post_or_404)):
    """获取文章详情 — 使用依赖注入"""
    return post


@router.delete("/{post_id}")
def delete_post(post: dict = Depends(get_post_or_404)):
    """删除文章"""
    posts_db.remove(post)
    return {"message": "删除成功"}
```

### 7.4 主应用

```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from routers import posts

app = FastAPI(title="博客系统 API", version="1.0")

# CORS 跨域支持
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# 注册路由
app.include_router(posts.router)


@app.get("/")
def root():
    return {
        "name": "博客系统 API",
        "version": "1.0",
        "docs": "/docs"
    }
```

---

## 八、常见问题

### Q1：APIRouter 和 app 直接写路由有什么区别？

功能上没有区别，`APIRouter` 的路由最终会注册到 `app` 上。区别在于**组织方式**：
- `app.get()` → 适合只有几个接口的小项目
- `APIRouter` → 适合中大型项目，按功能模块拆分

两者可以混用——主应用本身也可以直接定义路由（如根路径 `/`）。

### Q2：嵌套模型中的校验会自动生效吗？

是的。FastAPI/Pydantic 会递归校验嵌套模型的每个字段。如果 `OrderCreate` 包含 `Address`，那么 `Address` 中的约束同样会被校验。

### Q3：`Depends()` 和普通的函数调用有什么区别？

| 对比项 | `Depends()` | 普通调用 |
|--------|------------|----------|
| 执行时机 | FastAPI 自动调用 | 手动调用 |
| 结果缓存 | 同一请求中可共享 | 每次独立 |
| 类型提示 | 支持，文档可见 | 不影响文档 |
| 异常处理 | 异常自动返回 HTTP 错误 | 需手动 try/catch |

核心优势：**复用** + **自动文档** + **解耦**。

### Q4：`exclude_unset` 和 `exclude_none` 有什么区别？

```python
# exclude_unset=True  — 排除"未设置"的字段（未传入参数）
item = ItemUpdate()  # 没传任何参数
item.model_dump(exclude_unset=True)  # → {}（空字典）

# exclude_none=True  — 排除值为 None 的字段
item = ItemUpdate(name=None, price=10)
item.model_dump(exclude_none=True)  # → {"price": 10}

# 常用组合：PATCH 更新时同时排除未设置和 None
item.model_dump(exclude_unset=True, exclude_none=True)
```

### Q5：中间件和依赖注入该用哪个？

| 场景 | 推荐方案 |
|------|----------|
| 需要访问请求数据并影响响应（CORS、日志、计时） | 中间件 |
| 特定路由需要的逻辑（认证、分页、查数据库） | 依赖注入 |
| 全局影响所有路由 | 中间件 |
| 只影响部分路由 | 依赖注入 |

两者不冲突，可以同时使用。

---

## 九、练习题

### 练习一：学生成绩管理路由（基础）

使用 `APIRouter` 创建一个学生成绩管理模块，包含以下接口：

| 方法 | 路径 | 功能 |
|------|------|------|
| GET | `/students` | 获取学生列表，支持按 `class_name` 筛选 |
| GET | `/students/{id}` | 获取学生详情 |
| POST | `/students` | 添加学生（使用 Pydantic 模型 + 字段约束） |
| PUT | `/students/{id}` | 更新学生信息 |
| DELETE | `/students/{id}` | 删除学生 |

字段约束要求：姓名 2-20 字符、年龄 6-60、成绩 0-100 分。

### 练习二：商品分类嵌套模型（进阶）

设计一个商品接口，商品模型包含嵌套的"分类"和"规格"信息：

```python
# 要求：
# - Category: name（分类名）, parent（父分类，可选）
# - Spec: key（规格名）, value（规格值）, 如 {"key": "颜色", "value": "红色"}
# - Product: name, price, category（嵌套）, specs（嵌套列表）
# - 实现 GET/POST /products 接口
```

### 练习三：带认证的笔记 API（挑战）

创建一个笔记 API，要求：
1. 使用 `APIRouter` 组织路由（`/notes`）
2. 所有写操作（POST/PUT/DELETE）需要 Token 认证（用 `Depends` 实现认证依赖）
3. 笔记模型包含 `title`、`content`、`tags`（列表）、`is_public`（布尔值）
4. 实现分页、标签筛选、公开/私有筛选
5. 添加请求计时中间件

---

## 十、今日总结

| 知识点 | 关键内容 |
|--------|----------|
| APIRouter | 路由分组，`include_router()` 注册，`prefix`/`tags` 配置 |
| Field 约束 | `min_length`/`max_length`/`ge`/`le`/`pattern`/`description` |
| field_validator | 自定义字段校验（单字段） |
| model_validator | 跨字段校验（如密码确认） |
| 嵌套模型 | 模型中引用其他模型，自动递归校验 |
| 模型继承 | 公共字段放基类，避免重复定义 |
| model_config | `str_strip_whitespace`、`extra="forbid"` 等全局配置 |
| 依赖注入 | `Depends()` 复用逻辑、自动文档、结果缓存 |
| 中间件 | 请求前后执行，适合日志/CORS/计时 |
| CORS | 前后端分离必备，`CORSMiddleware` 配置 |

下一节（Day 48）我们将学习 **FastAPI 进阶**，包括异步数据库操作、WebSocket 实时通信、后台任务等高级功能。

---

## 十一、推荐学习资源

- [FastAPI 官方文档 — APIRouter](https://fastapi.tiangolo.com/tutorial/bigger-applications/) — 路由组织官方教程
- [FastAPI 官方文档 — 依赖注入](https://fastapi.tiangolo.com/tutorial/dependencies/) — Depends 详解
- [Pydantic V2 文档 — Validators](https://docs.pydantic.dev/latest/concepts/validators/) — 校验器完整指南
- [Pydantic V2 文档 — Field](https://docs.pydantic.dev/latest/api/fields/) — Field 参数大全
- [菜鸟教程 — FastAPI](https://www.runoob.com/fastapi/fastapi-tutorial.html) — 中文入门教程
