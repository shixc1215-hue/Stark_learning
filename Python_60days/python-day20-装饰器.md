# Python Day 20：装饰器（Decorator）

> **学习目标**：理解装饰器的概念与原理，掌握如何定义和使用装饰器，学会用装饰器优雅地为函数添加额外功能。

---

## 一、什么是装饰器？

装饰器（Decorator）是 Python 中一个非常重要的高级特性。简单来说，**装饰器就是一个函数，它用来"包装"另一个函数，在不修改原函数代码的前提下，给函数增加额外的功能**。

打个比方：
- 你买了一部手机（原函数）
- 装饰器就像是给手机套上了一个手机壳（增加保护功能），但手机本身的功能完全不变

装饰器在日常开发中非常常用，比如：
- 日志记录：自动记录函数的调用信息
- 性能计时：自动计算函数的运行时间
- 权限检查：在执行函数前检查用户权限
- 缓存：缓存函数的返回结果，避免重复计算

---

## 二、从闭包说起（装饰器的前置知识）

要理解装饰器，首先需要理解**闭包（Closure）**。

### 2.1 什么是闭包？

闭包就是一个**函数里面定义了另一个函数**，并且**内层函数引用了外层函数的变量**，外层函数把内层函数作为返回值返回。

```python
def outer(message):
    # message 是外层函数的变量
    def inner():
        # 内层函数引用了外层函数的 message
        print(f"消息是：{message}")
    return inner  # 返回内层函数（不是调用它）

# 使用
func = outer("你好，Python！")  # func 现在就是 inner 函数
func()  # 输出：消息是：你好，Python！
```

**关键点**：`outer` 函数执行完毕后，它的局部变量 `message` 本应被销毁。但因为 `inner` 函数引用了它，Python 会把 `message` 保留下来。这种"变量跟着函数走"的现象就是闭包。

### 2.2 带参数的闭包

```python
def make_multiplier(n):
    def multiplier(x):
        return x * n  # 引用外层变量 n
    return multiplier

double = make_multiplier(2)   # 乘以2的函数
triple = make_multiplier(3)   # 乘以3的函数

print(double(5))   # 输出：10
print(triple(5))   # 输出：15
```

> 💡 **记忆法**：闭包 = 内部函数 + 引用外部变量 + 返回内部函数

---

## 三、装饰器的基本用法

### 3.1 最简单的装饰器

装饰器本质上就是一个**接收函数作为参数，返回一个新函数**的闭包。

```python
def my_decorator(func):
    def wrapper():
        print("=== 函数执行前 ===")
        func()             # 调用原来的函数
        print("=== 函数执行后 ===")
    return wrapper

# 定义一个普通函数
def say_hello():
    print("Hello, World!")

# 用装饰器"包装"这个函数
say_hello = my_decorator(say_hello)

# 调用
say_hello()
# 输出：
# === 函数执行前 ===
# Hello, World!
# === 函数执行后 ===
```

### 3.2 @ 语法糖

每次手动写 `func = my_decorator(func)` 太麻烦了，Python 提供了 `@` 语法糖，效果完全一样：

```python
def my_decorator(func):
    def wrapper():
        print("=== 函数执行前 ===")
        func()
        print("=== 函数执行后 ===")
    return wrapper

@my_decorator  # 等价于 say_hello = my_decorator(say_hello)
def say_hello():
    print("Hello, World!")

say_hello()
```

`@my_decorator` 放在函数定义的上方，Python 会自动把下面的函数传给 `my_decorator`，然后用返回值替换原函数。

---

## 四、装饰带参数的函数

上面的装饰器只能装饰没有参数的函数。如果原函数有参数怎么办？

### 4.1 使用 *args 和 **kwargs

```python
def log_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"正在调用函数：{func.__name__}")
        print(f"参数：args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)  # 把参数原封不动传给原函数
        print(f"函数 {func.__name__} 执行完毕")
        return result  # 返回原函数的结果
    return wrapper

@log_decorator
def add(a, b):
    return a + b

@log_decorator
def greet(name, greeting="你好"):
    return f"{greeting}，{name}！"

# 调用
print(add(3, 5))
# 输出：
# 正在调用函数：add
# 参数：args=(3, 5), kwargs={}
# 函数 add 执行完毕
# 8

print(greet("小明", greeting="早上好"))
# 输出：
# 正在调用函数：greet
# 参数：args=('小明',), kwargs={'greeting': '早上好'}
# 函数 greet 执行完毕
# 早上好，小明！
```

**解析**：
- `*args`：接收任意数量的位置参数（以元组形式）
- `**kwargs`：接收任意数量的关键字参数（以字典形式）
- 这样不管原函数有多少参数，装饰器都能正确传递

---

## 五、实用的装饰器示例

### 5.1 计时装饰器（测量函数运行时间）

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()            # 记录开始时间
        result = func(*args, **kwargs)
        end = time.time()              # 记录结束时间
        print(f"函数 {func.__name__} 耗时：{end - start:.4f} 秒")
        return result
    return wrapper

@timer
def sum_numbers(n):
    """计算 1 到 n 的和"""
    total = 0
    for i in range(1, n + 1):
        total += i
    return total

@timer
def sum_numbers_fast(n):
    """用公式计算 1 到 n 的和"""
    return n * (n + 1) // 2

result1 = sum_numbers(1000000)
print(f"循环求和结果：{result1}")

result2 = sum_numbers_fast(1000000)
print(f"公式求和结果：{result2}")

# 输出示例：
# 循环求和结果：500000500000
# 函数 sum_numbers 耗时：0.0452 秒
# 公式求和结果：500000500000
# 函数 sum_numbers_fast 耗时：0.0000 秒
```

### 5.2 日志装饰器（记录函数调用）

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"[LOG] {func.__name__} 被调用了")
        result = func(*args, **kwargs)
        print(f"[LOG] {func.__name__} 返回了 {result}")
        return result
    return wrapper

@logger
def multiply(a, b):
    return a * b

multiply(4, 5)
# 输出：
# [LOG] multiply 被调用了
# [LOG] multiply 返回了 20
```

### 5.3 权限检查装饰器

```python
def login_required(func):
    def wrapper(username, is_logged_in=False, **kwargs):
        if not is_logged_in:
            print("❌ 请先登录！无法访问此功能。")
            return None
        return func(username, is_logged_in=True, **kwargs)
    return wrapper

@login_required
def view_dashboard(username, is_logged_in=False):
    print(f"欢迎 {username}！这是你的个人主页。")

# 已登录
view_dashboard("张三", is_logged_in=True)
# 输出：欢迎 张三！这是你的个人主页。

# 未登录
view_dashboard("李四", is_logged_in=False)
# 输出：❌ 请先登录！无法访问此功能。
```

---

## 六、带参数的装饰器

如果装饰器本身也需要接收参数呢？比如想在日志装饰器中指定日志级别。

这需要**再嵌套一层函数**：

```python
def repeat(n):
    """让函数重复执行 n 次的装饰器"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for i in range(n):
                print(f"第 {i + 1} 次执行...")
                result = func(*args, **kwargs)
                results.append(result)
            return results  # 返回所有结果组成的列表
        return wrapper
    return decorator  # 注意：这里返回的是 decorator，不是 wrapper

@repeat(3)  # 相当于 func = repeat(3)(func)
def roll_dice():
    import random
    return random.randint(1, 6)

results = roll_dice()
print(f"三次掷骰子结果：{results}")
# 输出示例：
# 第 1 次执行...
# 第 2 次执行...
# 第 3 次执行...
# 三次掷骰子结果：[3, 6, 1]
```

**三层结构拆解**：
```
repeat(3)           → 返回 decorator 函数
decorator(func)    → 返回 wrapper 函数
wrapper()          → 实际执行，重复调用 func 三次
```

---

## 七、保留原函数信息：functools.wraps

使用装饰器后，原函数的一些信息（如 `__name__`、`__doc__`）会被 wrapper 函数覆盖。用 `functools.wraps` 可以解决：

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # 自动复制原函数的名称、文档字符串等信息
    def wrapper(*args, **kwargs):
        print("执行前...")
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def say_hello():
    """向世界问好"""
    print("Hello!")

print(say_hello.__name__)   # 输出：say_hello（而不是 wrapper）
print(say_hello.__doc__)    # 输出：向世界问好
```

> 📌 **最佳实践**：写装饰器时，**始终加上 `@wraps(func)`**，这是规范做法。

---

## 八、多个装饰器的叠加

一个函数可以同时使用多个装饰器，执行顺序是**从下到上**（最靠近函数的先执行）：

```python
def bold(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold       # 外层：最后执行，包在最外面
@italic     # 内层：先执行
def say(text):
    return text

print(say("Hello"))
# 输出：<b><i>Hello</i></b>
```

执行过程：`say("Hello")` → `italic(say)` → `bold(italic(say))`

所以最终结果是最内层先包装，最外层最后包装。

---

## 九、类装饰器

装饰器不仅可以是函数，也可以是类。类装饰器需要实现 `__call__` 方法：

```python
class CountCalls:
    """统计函数被调用次数的装饰器"""
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"函数 {self.func.__name__} 已被调用 {self.count} 次")
        return self.func(*args, **kwargs)

@CountCalls
def greet(name):
    print(f"你好，{name}！")

greet("小明")  # 函数 greet 已被调用 1 次
greet("小红")  # 函数 greet 已被调用 2 次
greet("小刚")  # 函数 greet 已被调用 3 次
```

---

## 十、常见问题（FAQ）

### Q1：装饰器和直接修改函数代码有什么区别？

**装饰器的优势**：
- **不修改原函数代码**：遵循"开放-封闭原则"
- **可复用**：同一个装饰器可以应用到多个函数
- **可叠加**：可以给一个函数添加多个装饰器
- **可移除**：去掉 `@` 行即可恢复原函数行为

### Q2：装饰器会降低性能吗？

装饰器会引入一层函数调用，有极其微小的性能开销。但对于大多数应用来说，这个开销完全可以忽略不计。而且装饰器带来的代码简洁性和可维护性远大于这点开销。

### Q3：什么时候该用装饰器？

当你的需求是"**给多个函数添加相同的额外功能**"时，就该考虑用装饰器了。比如日志记录、计时、权限校验等"横切关注点"。

---

## 十一、练习题

### 练习 1：基本装饰器（★☆☆）

编写一个装饰器 `star_decorator`，在函数执行前后各打印一行 `*****`：

```python
@star_decorator
def display():
    print("  Hello Python  ")

# 期望输出：
# *****
#   Hello Python
# *****
```

<details>
<summary>参考答案</summary>

```python
def star_decorator(func):
    def wrapper(*args, **kwargs):
        print("*****")
        result = func(*args, **kwargs)
        print("*****")
        return result
    return wrapper

@star_decorator
def display():
    print("  Hello Python  ")

display()
```

</details>

### 练习 2：缓存装饰器（★★☆）

编写一个装饰器 `memoize`，缓存函数的返回结果。如果相同的参数再次调用，直接返回缓存结果，不重新计算：

```python
@memoize
def slow_add(a, b):
    print("  正在计算...")
    return a + b

print(slow_add(1, 2))   # 输出：正在计算... → 3
print(slow_add(1, 2))   # 直接返回 3，不打印"正在计算..."
print(slow_add(3, 4))   # 输出：正在计算... → 7
```

<details>
<summary>参考答案</summary>

```python
from functools import wraps

def memoize(func):
    cache = {}  # 用字典存储缓存
    @wraps(func)
    def wrapper(*args):
        if args in cache:
            return cache[args]
        result = func(*args)
        cache[args] = result
        return result
    return wrapper
```

</details>

### 练习 3：带参数的装饰器（★★★）

编写一个装饰器 `retry(max_retries)`，当函数抛出异常时自动重试，最多重试 `max_retries` 次：

```python
import random

@retry(max_retries=3)
def unstable_function():
    if random.random() < 0.7:
        raise ConnectionError("网络连接失败")
    return "操作成功！"

result = unstable_function()
print(result)  # 重试多次后输出：操作成功！
```

<details>
<summary>参考答案</summary>

```python
from functools import wraps

def retry(max_retries):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"  第 {attempt} 次尝试失败：{e}")
                    if attempt == max_retries:
                        print("  已达到最大重试次数，放弃。")
                        raise
            return None
        return wrapper
    return decorator
```

</details>

---

## 十二、今日小结

| 概念 | 说明 |
|------|------|
| **闭包** | 内部函数引用外部变量，外部函数返回内部函数 |
| **装饰器** | 接收函数作为参数，返回增强版函数的闭包 |
| **@ 语法糖** | `@decorator` 等价于 `func = decorator(func)` |
| **\*args / \*\*kwargs** | 让装饰器支持任意参数的原函数 |
| **带参数的装饰器** | 三层嵌套：参数层 → 装饰器层 → wrapper 层 |
| **@wraps(func)** | 保留原函数的元信息，推荐始终使用 |
| **类装饰器** | 实现 `__call__` 方法的类，可用作装饰器 |

**核心记忆**：装饰器 = 闭包 + 函数作为参数 + `@` 语法糖

---

## 十三、推荐学习资源

- 📖 [廖雪峰 Python 教程 — 装饰器](https://www.liaoxuefeng.com/wiki/1016959663602400/1017451662295584)
- 📖 [菜鸟教程 — Python 装饰器](https://www.runoob.com/w3cnote/python-func-decorators.html)
- 📖 [Python 官方文档 — functools](https://docs.python.org/zh-cn/3/library/functools.html)
- 🎥 [B站：Python 装饰器详解（搜索推荐）](https://search.bilibili.com/bili?keyword=python装饰器)

---

> **Day 20 完成！** 装饰器是 Python 中最优雅的特性之一，掌握它能让你的代码更加简洁、可复用。明天我们将进入新的阶段，开始学习**正则表达式**。
