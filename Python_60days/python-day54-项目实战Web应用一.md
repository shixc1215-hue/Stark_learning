# Python Day 54 - 项目实战：Web 应用（一）

> 前 53 天我们学了变量、函数、面向对象、FastAPI、数据库操作、前端基础……今天开始，我们要把这些知识**串联起来**，动手做一个真正的 Web 应用！本项目分两讲：今天搭建**后端 API + 数据库 + 前端页面**，Day 55 完善功能并部署。我们的实战项目是一个**在线记账本（MoneyTracker）**——它能帮你记录每笔收支、分类统计、查看趋势，是一个功能完整的个人财务工具。

---

## 一、项目概述

### 1.1 我们要做什么？

一个在线记账本，包含以下功能：

| 模块 | 功能 |
|------|------|
| 用户管理 | 注册、登录、退出 |
| 记账 | 添加收入/支出记录（金额、分类、备注、日期） |
| 查询 | 按日期范围、分类筛选记录 |
| 统计 | 月度收支汇总、分类占比 |
| 前端 | 美观的 HTML 页面，通过 API 与后端交互 |

### 1.2 技术栈回顾

```
┌─────────────┐     HTTP请求     ┌──────────────┐     SQL     ┌───────────┐
│  浏览器前端  │ ──────────────→ │  FastAPI后端  │ ─────────→ │  SQLite   │
│ HTML/CSS/JS │ ←────────────── │  Pydantic模型 │ ←───────── │  数据库    │
└─────────────┘     JSON响应     └──────────────┘             └───────────┘
```

- **后端**：FastAPI（Day 46-48）+ SQLAlchemy（Day 50）
- **数据库**：SQLite（Day 49）
- **前端**：HTML/CSS/JS（Day 53）
- **认证**：简单的 JWT Token（Day 48 提及）

### 1.3 项目目录结构

按照 Day 32 学过的项目结构规范来组织：

```
moneytracker/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI应用入口
│   ├── database.py          # 数据库连接配置
│   ├── models.py            # SQLAlchemy模型
│   ├── schemas.py           # Pydantic请求/响应模型
│   ├── auth.py              # 认证相关
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── users.py         # 用户注册/登录
│   │   └── records.py       # 记账记录CRUD
│   └── static/
│       ├── css/
│       │   └── style.css    # 样式文件
│       └── js/
│           └── app.js       # 前端逻辑
├── templates/
│   ├── base.html            # 基础模板
│   ├── index.html           # 首页/登录
│   └── dashboard.html        # 记账仪表盘
├── requirements.txt
└── run.py                   # 启动脚本
```

---

## 二、后端：数据库与模型

### 2.1 依赖安装

```bash
# requirements.txt
fastapi==0.111.0
uvicorn==0.30.1
sqlalchemy==2.0.31
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
jinja2==3.1.4
```

```bash
pip install -r requirements.txt
```

### 2.2 数据库连接（database.py）

```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

# SQLite数据库文件
DATABASE_URL = "sqlite:///./moneytracker.db"

# 创建引擎（echo=True 会打印SQL语句，方便调试）
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False}, echo=True)

# 创建会话工厂
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 声明基类
class Base(DeclarativeBase):
    pass


# 获取数据库会话的依赖函数（供FastAPI的Depends使用）
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

> **要点**：`connect_args={"check_same_thread": False}` 是 SQLite 特有的配置，允许多个线程共享同一个连接。`get_db()` 是一个**生成器函数**，用 `yield` 提供会话，`finally` 确保会话一定关闭——这就是 Day 27 学过的上下文管理思想。

### 2.3 数据模型（models.py）

```python
# app/models.py
from sqlalchemy import Column, Integer, String, Float, DateTime, ForeignKey, Enum
from sqlalchemy.orm import relationship
from datetime import datetime

from app.database import Base


class User(Base):
    """用户表"""
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, index=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.now)

    # 关联关系：一个用户有多条记账记录
    records = relationship("Record", back_populates="owner", cascade="all, delete-orphan")


class Record(Base):
    """记账记录表"""
    __tablename__ = "records"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    amount = Column(Float, nullable=False)           # 金额
    category = Column(String(50), nullable=False)   # 分类（如"餐饮"、"交通"）
    record_type = Column(String(10), nullable=False)  # 收入(income)或支出(expense)
    remark = Column(String(255), default="")         # 备注
    date = Column(DateTime, default=datetime.now)     # 记录日期

    # 关联关系
    owner = relationship("User", back_populates="records")
```

> **要点**：`relationship()` 建立了 User ↔ Record 的一对多关系。`cascade="all, delete-orphan"` 表示删除用户时，其所有记录也自动删除。

### 2.4 初始化数据库（main.py 入口部分）

```python
# app/main.py（先看前半部分）
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from fastapi.requests import Request

from app.database import engine, Base
from app.routers import users, records

# 创建所有数据库表
Base.metadata.create_all(bind=engine)

app = FastAPI(title="MoneyTracker", version="1.0")

# 注册路由（Day 47学的APIRouter）
app.include_router(users.router, prefix="/api/users", tags=["用户"])
app.include_router(records.router, prefix="/api/records", tags=["记账"])

# 挂载静态文件和模板
app.mount("/static", StaticFiles(directory="app/static"), name="static")
templates = Jinja2Templates(directory="templates")
```

> `Base.metadata.create_all()` 会在启动时自动创建所有表——如果表已存在则跳过。这在开发阶段很方便，生产环境通常用迁移工具（Alembic）。

---

## 三、后端：认证模块

### 3.1 密码哈希与Token生成（auth.py）

```python
# app/auth.py
from datetime import datetime, timedelta
from typing import Optional

from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.orm import Session

from app.database import get_db
from app.models import User

# 密码加密上下文
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# JWT配置
SECRET_KEY = "your-secret-key-change-in-production"  # 生产环境请用环境变量！
ALGORITHM = "HS256"
EXPIRE_MINUTES = 60 * 24  # Token有效期24小时

# OAuth2认证方案（从请求头中提取Bearer Token）
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/users/login")


def hash_password(password: str) -> str:
    """对密码进行哈希加密"""
    return pwd_context.hash(password)


def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码是否正确"""
    return pwd_context.verify(plain_password, hashed_password)


def create_token(user_id: int) -> str:
    """生成JWT Token"""
    expire = datetime.utcnow() + timedelta(minutes=EXPIRE_MINUTES)
    payload = {"sub": str(user_id), "exp": expire}
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)


def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    """从Token中获取当前登录用户（依赖注入）"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="无效的认证凭据",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: int = int(payload.get("sub"))
        if user_id is None:
            raise credentials_exception
    except (JWTError, ValueError):
        raise credentials_exception

    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise credentials_exception
    return user
```

> **要点**：
> - `pwd_context.hash()` 对密码进行**不可逆**加密，数据库中存储的永远是哈希值，即使数据库泄露，密码也安全。
> - JWT（JSON Web Token）是一种无状态的认证方式——服务端不存储会话，只通过 Token 验证身份。
> - `get_current_user` 是一个**依赖注入函数**，在需要认证的接口中用 `Depends(get_current_user)` 自动获取当前用户。

### 3.2 Pydantic 模型（schemas.py）

```python
# app/schemas.py
from pydantic import BaseModel, Field
from datetime import datetime
from typing import Optional


# ========== 用户相关 ==========

class UserCreate(BaseModel):
    """注册请求"""
    username: str = Field(min_length=3, max_length=50, description="用户名")
    password: str = Field(min_length=6, max_length=100, description="密码")


class UserLogin(BaseModel):
    """登录请求"""
    username: str
    password: str


class UserResponse(BaseModel):
    """用户信息响应"""
    id: int
    username: str
    created_at: datetime

    model_config = {"from_attributes": True}  # Pydantic v2语法


class TokenResponse(BaseModel):
    """Token响应"""
    access_token: str
    token_type: str = "bearer"


# ========== 记账记录相关 ==========

class RecordCreate(BaseModel):
    """创建记录请求"""
    amount: float = Field(gt=0, description="金额（必须大于0）")
    category: str = Field(min_length=1, max_length=50, description="分类")
    record_type: str = Field(description="类型：income或expense")
    remark: str = Field(default="", max_length=255, description="备注")
    date: Optional[datetime] = Field(default=None, description="记录日期")


class RecordUpdate(BaseModel):
    """更新记录请求（所有字段可选）"""
    amount: Optional[float] = Field(default=None, gt=0)
    category: Optional[str] = Field(default=None, min_length=1, max_length=50)
    record_type: Optional[str] = None
    remark: Optional[str] = Field(default=None, max_length=255)


class RecordResponse(BaseModel):
    """记录响应"""
    id: int
    amount: float
    category: str
    record_type: str
    remark: str
    date: datetime

    model_config = {"from_attributes": True}


class StatsResponse(BaseModel):
    """统计响应"""
    total_income: float = 0.0
    total_expense: float = 0.0
    balance: float = 0.0
    category_breakdown: dict = {}
```

---

## 四、后端：API 路由

### 4.1 用户路由（routers/users.py）

```python
# app/routers/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from app.database import get_db
from app.models import User
from app.schemas import UserCreate, UserLogin, UserResponse, TokenResponse
from app.auth import (
    hash_password, verify_password,
    create_token, get_current_user
)

router = APIRouter()


@router.post("/register", response_model=UserResponse, status_code=201)
def register(user_data: UserCreate, db: Session = Depends(get_db)):
    """用户注册"""
    # 检查用户名是否已存在
    existing = db.query(User).filter(User.username == user_data.username).first()
    if existing:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"用户名 '{user_data.username}' 已被注册"
        )

    # 创建用户（密码加密存储！）
    new_user = User(
        username=user_data.username,
        hashed_password=hash_password(user_data.password)
    )
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user


@router.post("/login", response_model=TokenResponse)
def login(user_data: UserLogin, db: Session = Depends(get_db)):
    """用户登录，返回JWT Token"""
    user = db.query(User).filter(User.username == user_data.username).first()
    if not user or not verify_password(user_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误"
        )

    # 生成Token
    token = create_token(user.id)
    return TokenResponse(access_token=token)


@router.get("/me", response_model=UserResponse)
def get_me(current_user: User = Depends(get_current_user)):
    """获取当前登录用户信息"""
    return current_user
```

> **要点**：`Depends(get_current_user)` 会自动执行认证逻辑——如果 Token 无效，请求直接被拦截并返回 401 错误。

### 4.2 记账记录路由（routers/records.py）

```python
# app/routers/records.py
from datetime import datetime
from typing import Optional

from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlalchemy.orm import Session
from sqlalchemy import func, and_

from app.database import get_db
from app.models import User, Record
from app.schemas import (
    RecordCreate, RecordUpdate, RecordResponse, StatsResponse
)
from app.auth import get_current_user

router = APIRouter()


@router.post("/", response_model=RecordResponse, status_code=201)
def create_record(
    data: RecordCreate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """添加一条记账记录"""
    record = Record(
        user_id=current_user.id,
        amount=data.amount,
        category=data.category,
        record_type=data.record_type,
        remark=data.remark,
        date=data.date or datetime.now()
    )
    db.add(record)
    db.commit()
    db.refresh(record)
    return record


@router.get("/", response_model=list[RecordResponse])
def list_records(
    start_date: Optional[str] = Query(None, description="开始日期 YYYY-MM-DD"),
    end_date: Optional[str] = Query(None, description="结束日期 YYYY-MM-DD"),
    category: Optional[str] = Query(None, description="筛选分类"),
    record_type: Optional[str] = Query(None, description="筛选类型"),
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=500),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """查询当前用户的记账记录（支持筛选和分页）"""
    query = db.query(Record).filter(Record.user_id == current_user.id)

    # 按条件筛选
    if start_date:
        query = query.filter(Record.date >= start_date)
    if end_date:
        query = query.filter(Record.date <= end_date)
    if category:
        query = query.filter(Record.category == category)
    if record_type:
        query = query.filter(Record.record_type == record_type)

    records = query.order_by(Record.date.desc()).offset(skip).limit(limit).all()
    return records


@router.get("/stats", response_model=StatsResponse)
def get_stats(
    year: int = Query(..., description="年份"),
    month: int = Query(..., ge=1, le=12, description="月份"),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """获取指定月份的收支统计"""
    # 本月所有记录
    records = db.query(Record).filter(
        and_(
            Record.user_id == current_user.id,
            func.strftime("%Y", Record.date) == str(year),
            func.strftime("%m", Record.date) == f"{month:02d}"
        )
    ).all()

    total_income = sum(r.amount for r in records if r.record_type == "income")
    total_expense = sum(r.amount for r in records if r.record_type == "expense")

    # 按分类统计
    category_breakdown = {}
    for r in records:
        if r.category not in category_breakdown:
            category_breakdown[r.category] = {"income": 0.0, "expense": 0.0}
        category_breakdown[r.category][r.record_type] += r.amount

    return StatsResponse(
        total_income=round(total_income, 2),
        total_expense=round(total_expense, 2),
        balance=round(total_income - total_expense, 2),
        category_breakdown=category_breakdown
    )


@router.put("/{record_id}", response_model=RecordResponse)
def update_record(
    record_id: int,
    data: RecordUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """修改一条记录（只能修改自己的）"""
    record = db.query(Record).filter(
        Record.id == record_id,
        Record.user_id == current_user.id
    ).first()
    if not record:
        raise HTTPException(status_code=404, detail="记录不存在")

    # 只更新提供的字段
    update_data = data.model_dump(exclude_unset=True)
    for field, value in update_data.items():
        setattr(record, field, value)

    db.commit()
    db.refresh(record)
    return record


@router.delete("/{record_id}", status_code=204)
def delete_record(
    record_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """删除一条记录（只能删除自己的）"""
    record = db.query(Record).filter(
        Record.id == record_id,
        Record.user_id == current_user.id
    ).first()
    if not record:
        raise HTTPException(status_code=404, detail="记录不存在")

    db.delete(record)
    db.commit()
```

### 4.3 启动脚本（run.py）

```python
# run.py
import uvicorn

if __name__ == "__main__":
    uvicorn.run("app.main:app", host="0.0.0.0", port=8000, reload=True)
```

```bash
# 启动命令
python run.py
# 或者
uvicorn app.main:app --reload
```

启动后访问：
- API文档：http://localhost:8000/docs
- ReDoc文档：http://localhost:8000/redoc

---

## 五、前端：HTML 模板

### 5.1 基础模板（templates/base.html）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}MoneyTracker{% endblock %}</title>
    <link rel="stylesheet" href="/static/css/style.css">
</head>
<body>
    <nav class="navbar">
        <div class="nav-brand">💰 MoneyTracker</div>
        <div class="nav-links" id="navLinks">
            <!-- 登录前显示 -->
            <span id="loginBtn"><a href="/login">登录</a></span>
            <span id="registerBtn"><a href="/register">注册</a></span>
            <!-- 登录后显示 -->
            <span id="dashboardBtn" style="display:none">
                <a href="/dashboard">我的账本</a>
            </span>
            <span id="logoutBtn" style="display:none">
                <a href="#" onclick="logout()">退出</a>
            </span>
        </div>
    </nav>

    <main class="container">
        {% block content %}{% endblock %}
    </main>

    <script src="/static/js/app.js"></script>
</body>
</html>
```

### 5.2 登录/注册页面（templates/index.html）

```html
<!-- templates/login.html -->
{% extends "base.html" %}

{% block title %}登录 - MoneyTracker{% endblock %}

{% block content %}
<div class="auth-card">
    <h2>🔐 登录</h2>
    <form id="loginForm" onsubmit="handleLogin(event)">
        <div class="form-group">
            <label for="username">用户名</label>
            <input type="text" id="username" required
                   placeholder="请输入用户名" minlength="3">
        </div>
        <div class="form-group">
            <label for="password">密码</label>
            <input type="password" id="password" required
                   placeholder="请输入密码" minlength="6">
        </div>
        <button type="submit" class="btn btn-primary">登录</button>
        <p class="auth-link">
            没有账号？<a href="/register">立即注册</a>
        </p>
        <p id="errorMsg" class="error-msg" style="display:none"></p>
    </form>
</div>
{% endblock %}
```

### 5.3 记账仪表盘（templates/dashboard.html）

```html
<!-- templates/dashboard.html -->
{% extends "base.html" %}

{% block title %}我的账本 - MoneyTracker{% endblock %}

{% block content %}
<div class="dashboard">
    <!-- 统计卡片 -->
    <div class="stats-cards">
        <div class="stat-card income">
            <h3>收入</h3>
            <p id="totalIncome">¥0.00</p>
        </div>
        <div class="stat-card expense">
            <h3>支出</h3>
            <p id="totalExpense">¥0.00</p>
        </div>
        <div class="stat-card balance">
            <h3>结余</h3>
            <p id="totalBalance">¥0.00</p>
        </div>
    </div>

    <!-- 添加记录 -->
    <div class="add-record">
        <h3>➕ 添加记录</h3>
        <form id="recordForm" onsubmit="handleAddRecord(event)">
            <div class="form-row">
                <div class="form-group">
                    <label>类型</label>
                    <select id="recordType" required>
                        <option value="expense">支出</option>
                        <option value="income">收入</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>金额</label>
                    <input type="number" id="amount" step="0.01"
                           min="0.01" required placeholder="0.00">
                </div>
                <div class="form-group">
                    <label>分类</label>
                    <select id="category" required>
                        <option value="餐饮">餐饮</option>
                        <option value="交通">交通</option>
                        <option value="购物">购物</option>
                        <option value="娱乐">娱乐</option>
                        <option value="住房">住房</option>
                        <option value="医疗">医疗</option>
                        <option value="教育">教育</option>
                        <option value="工资">工资</option>
                        <option value="奖金">奖金</option>
                        <option value="其他">其他</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>日期</label>
                    <input type="date" id="recordDate" required>
                </div>
            </div>
            <div class="form-row">
                <div class="form-group" style="flex:1">
                    <label>备注</label>
                    <input type="text" id="remark"
                           placeholder="选填备注信息" maxlength="255">
                </div>
                <button type="submit" class="btn btn-primary">添加</button>
            </div>
        </form>
    </div>

    <!-- 筛选栏 -->
    <div class="filter-bar">
        <input type="month" id="filterMonth" onchange="loadRecords()">
        <input type="text" id="filterCategory" placeholder="按分类筛选"
               onchange="loadRecords()">
    </div>

    <!-- 记录列表 -->
    <div class="record-list">
        <table id="recordTable">
            <thead>
                <tr>
                    <th>日期</th>
                    <th>类型</th>
                    <th>分类</th>
                    <th>金额</th>
                    <th>备注</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody id="recordBody">
                <!-- 动态填充 -->
            </tbody>
        </table>
    </div>
</div>
{% endblock %}
```

---

## 六、前端：CSS 样式（app/static/css/style.css）

```css
/* app/static/css/style.css */

/* ========== 全局样式 ========== */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, "Microsoft YaHei", sans-serif;
    background-color: #f5f7fa;
    color: #333;
    line-height: 1.6;
}

/* ========== 导航栏 ========== */
.navbar {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 0 2rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 60px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.nav-brand {
    font-size: 1.4rem;
    font-weight: bold;
}

.nav-links a {
    color: white;
    text-decoration: none;
    margin-left: 1.5rem;
    padding: 0.3rem 0.8rem;
    border-radius: 4px;
    transition: background 0.3s;
}

.nav-links a:hover {
    background: rgba(255, 255, 255, 0.2);
}

/* ========== 容器 ========== */
.container {
    max-width: 1000px;
    margin: 2rem auto;
    padding: 0 1rem;
}

/* ========== 登录/注册卡片 ========== */
.auth-card {
    max-width: 400px;
    margin: 3rem auto;
    background: white;
    padding: 2rem;
    border-radius: 12px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.auth-card h2 {
    text-align: center;
    margin-bottom: 1.5rem;
}

.form-group {
    margin-bottom: 1rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.3rem;
    font-weight: 500;
    color: #555;
}

.form-group input,
.form-group select {
    width: 100%;
    padding: 0.6rem;
    border: 1px solid #ddd;
    border-radius: 6px;
    font-size: 0.95rem;
    transition: border-color 0.3s;
}

.form-group input:focus,
.form-group select:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

/* ========== 按钮 ========== */
.btn {
    padding: 0.7rem 1.5rem;
    border: none;
    border-radius: 6px;
    font-size: 1rem;
    cursor: pointer;
    transition: transform 0.1s, box-shadow 0.2s;
}

.btn:active {
    transform: scale(0.97);
}

.btn-primary {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    box-shadow: 0 3px 10px rgba(102, 126, 234, 0.3);
}

.btn-primary:hover {
    box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.btn-danger {
    background: #e74c3c;
    color: white;
}

.btn-sm {
    padding: 0.3rem 0.6rem;
    font-size: 0.85rem;
}

.auth-link {
    text-align: center;
    margin-top: 1rem;
    font-size: 0.9rem;
}

.error-msg {
    color: #e74c3c;
    text-align: center;
    margin-top: 0.5rem;
    font-size: 0.9rem;
}

/* ========== 仪表盘 ========== */
.stats-cards {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    margin-bottom: 2rem;
}

.stat-card {
    background: white;
    padding: 1.5rem;
    border-radius: 10px;
    text-align: center;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.stat-card h3 {
    font-size: 0.9rem;
    color: #888;
    margin-bottom: 0.5rem;
}

.stat-card p {
    font-size: 1.5rem;
    font-weight: bold;
}

.stat-card.income p { color: #27ae60; }
.stat-card.expense p { color: #e74c3c; }
.stat-card.balance p { color: #2980b9; }

/* ========== 添加记录 ========== */
.add-record {
    background: white;
    padding: 1.5rem;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
    margin-bottom: 1.5rem;
}

.form-row {
    display: flex;
    gap: 1rem;
    align-items: flex-end;
}

.form-row .form-group {
    flex: 1;
}

/* ========== 筛选栏 ========== */
.filter-bar {
    display: flex;
    gap: 1rem;
    margin-bottom: 1rem;
}

.filter-bar input {
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 6px;
}

/* ========== 记录列表 ========== */
.record-list {
    background: white;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
    overflow-x: auto;
}

.record-list table {
    width: 100%;
    border-collapse: collapse;
}

.record-list th,
.record-list td {
    padding: 0.8rem 1rem;
    text-align: left;
    border-bottom: 1px solid #eee;
}

.record-list th {
    background: #f8f9fa;
    font-weight: 600;
    color: #555;
}

.record-list tr:hover {
    background: #f5f7ff;
}

.income-tag {
    color: #27ae60;
    font-weight: bold;
}

.expense-tag {
    color: #e74c3c;
    font-weight: bold;
}

/* ========== 响应式 ========== */
@media (max-width: 600px) {
    .stats-cards {
        grid-template-columns: 1fr;
    }
    .form-row {
        flex-direction: column;
    }
    .navbar {
        padding: 0 1rem;
    }
}
```

---

## 七、前端：JavaScript 逻辑（app/static/js/app.js）

```javascript
// app/static/js/app.js

// ========== 工具函数 ==========

// 从localStorage获取Token
function getToken() {
    return localStorage.getItem('token');
}

// 设置Token到localStorage
function setToken(token) {
    localStorage.setItem('token', token);
}

// 清除Token
function clearToken() {
    localStorage.removeItem('token');
}

// 带认证的API请求封装
async function apiRequest(url, options = {}) {
    const token = getToken();
    const headers = {
        'Content-Type': 'application/json',
        ...options.headers
    };

    if (token) {
        headers['Authorization'] = `Bearer ${token}`;
    }

    const response = await fetch(url, { ...options, headers });

    if (response.status === 401) {
        // Token过期，跳转登录
        clearToken();
        window.location.href = '/login';
        throw new Error('认证已过期，请重新登录');
    }

    return response;
}


// ========== 登录/注册 ==========

async function handleLogin(event) {
    event.preventDefault();

    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    const errorMsg = document.getElementById('errorMsg');

    try {
        const response = await fetch('/api/users/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ username, password })
        });

        if (!response.ok) {
            const data = await response.json();
            throw new Error(data.detail || '登录失败');
        }

        const data = await response.json();
        setToken(data.access_token);
        window.location.href = '/dashboard';

    } catch (error) {
        errorMsg.textContent = error.message;
        errorMsg.style.display = 'block';
    }
}

async function handleRegister(event) {
    event.preventDefault();

    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    const errorMsg = document.getElementById('errorMsg');

    try {
        const response = await fetch('/api/users/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ username, password })
        });

        if (!response.ok) {
            const data = await response.json();
            throw new Error(data.detail || '注册失败');
        }

        alert('注册成功！请登录');
        window.location.href = '/login';

    } catch (error) {
        errorMsg.textContent = error.message;
        errorMsg.style.display = 'block';
    }
}

function logout() {
    clearToken();
    window.location.href = '/login';
}


// ========== 记账记录操作 ==========

// 设置默认日期为今天
function initDateInput() {
    const dateInput = document.getElementById('recordDate');
    const today = new Date().toISOString().split('T')[0];
    if (dateInput) dateInput.value = today;
}

// 添加记录
async function handleAddRecord(event) {
    event.preventDefault();

    const data = {
        amount: parseFloat(document.getElementById('amount').value),
        category: document.getElementById('category').value,
        record_type: document.getElementById('recordType').value,
        remark: document.getElementById('remark').value,
        date: document.getElementById('recordDate').value
            ? document.getElementById('recordDate').value + 'T00:00:00'
            : undefined
    };

    try {
        const response = await apiRequest('/api/records/', {
            method: 'POST',
            body: JSON.stringify(data)
        });

        if (!response.ok) {
            const err = await response.json();
            throw new Error(err.detail || '添加失败');
        }

        // 添加成功：清空表单、刷新列表
        document.getElementById('recordForm').reset();
        initDateInput();
        loadRecords();
        loadStats();

    } catch (error) {
        alert(error.message);
    }
}

// 加载记录列表
async function loadRecords() {
    const tbody = document.getElementById('recordBody');
    if (!tbody) return;

    // 构建查询参数
    const monthInput = document.getElementById('filterMonth');
    const categoryInput = document.getElementById('filterCategory');
    let params = [];

    if (monthInput && monthInput.value) {
        const [year, month] = monthInput.value.split('-');
        // 计算月初和月末
        params.push(`start_date=${year}-${month}-01`);
        const lastDay = new Date(year, month, 0).getDate();
        params.push(`end_date=${year}-${month}-${lastDay}`);
    }
    if (categoryInput && categoryInput.value) {
        params.push(`category=${encodeURIComponent(categoryInput.value)}`);
    }

    const url = '/api/records/' + (params.length ? '?' + params.join('&') : '');

    try {
        const response = await apiRequest(url);
        const records = await response.json();

        if (records.length === 0) {
            tbody.innerHTML = '<tr><td colspan="6" style="text-align:center;color:#999;padding:2rem">暂无记录</td></tr>';
            return;
        }

        tbody.innerHTML = records.map(r => `
            <tr>
                <td>${formatDate(r.date)}</td>
                <td><span class="${r.record_type}-tag">${r.record_type === 'income' ? '收入' : '支出'}</span></td>
                <td>${r.category}</td>
                <td>¥${r.amount.toFixed(2)}</td>
                <td>${r.remark || '-'}</td>
                <td>
                    <button class="btn btn-danger btn-sm"
                            onclick="deleteRecord(${r.id})">删除</button>
                </td>
            </tr>
        `).join('');

    } catch (error) {
        tbody.innerHTML = `<tr><td colspan="6" style="text-align:center;color:red">加载失败: ${error.message}</td></tr>`;
    }
}

// 加载统计数据
async function loadStats() {
    const monthInput = document.getElementById('filterMonth');
    if (!monthInput) return;

    const month = monthInput.value || new Date().toISOString().slice(0, 7);
    const [year, m] = month.split('-');

    try {
        const response = await apiRequest(`/api/records/stats?year=${year}&month=${m}`);
        const stats = await response.json();

        document.getElementById('totalIncome').textContent = `¥${stats.total_income.toFixed(2)}`;
        document.getElementById('totalExpense').textContent = `¥${stats.total_expense.toFixed(2)}`;
        document.getElementById('totalBalance').textContent = `¥${stats.balance.toFixed(2)}`;

    } catch (error) {
        console.error('加载统计失败:', error);
    }
}

// 删除记录
async function deleteRecord(id) {
    if (!confirm('确定要删除这条记录吗？')) return;

    try {
        const response = await apiRequest(`/api/records/${id}`, {
            method: 'DELETE'
        });

        if (response.ok) {
            loadRecords();
            loadStats();
        }
    } catch (error) {
        alert('删除失败: ' + error.message);
    }
}

// 日期格式化
function formatDate(dateStr) {
    const date = new Date(dateStr);
    return `${date.getMonth() + 1}/${date.getDate()}`;
}

// 页面加载时初始化
document.addEventListener('DOMContentLoaded', () => {
    initDateInput();

    // 如果在仪表盘页面，加载数据
    if (document.getElementById('recordBody')) {
        // 设置默认月份
        const monthInput = document.getElementById('filterMonth');
        if (monthInput && !monthInput.value) {
            monthInput.value = new Date().toISOString().slice(0, 7);
        }
        loadRecords();
        loadStats();
    }
});
```

---

## 八、页面路由（main.py 补全）

在 `main.py` 中添加页面路由，用于渲染 HTML 模板：

```python
# app/main.py（补充页面路由部分）
from fastapi.responses import HTMLResponse, RedirectResponse
from fastapi import Request, Depends

@app.get("/", response_class=HTMLResponse)
async def root():
    """首页重定向到登录"""
    return RedirectResponse(url="/login")


@app.get("/login", response_class=HTMLResponse)
async def login_page(request: Request):
    """登录页面"""
    return templates.TemplateResponse("login.html", {"request": request})


@app.get("/register", response_class=HTMLResponse)
async def register_page(request: Request):
    """注册页面"""
    return templates.TemplateResponse("register.html", {"request": request})


@app.get("/dashboard", response_class=HTMLResponse)
async def dashboard_page(request: Request):
    """记账仪表盘"""
    return templates.TemplateResponse("dashboard.html", {"request": request})
```

---

## 九、运行与测试

### 9.1 启动步骤

```bash
# 1. 进入项目目录
cd moneytracker

# 2. 安装依赖
pip install -r requirements.txt

# 3. 启动服务
python run.py
```

### 9.2 测试流程

```
① 打开浏览器访问 http://localhost:8000
② 自动跳转到登录页 → 点击"注册"
③ 填写用户名和密码，注册成功后登录
④ 进入仪表盘 → 添加几条收入和支出记录
⑤ 切换月份查看不同月份的统计
⑥ 在分类筛选框输入"餐饮"，查看筛选结果
⑦ 点击"删除"按钮删除一条记录
⑧ 打开 http://localhost:8000/docs 查看完整的API文档
```

---

## 十、常见问题（FAQ）

**Q1：启动报错 `ModuleNotFoundError: No module named 'app'`？**
A：确保你在项目根目录（`moneytracker/`）下运行 `python run.py`，而不是在 `app/` 目录内。

**Q2：前端页面样式没有加载出来？**
A：检查 `app/main.py` 中的 `StaticFiles` 挂载路径是否正确，确保 `app/static/css/style.css` 文件存在。

**Q3：登录后刷新页面，Token还在吗？**
A：是的，Token 存储在浏览器的 `localStorage` 中，关闭浏览器后仍然保留。清除 Token 需要手动调用 `localStorage.removeItem('token')` 或退出登录。

**Q4：密码为什么不能明文存储？**
A：即使 SQLite 是本地数据库，明文存储密码也是极不安全的做法。万一数据库文件泄露（比如误上传到 GitHub），所有用户密码就暴露了。`bcrypt` 哈希是工业标准，即使获取到哈希值也很难反推密码。

**Q5：如何添加更多分类？**
A：修改 `templates/dashboard.html` 中 `<select id="category">` 的 `<option>` 列表即可。更好的做法是将分类配置存为 JSON 文件或数据库表，实现动态加载。

---

## 十一、练习题

### 练习1：添加编辑功能
在记录列表中，每条记录的"操作"列增加一个"编辑"按钮，点击后将该条记录的数据填入上方的表单中，提交时调用 `PUT /api/records/{id}` 接口更新。

### 练习2：增加用户头像
在用户表中添加 `avatar` 字段（存储图片URL），注册时允许用户选择头像，仪表盘顶部显示当前用户头像。

### 练习3：导出CSV
在仪表盘添加一个"导出"按钮，将当前筛选的记录导出为 CSV 文件（利用 Day 23 学过的 csv 模块知识）。

---

## 十二、今日学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| FastAPI官方文档 | https://fastapi.tiangolo.com/zh/ | 中文文档，非常全面 |
| SQLAlchemy 2.0文档 | https://docs.sqlalchemy.org/en/20/ | ORM权威参考 |
| Jinja2模板语法 | https://jinja.palletsprojects.com/en/3.1.x/templates/ | 模板渲染语法速查 |
| Fetch API MDN | https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API | 前端请求API参考 |
| Passlib文档 | https://passlib.readthedocs.io/ | 密码哈希方案大全 |

---

> **下一讲预告**：Day 55 我们将在这个项目基础上，添加数据可视化图表（用 Day 41 的 matplotlib）、分类饼图、月度趋势折线图，并学习如何将项目打包部署，让它在互联网上可以访问！

---

*学习进度：Day 54/60 | 距离完成还有 6 天*
