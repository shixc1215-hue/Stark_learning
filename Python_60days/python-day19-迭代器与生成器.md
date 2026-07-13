# Day 19 - 迭代器与生成器

> **学习目标**：理解迭代器（Iterator）和生成器（Generator）的概念，掌握 `yield` 关键字，学会用生成器处理大数据流。
>
> **前置知识**：Day 07 列表、Day 10 函数基础、Day 18 模块

---

## 一、什么是"可迭代"？

### 1.1 生活中的类比

想象你在看一本书：
- 你可以**一页一页翻**，每次读一页
- 你不需要把整本书**全部展开**才能开始阅读
- 翻到最后一页后，就**结束了**

这个"一页一页读"的过程，在 Python 中就叫**迭代（Iteration）**。

### 1.2 代码中的迭代

你已经用过很多次迭代了——`for` 循环就是最常见的迭代方式：

```python
# 遍历列表 —— 这就是迭代
for item in [1, 2, 3, 4, 5]:
    print(item)

# 遍历字符串 —— 也是迭代
for char in "Hello":
    print(char)

# 遍历字典 —— 还是迭代
for key in {"name": "小明", "age": 18}:
    print(key)
```

> **核心概念**：能被 `for` 循环遍历的对象，就叫**可迭代对象（Iterable）**。
>
> 列表、字符串、字典、元组、集合……都是可迭代对象。

### 1.3 判断一个对象是否可迭代

```python
from collections.abc import Iterable

# 判断是否可迭代
print(isinstance([1, 2, 3], Iterable))    # True  —— 列表可迭代
print(isinstance("hello", Iterable))       # True  —— 字符串可迭代
print(isinstance(123, Iterable))           # False —— 数字不可迭代
print(isinstance({"a": 1}, Iterable))      # True  —— 字典可迭代

# 用 try...except 也能判断
try:
    iter(123)  # 尝试获取迭代器
    print("可迭代")
except TypeError:
    print("不可迭代")  # ← 会输出这个
```

---

## 二、迭代器（Iterator）—— "逐个产出数据的机器"

### 2.1 迭代器是什么？

**可迭代对象**是一堆数据的集合（列表、字符串等），而**迭代器**是一个能"逐个取出数据"的机器。

二者的关系：
```
可迭代对象（Iterable）    →    迭代器（Iterator）
    "仓库"                      "取货员"

列表 [1, 2, 3]          →    iter(列表) → 每次取一个
字符串 "abc"            →    iter(字符串) → 每次取一个字符
```

### 2.2 获取和使用迭代器

```python
# 用 iter() 从可迭代对象获取迭代器
my_list = [10, 20, 30]
my_iterator = iter(my_list)

# 用 next() 逐个取出元素
print(next(my_iterator))  # 10  —— 取出第一个
print(next(my_iterator))  # 20  —— 取出第二个
print(next(my_iterator))  # 30  —— 取出第三个

# 再取就报错了！因为已经取完了
print(next(my_iterator))  # StopIteration 异常
```

> **关键点**：迭代器是**"一次性"**的。取完就没了，不能回头。

```python
# 迭代器取完后就空了
my_iter = iter([1, 2, 3])
next(my_iter)  # 1
next(my_iter)  # 2
next(my_iter)  # 3
# next(my_iter)  # ❌ StopIteration

# 你不能"重置"迭代器，要重来只能再创建一个
my_iter2 = iter([1, 2, 3])  # 新的迭代器
next(my_iter2)  # 1
```

### 2.3 for 循环的本质

其实，`for` 循环就是自动帮你做了"获取迭代器 + 反复 next + 捕获 StopIteration"这件事：

```python
# for 循环的写法
for item in [1, 2, 3]:
    print(item)

# 等价于下面这种手动写法（但 for 更简洁）
my_iter = iter([1, 2, 3])
while True:
    try:
        item = next(my_iter)
        print(item)
    except StopIteration:
        break  # 取完了，退出循环
```

### 2.4 自定义迭代器类

让一个类变成迭代器，需要实现两个"魔法方法"：
- `__iter__()` —— 返回迭代器对象（通常返回 `self`）
- `__next__()` —— 返回下一个元素，没有元素时抛出 `StopIteration`

```python
class Countdown:
    """倒计时迭代器：从 start 数到 1"""

    def __init__(self, start):
        self.current = start  # 当前值

    def __iter__(self):
        return self  # 返回自身作为迭代器

    def __next__(self):
        if self.current <= 0:
            raise StopIteration  # 数完了
        value = self.current
        self.current -= 1
        return value


# 使用自定义迭代器
countdown = Countdown(5)
for num in countdown:
    print(num)
# 输出: 5 4 3 2 1
```

再来一个更实用的例子——斐波那契数列迭代器：

```python
class Fibonacci:
    """斐波那契数列迭代器"""

    def __init__(self, max_count):
        self.max_count = max_count  # 最多生成多少个
        self.count = 0              # 已生成个数
        self.a, self.b = 0, 1       # 前两个值

    def __iter__(self):
        return self

    def __next__(self):
        if self.count >= self.max_count:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b  # 计算下一个
        self.count += 1
        return value


# 使用
fib = Fibonacci(10)
print(list(fib))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## 三、生成器（Generator）—— "更简单的迭代器"

### 3.1 为什么需要生成器？

自定义迭代器类（上面的 `Fibonacci`）需要写 `__iter__` 和 `__next__`，代码比较繁琐。

Python 提供了一种更简单的方式——**生成器函数**，只需要一个 `yield` 关键字就能创建迭代器。

### 3.2 第一个生成器

```python
def simple_generator():
    """最简单的生成器"""
    yield 1
    yield 2
    yield 3


# 使用生成器
gen = simple_generator()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
# next(gen)       # StopIteration

# 也可以用 for 循环
for value in simple_generator():
    print(value)
# 输出: 1 2 3
```

### 3.3 yield 和 return 的区别

```python
def with_return():
    return 1     # 函数结束，返回值
    return 2     # ❌ 永远不会执行


def with_yield():
    yield 1      # 暂停，返回 1，下次从这里继续
    yield 2      # 暂停，返回 2，下次从这里继续
    yield 3      # 暂停，返回 3


# return：函数结束，只能返回一次
result = with_return()
print(result)  # 1

# yield：函数暂停，可以"返回"多次
gen = with_yield()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
```

> **核心区别**：
> - `return` → 函数**结束**，值被返回，后续代码不执行
> - `yield` → 函数**暂停**，值被返回，下次调用时从暂停处继续执行

### 3.4 用生成器实现斐波那契数列

对比之前的类写法，生成器简洁得多：

```python
def fibonacci(max_count):
    """生成器版本的斐波那契数列"""
    a, b = 0, 1
    count = 0
    while count < max_count:
        yield a        # ← 暂停并返回当前值
        a, b = b, a + b  # ← 下次从这里继续
        count += 1


# 使用
for num in fibonacci(10):
    print(num, end=" ")
# 输出: 0 1 1 2 3 5 8 13 21 34

# 转成列表
fib_list = list(fibonacci(15))
print(fib_list)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
```

### 3.5 生成器表达式 —— 列表推导式的"懒加载"版本

之前学过列表推导式：

```python
# 列表推导式 —— 立刻生成所有数据
squares = [x ** 2 for x in range(1000000)]
print(len(squares))  # 1000000 —— 占用大量内存！
```

把方括号 `[]` 换成圆括号 `()`，就变成了**生成器表达式**：

```python
# 生成器表达式 —— 按需生成，几乎不占内存
squares_gen = (x ** 2 for x in range(1000000))

# 此时还没有计算任何值！
print(squares_gen)  # <generator object <genexpr> at 0x...>

# 用 next() 取值
print(next(squares_gen))  # 0  （计算 0**2）
print(next(squares_gen))  # 1  （计算 1**2）
print(next(squares_gen))  # 4  （计算 2**2）

# 或者用 for 循环
for val in (x ** 2 for x in range(5)):
    print(val, end=" ")
# 输出: 0 1 4 9 16
```

**对比一下**：

| 特性 | 列表推导式 `[...]` | 生成器表达式 `(...)` |
|------|-------------------|---------------------|
| 计算时机 | 立刻全部计算 | 按需逐个计算 |
| 内存占用 | 全部存入内存 | 几乎不占内存 |
| 可重复使用 | 可以（数据在内存中） | 不可以（一次性） |
| 支持索引 | 可以 `squares[0]` | 不可以 |
| 适用场景 | 数据量小，需要多次访问 | 数据量大，只需遍历一次 |

---

## 四、生成器的实战应用

### 4.1 逐行读取大文件

这是生成器最常见的应用场景之一：

```python
def read_large_file(filepath):
    """逐行读取大文件，不一次性加载到内存"""
    with open(filepath, "r", encoding="utf-8") as f:
        for line in f:
            # strip() 去掉末尾的换行符
            yield line.strip()


# 使用：即使文件有 10GB，内存也不会爆
for line in read_large_file("big_file.txt"):
    if "ERROR" in line:
        print(f"发现错误: {line}")
```

### 4.2 无限序列

生成器可以创建"无限序列"（当然，你需要自己在合适的时候停止）：

```python
def natural_numbers():
    """自然数序列：0, 1, 2, 3, ...（无限）"""
    n = 0
    while True:
        yield n
        n += 1


# 取前 10 个自然数
count = 0
for num in natural_numbers():
    if count >= 10:
        break
    print(num, end=" ")
    count += 1
# 输出: 0 1 2 3 4 5 6 7 8 9
```

### 4.3 数据处理管道

生成器可以串联起来，形成"数据处理管道"：

```python
def read_data(filepath):
    """阶段1：读取数据"""
    with open(filepath, "r", encoding="utf-8") as f:
        for line in f:
            yield line.strip()


def filter_errors(data_stream):
    """阶段2：只保留包含 ERROR 的行"""
    for line in data_stream:
        if "ERROR" in line:
            yield line


def extract_message(error_stream):
    """阶段3：提取错误消息（去掉 ERROR 前缀）"""
    for line in error_stream:
        message = line.replace("ERROR:", "").strip()
        yield message


def count_lines(data_stream):
    """阶段4：统计行数"""
    count = 0
    for _ in data_stream:
        count += 1
    return count


# 串联使用 —— 数据像流水线一样逐条通过
# pipeline = extract_message(filter_errors(read_data("log.txt")))
# error_count = count_lines(pipeline)
# print(f"共发现 {error_count} 个错误")

# 简单演示（不依赖实际文件）
def demo_data():
    for line in ["INFO: 系统启动", "ERROR: 磁盘满", "INFO: 用户登录", "ERROR: 网络超时"]:
        yield line

pipeline = extract_message(filter_errors(demo_data()))
for msg in pipeline:
    print(msg)
# 输出:
# 磁盘满
# 网络超时
```

### 4.4 分批处理大数据

```python
def batch_generator(data_list, batch_size):
    """将大列表分成小批次"""
    for i in range(0, len(data_list), batch_size):
        yield data_list[i : i + batch_size]


# 模拟 1000 条数据
data = list(range(1000))

# 每批处理 100 条
for i, batch in enumerate(batch_generator(data, 100)):
    print(f"第 {i + 1} 批: {len(batch)} 条数据，范围 {batch[0]}-{batch[-1]}")
# 第 1 批: 100 条数据，范围 0-99
# 第 2 批: 100 条数据，范围 100-199
# ...
# 第 10 批: 100 条数据，范围 900-999
```

---

## 五、yield 的进阶用法

### 5.1 yield 接收外部传值（send 方法）

`yield` 不只能"往外传值"，还能"接收外部的值"：

```python
def accumulator():
    """累加器：每次接收一个数，返回当前总和"""
    total = 0
    while True:
        # yield 返回 total，同时接收 send 传来的值
        value = yield total
        if value is not None:
            total += value


# 使用 send() 向生成器传值
gen = accumulator()

# 第一步：必须先 next() 或 send(None) "启动"生成器
print(next(gen))     # 0 （初始总和为 0）

# 以后可以用 send() 传值
print(gen.send(10))  # 10 （0 + 10 = 10）
print(gen.send(20))  # 30 （10 + 20 = 30）
print(gen.send(5))   # 35 （30 + 5 = 35）
print(gen.send(-15)) # 20 （35 + (-15) = 20）
```

> **注意**：第一次调用时，生成器还没执行到 `yield`，所以不能直接 `send(10)`，必须先 `next(gen)` 或 `gen.send(None)` 来"启动"它。

### 5.2 yield from —— 委托给另一个生成器

当你需要在一个生成器中"嵌入"另一个生成器时：

```python
# 不用 yield from —— 需要循环
def chain_generators_old():
    for value in fibonacci(3):
        yield value
    for value in fibonacci(4):
        yield value

# 用 yield from —— 简洁！
def chain_generators():
    yield from fibonacci(3)  # 委托给 fibonacci
    yield from fibonacci(4)  # 再委托一次


for val in chain_generators():
    print(val, end=" ")
# 输出: 0 1 1 0 1 1 2
```

### 5.3 实战：树形结构的遍历

```python
class TreeNode:
    """树节点"""
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []

    def __repr__(self):
        return f"Node({self.value})"


def traverse_tree(node):
    """生成器实现深度优先遍历"""
    yield node.value                    # 先访问当前节点
    for child in node.children:         # 再遍历每个子节点
        yield from traverse_tree(child) # 委托给递归调用


# 构建一棵树
#         A
#       / | \
#      B  C  D
#     / \    |
#    E   F   G
root = TreeNode("A", [
    TreeNode("B", [TreeNode("E"), TreeNode("F")]),
    TreeNode("C"),
    TreeNode("D", [TreeNode("G")])
])

# 遍历树
for value in traverse_tree(root):
    print(value, end=" ")
# 输出: A B E F C D G
```

---

## 六、 itertools —— Python 内置的迭代器工具箱

Python 标准库的 `itertools` 模块提供了大量强大的迭代器工具。

### 6.1 常用函数

```python
from itertools import count, cycle, repeat, islice, chain, product

# 1. count(start, step) —— 无限计数器
counter = count(10, 2)  # 从10开始，每次+2
print(list(islice(counter, 5)))  # [10, 12, 14, 16, 18]
# islice 截取前5个（因为 count 是无限的）

# 2. cycle(iterable) —— 无限循环
colors = cycle(["红", "绿", "蓝"])
print(list(islice(colors, 7)))  # ['红', '绿', '蓝', '红', '绿', '蓝', '红']

# 3. repeat(value, times) —— 重复值
print(list(repeat("Python", 3)))  # ['Python', 'Python', 'Python']

# 4. chain(*iterables) —— 连接多个可迭代对象
print(list(chain([1, 2], [3, 4], [5])))  # [1, 2, 3, 4, 5]

# 5. product(*iterables) —— 笛卡尔积（组合）
print(list(product([1, 2], ["a", "b"])))
# [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
```

### 6.2 实用例子

```python
from itertools import islice, takewhile, dropwhile, groupby

# takewhile —— 满足条件时才取值
numbers = [1, 2, 3, 4, 5, 1, 2]
result = list(takewhile(lambda x: x < 4, numbers))
print(result)  # [1, 2, 3]  —— 遇到4就停了

# dropwhile —— 跳过满足条件的值，直到第一个不满足的
result = list(dropwhile(lambda x: x < 4, numbers))
print(result)  # [4, 5, 1, 2]

# groupby —— 连续相同值的分组
data = sorted(["苹果", "香蕉", "草莓", "苹果", "草莓", "草莓"])
for key, group in groupby(data):
    print(f"{key}: {list(group)}")
# 苹果: ['苹果', '苹果']
# 草莓: ['草莓', '草莓', '草莓']
# 香蕉: ['香蕉']
```

> **注意**：`groupby` 只能对**连续**相同的值分组。所以用之前通常要 `sorted()`。

---

## 七、什么时候用生成器？

### 7.1 推荐使用生成器的场景

| 场景 | 例子 |
|------|------|
| 处理大文件 | 逐行读取日志、CSV |
| 大数据流 | 从数据库逐条查询 |
| 无限序列 | 自然数、素数序列 |
| 管道处理 | 读数据 → 过滤 → 转换 → 输出 |
| 需要惰性求值 | 只在需要时才计算 |

### 7.2 不推荐使用生成器的场景

| 场景 | 原因 |
|------|------|
| 数据量很小 | 生成器反而增加复杂度，直接用列表 |
| 需要多次遍历 | 生成器是一次性的 |
| 需要按索引访问 | 生成器不支持 `gen[0]` |
| 需要知道长度 | 生成器没有 `len()` |

### 7.3 简单判断法则

```
数据量超过 10,000 条？  →  考虑用生成器
只需要遍历一次？        →  用生成器
需要随机访问？          →  用列表
数据量很小？            →  用列表（简单直接）
```

---

## 八、练习题

### 练习1：素数生成器

```python
# 要求：写一个生成器 primes()，无限生成素数
# 提示：判断素数的方法 —— n 能被 2 到 sqrt(n) 整除就不是素数

# 示例输出：
# gen = primes()
# print([next(gen) for _ in range(10)])
# [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

### 练习2：扁平化嵌套列表

```python
# 要求：写一个生成器 flatten()，将任意深度的嵌套列表"展平"
# 提示：使用 yield from + 递归

# 示例：
# nested = [1, [2, 3], [4, [5, 6], 7], 8]
# print(list(flatten(nested)))
# [1, 2, 3, 4, 5, 6, 7, 8]

# 更深的情况：
# deep = [1, [2, [3, [4, 5]]]]
# print(list(flatten(deep)))
# [1, 2, 3, 4, 5]
```

### 练习3：合并多个有序列表

```python
# 要求：写一个生成器 merge_sorted(*lists)，将多个已排序的列表合并为一个有序序列
# 提示：每次从所有列表的"当前最小值"中取最小的

# 示例：
# a = [1, 4, 7]
# b = [2, 3, 6, 9]
# c = [0, 5, 8]
# print(list(merge_sorted(a, b, c)))
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

## 九、常见问题

**Q1：生成器和列表到底有什么区别？什么时候该用哪个？**

最核心的区别：**列表把所有数据都存在内存中，生成器每次只产生一个值。**

```python
# 列表：一次性全部生成，占用内存
nums_list = [x for x in range(1000000)]  # 占用约 8MB 内存

# 生成器：按需生成，几乎不占内存
nums_gen = (x for x in range(1000000))   # 占用约 100 字节
```

简单判断：数据量大 + 只需遍历一次 → 生成器。其他情况 → 列表。

**Q2：为什么生成器不能重复遍历？**

因为生成器的内部状态（执行位置、变量值）在遍历后就"消耗"完了。想再来一次？创建一个新的就好：

```python
def my_gen():
    yield 1
    yield 2

gen1 = my_gen()
list(gen1)  # [1, 2]
list(gen1)  # [] ← 已消耗完，空了！

gen2 = my_gen()  # 创建新的
list(gen2)  # [1, 2]
```

**Q3：`yield` 和 `yield from` 什么时候用哪个？**

- 只需要产出一个值 → 用 `yield`
- 需要把另一个生成器的所有值"透传"出来 → 用 `yield from`

```python
def gen_a():
    yield 1
    yield 2

# 用 yield —— 需要循环
def gen_b():
    for x in gen_a():
        yield x

# 用 yield from —— 简洁
def gen_c():
    yield from gen_a()
```

**Q4：生成器能用在列表推导式里吗？**

可以！生成器也是一种可迭代对象：

```python
gen = (x ** 2 for x in range(5))

# 在列表推导式中使用
result = [x * 10 for x in gen]
print(result)  # [0, 10, 40, 90, 160]
```

---

## 十、今日总结

```
┌─────────────────────────────────────────────────────┐
│               Day 19 知识地图                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│   可迭代对象 (Iterable)                             │
│     ├── 列表、字符串、字典、元组、集合               │
│     └── 能被 for 循环遍历                           │
│                                                     │
│   迭代器 (Iterator)                                 │
│     ├── iter() 获取，next() 取值                    │
│     ├── 一次性，取完就没了                           │
│     └── 自定义：__iter__ + __next__                 │
│                                                     │
│   生成器 (Generator)                                │
│     ├── 用 yield 创建的迭代器                       │
│     ├── yield → 暂停并返回值                        │
│     ├── send() → 向生成器传值                       │
│     ├── yield from → 委托给另一个生成器             │
│     └── 生成器表达式 (...) → 列表推导式的惰性版本   │
│                                                     │
│   itertools 模块                                    │
│     ├── count / cycle / repeat                      │
│     ├── chain / islice / product                    │
│     └── takewhile / dropwhile / groupby             │
│                                                     │
│   使用原则：大数据 + 单次遍历 → 生成器               │
│             小数据 + 多次访问 → 列表                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**今天掌握的技能让你能：**
- 理解 `for` 循环背后的迭代器机制
- 用 `yield` 创建轻量级的数据生成器
- 处理大文件和大数据流而不撑爆内存
- 使用 `itertools` 处理复杂的迭代场景

---

## 十一、免费学习资源

- [廖雪峰 Python 教程 - 生成器](https://www.liaoxuefeng.com/wiki/1016959663602400/1017318208210192) — 生成器概念的详细讲解
- [菜鸟教程 Python3 迭代器与生成器](https://www.runoob.com/python3/python3-iterator-generator.html) — 基础概念和实例
- [Python 官方文档 - itertools](https://docs.python.org/zh-cn/3/library/itertools.html) — 官方工具函数参考

---

> **Day 20 预告**：装饰器 —— 学习 Python 中最优雅的"函数增强"技巧，理解函数也能作为参数传递，掌握 `@` 语法的魔法。

> 💡 **学习小贴士**：生成器是 Python 的精华特性之一。如果你在之后学 pandas、处理日志、写爬虫时看到"生成器"或"惰性求值"这个词，今天学的知识就是它们的基础。试着用生成器重写之前的一些练习，比如把 Day 12 的文件读取改成生成器版本。