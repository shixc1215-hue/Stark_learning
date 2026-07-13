# Python Day 34：配置管理

> 学会管理配置，让你的程序告别"硬编码"——不同环境（开发/测试/生产）自由切换，敏感信息安全隔离。

---

## 一、为什么需要配置管理？

先看一个反面教材：

```python
# ❌ 硬编码：把配置直接写死在代码里
def send_email(to: str, message: str):
    import smtplib
    server = smtplib.SMTP("smtp.qq.com", 587)  # 写死了邮箱服务器
    server.login("my_email@qq.com", "abc123")   # 密码暴露在代码里！
    server.sendmail("my_email@qq.com", to, message)
    server.quit()
```

这样做有三大问题：

| 问题 | 说明 |
|------|------|
| **不安全** | 密码、密钥等敏感信息暴露在代码中，推送到 Git 仓库就泄露了 |
| **不灵活** | 换个环境就要改代码、重新发布，费时费力 |
| **难维护** | 配置散落在各个文件里，找起来像大海捞针 |

**正确做法**：把配置从代码中分离出来，存到独立的配置文件中。

---

## 二、Python 内置的 configparser

`configparser` 是 Python 标准库自带的配置管理模块，使用经典的 INI 格式。

### 2.1 INI 配置文件格式

```ini
; config.ini — 项目配置文件
; 以分号开头的是注释

[database]          ; section（节）
host = 127.0.0.1
port = 3306
name = mydb
username = admin
password = secret123

[server]
host = 0.0.0.0
port = 8000
debug = true

[email]
smtp_server = smtp.qq.com
smtp_port = 465
sender = notify@example.com
```

> **格式要点**：用 `[section]` 分组，`key = value` 写配置值。分号 `;` 开头是注释。

### 2.2 读取配置文件

```python
import configparser
from pathlib import Path

# 创建配置解析器
config = configparser.ConfigParser()

# 读取配置文件
config_path = Path("config.ini")
config.read(config_path, encoding="utf-8")

# 读取 database 节下的配置
db_host = config.get("database", "host")        # "127.0.0.1"
db_port = config.getint("database", "port")      # 3306（自动转整数）
db_debug = config.getboolean("server", "debug")   # True（自动转布尔值）

print(f"数据库地址: {db_host}:{db_port}")
print(f"调试模式: {db_debug}")
```

### 2.3 常用的读取方法

```python
# get()      — 返回字符串
# getint()   — 返回整数
# getfloat() — 返回浮点数
# getboolean()— 返回布尔值（支持 true/false/yes/no/1/0）

# 检查节和键是否存在
print(config.has_section("database"))              # True
print(config.has_option("database", "host"))        # True

# 获取所有节名
print(config.sections())   # ['database', 'server', 'email']

# 获取某个节下所有键值对
for key, value in config["database"].items():
    print(f"  {key} = {value}")
```

### 2.4 写入和修改配置

```python
import configparser

config = configparser.ConfigParser()

# 方式一：通过赋值写入
config["app"] = {}                          # 创建新节
config["app"]["name"] = "我的应用"
config["app"]["version"] = "1.0.0"
config["app"]["max_users"] = "100"

# 方式二：通过 add_section + set 写入
config.add_section("feature")
config.set("feature", "dark_mode", "true")

# 写入文件
with open("config.ini", "w", encoding="utf-8") as f:
    config.write(f)

print("配置文件已保存！")
```

### 2.5 configparser 的局限性

```python
# configparser 不支持嵌套结构
# ❌ 不支持这种格式：
[database.mysql]
host = localhost

# ❌ 不支持列表/数组类型
servers = ["a.com", "b.com", "c.com"]

# 如果需要复杂结构，推荐使用 JSON 或 YAML
```

---

## 三、JSON 配置文件

JSON 格式支持嵌套结构，Python 标准库直接支持，非常适合中等复杂度的配置。

### 3.1 创建 JSON 配置文件

```json
{
    "database": {
        "host": "127.0.0.1",
        "port": 3306,
        "name": "mydb",
        "username": "admin",
        "password": "secret123"
    },
    "features": {
        "dark_mode": true,
        "max_users": 100,
        "allowed_roles": ["admin", "editor", "viewer"]
    },
    "logging": {
        "level": "INFO",
        "file": "app.log"
    }
}
```

### 3.2 读取 JSON 配置

```python
import json
from pathlib import Path

def load_config(path: str = "config.json") -> dict:
    """加载 JSON 配置文件"""
    config_path = Path(path)
    if not config_path.exists():
        raise FileNotFoundError(f"配置文件不存在: {path}")

    with open(config_path, "r", encoding="utf-8") as f:
        return json.load(f)

# 使用配置
config = load_config()
db = config["database"]
print(f"数据库: {db['username']}@{db['host']}:{db['port']}")
print(f"允许的角色: {config['features']['allowed_roles']}")
```

### 3.3 动态修改并保存

```python
import json

# 加载 → 修改 → 保存
config = load_config()
config["features"]["dark_mode"] = False          # 修改值
config["features"]["allowed_roles"].append("guest")  # 添加列表项

with open("config.json", "w", encoding="utf-8") as f:
    json.dump(config, f, ensure_ascii=False, indent=4)

print("配置已更新！")
```

### 3.4 JSON 的优缺点

| 优点 | 缺点 |
|------|------|
| Python 原生支持，无需额外安装 | 不支持注释（不能在 JSON 中写说明） |
| 支持嵌套结构、列表、布尔值 | 书写格式严格，逗号/引号容易出错 |
| 可读性较好 | 多行字符串不便处理 |

---

## 四、YAML 配置文件（推荐）

YAML 是目前最流行的配置文件格式——可读性强、支持注释、语法简洁。

### 4.1 安装 PyYAML

```bash
pip install pyyaml
```

### 4.2 YAML 配置文件示例

```yaml
# config.yaml — 项目配置
# YAML 用 # 号写注释（比 JSON 方便多了！）

database:
  host: 127.0.0.1
  port: 3306
  name: mydb
  username: admin
  password: secret123

server:
  host: 0.0.0.0
  port: 8000
  workers: 4    # 整数类型自动识别
  debug: true    # 布尔值自动识别

features:
  dark_mode: true
  allowed_roles:          # 列表写法（短格式）
    - admin
    - editor
    - viewer

logging:
  level: INFO
  handlers:              # 列表写法（方括号格式）
    [console, file, email]
```

### 4.3 读取 YAML 配置

```python
import yaml
from pathlib import Path

def load_yaml_config(path: str = "config.yaml") -> dict:
    """加载 YAML 配置文件"""
    config_path = Path(path)
    if not config_path.exists():
        raise FileNotFoundError(f"配置文件不存在: {path}")

    with open(config_path, "r", encoding="utf-8") as f:
        return yaml.safe_load(f)   # safe_load 比 load 更安全

# 使用配置
config = load_yaml_config()
db = config["database"]
print(f"数据库: {db['host']}:{db['port']}")
print(f"工作进程数: {config['server']['workers']}")
print(f"日志处理器: {config['logging']['handlers']}")
```

> **重要提示**：始终使用 `yaml.safe_load()` 而不是 `yaml.load()`。`load()` 可以执行任意 Python 代码，存在安全风险！

### 4.4 写入 YAML 配置

```python
import yaml

config = {
    "app": {
        "name": "我的应用",
        "version": "1.0.0",
    },
    "database": {
        "host": "localhost",
        "port": 5432,
    }
}

with open("config.yaml", "w", encoding="utf-8") as f:
    yaml.dump(config, f, allow_unicode=True, default_flow_style=False)

# allow_unicode=True: 允许写入中文
# default_flow_style=False: 使用块样式（更易读）
```

### 4.5 三种格式对比

| 对比项 | INI (configparser) | JSON | YAML |
|--------|-------------------|------|------|
| 嵌套结构 | ❌ 不支持 | ✅ 支持 | ✅ 支持 |
| 注释 | ✅ 分号 | ❌ 不支持 | ✅ 井号 |
| 安装依赖 | 无（标准库） | 无（标准库） | 需要 pyyaml |
| 列表/数组 | ❌ | ✅ | ✅ |
| 可读性 | ★★★ | ★★ | ★★★★★ |
| 适合场景 | 简单配置 | 中等复杂度 | 复杂项目配置 |

---

## 五、环境变量与 .env 文件

### 5.1 为什么需要环境变量？

有些配置信息**绝对不能写在代码或配置文件中**，比如：
- 数据库密码
- API 密钥（如百度AI、微信支付）
- 邿箱账号密码
- 云服务密钥

这些信息应该通过**环境变量**传递，或者存在 `.env` 文件中（该文件加入 `.gitignore`，不提交到仓库）。

### 5.2 使用 os.environ 读取环境变量

```python
import os

# 读取环境变量（不存在则返回 None）
db_host = os.environ.get("DB_HOST")
db_port = os.environ.get("DB_PORT")

# 读取并设置默认值
debug = os.environ.get("DEBUG", "false")

# 如果必需的环境变量不存在，直接报错
api_key = os.environ["API_KEY"]  # KeyError if missing!
```

### 5.3 使用 python-dotenv 管理 .env 文件

```bash
pip install python-dotenv
```

创建 `.env` 文件：

```ini
# .env — 环境变量配置文件
# 注意：这个文件绝对不要提交到 Git！

DB_HOST=127.0.0.1
DB_PORT=3306
DB_PASSWORD=my_secret_password
API_KEY=sk-xxxxxxxxxxxxxxxxx
DEBUG=true
```

在 Python 中使用：

```python
from pathlib import Path
from dotenv import load_dotenv
import os

# 加载 .env 文件
env_path = Path(".") / ".env"
load_dotenv(dotenv_path=env_path)

# 现在可以直接用 os.environ 访问了
db_host = os.environ.get("DB_HOST")
db_port = os.environ.get("DB_PORT")
api_key = os.environ.get("API_KEY")
debug = os.environ.get("DEBUG", "false")

print(f"数据库: {db_host}:{db_port}")
print(f"API密钥: {api_key[:6]}...")   # 只显示前6位，保护安全
```

### 5.4 .env 文件安全规则

```bash
# .gitignore 中必须包含以下内容：

# 环境变量文件（包含敏感信息，禁止提交）
.env
.env.local
.env.production
.env.*.local

# 配置文件（包含密码的也排除）
config.local.yaml
secrets.json
```

---

## 六、实战：构建完整的配置管理系统

把上面学到的知识整合起来，构建一个灵活、安全、分层的配置系统。

### 6.1 项目结构

```
my_project/
├── config/
│   ├── default.yaml          # 默认配置（提交到 Git）
│   ├── development.yaml       # 开发环境配置（提交到 Git）
│   └── production.yaml        # 生产环境配置（提交到 Git）
├── .env                       # 环境变量（不提交到 Git！）
├── .env.example               # 环境变量模板（提交到 Git）
├── app.py                     # 应用入口
└── settings.py                # 配置管理模块
```

### 6.2 settings.py — 配置管理模块

```python
"""
settings.py — 统一配置管理模块
加载顺序：环境变量 > 环境配置文件 > 默认配置
"""
import os
import yaml
from pathlib import Path
from dotenv import load_dotenv
from dataclasses import dataclass, field
from typing import Optional


def _deep_merge(base: dict, override: dict) -> dict:
    """深度合并两个字典，override 的值优先"""
    result = base.copy()
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = _deep_merge(result[key], value)
        else:
            result[key] = value
    return result


def load_yaml(path: str) -> dict:
    """安全加载 YAML 文件，不存在则返回空字典"""
    file_path = Path(path)
    if not file_path.exists():
        return {}
    with open(file_path, "r", encoding="utf-8") as f:
        return yaml.safe_load(f) or {}


class Settings:
    """应用配置管理类"""

    def __init__(self):
        # 1. 加载 .env 环境变量
        load_dotenv()

        # 2. 确定当前运行环境
        self.env = os.environ.get("APP_ENV", "development")

        # 3. 加载配置文件（默认 → 环境特定）
        config_dir = Path("config")
        default_config = load_yaml(config_dir / "default.yaml")
        env_config = load_yaml(config_dir / f"{self.env}.yaml")

        # 合并配置（环境配置覆盖默认配置）
        self._config = _deep_merge(default_config, env_config)

        # 4. 环境变量覆盖配置文件（最高优先级）
        self._override_from_env()

    def _override_from_env(self):
        """环境变量覆盖：以 APP_ 前缀的变量会覆盖对应配置"""
        for key, value in os.environ.items():
            if key.startswith("APP_"):
                # APP_DB_HOST → database.host
                config_key = key[4:].lower().split("_")
                # 简化处理：只支持一级覆盖
                if len(config_key) >= 2:
                    section = config_key[0]
                    prop = "_".join(config_key[1:])
                    if section in self._config:
                        self._config[section][prop] = value

    def get(self, key: str, default=None):
        """获取配置值，支持点号路径，如 'database.host'"""
        keys = key.split(".")
        value = self._config
        for k in keys:
            if isinstance(value, dict) and k in value:
                value = value[k]
            else:
                return default
        return value

    @property
    def database(self) -> dict:
        return self._config.get("database", {})

    @property
    def server(self) -> dict:
        return self._config.get("server", {})

    def __repr__(self) -> str:
        return f"<Settings env={self.env}>"


# 全局配置实例（整个项目共用一个）
settings = Settings()


# === 使用示例 ===
if __name__ == "__main__":
    print(f"当前环境: {settings.env}")
    print(f"数据库: {settings.get('database.host')}:{settings.get('database.port')}")
    print(f"服务器端口: {settings.get('server.port')}")
    print(f"调试模式: {settings.get('server.debug')}")
```

### 6.3 default.yaml — 默认配置

```yaml
# config/default.yaml — 默认配置（所有环境共用）

app:
  name: "我的应用"
  version: "1.0.0"

database:
  host: 127.0.0.1
  port: 3306
  name: mydb
  pool_size: 5

server:
  host: 0.0.0.0
  port: 8000
  debug: false
  workers: 1

logging:
  level: WARNING
  file: app.log
```

### 6.4 development.yaml — 开发环境

```yaml
# config/development.yaml — 开发环境专用配置

database:
  host: 127.0.0.1
  name: mydb_dev

server:
  debug: true
  port: 8000

logging:
  level: DEBUG
```

### 6.5 production.yaml — 生产环境

```yaml
# config/production.yaml — 生产环境专用配置

database:
  host: 10.0.1.100       # 生产数据库地址
  name: mydb_prod
  pool_size: 20

server:
  debug: false
  port: 80
  workers: 8              # 生产环境启用多进程

logging:
  level: WARNING
  file: /var/log/app/app.log
```

### 6.6 使用配置

```python
# app.py — 应用入口
from settings import settings

def main():
    print(f"🚀 启动 {settings.get('app.name')} v{settings.get('app.version')}")
    print(f"   环境: {settings.env}")

    db = settings.database
    print(f"   数据库: {db['host']}:{db['port']}/{db['name']}")

    if settings.get("server.debug"):
        print("   ⚠️ 调试模式已开启（仅开发环境）")

if __name__ == "__main__":
    main()
```

### 6.7 切换环境

```bash
# 开发环境（默认）
python app.py

# 切换到生产环境
set APP_ENV=production
python app.py

# 临时覆盖某个配置
set APP_SERVER_PORT=9000
python app.py
```

---

## 七、配置加载优先级

```
优先级从高到低：

1. 环境变量（APP_ 前缀）     ← 最高优先，动态覆盖一切
2. 环境特定配置文件            ← development.yaml / production.yaml
3. 默认配置文件                ← default.yaml
4. 代码中的默认值              ← 最低优先，作为兜底
```

这个优先级设计的好处：
- 开发用 `development.yaml`，部署用 `production.yaml`
- 紧急修改不用改文件，设个环境变量就行
- 新增配置有默认值兜底，不会报错

---

## 八、常见问题

### Q1：配置文件放在项目根目录还是 config 目录？

**推荐放在 `config/` 目录下**，理由：
- 根目录文件太多会混乱
- 配置文件和代码分离，结构更清晰
- 可以按环境拆分多个文件

### Q2：YAML 文件中的布尔值怎么写？

```yaml
# 以下写法都是 True
debug: true
debug: True
debug: TRUE
debug: yes
debug: on

# 以下写法都是 False
debug: false
debug: False
debug: FALSE
debug: no
debug: off
```

> 注意：`yes` 和 `no` 在 YAML 中会被解析为布尔值！如果需要字符串 "yes"，加引号：`answer: "yes"`

### Q3：如何处理多环境部署？

```python
# 推荐做法：通过 APP_ENV 环境变量控制
# 开发机: APP_ENV=development
# 测试服务器: APP_ENV=testing
# 生产服务器: APP_ENV=production

# 每个环境对应一个配置文件:
# config/development.yaml
# config/testing.yaml
# config/production.yaml
```

### Q4：配置文件应该提交到 Git 吗？

| 文件类型 | 是否提交 | 原因 |
|---------|---------|------|
| `default.yaml` | ✅ 提交 | 所有环境共用，不含敏感信息 |
| `development.yaml` | ✅ 提交 | 开发环境配置，不含敏感信息 |
| `production.yaml` | ✅ 提交 | 生产环境配置，**不含密码** |
| `.env` | ❌ 不提交 | 包含密码和密钥 |
| `.env.example` | ✅ 提交 | 模板文件，展示需要哪些变量 |

### Q5：如何避免密码泄露到 Git？

```bash
# 1. 创建 .env.example（不含真实密码）
# .env.example
DB_PASSWORD=<your_password_here>
API_KEY=<your_api_key_here>

# 2. 把 .env 加入 .gitignore
echo ".env" >> .gitignore

# 3. 团队新成员复制模板并填入自己的密码
cp .env.example .env
```

---

## 九、练习题

### 练习一：基础配置读取

创建一个 `settings.ini` 文件，包含 `[app]` 和 `[database]` 两个节，用 `configparser` 读取并打印所有配置项。

```python
# 提示代码结构
import configparser

config = configparser.ConfigParser()
config.read("settings.ini", encoding="utf-8")

# TODO: 打印所有节名
# TODO: 遍历每个节，打印其键值对
```

### 练习二：多格式转换

写一个程序，将 `config.json` 转换为 `config.yaml`，再转换为 `config.ini`。

```python
# 提示：json.load() → dict → yaml.dump()
#      dict → configparser 写入
```

### 练习三：安全配置管理器

实现一个 `SafeConfig` 类，满足以下要求：
1. 从 `.env` 加载敏感信息（密码、密钥）
2. 从 `config.yaml` 加载普通配置
3. 提供 `get(key, default=None)` 方法读取配置
4. 读取密码类字段时，打印日志记录访问行为（不打印密码值）

```python
# 提示代码结构
class SafeConfig:
    SECRET_KEYS = {"password", "secret", "api_key", "token"}

    def get(self, key: str, default=None):
        # TODO: 检查是否为敏感字段
        # TODO: 如果是敏感字段，记录日志（不记录值）
        # TODO: 返回配置值
        pass
```

---

## 十、推荐学习资源

- **廖雪峰 Python 教程**：[模块和包](https://www.liaoxuefeng.com/wiki/1016959663602400/1017454230018560)
- **菜鸟教程 - configparser**：[https://www.runoob.com/python3/python3-configparser.html](https://www.runoob.com/python3/python3-configparser.html)
- **PyYAML 官方文档**：[https://pyyaml.org/wiki/PyYAMLDocumentation](https://pyyaml.org/wiki/PyYAMLDocumentation)
- **The Twelve-Factor App**（配置管理经典理论）：[https://12factor.net/zh_cn/config](https://12factor.net/zh_cn/config)
- **python-dotenv GitHub**：[https://github.com/theskumar/python-dotenv](https://github.com/theskumar/python-dotenv)

---

> **下期预告**：Day 35 将学习 **Git 版本控制**——用 Git 管理你的代码，告别"最终版v3改了又改的终极版"。
