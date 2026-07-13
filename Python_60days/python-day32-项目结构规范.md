# Python Day 32：项目结构规范

> **学习目标**：掌握 Python 项目标准目录结构，理解 `__init__.py`、`__all__` 的作用，学会绝对导入与相对导入，了解 `pyproject.toml` 现代项目配置，能独立搭建规范可维护的项目骨架。

---

## 一、为什么要学项目结构？

学完前 31 天，你已经掌握了不少 Python 技能——面向对象、装饰器、单元测试、pip、虚拟环境……但有一个问题还没解决：**代码写多了放哪里？**

很多人的做法是把所有 `.py` 文件丢在一个文件夹里。5 个文件时没问题，50 个文件时就乱了。一个真实的项目通常包含：

- 源代码（核心逻辑）
- 测试代码（单元测试）
- 配置文件（数据库连接、API 密钥等）
- 文档（README、开发手册）
- 依赖声明（requirements.txt）
- 工具脚本（数据迁移、部署脚本等）

没有规范的结构，项目就像一个没整理过的房间——找东西全凭记忆，协作时同事一头雾水。

**今天的目标**：学会用 Python 社区公认的最佳实践来组织项目，让你的代码从一开始就"长得像样"。

---

## 二、Python 项目标准目录结构

### 2.1 一个完整的规范项目长什么样

```
my_project/                     # 项目根目录（名称可自定义）
├── src/                        # 源代码目录（推荐方式）
│   └── my_package/             # 包目录（实际包名）
│       ├── __init__.py        # 包初始化文件
│       ├── core.py            # 核心逻辑
│       ├── models.py          # 数据模型
│       ├── utils.py           # 工具函数
│       └── config.py          # 配置管理
├── tests/                      # 测试目录
│   ├── __init__.py             # 让 tests 也成为包
│   ├── conftest.py             # pytest 共享 fixtures
│   ├── test_core.py            # 核心逻辑测试
│   └── test_models.py         # 模型测试
├── docs/                       # 项目文档
│   └── usage.md                # 使用说明
├── scripts/                    # 工具脚本
│   ├── migrate_db.py           # 数据库迁移脚本
│   └── deploy.sh               # 部署脚本
├── .gitignore                  # Git 忽略规则
├── README.md                   # 项目说明文件
├── requirements.txt            # 依赖清单
├── requirements-dev.txt        # 开发环境依赖
├── pyproject.toml              # 现代项目配置文件
└── setup.py 或 setup.cfg       # 包安装配置（传统方式）
```

### 2.2 各目录/文件的作用

| 目录/文件 | 作用 | 是否提交到 Git |
|-----------|------|---------------|
| `src/` | 存放源代码 | ✅ 是 |
| `tests/` | 存放测试代码 | ✅ 是 |
| `docs/` | 项目文档 | ✅ 是 |
| `scripts/` | 工具/部署脚本 | ✅ 是 |
| `venv/` | 虚拟环境 | ❌ 否（在 .gitignore 中排除） |
| `__pycache__/` | Python 缓存 | ❌ 否 |
| `README.md` | 项目说明 | ✅ 是 |
| `requirements.txt` | 依赖清单 | ✅ 是 |
| `pyproject.toml` | 项目元数据 | ✅ 是 |
| `.gitignore` | Git 忽略规则 | ✅ 是 |

### 2.3 为什么用 `src/` 嵌套？

你可能注意到，源代码不是直接放在项目根目录，而是放在 `src/my_package/` 下。这叫 **src layout**，是 Python 社区推荐的做法。

**原因**：避免"意外导入"问题。如果不加 `src/`，你在项目根目录运行 `python` 时，Python 会把当前目录加入 `sys.path`。这意味着你可能导入了未安装的本地代码，而不是通过 `pip install` 安装的版本。使用 `src/` 布局可以强制你在开发时也通过 `pip install -e .` 安装，确保导入路径和生产环境一致。

> **简化版项目**：对于个人学习或小型工具项目，可以省略 `src/` 目录，直接在根目录下放包文件夹即可。重要的是**保持一致**。

---

## 三、`__init__.py` 详解

### 3.1 什么是 `__init__.py`

`__init__.py` 是一个特殊文件，它的存在告诉 Python："这个目录不是一个普通文件夹，而是一个 Python 包（Package）"。没有这个文件，Python 就无法导入该目录下的模块。

```
my_package/
├── __init__.py     ← 有这个文件，my_package 就是一个包
├── module_a.py
└── module_b.py
```

### 3.2 Python 3.3+ 的变化：命名空间包

从 Python 3.3 开始，引入了**命名空间包（Namespace Package）**。即使没有 `__init__.py`，Python 也能识别目录为包（通过 `__path__` 属性）。

**但是**，强烈建议仍然保留 `__init__.py`，原因：
1. 明确标识——一眼就知道这是个包
2. 兼容旧版 Python 和工具
3. 可以在里面写初始化代码
4. 某些打包工具（如 setuptools）仍然依赖它

### 3.3 `__init__.py` 中可以写什么

```python
# my_package/__init__.py

# 1. 包级别的文档字符串
"""
my_package - 一个示例 Python 包

提供核心业务逻辑和工具函数。
"""

# 2. 定义包的版本号
__version__ = "1.0.0"
__author__ = "张三"

# 3. 简化导入：让用户可以从包直接导入常用对象
from .core import process_data, calculate_score
from .models import User, Order
from .utils import format_output

# 4. 包初始化时执行（谨慎使用）
import logging
logger = logging.getLogger(__name__)
logger.info("my_package 已加载")

# 5. 控制导出哪些内容
__all__ = ["process_data", "calculate_score", "User", "Order", "format_output"]
```

有了第 3 步的简化导入，用户可以：

```python
# 不用写完整路径
from my_package import process_data, User

# 而不是
from my_package.core import process_data
from my_package.models import User
```

### 3.4 `__all__` 的作用

`__all__` 是一个字符串列表，定义了 `from package import *` 时导出哪些名称：

```python
# my_package/utils.py
__all__ = ["format_output", "validate_input"]  # 只有这两个会被 import * 导入


def format_output(data):
    """格式化输出"""
    ...

def validate_input(data):
    """验证输入"""
    ...

def _internal_helper(data):
    """内部辅助函数（以下划线开头，约定为私有）"""
    ...
```

```python
# 使用时
from my_package.utils import *

# format_output ✅ 能导入
# validate_input ✅ 能导入
# _internal_helper ❌ 不会被导入（不在 __all__ 中）
```

> **最佳实践**：始终在 `__init__.py` 中定义 `__all__`，明确声明你的包对外暴露的公共 API。这既是一种文档，也是一种约定——告诉使用者"这些是你可以安全使用的"。

---

## 四、导入机制详解

### 4.1 绝对导入（推荐）

绝对导入从项目根目录或已安装的包开始，路径完整明确：

```python
# 绝对导入示例

# 从项目自己的包导入
from my_package.core import process_data
from my_package.models import User
from my_package.utils import format_output

# 从标准库导入
import os
import json
from datetime import datetime

# 从第三方库导入
import requests
import pandas as pd
```

**优点**：
- 路径清晰，一眼看出来源
- 重构时不容易出错
- IDE 自动补全更准确

### 4.2 相对导入

相对导入使用 `.` 表示当前包，`..` 表示上级包：

```python
# 假设目录结构：
# my_package/
# ├── __init__.py
# ├── core.py
# ├── models.py
# └── utils.py

# 在 my_package/utils.py 中使用相对导入：
from .models import User          # 从同级 models.py 导入
from .core import process_data   # 从同级 core.py 导入

# 在 my_package/sub_package/helper.py 中：
from ..models import User         # 从上级包的 models 导入
from ..utils import format_output  # 从上级包的 utils 导入
```

**相对导入规则**：
- `.` = 当前包
- `..` = 上级包
- `...` = 上上级包（以此类推）

**相对导入的限制**：
```python
# ❌ 不能在直接运行的脚本中使用相对导入
# python src/my_package/utils.py → 直接运行会报错！
# ImportError: attempted relative import with no known parent package

# ✅ 只能通过导入包的方式使用
from my_package.utils import format_output
```

### 4.3 绝对导入 vs 相对导入对比

| 特性 | 绝对导入 | 相对导入 |
|------|---------|---------|
| 语法 | `from my_package.utils import X` | `from .utils import X` |
| 可读性 | 高（路径完整） | 中（需要理解目录结构） |
| 重构友好 | 高 | 中（移动文件时相对路径变化） |
| 官方推荐 | ✅ 推荐 | 可用但不推荐作为首选 |
| 直接运行脚本 | ✅ 可以 | ❌ 不可以 |

> **建议**：绝大多数情况下使用**绝对导入**。只在包内部的深度嵌套模块中，且路径非常长时，才考虑使用相对导入来简化代码。

### 4.4 避免循环导入

当两个模块互相导入时，会形成**循环导入**，导致 `ImportError`：

```
# a.py
from b import func_b    # ← 导入 b

# b.py
from a import func_a    # ← 又导入 a → 循环！
```

**解决方案**：

```python
# 方法1：延迟导入（在函数内部导入）
# a.py
def func_a():
    from b import func_b   # 在函数内部导入，此时 b 已经初始化完成
    return func_b()


# 方法2：合并到第三个模块
# common.py — 把共享的代码提取到这里
# a.py 和 b.py 都从 common.py 导入


# 方法3：使用 TYPE_CHECKING 做类型注解时的延迟导入
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from b import User  # 仅用于类型检查，运行时不执行

def greet(user: "User"):  # 用字符串形式引用
    ...
```

---

## 五、现代项目配置：pyproject.toml

### 5.1 什么是 pyproject.toml

`pyproject.toml` 是 Python 社区正在推广的**统一项目配置文件**。过去，Python 项目需要 `setup.py`、`setup.cfg`、`requirements.txt`、`tox.ini`、`.flake8` 等多个配置文件，非常分散。`pyproject.toml` 的目标是**一个文件搞定所有配置**。

这是 Python 打包标准（PEP 518/621/517）定义的格式。

### 5.2 基本示例

```toml
# pyproject.toml

[build-system]
# 构建系统配置：告诉 pip 用什么工具来构建这个包
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
# 项目基本信息
name = "my-package"
version = "1.0.0"
description = "一个示例 Python 包"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.10"
authors = [
    {name = "张三", email = "zhangsan@example.com"}
]

# 依赖
dependencies = [
    "requests>=2.31.0",
    "pandas>=2.0.0",
]

# 可选依赖分组
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov",
    "black",
    "flake8",
]
docs = [
    "mkdocs",
    "mkdocs-material",
]

# 项目入口（让用户可以直接用命令行调用）
[project.scripts]
my-cli = "my_package.cli:main"

# 工具配置
[tool.black]
line-length = 88

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=my_package"

[tool.coverage.run]
source = ["src"]
```

### 5.3 pyproject.toml vs requirements.txt

| 特性 | pyproject.toml | requirements.txt |
|------|----------------|-----------------|
| 用途 | 项目元数据 + 构建配置 + 依赖 | 仅列出依赖 |
| 是否可 pip install | ✅ `pip install -e .` | ✅ `pip install -r` |
| 版本规范 | ✅ 支持丰富格式 | ✅ 但格式有限 |
| 包含项目信息 | ✅ 名称、版本、作者等 | ❌ |
| 工具统一配置 | ✅ pytest/black/ruff 等 | ❌ |

> **建议**：新项目直接用 `pyproject.toml`；如果维护老项目，可以两者共存（`pyproject.toml` 放元数据，`requirements.txt` 放精确依赖版本）。

### 5.4 使用 pip install -e .

在开发中，你希望代码修改后立刻生效，不用每次都重新安装。`-e` 参数（editable 可编辑模式）帮你实现：

```bash
# 在项目根目录（含 pyproject.toml）执行：
pip install -e .

# 等价于在开发模式下安装你的包
# 修改代码后无需重新安装，立刻生效

# 安装开发依赖：
pip install -e ".[dev]"
```

---

## 六、实战：搭建一个完整的项目骨架

让我们一步步创建一个规范的学生成绩管理系统项目。

### 6.1 项目结构

```
student_manager/
├── src/
│   └── student_manager/
│       ├── __init__.py
│       ├── models.py          # 数据模型
│       ├── service.py         # 业务逻辑
│       └── utils.py           # 工具函数
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_service.py
├── .gitignore
├── README.md
├── pyproject.toml
└── requirements.txt
```

### 6.2 各文件内容

**src/student_manager/__init__.py**：

```python
"""
student_manager - 学生成绩管理系统

提供学生信息管理、成绩统计与分析功能。
"""

__version__ = "1.0.0"

from .models import Student
from .service import StudentManager
from .utils import format_report

__all__ = ["Student", "StudentManager", "format_report"]
```

**src/student_manager/models.py**：

```python
"""数据模型定义"""
from dataclasses import dataclass, field
from typing import Optional


@dataclass
class Student:
    """学生模型"""
    name: str
    student_id: str
    age: int
    scores: dict[str, float] = field(default_factory=dict)
    # scores 格式: {"数学": 95.0, "英语": 87.5, ...}

    @property
    def average_score(self) -> float:
        """计算平均分"""
        if not self.scores:
            return 0.0
        return sum(self.scores.values()) / len(self.scores)

    @property
    def total_score(self) -> float:
        """计算总分"""
        return sum(self.scores.values())

    def __str__(self) -> str:
        return f"[{self.student_id}] {self.name} - 平均分: {self.average_score:.1f}"
```

**src/student_manager/service.py**：

```python
"""业务逻辑层"""
from __future__ import annotations
from .models import Student
from .utils import format_report


class StudentManager:
    """学生成绩管理器"""

    def __init__(self) -> None:
        self._students: dict[str, Student] = {}

    def add_student(self, student: Student) -> None:
        """添加学生"""
        if student.student_id in self._students:
            raise ValueError(f"学号 {student.student_id} 已存在")
        self._students[student.student_id] = student

    def get_student(self, student_id: str) -> Student | None:
        """查询学生"""
        return self._students.get(student_id)

    def list_students(self) -> list[Student]:
        """列出所有学生"""
        return list(self._students.values())

    def get_ranking(self) -> list[Student]:
        """按平均分排名"""
        return sorted(
            self._students.values(),
            key=lambda s: s.average_score,
            reverse=True,
        )

    def generate_report(self) -> str:
        """生成成绩报告"""
        ranking = self.get_ranking()
        return format_report(ranking)
```

**src/student_manager/utils.py**：

```python
"""工具函数"""
from .models import Student


def format_report(ranking: list[Student]) -> str:
    """格式化成绩排名报告"""
    lines = ["=" * 50, "  学生成绩排名", "=" * 50]
    for idx, student in enumerate(ranking, 1):
        line = f"  第{idx}名: {student.name} ({student.student_id})"
        line += f" - 总分: {student.total_score:.1f}"
        line += f" - 平均分: {student.average_score:.1f}"
        lines.append(line)
    lines.append("=" * 50)
    return "\n".join(lines)
```

**tests/conftest.py**：

```python
"""pytest 共享 fixtures"""
import pytest
from student_manager.models import Student
from student_manager.service import StudentManager


@pytest.fixture
def sample_students() -> list[Student]:
    """创建示例学生数据"""
    return [
        Student(name="张三", student_id="S001", age=20,
                scores={"数学": 95.0, "英语": 87.5, "物理": 92.0}),
        Student(name="李四", student_id="S002", age=21,
                scores={"数学": 88.0, "英语": 91.0, "物理": 85.0}),
        Student(name="王五", student_id="S003", age=19,
                scores={"数学": 76.0, "英语": 82.0, "物理": 98.0}),
    ]


@pytest.fixture
def manager(sample_students: list[Student]) -> StudentManager:
    """创建带数据的管理器"""
    mgr = StudentManager()
    for s in sample_students:
        mgr.add_student(s)
    return mgr
```

**tests/test_service.py**：

```python
"""业务逻辑测试"""
import pytest
from student_manager.service import StudentManager
from student_manager.models import Student


class TestStudentManager:
    """测试 StudentManager"""

    def test_add_and_get(self, manager: StudentManager):
        """测试添加和查询学生"""
        student = manager.get_student("S001")
        assert student is not None
        assert student.name == "张三"

    def test_duplicate_id_raises(self):
        """测试重复学号抛出异常"""
        mgr = StudentManager()
        s1 = Student("张三", "S001", 20)
        s2 = Student("李四", "S001", 21)
        mgr.add_student(s1)
        with pytest.raises(ValueError, match="已存在"):
            mgr.add_student(s2)

    def test_ranking_order(self, manager: StudentManager):
        """测试排名按平均分降序"""
        ranking = manager.get_ranking()
        assert ranking[0].name == "张三"  # 平均分最高
        scores = [s.average_score for s in ranking]
        assert scores == sorted(scores, reverse=True)

    def test_generate_report(self, manager: StudentManager):
        """测试报告生成"""
        report = manager.generate_report()
        assert "张三" in report
        assert "排名" in report
```

**pyproject.toml**：

```toml
[build-system]
requires = ["setuptools>=68.0"]
build-backend = "setuptools.build_meta"

[project]
name = "student-manager"
version = "1.0.0"
description = "学生成绩管理系统"
requires-python = ">=3.10"
dependencies = []

[project.optional-dependencies]
dev = ["pytest>=7.0", "pytest-cov"]

[tool.setuptools.packages.find]
where = ["src"]

[tool.pytest.ini_options]
testpaths = ["tests"]
```

**.gitignore**：

```text
venv/
__pycache__/
*.pyc
.pytest_cache/
htmlcov/
.coverage
.vscode/
.idea/
.env
dist/
*.egg-info/
```

### 6.3 搭建与运行

```bash
# 1. 创建目录结构
mkdir student_manager && cd student_manager
mkdir -p src/student_manager tests

# 2. 创建并激活虚拟环境
python -m venv venv
venv\Scripts\Activate.ps1          # Windows

# 3. 以可编辑模式安装项目
pip install -e ".[dev]"

# 4. 运行测试
python -m pytest tests/ -v

# 5. 验证导入
python -c "from student_manager import Student; print('导入成功')"
```

---

## 七、练习题

### 练习1：创建包并练习导入

创建一个 `calculator` 包，结构如下：

```
calculator/
├── __init__.py
├── basic.py        # 加减乘除
└── advanced.py     # 幂运算、开方
```

要求：
1. `basic.py` 中实现 `add(a, b)`、`subtract(a, b)`、`multiply(a, b)`、`divide(a, b)`
2. `advanced.py` 中实现 `power(base, exp)`、`sqrt(number)`
3. 在 `__init__.py` 中用 `__all__` 导出所有公共函数
4. 测试 `from calculator import add, power` 是否正常工作

### 练习2：修复循环导入

以下代码存在循环导入问题，请找出并修复：

```python
# user.py
from order import Order

class User:
    def __init__(self, name):
        self.name = name
        self.orders: list[Order] = []
```

```python
# order.py
from user import User

class Order:
    def __init__(self, user: User, amount: float):
        self.user = user
        self.amount = amount
```

提示：使用延迟导入或 TYPE_CHECKING。

### 练习3：搭建完整项目

按照第六节的步骤，搭建 `student_manager` 项目。完成后：
1. 添加一个 `remove_student` 方法到 `StudentManager`
2. 为该方法编写单元测试
3. 确保所有测试通过

---

## 八、常见问题（FAQ）

### Q1：项目目录和包目录可以同名吗？

可以，这其实是最常见的做法。比如项目叫 `my_package`，根目录也是 `my_package/`，里面的包目录也在 `src/my_package/`。但要注意区分：项目根目录是一个文件夹，`src/my_package/` 才是 Python 包。

### Q2：什么时候用 `requirements.txt`，什么时候用 `pyproject.toml`？

- `requirements.txt`：纯粹列出依赖，适合简单脚本、快速原型
- `pyproject.toml`：完整的项目定义，适合正式项目、要发布到 PyPI 的包
- 两者可以共存：`pyproject.toml` 定义项目元数据和构建方式，`requirements.txt`（或 `requirements-dev.txt`）锁定精确版本

### Q3：导入自己的模块时报 `ModuleNotFoundError`？

常见原因和解决：
```python
# 1. 忘记创建 __init__.py
#    解决：在包目录下创建一个空的 __init__.py

# 2. Python 找不到包的路径
#    解决：用 pip install -e . 以可编辑模式安装

# 3. 运行方式不对
#    python src/my_package/main.py  ← 可能找不到其他模块
#    python -m my_package.main      ← 正确方式（从包运行）
```

### Q4：`__init__.py` 可以为空吗？

完全可以。一个空的 `__init__.py` 只是告诉 Python"这是一个包"。你可以在后续开发中逐渐往里面添加版本号、简化导入等内容。

### Q5：什么是 `conftest.py`？

`conftest.py` 是 pytest 框架的特殊文件，用于定义**共享的 fixture**（测试夹具）。pytest 会自动发现各级目录中的 `conftest.py`，其中的 fixture 无需导入就能在同级及子级测试文件中使用。

---

## 九、总结

| 概念 | 作用 | 关键点 |
|------|------|--------|
| 项目目录结构 | 组织代码、便于协作 | src layout 是社区推荐方式 |
| `__init__.py` | 标识包、简化导入 | 始终保留，定义 `__all__` |
| 绝对导入 | `from pkg.module import X` | 推荐，路径清晰 |
| 相对导入 | `from .module import X` | 避免在直接运行的脚本中使用 |
| `pyproject.toml` | 统一项目配置 | 新项目标配 |
| `pip install -e .` | 可编辑模式安装 | 开发时必备 |
| 循环导入 | 两个模块互相导入 | 用延迟导入或 TYPE_CHECKING 解决 |

**今天的关键收获**：
1. 规范的目录结构让项目更易维护、更易协作
2. `__init__.py` 不仅是标识，更是包的"门面"——可以简化导入、定义版本号
3. 绝对导入优于相对导入，路径清晰且不容易出错
4. `pyproject.toml` 是现代 Python 项目的标准配置方式
5. 开发时使用 `pip install -e .` 安装自己的包，确保导入行为和生产一致

---

## 十、免费学习资源

- [廖雪峰 - 模块和包](https://www.liaoxuefeng.com/wiki/1016959663602400/1017454145014400)
- [Python 官方文档 - 包](https://docs.python.org/zh-cn/3/tutorial/modules.html#packages)
- [Real Python - Absolute vs Relative Imports](https://realpython.com/absolute-vs-relative-python-imports/)
- [Python 官方文档 - pyproject.toml](https://packaging.python.org/zh-cn/latest/guides/writing-pyproject-toml/)
- [Cookiecutter 模板](https://github.com/audreyr/cookiecutter-pypackage) — 一键生成规范项目骨架

---

> **下一篇预告**：Day 33 — logging 日志，学习 Python 标准日志模块的使用，掌握日志级别、日志格式、文件输出、日志配置等内容，让你的程序运行状态可追溯。
