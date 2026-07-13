# Python Day 27：上下文管理器

> **学习目标**：理解 `with` 语句背后的原理，掌握自定义上下文管理器的两种方式（基于类和基于生成器），学会用上下文管理器优雅地管理资源（文件、网络连接、锁等）。

---

## 一、从一个熟悉的问题说起

你在前面的学习中，已经无数次写过这样的代码：

```python
# 打开文件 → 操作 → 关闭文件
f = open("data.txt", "r", encoding="utf-8")
try:
    content = f.read()
    print(content)
finally:
    f.close()  # 无论是否出错，都要关闭文件
```

这段代码有一个问题：**太啰嗦了**。每次都要手动 `open`、`try/finally`、`close`，容易忘记关闭，代码也不够优雅。

Python 提供了一个更简洁的写法：

```python
# 用 with 语句，文件会自动关闭
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)
# 离开 with 块后，文件已经自动关闭了，不需要手动 f.close()
```

`with` 语句背后的魔法，就是**上下文管理器（Context Manager）**。今天我们就来揭开它的面纱。

---

## 二、什么是上下文管理器？

### 2.1 概念

上下文管理器是一个对象，它定义了**进入**和**退出**某个"上下文"时应该执行的操作。最常见的用途就是资源管理：

| 场景 | 进入时（获取资源） | 退出时（释放资源） |
|------|-------------------|-------------------|
| 文件操作 | `open()` 打开文件 | `close()` 关闭文件 |
| 网络请求 | 建立连接 | 关闭连接 |
| 数据库操作 | 获取连接/游标 | 释放连接/游标 |
| 线程锁 | `acquire()` 获取锁 | `release()` 释放锁 |
| 临时目录 | 创建临时文件夹 | 删除临时文件夹 |

### 2.2 `with` 语句的执行流程

```python
with 表达式 as 变量:
    # 在这个代码块中，资源是可用的
    pass
# 离开代码块后，资源被自动清理
```

执行顺序：
1. **执行** `表达式`，得到一个上下文管理器对象
2. **调用** 上下文管理器的 `__enter__()` 方法，返回值赋给 `as` 后面的 `变量`
3. **执行** `with` 缩进块内的代码
4. **离开** 缩进块时，自动调用 `__exit__()` 方法进行清理
5. 无论缩进块内是否发生异常，`__exit__()` **都会被执行**

---

## 三、方式一：基于类实现上下文管理器

一个对象要成为上下文管理器，需要实现两个"魔术方法"：

```python
class MyContext:
    def __enter__(self):
        """进入上下文时调用，返回值通过 as 赋给变量"""
        print(">>> 进入上下文")
        return self  # 通常返回 self 或资源对象

    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出上下文时调用（无论如何都会执行）"""
        print("<<< 退出上下文")
        if exc_type:  # 如果有异常发生
            print(f"捕获到异常：{exc_type.__name__}: {exc_val}")
        return True   # 返回 True 表示异常已被处理，不再向外抛出
```

### 3.1 基本使用

```python
# 使用自定义的上下文管理器
with MyContext() as ctx:
    print("  --- 在上下文中执行代码 ---")
print("已经离开上下文了")
```

输出：
```
>>> 进入上下文
  --- 在上下文中执行代码 ---
<<< 退出上下文
已经离开上下文了
```

### 3.2 异常处理：`__exit__` 的三个参数

`__exit__` 方法接收三个参数，当 `with` 块内**没有异常**时，它们都是 `None`：

| 参数 | 含义 | 无异常时 | 有异常时 |
|------|------|---------|---------|
| `exc_type` | 异常类型 | `None` | 如 `ValueError` |
| `exc_val` | 异常实例 | `None` | 异常对象本身 |
| `exc_tb` | 异常追踪信息 | `None` | traceback 对象 |

```python
class SafeDivision:
    """安全的除法运算上下文"""

    def __enter__(self):
        print("开始安全除法运算")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            print(f"运算出错：{exc_type.__name__}: {exc_val}")
            print("错误已记录，程序继续运行")
            return True  # 吞掉异常
        print("运算顺利完成")
        return False  # 没有异常，正常退出

# 测试：正常情况
with SafeDivision():
    result = 10 / 2
    print(f"10 ÷ 2 = {result}")

print("---")

# 测试：异常情况
with SafeDivision():
    result = 10 / 0  # 会抛出 ZeroDivisionError
    print("这行不会执行")
print("程序继续运行，没有被中断")
```

输出：
```
开始安全除法运算
10 ÷ 2 = 5.0
运算顺利完成
---
开始安全除法运算
运算出错：ZeroDivisionError: division by zero
错误已记录，程序继续运行
程序继续运行，没有被中断
```

> **注意**：`__exit__` 返回 `True` 表示异常被"吞掉"，不会继续向上传播。返回 `False`（或不返回值）则异常会继续向上抛出。**通常建议只在你能正确处理异常时才返回 `True`**。

### 3.3 实战：自动计时的上下文管理器

```python
import time

class Timer:
    """计时器：自动记录代码块的执行时间"""

    def __init__(self, name="代码块"):
        self.name = name
        self.start_time = 0
        self.elapsed = 0

    def __enter__(self):
        self.start_time = time.time()
        print(f"[{self.name}] 开始计时...")
        return self  # 返回 self，方便外部获取时间

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.time() - self.start_time
        print(f"[{self.name}] 执行完毕，耗时：{self.elapsed:.4f} 秒")
        if exc_type:
            print(f"[{self.name}] 执行过程中出现异常：{exc_type.__name__}")
        return False  # 不吞异常，让它正常传播

# 使用示例
with Timer("数据加载") as t:
    time.sleep(1.5)  # 模拟耗时操作
    print(f"计时器对象也可以通过 as 变量访问")

# 退出 with 后，依然可以读取耗时
print(f"总共耗时：{t.elapsed:.4f} 秒")
```

### 3.4 实战：临时修改工作目录

```python
import os

class ChangeDirectory:
    """临时切换工作目录，退出后自动恢复"""

    def __init__(self, target_path):
        self.target_path = target_path
        self.original_path = ""

    def __enter__(self):
        self.original_path = os.getcwd()
        os.chdir(self.target_path)
        print(f"工作目录切换为：{os.getcwd()}")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        os.chdir(self.original_path)
        print(f"工作目录恢复为：{os.getcwd()}")
        return False

# 使用示例
print(f"当前目录：{os.getcwd()}")
with ChangeDirectory("C:/Windows"):
    print(f"在 with 块内：{os.getcwd()}")
print(f"退出后：{os.getcwd()}")
```

---

## 四、方式二：`contextlib.contextmanager` 装饰器

每次都要写一个类、定义 `__enter__` 和 `__exit__`，有时候显得太重了。Python 标准库的 `contextlib` 模块提供了一个更简洁的方式——**用生成器函数来创建上下文管理器**。

### 4.1 基本用法

```python
from contextlib import contextmanager

@contextmanager
def simple_context():
    # 这部分相当于 __enter__ 之前的准备工作
    print(">>> 进入上下文")
    yield           # ⭐ yield 之前的代码 = __enter__ 的逻辑
    # 这部分相当于 __exit__ 的逻辑
    print("<<< 退出上下文")
```

**关键规则**：
- `yield` 之前的代码 → 进入上下文时执行（相当于 `__enter__`）
- `yield` 本身 → `with ... as` 接收的值
- `yield` 之后的代码 → 退出上下文时执行（相当于 `__exit__`）
- 如果有异常，`yield` 处会接收到异常；可以用 `try/finally` 包裹

### 4.2 使用示例

```python
with simple_context() as val:
    print(f"  val 的值是：{val}")  # val 是 yield 的返回值
```

输出：
```
>>> 进入上下文
  val 的值是：None
<<< 退出上下文
```

### 4.3 yield 一个有意义的值

```python
from contextlib import contextmanager

@contextmanager
def open_file(filename, mode="r"):
    """自己实现一个简化版的文件打开上下文"""
    f = open(filename, mode, encoding="utf-8")
    try:
        yield f          # 把文件对象交给 with 块使用
    finally:
        f.close()        # 无论如何都关闭文件

# 使用起来和内置的 open 一样
with open_file("test.txt", "w") as f:
    f.write("Hello, ContextManager!")
# 文件已自动关闭
```

### 4.4 带异常处理的生成器上下文

```python
from contextlib import contextmanager

@contextmanager
def safe_divide():
    """安全的除法运算上下文（生成器版）"""
    try:
        yield
    except ZeroDivisionError as e:
        print(f"捕获到异常：{e}，程序继续运行")

# 正常情况
with safe_divide():
    result = 10 / 2
    print(f"结果：{result}")

# 异常情况
with safe_divide():
    result = 10 / 0  # 异常被安全捕获
print("程序没有被中断")
```

### 4.5 对比两种方式

| 特性 | 基于类 | 基于 `@contextmanager` |
|------|--------|----------------------|
| 代码量 | 较多（需定义类） | 较少（一个函数即可） |
| 适用场景 | 复杂状态管理 | 简单的资源获取/释放 |
| 异常处理 | `__exit__` 的三个参数 | `try/except/finally` |
| 可重入 | 可以 | 通常不行 |
| 可维护性 | 复杂场景更清晰 | 简单场景更直观 |

**选择建议**：
- 简单的资源开关（打开/关闭） → 用 `@contextmanager`
- 需要维护状态、或 `__enter__`/`__exit__` 逻辑复杂 → 用类

---

## 五、`contextlib` 模块的实用工具

Python 标准库 `contextlib` 提供了好几个开箱即用的工具，非常实用：

### 5.1 `suppress` — 忽略指定异常

```python
from contextlib import suppress

# 以前要写
try:
    os.remove("不存在的文件.txt")
except FileNotFoundError:
    pass

# 现在可以写
with suppress(FileNotFoundError):
    os.remove("不存在的文件.txt")
```

`suppress` 相当于一个"静默的 except"，只忽略你指定的异常类型。

```python
# 忽略多种异常
with suppress(FileNotFoundError, PermissionError):
    os.remove("/protected/file.txt")
```

### 5.2 `closing` — 确保对象被关闭

```python
from contextlib import closing
from urllib.request import urlopen

# closing 会自动调用对象的 close() 方法
with closing(urlopen("https://www.example.com")) as page:
    content = page.read(100)
    print(content)
# 离开 with 块后，page.close() 被自动调用
```

### 5.3 `redirect_stdout` / `redirect_stderr` — 重定向输出

```python
from contextlib import redirect_stdout
import io

# 把 print 的输出捕获到字符串中
f = io.StringIO()
with redirect_stdout(f):
    print("这行不会显示在控制台")
    print("而是被重定向到了 StringIO 中")

captured = f.getvalue()
print(f"捕获到的内容：{captured}")
# 输出：捕获到的内容：这行不会显示在控制台\n而是被重定向到了 StringIO 中\n
```

### 5.4 `nullcontext` — 空上下文（Python 3.7+）

当你需要一个"什么都不做"的上下文管理器时（通常用于条件判断）：

```python
from contextlib import nullcontext

use_lock = False

# 根据条件选择是否使用锁
if use_lock:
    ctx = threading.Lock()
else:
    ctx = nullcontext()  # 空操作，什么都不做

with ctx:
    print("这段代码可能上锁，也可能不上锁")
```

---

## 六、综合实战：数据库连接池管理器

让我们把今天学的知识综合起来，实现一个模拟的数据库连接池上下文管理器：

```python
import time
from contextlib import contextmanager

class ConnectionPool:
    """模拟的数据库连接池"""

    def __init__(self, max_connections=3):
        self.max_connections = max_connections
        self.available = []  # 可用连接列表
        self.in_use = set()  # 正在使用的连接ID
        self._next_id = 0

    def _create_connection(self):
        """创建新连接"""
        self._next_id += 1
        conn_id = f"conn-{self._next_id}"
        print(f"  [连接池] 创建新连接：{conn_id}")
        return conn_id

    def acquire(self):
        """获取一个连接"""
        if self.available:
            conn_id = self.available.pop()
            print(f"  [连接池] 复用已有连接：{conn_id}")
        elif len(self.in_use) < self.max_connections:
            conn_id = self._create_connection()
        else:
            raise RuntimeError("连接池已满，无法获取新连接！")

        self.in_use.add(conn_id)
        return conn_id

    def release(self, conn_id):
        """释放连接回连接池"""
        if conn_id in self.in_use:
            self.in_use.remove(conn_id)
            self.available.append(conn_id)
            print(f"  [连接池] 释放连接：{conn_id}")


@contextmanager
def get_connection(pool):
    """从连接池获取连接的上下文管理器"""
    conn_id = pool.acquire()
    try:
        print(f"  [业务] 使用连接 {conn_id} 执行查询...")
        yield conn_id          # 把连接交给业务代码使用
    finally:
        pool.release(conn_id)  # 无论如何都释放连接


# ===== 使用演示 =====
pool = ConnectionPool(max_connections=2)
print("=== 模拟多个业务操作 ===\n")

# 操作1
print("操作1：")
with get_connection(pool) as conn:
    print(f"  [业务] 拿到的连接：{conn}")

# 操作2（复用连接）
print("\n操作2：")
with get_connection(pool) as conn:
    print(f"  [业务] 拿到的连接：{conn}")

# 操作3（模拟异常）
print("\n操作3（模拟出错）：")
with get_connection(pool) as conn:
    print(f"  [业务] 拿到的连接：{conn}")
    raise ValueError("模拟数据库查询出错！")
except ValueError as e:
    print(f"  [业务] 捕获异常：{e}")

print("\n=== 所有操作完成 ===")
```

输出：
```
=== 模拟多个业务操作 ===

操作1：
  [连接池] 创建新连接：conn-1
  [业务] 使用连接 conn-1 执行查询...
  [业务] 拿到的连接：conn-1
  [连接池] 释放连接：conn-1

操作2：
  [连接池] 复用已有连接：conn-1
  [业务] 使用连接 conn-1 执行查询...
  [业务] 拿到的连接：conn-1
  [连接池] 释放连接：conn-1

操作3（模拟出错）：
  [连接池] 复用已有连接：conn-1
  [业务] 使用连接 conn-1 执行查询...
  [业务] 拿到的连接：conn-1
  [连接池] 释放连接：conn-1
  [业务] 捕获异常：模拟数据库查询出错！

=== 所有操作完成 ===
```

注意：即使操作3中抛出了异常，连接依然被正确释放了——这就是上下文管理器的价值。

---

## 七、练习题

### 练习1：文件写入管理器（基础）

编写一个 `WriteSafe` 上下文管理器，要求：
- 进入时创建/打开文件
- 退出时自动关闭文件
- 如果写入过程中出错，打印错误信息但不要让程序崩溃
- 统计写入的字节数

```python
# 提示框架
class WriteSafe:
    def __init__(self, filename):
        self.filename = filename
        self.file = None
        self.bytes_written = 0

    def __enter__(self):
        # TODO: 打开文件
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # TODO: 关闭文件，处理异常
        pass

# 测试代码
with WriteSafe("output.txt") as ws:
    ws.file.write("Hello, Python!")
# 退出后检查 ws.bytes_written
```

### 练习2：临时目录管理器（进阶）

用 `@contextmanager` 实现一个 `temp_directory` 上下文管理器：
- 进入时创建一个临时目录
- 退出时删除该临时目录及其中所有文件
- 即使目录创建失败也不要崩溃

```python
import tempfile
import shutil
from contextlib import contextmanager

@contextmanager
def temp_directory(prefix="tmp_"):
    # TODO: 实现这个函数
    pass

# 测试：在临时目录中创建文件，退出后目录应该被删除
with temp_directory() as tmp_dir:
    print(f"临时目录：{tmp_dir}")
    # 在临时目录中创建文件
    with open(f"{tmp_dir}/test.txt", "w") as f:
        f.write("临时文件")
print(f"目录还存在吗？{os.path.exists(tmp_dir)}")  # 应该输出 False
```

### 练习3：重试上下文管理器（挑战）

实现一个 `retry` 上下文管理器，当代码块抛出指定异常时自动重试：

```python
from contextlib import contextmanager

@contextmanager
def retry(max_attempts=3, exceptions=(Exception,), delay=1):
    """最多重试 max_attempts 次，每次间隔 delay 秒"""
    # TODO: 实现自动重试逻辑
    pass

# 测试：模拟一个不稳定的操作
import random

attempt_count = 0
with retry(max_attempts=5, exceptions=(ValueError,), delay=0.5):
    attempt_count += 1
    if attempt_count < 3:
        raise ValueError("服务暂时不可用")
    print("操作成功！")
```

---

## 八、常见问题（FAQ）

### Q1：`with` 语句中 `as` 部分可以省略吗？
**可以**。如果不需要使用 `__enter__` 返回的值，直接省略 `as` 即：

```python
with open("data.txt", "r"):
    content = ...  # 但这样拿不到文件对象 f
```

不过通常文件操作都会需要 `as f`。对于不需要返回值的场景更常见，比如 `with suppress(FileNotFoundError):`。

### Q2：`__exit__` 返回 `True` 和 `False` 有什么区别？
- `return True` → 异常被"吞掉"，`with` 块外面不会看到这个异常
- `return False` 或不返回值 → 异常会继续向上传播

**绝大多数情况下，返回 `False` 是更好的选择**。只在你能完全处理异常、且调用者不需要知道时才返回 `True`。

### Q3：`@contextmanager` 装饰的函数中，`yield` 后面的代码一定会执行吗？
**如果 `with` 块内发生了异常，`yield` 后面的代码可能不会执行**——除非你用 `try/finally` 包裹 `yield`：

```python
from contextlib import contextmanager

@contextmanager
def buggy_context():
    print("进入")
    yield
    print("退出")  # ⚠️ 如果有异常，这行可能不执行！

# 正确写法：用 try/finally
@contextmanager
def safe_context():
    print("进入")
    try:
        yield
    finally:
        print("退出")  # ✅ 无论如何都会执行
```

**最佳实践**：`yield` 始终放在 `try` 块中，`finally` 中放清理代码。

### Q4：上下文管理器可以嵌套使用吗？
**完全可以**，这是非常常见的用法：

```python
# 同时管理多个文件
with open("input.txt") as fin, open("output.txt", "w") as fout:
    data = fin.read()
    fout.write(data.upper())

# 或者分行写（更清晰）
with open("input.txt") as fin:
    with open("output.txt", "w") as fout:
        data = fin.read()
        fout.write(data.upper())
```

### Q5：什么时候该用类，什么时候该用 `@contextmanager`？
一条简单的判断标准：

- **需要维护内部状态**（如连接池、计数器）→ 用类
- **只是简单的"获取 + 清理"**（打开文件、获取锁）→ 用 `@contextmanager`
- **不确定** → 用 `@contextmanager`，它通常够用且更简洁

---

## 九、本节总结

| 概念 | 说明 |
|------|------|
| 上下文管理器 | 定义了资源获取和释放逻辑的对象 |
| `with` 语句 | 使用上下文管理器的语法糖 |
| `__enter__` | 进入上下文时调用，返回值通过 `as` 赋给变量 |
| `__exit__` | 退出上下文时调用，接收三个异常参数 |
| `@contextmanager` | 用生成器函数创建上下文管理器的装饰器 |
| `yield` 之前 | 相当于 `__enter__` 的逻辑 |
| `yield` 之后 | 相当于 `__exit__` 的逻辑（需用 `try/finally`） |
| `suppress` | 忽略指定异常的上下文管理器 |
| `closing` | 自动调用对象 `close()` 方法的上下文管理器 |

**核心思想**：上下文管理器让你把"资源获取"和"资源释放"绑定在一起，永远不会忘记释放资源。

---

## 十、下节预告

**Day 28：类型注解（Type Hints）**——Python 3.5+ 引入了类型注解，让你的代码更易读、更易维护。我们将学习如何为变量、函数参数、返回值添加类型注解，以及 `typing` 模块中常用类型的使用方法。

---

## 参考资源

- [廖雪峰 Python 教程 - 错误处理](https://www.liaoxuefeng.com/wiki/1016959663602400/1017598873224624)
- [菜鸟教程 - Python with 语句](https://www.runoob.com/python3/python-with.html)
- [Python 官方文档 - contextlib 模块](https://docs.python.org/zh-cn/3/library/contextlib.html)
- [Python 官方文档 - with 语句](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#the-with-statement)
