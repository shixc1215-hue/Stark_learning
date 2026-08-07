# Python Day 58 - 代码优化与重构

> 经过 57 天的学习，你已经写了很多 Python 代码。今天不学新框架，而是回头审视代码质量——**怎样写出更快、更优雅、更易维护的代码**？这是从"能写代码"到"写好代码"的关键一步。

---

## 一、为什么要优化和重构？

**重构（Refactoring）**：在不改变代码外部行为的前提下，改善内部结构，使其更易读、更易维护。

**优化（Optimization）**：提升代码的运行速度或降低资源消耗。

| 维度 | 重构关注 | 优化关注 |
|------|---------|---------|
| 目标 | 可读性、可维护性 | 性能、效率 |
| 时机 | 日常开发中持续进行 | 发现瓶颈后再做 |
| 原则 | "让代码为自己说话" | "先确保正确，再追求快" |
| 工具 | 静态分析、Code Review | Profiler 基准测试 |

> **黄金法则**：不要过早优化。—— Donald Knuth
> 先让代码跑对，再通过 Profile 工具找到真正的瓶颈，针对性地优化。

---

## 二、性能分析工具——用数据说话

### 2.1 time 模块——最简单的计时

```python
import time

def sum_with_loop(n: int) -> int:
    """用 for 循环求和"""
    total = 0
    for i in range(n):
        total += i
    return total

def sum_with_builtin(n: int) -> int:
    """用内置函数求和"""
    return sum(range(n))

# 计时对比
start = time.perf_counter()       # 高精度计时器（推荐）
sum_with_loop(1_000_000)
elapsed_loop = time.perf_counter() - start

start = time.perf_counter()
sum_with_builtin(1_000_000)
elapsed_builtin = time.perf_counter() - start

print(f"for 循环:  {elapsed_loop:.4f} 秒")
print(f"sum 内置:  {elapsed_builtin:.4f} 秒")
print(f"内置函数快了 {elapsed_loop / elapsed_builtin:.1f} 倍")
```

### 2.2 timeit 模块——精准基准测试

```python
import timeit

# timeit 会自动多次运行取平均值，减少误差
loop_time = timeit.timeit(
    "sum(range(10000))",
    number=1000       # 重复执行 1000 次
)
print(f"sum(range(10000)) 执行 1000 次耗时: {loop_time:.4f} 秒")

# 对比两种写法
code_a = timeit.timeit(
    "result = [i**2 for i in range(1000)]",
    number=1000
)
code_b = timeit.timeit(
    "result = list(map(lambda i: i**2, range(1000)))",
    number=1000
)
print(f"列表推导式:  {code_a:.4f}s")
print(f"map+lambda:  {code_b:.4f}s")
```

### 2.3 cProfile——找出性能瓶颈

```python
import cProfile

def slow_function():
    """模拟一个有性能问题的函数"""
    result = []
    for i in range(10000):
        if i % 2 == 0:
            result.append(i ** 2)
    return sorted(result, reverse=True)

def main():
    data = slow_function()
    return sum(data)

# 使用 cProfile 分析——看哪行代码最耗时
cProfile.run("main()", sort="cumulative")
```

**输出解读要点：**

| 列名 | 含义 |
|------|------|
| `ncalls` | 调用次数 |
| `tottime` | 函数本身耗时（不含子函数） |
| `percall` | `tottime / ncalls` |
| `cumtime` | 累计耗时（含子函数） |
| `filename:lineno(function)` | 代码位置 |

**技巧**：`sort="cumtime"` 按累计耗时排序，快速定位最耗时的函数。

### 2.4 line_profiler——逐行分析（按行找瓶颈）

```python
# 安装: pip install line_profiler

# 在函数上添加 @profile 装饰器
@profile
def process_data(data):
    total = 0
    for item in data:              # <-- line_profiler 会告诉你这行耗时多少
        total += item
    result = [x * 2 for x in data]
    return total, result

# 命令行运行：
# kernprof -l -v your_script.py
```

> `line_profiler` 精确到每一行，比 `cProfile` 更适合定位具体哪行代码慢。

---

## 三、Python 性能优化 10 条实战技巧

### 技巧 1：用列表推导式替代循环拼接

```python
# ❌ 慢——反复 append 导致列表多次扩容
result = []
for i in range(10000):
    result.append(i ** 2)

# ✅ 快——列表推导式预分配内存
result = [i ** 2 for i in range(10000)]

# ❌ 慢——循环中字符串拼接（字符串不可变，每次创建新对象）
text = ""
for word in words:
    text += word + " "

# ✅ 快——join 一次性合并
text = " ".join(words)
```

### 技巧 2：用集合替代列表做"存在性检查"

```python
data_list = list(range(10000))
data_set = set(range(10000))

# 列表查找: O(n) —— 需要逐个比较
# %timeit 9999 in data_list   → 约 0.1 毫秒

# 集合查找: O(1) —— 哈希表直接定位
# %timeit 9999 in data_set    → 约 0.00003 毫秒
# 快了 3000 倍！
```

**适用场景**：频繁使用 `in` 检查元素是否存在时，优先用 `set`。

### 技巧 3：用局部变量加速属性访问

```python
import math

# ❌ 每次循环都要查找 math.sqrt
result = [math.sqrt(i) for i in range(10000)]

# ✅ 局部变量引用——减少属性查找开销
sqrt = math.sqrt  # 只做一次属性查找
result = [sqrt(i) for i in range(10000)]
```

> Python 查找局部变量比查找全局变量/对象属性快。在循环中频繁调用的函数，可以先赋值给局部变量。

### 技巧 4：用生成器节省内存

```python
# ❌ 一次性加载全部数据到内存
numbers = [i ** 2 for i in range(1_000_000)]  # 约 8MB 内存

# ✅ 按需生成——内存几乎为零
numbers = (i ** 2 for i in range(1_000_000))   # 生成器表达式

# 处理大文件时同理：
# ❌ lines = f.readlines()         # 全部读入内存
# ✅ for line in f: ...            # 逐行读取
```

### 技巧 5：避免在循环中重复计算

```python
# ❌ len(data) 每次循环都调用
for i in range(len(data)):
    if i < len(data) - 1:
        print(data[i], data[i + 1])

# ✅ 提前计算一次
n = len(data)
for i in range(n):
    if i < n - 1:
        print(data[i], data[i + 1])
```

### 技巧 6：用 `str.format` 或 f-string 替代 `%` 和 `+`

```python
name = "Python"
version = 3

# ⚠️ 旧式 % 格式化
msg = "Welcome to %s %d" % (name, version)

# ✅ f-string（最快、最可读）
msg = f"Welcome to {name} {version}"
```

> f-string 在 CPython 3.12+ 中性能最优，同时也是最易读的格式化方式。

### 技巧 7：用 `collections` 模块替代手写逻辑

```python
from collections import Counter, defaultdict

# ❌ 手写计数
counts = {}
for item in data:
    if item in counts:
        counts[item] += 1
    else:
        counts[item] = 1

# ✅ Counter 一行搞定
counts = Counter(data)

# ❌ 手写分组
groups = {}
for key, value in pairs:
    if key not in groups:
        groups[key] = []
    groups[key].append(value)

# ✅ defaultdict 自动初始化
groups = defaultdict(list)
for key, value in pairs:
    groups[key].append(value)
```

### 技巧 8：用 `itertools` 替代嵌套循环

```python
import itertools

# ❌ 双重循环
pairs = []
for a in [1, 2, 3]:
    for b in ['x', 'y']:
        pairs.append((a, b))

# ✅ itertools.product（笛卡尔积）
pairs = list(itertools.product([1, 2, 3], ['x', 'y']))

# ✅ itertools.combinations（组合）
from itertools import combinations
combos = list(combinations([1, 2, 3, 4], 2))
# [(1,2), (1,3), (1,4), (2,3), (2,4), (3,4)]
```

### 技巧 9：用 `__slots__` 减少类实例内存

```python
class Point:
    """普通类——每个实例有一个 __dict__ 字典（额外内存开销）"""
    def __init__(self, x, y):
        self.x = x
        self.y = y

class SlotPoint:
    """使用 __slots__ —— 禁止动态属性，节省 40-50% 内存"""
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y

# 对比内存占用
import sys
p1 = Point(1, 2)
p2 = SlotPoint(1, 2)
print(f"普通类实例:  {sys.getsizeof(p1.__dict__)} 字节")
print(f"__slots__实例: {sys.getsizeof(p2)} 字节")
```

### 技巧 10：缓存重复计算结果

```python
import functools

# 方法一：@lru_cache（适合纯函数）
@functools.lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))  # 毫秒级完成（无缓存时几乎不可能）

# 方法二：手动缓存（适合类方法）
class DataService:
    def __init__(self):
        self._cache = {}

    def get_user(self, user_id: int) -> dict:
        if user_id in self._cache:
            return self._cache[user_id]  # 命中缓存
        user = self._fetch_from_db(user_id)
        self._cache[user_id] = user
        return user
```

---

## 四、代码重构原则与实战

### 4.1 提取函数（Extract Function）

**核心原则**：一个函数只做一件事（单一职责原则）。

```python
# ❌ 一个函数做了太多事
def process_order(order):
    # 验证
    if not order.get("items"):
        raise ValueError("订单为空")
    for item in order["items"]:
        if item["price"] < 0:
            raise ValueError("价格不能为负")

    # 计算金额
    total = sum(item["price"] * item["qty"] for item in order["items"])
    discount = total * 0.1 if total > 500 else 0
    final_price = total - discount

    # 保存
    order["total"] = final_price
    save_to_db(order)

    # 通知
    send_email(order["user_email"], f"订单 {order['id']} 已处理")

    return order

# ✅ 拆分为职责明确的小函数
def validate_order(order: dict) -> None:
    """验证订单数据"""
    if not order.get("items"):
        raise ValueError("订单为空")
    for item in order["items"]:
        if item["price"] < 0:
            raise ValueError("价格不能为负")


def calculate_total(items: list) -> float:
    """计算订单总金额（含折扣）"""
    total = sum(item["price"] * item["qty"] for item in items)
    discount = total * 0.1 if total > 500 else 0
    return total - discount


def save_order(order: dict) -> None:
    """保存订单到数据库"""
    save_to_db(order)


def notify_user(email: str, order_id: str) -> None:
    """发送订单通知"""
    send_email(email, f"订单 {order_id} 已处理")


def process_order(order: dict) -> dict:
    """处理订单——主流程一目了然"""
    validate_order(order)
    order["total"] = calculate_total(order["items"])
    save_order(order)
    notify_user(order["user_email"], order["id"])
    return order
```

### 4.2 消除重复代码（DRY 原则）

```python
# ❌ 大量重复逻辑
def show_user_info():
    conn = sqlite3.connect("db.sqlite")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    data = cursor.fetchall()
    conn.close()
    return data

def show_order_info():
    conn = sqlite3.connect("db.sqlite")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM orders")
    data = cursor.fetchall()
    conn.close()
    return data

# ✅ 提取公共逻辑（可以用上下文管理器）
from contextlib import contextmanager

@contextmanager
def get_cursor(db_path: str = "db.sqlite"):
    """数据库游标上下文管理器——自动关闭连接"""
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    try:
        yield cursor
    finally:
        conn.close()

def show_user_info():
    with get_cursor() as cursor:
        cursor.execute("SELECT * FROM users")
        return cursor.fetchall()

def show_order_info():
    with get_cursor() as cursor:
        cursor.execute("SELECT * FROM orders")
        return cursor.fetchall()
```

### 4.3 用字典替代多重 if-elif

```python
# ❌ 冗长的 if-elif 链
def get_handler(action: str):
    if action == "create":
        return handle_create
    elif action == "read":
        return handle_read
    elif action == "update":
        return handle_update
    elif action == "delete":
        return handle_delete
    else:
        raise ValueError(f"未知操作: {action}")

# ✅ 分发字典——清晰高效
HANDLERS = {
    "create": handle_create,
    "read": handle_read,
    "update": handle_update,
    "delete": handle_delete,
}

def get_handler(action: str):
    handler = HANDLERS.get(action)
    if handler is None:
        raise ValueError(f"未知操作: {action}")
    return handler
```

### 4.4 用枚举替代魔法字符串

```python
# ❌ 魔法字符串——容易拼错、不易维护
def set_status(order, status):
    if status == "pending":
        ...
    elif status == "shipped":
        ...

# ✅ 枚举——IDE 自动补全、类型安全
from enum import Enum, auto

class OrderStatus(Enum):
    PENDING = auto()
    PAID = auto()
    SHIPPED = auto()
    DELIVERED = auto()
    CANCELLED = auto()

def set_status(order, status: OrderStatus):
    if status == OrderStatus.PENDING:
        ...
    elif status == OrderStatus.SHIPPED:
        ...

# 使用时：set_status(order, OrderStatus.SHIPPED)
```

### 4.5 用 dataclass 简化数据容器

```python
# ❌ 手写 __init__ + __repr__
class Student:
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

    def __repr__(self):
        return f"Student(name={self.name}, age={self.age}, score={self.score})"

# ✅ dataclass——自动生成 __init__、__repr__、__eq__ 等
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    age: int
    score: float
```

---

## 五、代码风格与可读性

### 5.1 命名规范

```python
# ✅ 变量名要有意义
# ❌ x, y, tmp, data, info —— 读了不知道是什么
# ✅ user_name, total_price, is_valid, file_path

# ✅ 函数名用动词开头
# ❌ data() —— 不知道做什么
# ✅ get_user(), calculate_total(), validate_input()

# ✅ 布尔变量用 is/has/can 开头
is_active = True
has_permission = False
can_edit = user.role == "admin"

# ✅ 常量用全大写
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30
API_BASE_URL = "https://api.example.com"
```

### 5.2 函数设计原则

```python
# 原则 1：函数不超过 20 行（如果超过，考虑拆分）
# 原则 2：参数不超过 4 个（多了用 dataclass 或字典）
# 原则 3：返回值类型一致（不要有时返回 int 有时返回 None）

# ❌ 参数太多、返回值不一致
def create_user(name, age, email, phone, address, role):
    if role == "admin":
        return {"error": "不能直接创建管理员"}
    return {"id": 1, "name": name}

# ✅ 用 dataclass 封装参数，统一返回
from dataclasses import dataclass

@dataclass
class CreateUserParams:
    name: str
    age: int
    email: str
    phone: str
    address: str
    role: str

def create_user(params: CreateUserParams) -> dict:
    if params.role == "admin":
        raise ValueError("不能直接创建管理员")
    return {"id": 1, "name": params.name}
```

### 5.3 用 Type Hints 增强可读性

```python
from typing import Optional

# ❌ 没有类型提示——需要看实现才知道参数类型
def search_user(name, age_limit):

# ✅ 类型提示——函数签名即文档
def search_user(
    name: str,
    age_limit: Optional[int] = None,
) -> list[dict]:
```

---

## 六、综合重构实战——优化一个"脏"函数

下面是一个典型的"需要重构"的函数，我们一步步改进：

```python
# ====== 原始版本：功能正确但代码质量差 ======
def getTop10(d):
    result = {}
    for k in d:
        if isinstance(d[k], (int, float)):
            result[k] = d[k]
    items = list(result.items())
    items.sort(key=lambda x: x[1], reverse=True)
    return items[:10]
```

**问题**：命名差、没有类型提示、没有文档、不够 Pythonic。

```python
# ====== 重构后 ======
from typing import Any

def get_top_values(
    data: dict[str, Any],
    top_n: int = 10,
) -> list[tuple[str, float]]:
    """
    从字典中提取数值最大的前 N 个键值对。

    Args:
        data: 待分析的字典（可能包含非数值值）
        top_n: 返回前 N 个结果（默认 10）

    Returns:
        按值降序排列的 (key, value) 列表

    Example:
        >>> get_top_values({"a": 100, "b": "hello", "c": 50}, top_n=2)
        [("a", 100), ("c", 50)]
    """
    numeric_items = [
        (key, float(value))
        for key, value in data.items()
        if isinstance(value, (int, float))
    ]
    return sorted(numeric_items, key=lambda x: x[1], reverse=True)[:top_n]
```

**改进点总结：**

| 维度 | 原版 | 重构后 |
|------|------|--------|
| 命名 | `getTop10(d)` 驼峰 | `get_top_values(data)` 蛇形 |
| 类型提示 | 无 | 完整标注 |
| 文档 | 无 | 含 Args/Returns/Example |
| 灵活性 | 固定 Top 10 | 参数控制 `top_n` |
| 可读性 | 命令式循环 | 列表推导式 + sorted |

---

## 七、静态代码检查工具

```bash
# mypy —— 类型检查
pip install mypy
mypy your_script.py

# ruff —— 超快代码格式化 + 检查（替代 flake8 + black）
pip install ruff
ruff check your_script.py        # 检查
ruff format your_script.py       # 格式化

# pylint —— 综合代码质量检查
pip install pylint
pylint your_script.py
```

**推荐工作流**：保存代码 → `ruff format` 自动格式化 → `ruff check` 检查问题 → `mypy` 类型检查。

---

## 八、练习题

### 练习 1：性能对比（基础）

```python
# 题目：用 timeit 对比以下三种方式将字符串列表转为整数的速度
# 方法 A：for 循环
# 方法 B：列表推导式
# 方法 C：map(int, ...)
# 哪种最快？快多少倍？

strings = [str(i) for i in range(10000)]
# 请写出三种实现并用 timeit.timeit() 对比
```

### 练习 2：重构练习（中级）

```python
# 题目：重构以下函数，使其符合今天学到的原则
# （合理命名、提取函数、添加类型提示和文档）

def f(lst, t):
    r = []
    for x in lst:
        if x > t:
            r.append(x * 2)
        else:
            r.append(x)
    s = sum(r)
    a = s / len(r)
    return {"items": r, "total": s, "avg": a}
```

### 练习 3：Profile 实战（挑战）

```python
# 题目：下面这个函数在数据量大时很慢
# 1. 用 cProfile 找出瓶颈
# 2. 用至少 3 种今天学到的技巧优化它
# 3. 对比优化前后的运行时间

def find_duplicates(data):
    """找出列表中的重复元素及其出现次数"""
    duplicates = []
    seen = []
    for i, item in enumerate(data):
        is_dup = False
        for j in range(i):
            if data[j] == item:
                is_dup = True
                break
        if is_dup:
            count = 0
            for x in data:
                if x == item:
                    count += 1
            if item not in seen:
                seen.append(item)
                duplicates.append((item, count))
    return duplicates
```

---

## 九、常见问题

### Q1：重构时怎么保证不引入 bug？

**A**：重构前先写测试（单元测试），重构后运行测试确保行为不变。Day 30 学的 `unittest` 正好派上用场。也可以用 `git commit` 先保存当前版本，重构后如果出问题可以快速回退。

### Q2：性能优化做到什么程度就够了？

**A**：遵循"够用就好"原则。用户感知不到的差异（毫秒级以内）不需要优化。优化的投入要与收益匹配——花 2 小时优化一个只运行 0.01 秒的函数是不值得的。

### Q3：ruff 和 black、flake8 是什么关系？

**A**：`ruff` 是新一代工具，用 Rust 编写，一个工具同时替代了 `black`（格式化）和 `flake8`（代码检查），速度快 10-100 倍。推荐直接用 `ruff`。

### Q4：代码优化和可读性冲突时怎么办？

**A**：**可读性优先**。除非这个代码是性能关键路径（如被调用百万次），否则保持代码清晰比省几微秒更重要。在需要优化但会降低可读性的地方，添加注释说明原因。

### Q5：__slots__ 什么时候该用？

**A**：当你需要创建大量（数万+）同类型的简单对象时，`__slots__` 可以显著减少内存。普通应用中不需要刻意使用——它禁止了动态添加属性，灵活性降低。

---

## 十、推荐资源

- **书籍**：《重构：改善既有代码的设计》（Martin Fowler）—— 经典必读
- **书籍**：《代码整洁之道》（Robert C. Martin）
- **在线**：[Python 官方性能优化指南](https://docs.python.org/3/howto/performance.html)
- **在线**：[Refactoring Guru](https://refactoring.guru/) —— 重构模式图文讲解
- **在线**：[廖雪峰 Python 教程 - 高级特性](https://www.liaoxuefeng.com/wiki/1016959663602400/1017454145019696)
- **工具**：[Ruff 官方文档](https://docs.astral.sh/ruff/)

---

> **今日小结**：代码优化不是炫技，而是用最简洁的方式解决问题。性能分析让优化有据可依，重构让代码可持续演进。明天我们将学习项目部署——把代码变成真正可用的产品！
