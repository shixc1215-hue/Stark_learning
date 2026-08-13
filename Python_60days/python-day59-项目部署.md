# Python Day 59 - 项目部署

> 代码写好了，运行在本地没问题——但如何让别人也能访问？今天学习**项目部署**，让你的 Python 项目从"本地脚本"变成"上线可用"的服务。

---

## 一、部署概述——从本地到线上

### 1.1 什么是部署？

**部署（Deployment）**就是把你在本地开发好的项目，放到服务器上运行，让用户可以通过网络访问。

```
本地开发                          线上部署
┌─────────────┐                ┌─────────────┐
│ 你的电脑     │                │  云服务器    │
│ localhost    │  ──部署──>     │ 公网IP/域名  │
│ 只自己能用   │                │ 任何人可访问 │
└─────────────┘                └─────────────┘
```

### 1.2 常见部署方式对比

| 方式 | 难度 | 成本 | 适用场景 |
|------|------|------|---------|
| **PythonAnywhere** | ⭐ | 免费/低价 | 学习项目、简单的 Web 应用 |
| **Streamlit Cloud** | ⭐ | 免费 | Streamlit 数据应用 |
| **Docker + 云服务器** | ⭐⭐⭐ | 中等 | 生产环境、企业项目 |
| **云函数（Serverless）** | ⭐⭐ | 按用量计费 | 轻量 API、定时任务 |
| **Railway / Render** | ⭐⭐ | 免费/低价 | FastAPI、数据库应用 |

> 今天我们重点掌握 **PythonAnywhere**（最简单）和 **Docker**（最通用）两种方式。

---

## 二、部署前的准备——确保项目可迁移

### 2.1 项目结构标准化

你的项目应该有清晰的目录结构：

```
my_project/
├── app/                    # 主应用代码
│   ├── __init__.py
│   ├── main.py             # 入口文件
│   └── ...
├── templates/              # HTML 模板（Web 应用）
├── static/                 # 静态文件（CSS/JS/图片）
├── tests/                  # 测试
├── requirements.txt        # 依赖清单（重要！）
├── .gitignore              # Git 忽略文件
├── README.md               # 项目说明
└── config.py               # 配置文件
```

### 2.2 生成 requirements.txt

`requirements.txt` 记录项目依赖的所有第三方库，服务器需要根据它来安装：

```bash
# 导出当前环境已安装的包
pip freeze > requirements.txt
```

生成后内容示例：

```
fastapi==0.115.0
uvicorn==0.30.0
pandas==2.2.0
sqlalchemy==2.0.30
python-multipart==0.0.9
```

> **注意**：`pip freeze` 会导出环境中所有包，建议手动精简只保留项目真正需要的。也可以用 `pipreqs` 工具自动扫描代码中的 import：

```bash
pip install pipreqs
pipreqs . --force  # 自动扫描并生成 requirements.txt
```

### 2.3 配置管理——环境变量

密码、密钥等敏感信息不应写死在代码中，而应通过**环境变量**管理：

```python
# config.py —— 从环境变量读取配置
import os

class Config:
    """基础配置"""
    DEBUG = False

class DevelopmentConfig(Config):
    """开发环境"""
    DEBUG = True
    SECRET_KEY = "dev-secret-key-123"  # 开发用

class ProductionConfig(Config):
    """生产环境"""
    SECRET_KEY = os.environ.get("SECRET_KEY")  # 从环境变量读取
    DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///app.db")

# 根据环境变量选择配置
config = {
    "development": DevelopmentConfig,
    "production": ProductionConfig,
}
current_config = config.get(os.environ.get("ENV", "development"))
```

使用时：

```python
# main.py
from config import current_config as config

print(f"SECRET_KEY: {config.SECRET_KEY}")
print(f"DEBUG: {config.DEBUG}")
```

在服务器上设置环境变量：

```bash
# Linux / macOS
export SECRET_KEY="your-real-secret-key"
export ENV="production"
```

---

## 三、方式一：PythonAnywhere 部署（推荐新手）

### 3.1 PythonAnywhere 是什么？

PythonAnywhere 是一个**免费的 Python 在线托管平台**，支持运行 Web 应用、定时任务和交互式 Python 控制台。

**优点**：
- 免费额度足够学习使用
- 无需配置服务器，网页操作即可
- 内置 Python 环境，自带常用库

**限制**：
- 免费版不支持自定义域名
- 流量和存储有上限
- 不支持 WebSocket 等高级特性

### 3.2 部署步骤

**第一步：注册账号**

访问 [https://www.pythonanywhere.com](https://www.pythonanywhere.com)，点击注册（建议选择免费版 Beginner 账户）。

**第二步：上传代码**

在 Dashboard 页面点击 "Files" 标签页，然后：

```
方式一：直接上传 zip 压缩包
  1. 本地将项目打包：zip -r myproject.zip myproject/
  2. 点击 "Upload a file" 上传 zip
  3. 打开 Bash 控制台，解压：unzip myproject.zip

方式二：通过 Git 克隆（推荐）
  $ git clone https://github.com/yourname/myproject.git
```

**第三步：安装依赖**

打开 Bash 控制台（"Consoles" → "Bash"），执行：

```bash
# 创建虚拟环境
python3 -m venv myenv
source myenv/bin/activate

# 安装依赖
pip install -r myproject/requirements.txt
```

**第四步：配置 Web 应用**

1. 进入 "Web" 标签页
2. 点击 "Add a new web app"
3. 选择你的域名（免费版是 `yourname.pythonanywhere.com`）
4. 选择 "Manual configuration"
5. Python 版本选 3.10 或更高

**第五步：编辑 WSGI 文件**

PythonAnywhere 会自动创建一个 WSGI 文件（如 `var/www/yourname_pythonanywhere_com_wsgi.py`），修改它：

```python
# 如果是 FastAPI 项目
import sys
# 添加项目路径
sys.path.insert(0, "/home/yourname/myproject")
sys.path.insert(0, "/home/yourname/myenv/lib/python3.10/site-packages")

from app.main import app  # 导入 FastAPI 实例

# 注意：FastAPI 不能直接用 WSGI，需要 ASGI
# PythonAnywhere 免费版不支持 ASGI，建议用 Flask

# 如果是 Flask 项目
from app.main import create_app
application = create_app()  # WSGI 需要 application 变量
```

**第六步：重载应用**

在 "Web" 标签页点击 "Reload" 按钮，访问你的应用 URL 即可。

### 3.3 FastAPI 的替代方案

PythonAnywhere 免费版**只支持 WSGI**，不支持 ASGI（FastAPI 需要）。如果你要用 FastAPI，有两个选择：

1. **用 Flask 替代**（免费版最简单）
2. **升级到付费版**（支持 ASGI）

```python
# 将 FastAPI 改为 Flask 的最小示例
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/api/tasks")
def get_tasks():
    return jsonify({"tasks": ["学习Python", "写项目", "部署上线"]})

if __name__ == "__main__":
    app.run(debug=True)
```

### 3.4 定时任务（Cron Jobs）

PythonAnywhere 支持设置定时任务（比如每天运行一次脚本）：

1. 进入 "Tasks" 标签页
2. 设置执行时间（小时:分钟）
3. 填入命令，例如：

```bash
/home/yourname/myenv/bin/python /home/yourname/myproject/daily_report.py
```

---

## 四、方式二：Docker 容器化部署（推荐进阶）

### 4.1 Docker 是什么？

**Docker** 是一个容器化工具，把应用及其所有依赖打包成一个**镜像（Image）**，在任何安装了 Docker 的机器上都能一致运行。

```
传统部署                              Docker 部署
┌──────────────────┐                ┌──────────────────┐
│ 服务器A：Python 3.9│               │ 容器A（完全相同） │
│ 服务器B：Python 3.8│  ──镜像──>    │ 容器B（完全相同） │
│ "在我电脑能跑啊"   │               │ 任何机器都一样    │
└──────────────────┘                └──────────────────┘
```

### 4.2 安装 Docker

```bash
# Windows：下载 Docker Desktop
# https://www.docker.com/products/docker-desktop

# Linux（Ubuntu）：
curl -fsSL https://get.docker.com | sh

# macOS：下载 Docker Desktop

# 验证安装
docker --version
docker run hello-world  # 运行测试容器
```

### 4.3 编写 Dockerfile

`Dockerfile` 是构建镜像的"配方"，放在项目根目录：

```dockerfile
# 1. 基础镜像——使用官方 Python 镜像
FROM python:3.11-slim

# 2. 设置工作目录
WORKDIR /app

# 3. 设置环境变量（防止 Python 生成 .pyc 文件，日志实时输出）
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# 4. 先复制依赖文件（利用 Docker 缓存层，依赖不变时不用重装）
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 5. 复制项目代码
COPY . .

# 6. 暴露端口
EXPOSE 8000

# 7. 启动命令
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 4.4 .dockerignore 文件

类似 `.gitignore`，排除不需要复制到容器中的文件：

```
.git
__pycache__
*.pyc
.env
venv/
myenv/
.vscode/
.idea/
*.md
tests/
```

### 4.5 构建和运行

```bash
# 构建镜像（-t 指定名称和标签）
docker build -t myproject:latest .

# 查看镜像列表
docker images

# 运行容器（-d 后台运行，-p 端口映射，--env 环境变量）
docker run -d \
  --name myproject-app \
  -p 8000:8000 \
  --env SECRET_KEY=your-secret-key \
  --env ENV=production \
  myproject:latest

# 查看运行中的容器
docker ps

# 查看日志
docker logs myproject-app

# 停止容器
docker stop myproject-app

# 删除容器
docker rm myproject-app
```

### 4.6 Docker Compose——管理多容器

当项目需要**多个服务**（如 Web 应用 + 数据库 + Redis），用 `docker-compose.yml` 一键管理：

```yaml
# docker-compose.yml
version: "3.8"

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - SECRET_KEY=your-secret-key
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      - db        # 先启动数据库再启动 Web
    restart: always  # 崩溃后自动重启

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data  # 数据持久化
    restart: always

volumes:
  postgres_data:  # 命名卷，数据不会丢失
```

常用命令：

```bash
# 一键启动所有服务（-d 后台）
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f web

# 停止所有服务
docker-compose down

# 停止并删除数据卷（慎用！）
docker-compose down -v
```

### 4.7 Docker 常用命令速查

| 命令 | 作用 |
|------|------|
| `docker build -t name .` | 构建镜像 |
| `docker images` | 查看本地镜像 |
| `docker run -it image bash` | 进入容器终端 |
| `docker ps` | 查看运行中容器 |
| `docker ps -a` | 查看所有容器 |
| `docker logs <name>` | 查看容器日志 |
| `docker exec -it <name> bash` | 在运行中的容器执行命令 |
| `docker stop <name>` | 停止容器 |
| `docker rm <name>` | 删除容器 |
| `docker rmi <image>` | 删除镜像 |
| `docker system prune` | 清理无用资源 |

---

## 五、方式三：Streamlit Cloud 部署（Streamlit 应用）

如果你的项目是基于 Streamlit 的数据应用，部署到 Streamlit Cloud 最简单：

### 5.1 部署步骤

1. **将项目推送到 GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/yourname/my-streamlit-app.git
   git push -u origin main
   ```

2. **访问 Streamlit Cloud**：[https://streamlit.io/cloud](https://streamlit.io/cloud)

3. **登录 GitHub 并选择仓库**

4. **配置**：
   - **Main file path**：`app.py`（Streamlit 入口文件）
   - **Python version**：3.11
   - **Requirements**：自动识别 `requirements.txt`

5. **点击 Deploy**，等待构建完成即可访问

### 5.2 入口文件示例

```python
# app.py —— Streamlit 入口
import streamlit as st
import pandas as pd
import numpy as np

st.set_page_config(page_title="我的数据分析工具", layout="wide")
st.title("📊 数据分析工具")

# 上传数据
uploaded_file = st.file_uploader("上传CSV文件", type=["csv"])

if uploaded_file:
    df = pd.read_csv(uploaded_file)
    st.dataframe(df.head())

    # 简单统计
    st.metric("总行数", len(df))
    st.metric("总列数", len(df.columns))
```

---

## 六、方式四：云服务器部署（Linux）

### 6.1 购买服务器

常见云服务商：
- **阿里云**（ECS）
- **腾讯云**（CVM）
- **华为云**（ECS）

选择配置：
- 学习/测试：1核2G，约 50-100 元/月
- 生产环境：2核4G，约 150-300 元/月

### 6.2 连接服务器

```bash
# 使用 SSH 连接
ssh root@你的服务器IP

# 首次连接需输入密码（或配置 SSH 密钥免密登录）
```

### 6.3 部署 FastAPI

```bash
# 1. 安装 Python 和 pip
apt update
apt install python3 python3-pip python3-venv -y

# 2. 克隆项目
git clone https://github.com/yourname/myproject.git
cd myproject

# 3. 创建虚拟环境
python3 -m venv venv
source venv/bin/activate

# 4. 安装依赖
pip install -r requirements.txt

# 5. 启动服务（测试）
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### 6.4 使用 Gunicorn + Uvicorn Worker（生产环境）

生产环境不用直接 `uvicorn`，而是通过 **Gunicorn** 管理多个 worker 进程：

```bash
pip install gunicorn

# 启动（4个worker进程）
gunicorn app.main:app \
  --workers 4 \
  --worker-class uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --timeout 120
```

### 6.5 配置 Nginx 反向代理

Nginx 作为前端服务器，处理 HTTPS、静态文件，并将请求转发给 Python 应用：

```bash
# 安装 Nginx
apt install nginx -y

# 编辑配置文件
nano /etc/nginx/sites-available/myproject
```

Nginx 配置：

```nginx
server {
    listen 80;
    server_name your-domain.com;  # 你的域名

    # 静态文件直接由 Nginx 处理
    location /static/ {
        alias /home/user/myproject/static/;
    }

    # 其他请求转发给 Python 应用
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```bash
# 创建软链接并重启
ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled/
nginx -t  # 测试配置是否正确
systemctl restart nginx
```

### 6.6 使用 Systemd 守护进程

让应用在后台持续运行，服务器重启后自动恢复：

```ini
# /etc/systemd/system/myproject.service
[Unit]
Description=My Python Web App
After=network.target

[Service]
User=user
Group=user
WorkingDirectory=/home/user/myproject
Environment="PATH=/home/user/myproject/venv/bin"
ExecStart=/home/user/myproject/venv/bin/gunicorn \
    app.main:app \
    --workers 4 \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
# 启动并设置开机自启
systemctl daemon-reload
systemctl start myproject
systemctl enable myproject

# 查看状态
systemctl status myproject

# 查看日志
journalctl -u myproject -f
```

---

## 七、部署安全清单

| 项目 | 说明 |
|------|------|
| ✅ 不在代码中硬编码密钥 | 使用环境变量或 `.env` 文件 |
| ✅ `.env` 加入 `.gitignore` | 防止密钥泄露到 GitHub |
| ✅ 关闭 DEBUG 模式 | 生产环境 `DEBUG = False` |
| ✅ 使用 HTTPS | 通过 Nginx 配置 SSL 证书（Let's Encrypt 免费申请） |
| ✅ 限制 CORS | 只允许特定域名访问你的 API |
| ✅ 设置防火墙 | 只开放 80/443 端口，SSH 用密钥登录 |
| ✅ 定期备份数据 | 数据库定时备份到其他位置 |
| ✅ 更新依赖 | 定期 `pip install --upgrade` 修复安全漏洞 |

---

## 八、综合实战——将 MoneyTracker 部署上线

将 Day 54-55 开发的在线记账本项目部署到云服务器：

### 步骤总结

```bash
# === 1. 本地准备 ===
# 确保 requirements.txt 完整
pip freeze > requirements.txt

# 确保 .gitignore 包含敏感文件
echo ".env" >> .gitignore
echo "*.db" >> .gitignore

# 提交代码
git add .
git commit -m "Deploy v1.0"
git push

# === 2. 服务器配置 ===
# 克隆代码
git clone https://github.com/yourname/money-tracker.git
cd money-tracker

# 创建虚拟环境并安装依赖
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 设置环境变量
export SECRET_KEY="your-random-secret-key-here"
export ENV="production"

# === 3. 配置 Nginx 反向代理 ===
sudo nano /etc/nginx/sites-available/money-tracker
# （填入上面 6.5 的配置内容）

# === 4. 配置 Systemd 守护进程 ===
sudo nano /etc/systemd/system/money-tracker.service
# （填入上面 6.6 的配置内容）

# === 5. 启动服务 ===
sudo systemctl daemon-reload
sudo systemctl start money-tracker
sudo systemctl enable money-tracker
sudo systemctl restart nginx

# === 6. 验证 ===
curl http://localhost:8000/docs   # FastAPI 文档页
curl http://你的服务器IP          # 网页访问
```

---

## 九、练习题

### 练习一：生成 requirements.txt（基础）

检查你之前做的 FastAPI 项目（Day 46-55）或 Streamlit 项目（Day 51-57），生成一个干净的 `requirements.txt` 文件。

```bash
# 提示：手动列出项目真正用到的库
# 而不是 pip freeze 直接导出
```

### 练习二：编写 Dockerfile（进阶）

为你之前做的 FastAPI 记账本项目编写一个完整的 Dockerfile，包含：
- Python 3.11-slim 基础镜像
- 依赖安装
- 端口暴露 8000
- 正确的启动命令

```dockerfile
# 提示：注意 Docker 层缓存优化
# 先 COPY requirements.txt 再 COPY 其他文件
```

### 练习三：配置管理改造（挑战）

将你之前项目中的硬编码配置改为环境变量方式：

```python
# 改造前
SECRET_KEY = "my-secret-key"
DATABASE_URL = "sqlite:///app.db"

# 改造后
import os
SECRET_KEY = os.environ.get("SECRET_KEY", "default-dev-key")
DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///app.db")
```

同时创建 `.env` 文件（仅开发用）和 `.env.example`（提交到 Git 的模板）。

---

## 十、常见问题

### Q1：部署时提示 "ModuleNotFoundError"？
**A**：通常是依赖没装全。检查 `requirements.txt` 是否包含所有用到的库，确认虚拟环境已激活再安装。可以用 `pip list` 对比本地和服务器的包列表。

### Q2：Docker 构建很慢？
**A**：注意利用 Docker 层缓存。把 `requirements.txt` 的 COPY 放在代码 COPY 之前——依赖不变时 Docker 会跳过安装步骤，直接使用缓存。

### Q3：服务器上无法访问 8000 端口？
**A**：检查防火墙（`ufw status`）、云服务商安全组是否开放了 8000 端口。生产环境建议用 Nginx 代理，只开放 80/443。

### Q4：.env 文件要提交到 Git 吗？
**A**：**绝对不要！** `.env` 包含密钥和密码，必须加入 `.gitignore`。可以创建 `.env.example` 作为模板提交到 Git（里面只有键名没有值）。

### Q5：免费部署平台有哪些推荐？
**A**：
- **PythonAnywhere**：适合 Flask Web 应用和定时任务
- **Streamlit Cloud**：适合 Streamlit 数据应用
- **Railway**：支持 FastAPI + 数据库
- **Vercel**（通过适配器）：支持 FastAPI
- **Render**：支持多种框架

---

## 十一、免费学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| Docker 官方教程 | https://docs.docker.com/get-started/ | 交互式入门 |
| Docker 从入门到实践 | https://yeasy.gitbook.io/docker_practice/ | 中文免费书 |
| PythonAnywhere 文档 | https://help.pythonanywhere.com/ | 平台使用指南 |
| Nginx 入门教程 | https://runoob.com/nginx/nginx-tutorial.html | 菜鸟教程 |
| Let's Encrypt | https://letsencrypt.org/ | 免费 SSL 证书 |
| Streamlit 部署文档 | https://docs.streamlit.io/deploy | 官方部署指南 |

---

> **明日预告**：Day 60 —— **总结与进阶路线**。60 天 Python 之旅即将画上句号，我们回顾所学，规划未来的技术成长路线！
