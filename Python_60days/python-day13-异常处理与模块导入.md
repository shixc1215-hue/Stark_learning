# Python Day 13：异常处理 + 模块导入 —— 写出"不崩"的程序

> 🎯 今日目标：掌握异常处理机制，学会用模块组织代码和安装第三方库
> ⏱ 预计用时：60 分钟 | 📅 进度：Day 13/60
> 🏷 阶段：第一阶段 · 基础语法入门（Day 1-14）

---

## 一、为什么学这两样？

| 问题 | 没有它 | 有了它 |
|------|--------|--------|
| 用户输入了字母当数字 | 程序直接崩溃 💥 | 优雅提示"请输入数字" |
| 文件不存在 | 程序崩溃 | 自动创建或跳过 |
| 代码全写一个文件 | 2000行找bug崩溃 | 拆成模块，清晰可维护 |
| 想用别人写好的功能 | 从零造轮子 | `pip install` 一行搞定 |

**一句话：异常处理让程序"抗揍"，模块导入让你"站在巨人肩膀上"。**

---

## 二、异常处理（try/except）

### 2.1 先看崩溃现场

```python
# 用户可能输入的不是数字
num = int(input("请输入一个数字: "))  # 输入 "abc" → 程序崩溃！
print("结果是:", num * 2)
```

```
请输入一个数字: abc
Traceback (most recent call last):
ValueError: invalid literal for int() with base 10: 'abc'
```

### 2.2 基础语法：try/except

```python
try:
    num = int(input("请输入一个数字: "))
    print("结果是:", num * 2)
except ValueError:
    print("⚠️ 输入的不是数字，请重新运行！")
```

**核心逻辑**：try 块里写"可能出错"的代码，except 块写"出错了怎么办"。

### 2.3 捕获多种异常

```python
try:
    a = int(input("请输入被除数: "))
    b = int(input("请输入除数: "))
    result = a / b
    print(f"{a} ÷ {b} = {result}")
except ValueError:
    print("⚠️ 请输入有效的整数！")
except ZeroDivisionError:
    print("⚠️ 除数不能为零！")
```

> 🔑 **常见异常速查表**：

| 异常类型 | 触发场景 | 示例 |
|----------|----------|------|
| `ValueError` | 类型对但值不对 | `int("abc")` |
| `TypeError` | 类型不匹配 | `"hello" + 5` |
| `ZeroDivisionError` | 除以零 | `10 / 0` |
| `FileNotFoundError` | 文件不存在 | `open("不存在的文件.txt")` |
| `IndexError` | 列表索引越界 | `lst[100]` |
| `KeyError` | 字典键不存在 | `d["不存在的键"]` |
| `NameError` | 变量未定义 | `print(undefined_var)` |

### 2.4 万能捕获（谨慎使用）

```python
try:
    # 任何可能出错的代码
    risky_operation()
except Exception as e:        # Exception 是大部分异常的"父类"
    print(f"出错了: {e}")     # e 是异常对象，包含错误信息
```

> ⚠️ **避坑**：不要在所有地方都用 `except Exception`！它会掩盖真正的 bug。优先捕获具体异常类型。

### 2.5 else 和 finally

```python
try:
    num = int(input("输入数字: "))
    result = 100 / num
except ValueError:
    print("输入的不是数字")
except ZeroDivisionError:
    print("不能除以零")
else:
    # try 块没出错才会执行
    print(f"✅ 计算成功！100 ÷ {num} = {result}")
finally:
    # 不管出不出错，一定会执行
    print("程序运行结束。")
```

| 块 | 什么时候执行 |
|----|-------------|
| `try` | 尝试执行 |
| `except` | try 出错时 |
| `else` | try 没出错时 |
| `finally` | **无论如何都执行**（常用于关闭文件/释放资源） |

### 2.6 raise：主动抛出异常

```python
def check_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数！")
    if age > 150:
        raise ValueError("年龄不可能超过150！")
    return f"{age}岁"

print(check_age(25))    # ✅ 正常
print(check_age(-5))    # ❌ 抛出异常
```

### 2.7 实战：健壮的记账程序

```python
def add_record():
    """添加一条记账记录，带完整异常处理"""
    try:
        date = input("日期 (YYYY-MM-DD): ").strip()
        amount = float(input("金额: "))
        category = input("分类: ").strip()

        if amount <= 0:
            raise ValueError("金额必须大于0！")
        if not category:
            raise ValueError("分类不能为空！")

        return {"date": date, "amount": amount, "category": category}

    except ValueError as e:
        print(f"⚠️ 输入错误: {e}")
        return None
    except KeyboardInterrupt:
        print("\n👋 用户取消输入")
        return None
```

---

## 三、模块导入

### 3.1 什么是模块？

> 一个 `.py` 文件就是一个模块。把相关功能放在一起，需要时 `import` 即可。

```
项目结构：
my_project/
├── main.py          # 主程序
├── utils.py         # 工具函数模块
└── calculator.py    # 计算器模块
```

### 3.2 四种导入方式

```python
# 方式1：导入整个模块
import math
print(math.sqrt(16))   # 4.0
print(math.pi)         # 3.14159...

# 方式2：导入特定函数
from math import sqrt, pi
print(sqrt(16))        # 4.0（不用写 math.）
print(pi)              # 3.14159...

# 方式3：导入并起别名（最常用）
import numpy as np
import pandas as pd
arr = np.array([1, 2, 3])

# 方式4：导入全部（不推荐，容易命名冲突）
from math import *
print(sin(0))          # 不知道 sin 从哪来的
```

### 3.3 创建自己的模块

**第一步**：创建 `utils.py`

```python
# utils.py —— 我的工具模块
def greet(name):
    """打招呼"""
    return f"你好，{name}！"

def add(a, b):
    """加法"""
    return a + b

PI = 3.14159

# 模块内部测试代码（只有直接运行 utils.py 时才执行）
if __name__ == "__main__":
    print("测试 greet:", greet("小明"))
    print("测试 add:", add(3, 5))
```

**第二步**：在 `main.py` 中导入

```python
# main.py
import utils

print(utils.greet("星辰"))   # 你好，星辰！
print(utils.add(2, 3))       # 5
print(utils.PI)              # 3.14159
```

> 🔑 **`if __name__ == "__main__"`** 的作用：这个文件作为模块被导入时，`__name__` 是模块名（`"utils"`），不等于 `"__main__"`，所以测试代码不会执行。直接运行 `utils.py` 时，`__name__` 才是 `"__main__"`，测试代码才会执行。

### 3.4 Python 常用内置模块速览

| 模块 | 用途 | 快速示例 |
|------|------|----------|
| `math` | 数学函数 | `math.sqrt()`, `math.ceil()` |
| `random` | 随机数 | `random.randint(1, 10)` |
| `datetime` | 日期时间 | `datetime.datetime.now()` |
| `os` | 操作系统 | `os.path.exists()`, `os.listdir()` |
| `json` | JSON处理 | `json.dumps()`, `json.loads()` |
| `csv` | CSV处理 | `csv.reader()`, `csv.writer()` |
| `re` | 正则表达式 | `re.search()`, `re.sub()` |
| `sys` | 系统参数 | `sys.argv`, `sys.exit()` |
| `collections` | 高级容器 | `defaultdict`, `Counter` |
| `itertools` | 迭代工具 | `product()`, `combinations()` |

### 3.5 导入时最容易踩的坑

**坑1：循环导入**

```python
# a.py
from b import func_b       # ❌ a 导入 b
def func_a(): pass

# b.py
from a import func_a       # ❌ b 导入 a → 死循环！
def func_b(): pass
```

> **解决**：把公共部分提取到第三个模块 `common.py`。

**坑2：模块名和标准库重名**

```python
# 你自己的文件叫 random.py
import random   # ❌ 导入的是你自己的 random.py，不是标准库！
```

> **解决**：不要用标准库的名字命名自己的文件！

**坑3：`from module import *` 污染命名空间**

```python
from os import *     # ❌ 导入了 os 模块的 100+ 个名字
# 现在你根本不知道 open() 是内置的还是 os 的
```

---

## 四、pip：安装第三方库

### 4.1 什么是 pip？

Python 的包管理器，用来安装别人写好的库。就像手机"应用商店"。

### 4.2 常用命令

```bash
# 安装
pip install requests          # 安装 requests 库
pip install numpy pandas      # 一次安装多个

# 查看已安装
pip list                      # 列出所有已安装的包
pip show requests             # 查看某个包的详细信息

# 卸载
pip uninstall requests

# 导出/还原环境（项目迁移必备）
pip freeze > requirements.txt      # 导出依赖列表
pip install -r requirements.txt    # 从列表安装所有依赖
```

### 4.3 新手必装库推荐

| 库 | 一句话用途 | 安装 |
|----|-----------|------|
| `requests` | HTTP 请求（爬虫/API） | `pip install requests` |
| `pandas` | 数据分析 | `pip install pandas` |
| `openpyxl` | 读写 Excel | `pip install openpyxl` |
| `python-dotenv` | 管理环境变量/密钥 | `pip install python-dotenv` |
| `rich` | 终端彩色输出 | `pip install rich` |

### 4.4 venv 虚拟环境（避坑必读）

> 你之前的笔记中提到了 venv 混淆问题，这里再强调一遍：

```bash
# 创建虚拟环境
python -m venv myenv

# 激活（Windows PowerShell）
myenv\Scripts\Activate.ps1

# 激活（Windows CMD）
myenv\Scripts\activate.bat

# 激活（macOS/Linux）
source myenv/bin/activate

# 退出
deactivate
```

**为什么用 venv？** 不同项目可能需要不同版本的库。venv 把每个项目的依赖隔离开，互不干扰。PyCharm 自动创建的 `.venv` 本质也是 venv，不要和手动创建的混用。

---

## 五、今日练习

### 练习1：安全计算器 ⭐

写一个除法计算器，处理用户可能输入的各种错误：
- 输入非数字 → 提示重新输入
- 除数为0 → 提示不能除以0
- 输入 q 退出程序

```python
# 你的代码
```

### 练习2：模块拆分 ⭐⭐

把 Day 12 的记账本程序拆分成两个文件：
- `account_book.py` —— 核心逻辑（添加/查看/统计）
- `main.py` —— 主程序入口 + 用户交互

### 练习3：健壮的 CSV 读取器 ⭐⭐

写一个函数 `safe_read_csv(filename)`，用异常处理保证：
- 文件不存在时返回空列表，不崩溃
- CSV 格式错误时打印具体哪一行出错
- 空文件返回空列表

```python
import csv

def safe_read_csv(filename):
    """安全读取CSV文件，处理所有可能的异常"""
    # 你的代码
    pass
```

---

## 六、常见问题 & 避坑

| 问题 | 原因 | 解决 |
|------|------|------|
| `except` 捕获了不该捕获的异常 | 用了太宽泛的异常类型 | 优先用具体类型：`except ValueError` |
| 异常处理后程序悄悄吞掉错误 | `except` 里写了 `pass` | 至少要 `print` 或 `log` 错误信息 |
| 导入时报 `ModuleNotFoundError` | 没安装 / 路径不对 | `pip install 包名` 或检查文件路径 |
| 自己的模块导入失败 | 不在同一目录 / 没加 `sys.path` | 确保文件在同一目录，或用 `sys.path.append()` |
| `pip install` 报权限错误 | 用了系统 Python | 用 venv 虚拟环境，或加 `--user` |
| finally 里 return 覆盖了异常 | finally 的 return 优先执行 | 不要在 finally 里写 return |

---

## 七、今日总结

```
✅ try/except/else/finally 四种异常处理结构
✅ raise 主动抛出异常
✅ import 四种导入方式（import / from...import / as / *）
✅ 创建自己的模块 + if __name__ == "__main__" 原理
✅ 常用内置模块速查（math/random/datetime/os/json）
✅ pip 安装/卸载/导出依赖
✅ venv 虚拟环境基本操作
```

---

## 📌 明日预告

> **Day 14：第一阶段总结 + 综合大练习** —— 用学过的所有知识做一个完整的"学生管理系统"

---

> 💡 **学习建议**：
> 1. 先看完语法部分（30 分钟）
> 2. 动手敲练习 1 和练习 2（20 分钟）
> 3. 尝试用 `pip install requests` 安装第一个第三方库（10 分钟）
> 4. 下周即将进入**第二阶段：项目实战**，打好基础！
