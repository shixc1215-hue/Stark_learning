# Python 60天学习计划 — Day 18：模块与包

> **学习目标**：学会用模块和包组织代码，理解 `import` 的各种写法，掌握 `__name__` 和 `__all__` 的用法，能创建自己的可复用模块。

---

## 一、什么是模块？

### 1.1 基本概念

你之前其实已经用过很多"模块"了！每次写 `import random`、`import math`，你就是在使用 Python 提供的**标准库模块**。

简单来说：

- **一个 `.py` 文件就是一个模块**
- 模块 = 把相关的函数、类、变量**组织到一个文件里**
- 包 = 把相关的模块**组织到一个文件夹里**

```
项目结构示意：

my_project/
├── main.py              ← 主程序（也是一个模块）
├── utils.py             ← 工具模块
├── models.py            ← 数据模型模块
└── package/             ← 包（文件夹）
    ├── __init__.py      ← 包的标记文件
    ├── db.py            ← 数据库模块
    └── config.py        ← 配置模块
```

### 1.2 为什么需要模块？

```python
# ❌ 不用模块：所有代码堆在一个文件里，几百上千行，维护噩梦
# main.py（假设有 500 行）
def calculate_tax():
    ...
def format_report():
    ...
def connect_db():
    ...
class User:
    ...
class Order:
    ...
# ... 还有很多很多 ...

# ✅ 用模块：按功能拆分，清晰有序
# utils.py      → 放工具函数
# models.py     → 放类定义
# database.py   → 放数据库操作
# main.py       → 只负责主流程，简洁清晰
```

**模块的好处**：
- 🔹 **代码组织**：相关功能放一起，一目了然
- 🔹 **可复用**：写好的模块可以在多个项目中使用
- 🔹 **命名空间**：避免函数/变量名冲突
- 🔹 **协作友好**：团队成员可以同时开发不同模块

---

## 二、导入模块的几种方式

### 2.1 `import 模块名` — 导入整个模块

```python
# 导入整个 math 模块
import math

# 使用时需要加上模块名前缀
print(math.pi)           # 3.141592653589793
print(math.sqrt(16))     # 4.0
print(math.ceil(3.2))    # 4（向上取整）
print(math.floor(3.8))   # 3（向下取整）
```

### 2.2 `from 模块名 import 名称` — 导入指定功能

```python
# 只导入 pi 和 sqrt
from math import pi, sqrt

# 直接使用，不需要加前缀
print(pi)          # 3.141592653589793
print(sqrt(25))     # 5.0

# ⚠️ 注意：如果同时导入了同名的变量，后导入的会覆盖先导入的
pi = 3.14              # 自己定义的 pi
from math import pi   # 覆盖了上面的 pi
print(pi)              # 3.141592653589793
```

### 2.3 `from 模块名 import *` — 导入所有内容（不推荐）

```python
from math import *

# 可以直接用 math 中的所有东西
print(sqrt(9))    # 3.0
print(sin(0))     # 0.0

# ⚠️ 但这种写法有严重问题！
# 1. 不知道哪些名字被导入了，容易冲突
# 2. 污染命名空间
# 3. 影响代码可读性
```

> **最佳实践**：永远不要用 `from xxx import *`。明确导入你需要的内容，代码更清晰、更安全。

### 2.4 `import 模块名 as 别名` — 给模块起别名

```python
# 别名让代码更简短，也更清晰
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# 用别名来使用
# print(np.array([1, 2, 3]))  # numpy 的内容（后续课程会学）
```

```python
# 也可以给导入的具体功能起别名
from math import factorial as fac
from collections import Counter as Cnt

print(fac(5))     # 120（5!）
```

### 2.5 导入方式对比

| 导入方式 | 语法 | 适用场景 |
|---------|------|---------|
| `import 模块` | `math.sqrt()` | 模块中要用很多功能 |
| `from 模块 import 名称` | `sqrt()` | 只需要一两个功能 |
| `import 模块 as 别名` | `np.sqrt()` | 模块名太长，想简写 |
| `from 模块 import 名称 as 别名` | `fac()` | 功能名和已有变量冲突 |

---

## 三、创建自己的模块

### 3.1 最简单的模块

创建一个 `utils.py` 文件：

```python
# utils.py —— 我的工具模块

def greet(name):
    """打招呼函数"""
    return f"你好，{name}！欢迎学习 Python！"

def calculate_average(numbers):
    """计算平均值"""
    if not numbers:
        return 0
    return sum(numbers) / len(numbers)

PI = 3.14159        # 模块级别的变量
VERSION = "1.0"     # 模块版本号
```

然后在另一个文件中使用它：

```python
# main.py —— 使用自定义模块
import utils

print(utils.greet("小明"))              # 你好，小明！欢迎学习 Python！
print(utils.calculate_average([80, 90, 100]))  # 90.0
print(utils.PI)                         # 3.14159
```

### 3.2 `from ... import ...` 使用自定义模块

```python
# 直接导入需要的函数
from utils import greet, calculate_average, PI

print(greet("小红"))                      # 你好，小红！欢迎学习 Python！
print(calculate_average([1, 2, 3, 4, 5]))  # 3.0
print(f"PI 的值是 {PI}")                   # PI 的值是 3.14159
```

### 3.3 模块就是文件 —— `__file__` 属性

每个模块都有一个 `__file__` 属性，告诉你它在磁盘上的位置：

```python
import utils

print(utils.__file__)
# 输出类似: D:\数据\Python_60days\utils.py
```

### 3.4 查看模块里有什么

```python
import math

# dir() 列出模块的所有属性
print(dir(math))
# 输出: ['__doc__', '__file__', '__loader__', '__name__', '__package__',
#         '__spec__', 'acos', 'asin', 'atan', 'atan2', 'ceil', 'comb',
#         'copysign', 'cos', 'degrees', 'dist', 'e', 'erf', 'erfc', ...]

# 常用技巧：过滤掉下划线开头的"内部属性"
public_items = [x for x in dir(math) if not x.startswith('_')]
print(public_items)
# ['acos', 'acosh', 'asin', 'asinh', 'atan', 'atan2', 'atanh', ...]
```

---

## 四、`__name__` — 模块的"身份证"

### 4.1 什么是 `__name__`？

每个 Python 模块都有一个内置变量 `__name__`：

- **直接运行这个文件时**，`__name__` 的值是 `"__main__"`
- **被其他文件导入时**，`__name__` 的值是**模块名**（文件名）

```python
# demo.py
print(f"demo.py 的 __name__ 是: {__name__}")
```

```bash
# 直接运行
python demo.py
# 输出: demo.py 的 __name__ 是: __main__
```

```python
# 在其他文件中导入
import demo
# 输出: demo.py 的 __name__ 是: demo
```

### 4.2 经典用法：`if __name__ == "__main__":`

这个技巧让你写出的模块既能**直接运行测试**，又能**被安全导入**：

```python
# calculator.py —— 计算器模块

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("除数不能为零！")
    return a / b


# === 下面的代码只在直接运行时执行，被导入时不会执行 ===
if __name__ == "__main__":
    print("=== 计算器模块测试 ===")
    print(f"3 + 5 = {add(3, 5)}")
    print(f"10 - 4 = {subtract(10, 4)}")
    print(f"6 × 7 = {multiply(6, 7)}")
    print(f"15 ÷ 3 = {divide(15, 3)}")
    print("=== 所有测试通过！===")
```

```python
# main.py —— 导入 calculator，不会触发测试代码
from calculator import add, multiply

result = add(10, 20)
print(result)  # 30
# 不会看到"=== 计算器模块测试 ==="的输出！
```

> **养成习惯**：每个模块都加上 `if __name__ == "__main__":` 来放测试代码。这是 Python 程序员的"好习惯"之一。

---

## 五、`__all__` — 控制 `import *` 时的导出

### 5.1 基本用法

```python
# string_tools.py —— 字符串工具模块

__all__ = ["capitalize_words", "reverse_string", "count_words"]
# ↑ 告诉别人：这个模块公开的只有这三个功能


def capitalize_words(text):
    """每个单词首字母大写"""
    return " ".join(word.capitalize() for word in text.split())

def reverse_string(text):
    """反转字符串"""
    return text[::-1]

def count_words(text):
    """统计单词数"""
    return len(text.split())

def _internal_helper(text):
    """内部辅助函数（以下划线开头，表示不对外公开）"""
    return text.strip().lower()
```

```python
# 使用模块
from string_tools import *

# 只有 __all__ 中列出的三个函数可以用
print(capitalize_words("hello world"))  # Hello World
print(reverse_string("abc"))            # cba
print(count_words("a b c d"))           # 4
```

### 5.2 不用 `__all__` 的情况

如果不定义 `__all__`，`from xxx import *` 会导出所有**不以 `_` 开头**的名称。

```python
# 没有 __all__ 的模块
from my_module import *
# 导入了所有不以下划线开头的名称

# 有 __all__ 的模块
from my_module import *
# 只导入了 __all__ 中列出的名称
```

> **最佳实践**：在每个模块中定义 `__all__`，明确告知使用者哪些是公开 API。

---

## 六、第三方模块 — pip 安装

### 6.1 什么是第三方模块？

Python 社区有数十万个免费的第三方模块，覆盖各种需求：

| 领域 | 常用模块 |
|------|---------|
| 数据分析 | `pandas`、`numpy` |
| 可视化 | `matplotlib`、`seaborn` |
| Web 开发 | `flask`、`django`、`fastapi` |
| 爬虫 | `requests`、`beautifulsoup4` |
| 机器学习 | `scikit-learn`、`tensorflow` |
| 自动化 | `selenium`、`playwright` |

### 6.2 使用 pip 安装（概念介绍）

> 💡 Day 31 会详细讲 pip 和虚拟环境，这里先了解基本概念。

```bash
# 安装第三方模块
pip install requests

# 查看已安装的模块
pip list

# 升级模块
pip install --upgrade requests

# 卸载模块
pip uninstall requests
```

安装后就可以像标准库一样导入使用：

```python
import requests    # 安装后直接 import

response = requests.get("https://www.example.com")
print(response.status_code)  # 200
```

### 6.3 常用标准库模块一览

Python 自带了大量标准库（不需要安装），以下是常用的：

```python
# === 文件与路径 ===
import os           # 操作系统接口（文件、目录操作）
import pathlib      # 面向对象的路径操作（推荐）

# === 数据处理 ===
import json         # JSON 数据读写
import csv          # CSV 文件读写
import re           # 正则表达式

# === 日期时间 ===
import datetime     # 日期和时间处理
import time         # 时间戳和计时

# === 数学计算 ===
import math         # 数学函数
import random       # 随机数生成

# === 集合类型 ===
from collections import Counter, defaultdict, OrderedDict

# === 网络通信 ===
import urllib.request   # URL 请求

# === 复制与比较 ===
import copy        # 深拷贝、浅拷贝
import itertools   # 高级迭代工具

# === 调试 ===
import logging     # 日志记录
```

---

## 七、包（Package）— 模块的文件夹

### 7.1 什么是包？

当模块越来越多时，把它们放到文件夹里进行分类管理，这个文件夹就是"包"。

**关键**：包文件夹中必须有一个 `__init__.py` 文件（可以是空的）。

```
my_package/
├── __init__.py      ← 必须有！告诉 Python 这是个包
├── math_tools.py    ← 数学工具模块
├── string_tools.py  ← 字符串工具模块
└── file_tools.py    ← 文件工具模块
```

### 7.2 创建一个包

```python
# my_package/__init__.py —— 可以留空，也可以放初始化代码
# 这个文件在导入包时会自动执行

print("my_package 已被导入！")

# 也可以在这里定义一些方便导入的内容
from .math_tools import add, multiply
from .string_tools import reverse_string
```

```python
# my_package/math_tools.py
def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

def power(base, exp):
    return base ** exp
```

```python
# my_package/string_tools.py
def reverse_string(text):
    return text[::-1]

def to_upper(text):
    return text.upper()
```

### 7.3 导入包中的模块

```python
# 方式1：导入包中的具体模块
import my_package.math_tools
print(my_package.math_tools.add(3, 5))         # 8
print(my_package.math_tools.power(2, 10))        # 1024

# 方式2：从包中导入模块
from my_package import math_tools, string_tools
print(math_tools.multiply(4, 5))                # 20
print(string_tools.reverse_string("hello"))     # olleh

# 方式3：从包的模块中导入具体函数
from my_package.math_tools import add, power
print(add(10, 20))          # 30
print(power(3, 3))          # 27

# 方式4：如果 __init__.py 中已经导入，可以直接从包导入
from my_package import add, reverse_string
print(add(1, 1))             # 2
```

### 7.4 相对导入（包内部互相引用）

在包内部的模块中，可以用 `.` 表示当前包：

```python
# my_package/math_tools.py
from .string_tools import reverse_string
# . 表示"当前包"，即 my_package

def process(text):
    """处理文本的数学统计"""
    reversed_text = reverse_string(text)
    return len(reversed_text)
```

```python
# 也可以用 .. 表示上一级包
# from ..another_package import something
```

> **注意**：相对导入只能在包内部使用，不能在普通脚本中使用。

---

## 八、模块搜索路径

### 8.1 Python 怎么找到模块？

当你写 `import xxx` 时，Python 会按以下顺序搜索：

```
1. 当前目录（脚本所在目录）
2. PYTHONPATH 环境变量指定的目录
3. Python 安装目录的标准库
4. 第三方包安装目录（site-packages）
```

```python
# 查看完整的搜索路径
import sys
print(sys.path)
# 输出类似:
# ['D:\\数据\\Python_60days',
#  'C:\\Users\\21567\\.workbuddy\\binaries\\python\\versions\\3.13.12\\python313.zip',
#  'C:\\Users\\21567\\.workbuddy\\binaries\\python\\versions\\3.13.12\\DLLs',
#  'C:\\Users\\21567\\.workbuddy\\binaries\\python\\versions\\3.13.12\\lib',
#  ...]
```

### 8.2 动态添加搜索路径

```python
import sys

# 临时添加一个搜索路径
sys.path.append("D:\\my_modules")
import my_module  # 现在可以从这个目录导入了

# ⚠️ 这种方式只对当前程序有效，程序关闭后就失效了
```

> **实际开发中**，不需要手动管理路径。使用 pip 安装的包会自动放到正确位置，项目结构合理的代码也都能被正确找到。

---

## 九、综合实战：学生成绩管理系统（模块化）

把之前学过的知识用模块化的方式组织起来：

```
student_manager/
├── main.py              ← 主程序入口
├── models.py            ← 学生类定义
├── utils.py             ← 工具函数
└── data_manager.py      ← 数据存取
```

### 9.1 models.py — 数据模型

```python
# models.py —— 学生数据模型

__all__ = ["Student"]


class Student:
    """学生类"""

    def __init__(self, name, student_id):
        self.name = name
        self.student_id = student_id
        self._scores = {}  # 科目: 分数

    def add_score(self, subject, score):
        """添加成绩"""
        if not (0 <= score <= 100):
            raise ValueError(f"分数必须在0-100之间，当前值: {score}")
        self._scores[subject] = score

    def get_average(self):
        """计算平均分"""
        if not self._scores:
            return 0
        return sum(self._scores.values()) / len(self._scores)

    def __str__(self):
        scores_str = ", ".join(f"{k}: {v}" for k, v in self._scores.items())
        return f"{self.name}（{self.student_id}）— {scores_str}"


if __name__ == "__main__":
    # 测试代码
    s = Student("小明", "2024001")
    s.add_score("数学", 90)
    s.add_score("英语", 85)
    print(s)
    print(f"平均分: {s.get_average()}")
```

### 9.2 utils.py — 工具函数

```python
# utils.py —— 工具函数

__all__ = ["calculate_rank", "format_score", "validate_id"]


def calculate_rank(average):
    """根据平均分返回等级"""
    if average >= 90:
        return "A（优秀）"
    elif average >= 80:
        return "B（良好）"
    elif average >= 70:
        return "C（中等）"
    elif average >= 60:
        return "D（及格）"
    else:
        return "F（不及格）"


def format_score(score, total=100):
    """格式化分数显示"""
    percentage = score / total * 100
    return f"{score}/{total}（{percentage:.1f}%）"


def validate_id(student_id):
    """验证学号格式（6位数字）"""
    import re
    return bool(re.match(r"^\d{6}$", student_id))


if __name__ == "__main__":
    print(calculate_rank(95))   # A（优秀）
    print(calculate_rank(75))   # C（中等）
    print(format_score(85))     # 85/100（85.0%）
    print(validate_id("123456"))  # True
    print(validate_id("abc"))     # False
```

### 9.3 data_manager.py — 数据管理

```python
# data_manager.py —— 数据存取

__all__ = ["save_students", "load_students"]


def save_students(students, filepath="students.json"):
    """保存学生数据到 JSON 文件"""
    import json
    data = []
    for s in students:
        data.append({
            "name": s.name,
            "student_id": s.student_id,
            "scores": s._scores,
            "average": s.get_average()
        })
    with open(filepath, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"已保存 {len(students)} 名学生数据到 {filepath}")


def load_students(filepath="students.json"):
    """从 JSON 文件加载学生数据"""
    import json
    from models import Student

    try:
        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)

        students = []
        for item in data:
            s = Student(item["name"], item["student_id"])
            for subject, score in item["scores"].items():
                s.add_score(subject, score)
            students.append(s)

        print(f"已加载 {len(students)} 名学生数据")
        return students
    except FileNotFoundError:
        print(f"文件 {filepath} 不存在，返回空列表")
        return []


if __name__ == "__main__":
    # 测试保存和加载
    from models import Student
    s1 = Student("小明", "2024001")
    s1.add_score("数学", 90)
    s1.add_score("英语", 85)
    save_students([s1])
```

### 9.4 main.py — 主程序

```python
# main.py —— 学生成绩管理系统主程序

from models import Student
from utils import calculate_rank, validate_id
from data_manager import save_students, load_students


def display_report(student):
    """显示学生成绩报告"""
    avg = student.get_average()
    rank = calculate_rank(avg)
    print("=" * 40)
    print(f"  学生成绩报告")
    print("=" * 40)
    print(f"  姓名: {student.name}")
    print(f"  学号: {student.student_id}")
    print(f"  成绩: {student._scores}")
    print(f"  平均分: {avg:.1f}")
    print(f"  等级: {rank}")
    print("=" * 40)


def main():
    """主函数"""
    print("=== 学生成绩管理系统 ===\n")

    # 创建学生
    s1 = Student("小明", "2024001")
    s1.add_score("数学", 95)
    s1.add_score("英语", 88)
    s1.add_score("物理", 92)

    s2 = Student("小红", "2024002")
    s2.add_score("数学", 78)
    s2.add_score("英语", 96)
    s2.add_score("物理", 85)

    s3 = Student("小刚", "2024003")
    s3.add_score("数学", 62)
    s3.add_score("英语", 71)
    s3.add_score("物理", 58)

    # 显示报告
    for student in [s1, s2, s3]:
        display_report(student)
        print()

    # 保存数据
    save_students([s1, s2, s3])

    # 加载数据
    loaded = load_students()
    print(f"\n从文件加载了 {len(loaded)} 名学生")


if __name__ == "__main__":
    main()
```

运行效果：

```
=== 学生成绩管理系统 ===

========================================
  学生成绩报告
========================================
  姓名: 小明
  学号: 2024001
  成绩: {'数学': 95, '英语': 88, '物理': 92}
  平均分: 91.7
  等级: A（优秀）
========================================

...（依次显示小红、小刚的报告）

已保存 3 名学生数据到 students.json

已加载 3 名学生数据

从文件加载了 3 名学生
```

---

## 十、练习题

### 练习1：创建一个日期工具模块

```python
# date_utils.py
# 要求：
# 1. 实现 format_date(date, fmt) 函数，支持格式化日期
# 2. 实现 days_between(date1, date2) 函数，计算两个日期之间的天数
# 3. 实现 is_weekend(date) 函数，判断是否是周末
# 4. 加上 if __name__ == "__main__": 的测试代码
# 5. 定义 __all__

# 提示：使用 datetime 模块
# from datetime import datetime, date

# 测试：
# today = date(2026, 6, 17)
# print(is_weekend(today))  # False（周三）
# print(days_between(date(2026, 1, 1), date(2026, 12, 31)))  # 364
```

### 练习2：创建一个包

```
text_tools/
├── __init__.py
├── chinese.py      ← 处理中文的函数
└── english.py     ← 处理英文的函数
```

要求：
- `chinese.py` 中实现 `count_chinese_chars(text)` 统计中文字符数
- `english.py` 中实现 `count_words(text)` 统计英文单词数
- `__init__.py` 中导入两个函数，让用户可以 `from text_tools import count_chinese_chars`

提示：中文字符的 Unicode 范围大约是 `\u4e00` 到 `\u9fff`

### 练习3：模块探索

使用 `dir()` 和 `help()` 探索以下标准库模块，找出 5 个你觉得有用的函数：

- `os` 模块
- `random` 模块
- `collections` 模块

```python
import os
import random
import collections

# 用 dir() 查看可用函数
# 用 help(函数名) 查看详细说明
# 写下你发现的 5 个有用函数及其用途
```

---

## 十一、常见问题

**Q1：`import` 和 `from ... import` 到底该用哪个？**

简单原则：
- 如果模块名短（如 `os`、`sys`、`math`）→ `import os`，用 `os.path.join()`
- 如果只需要少量功能 → `from math import sqrt`
- 如果模块名长 → `import numpy as np`

不要纠结太多，保持**一致性**最重要——整个项目里统一风格。

**Q2：循环导入（两个模块互相 import）怎么办？**

```python
# a.py
import b    # 导入 b
def func_a():
    b.func_b()

# b.py
import a    # 导入 a
def func_b():
    a.func_a()
# ❌ 这会导致循环导入错误！
```

**解决方法**：
- 重新设计模块结构，避免互相依赖
- 把 import 放到函数内部（而不是文件顶部）
- 把共享的代码提取到第三个模块中

**Q3：`__init__.py` 可以是空文件吗？**

可以！Python 3.3+ 支持"命名空间包"（namespace packages），技术上不需要 `__init__.py`。但**强烈建议保留空文件**：
- 明确告诉别人"这是包，不是普通文件夹"
- 可以放初始化代码
- 兼容性更好

**Q4：模块名和变量名冲突了怎么办？**

```python
# 如果模块名和本地变量名冲突
import math    # math 是模块
math = 42      # math 变成了数字！模块被覆盖了

# 解决方案1：用别名
import math as m
# m.sqrt(16) 仍然可用

# 解决方案2：用 from import
from math import sqrt
# sqrt(16) 仍然可用
```

---

## 十二、今日总结

```
┌─────────────────────────────────────────────────┐
│              Day 18 知识地图                      │
├─────────────────────────────────────────────────┤
│                                                  │
│   模块 ── 一个 .py 文件就是一个模块               │
│   包 ── 包含 __init__.py 的文件夹                 │
│                                                  │
│   import 模块          → 模块名.函数名()          │
│   from 模块 import 函数 → 直接使用函数名()         │
│   import 模块 as 别名   → 别名.函数名()           │
│                                                  │
│   __name__ == "__main__" → 直接运行时执行         │
│   __all__               → 控制 * 导出的内容       │
│   __file__              → 模块的文件路径           │
│                                                  │
│   sys.path              → 模块搜索路径             │
│   pip install           → 安装第三方模块            │
│                                                  │
└─────────────────────────────────────────────────┘
```

**今天掌握的技能让你能：**
- 将代码按功能拆分成模块，告别"一个文件 500 行"
- 使用 `if __name__ == "__main__":` 写出可运行又可导入的模块
- 创建包来组织大型项目
- 安装和使用社区海量的第三方库

---

## 十三、免费学习资源

- [廖雪峰 Python 教程 - 模块](https://www.liaoxuefeng.com/wiki/1016959663602400/1017454145014384) — 模块和包的详细讲解
- [菜鸟教程 Python 模块](https://www.runoob.com/python3/python3-module.html) — 基础概念和实例
- [Python 官方文档 - 模块](https://docs.python.org/zh-cn/3/tutorial/modules.html) — 官方权威教程

---

> **Day 19 预告**：迭代器与生成器 —— 学习 Python 中处理数据流的强大工具，理解 `yield` 关键字，写出更高效的代码。

> 💡 **学习小贴士**：今天学完后，试着把你之前写过的练习代码整理成模块。比如把 Day 17 的"智能商品库存系统"拆分成 `models.py`（类定义）和 `main.py`（主程序），体会模块化带来的好处。
