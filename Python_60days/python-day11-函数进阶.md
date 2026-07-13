# 🐍 Python 60天学习计划 · Day 11

> **阶段**：第一阶段 · 基础语法入门（Day 1-14）
> **今日主题**：函数进阶 — lambda / 高阶函数 / 递归入门
> **预计时间**：约 60 分钟
> **进度**：Day 11 / 60 ▓▓▓▓░░░░░░░░░░░░░░░░ 18%

---

## 一、今日目标

- ✅ 理解 lambda 匿名函数的语法和使用场景
- ✅ 掌握高阶函数 map / filter / sorted 的用法
- ✅ 了解函数作为参数传递（函数是一等公民）
- ✅ 理解递归的概念，能写简单的递归函数
- ✅ 完成实战练习

---

## 二、lambda 匿名函数 — 一行搞定简单逻辑

### 2.1 什么是 lambda？

lambda 就是"一句话函数"——当你需要一个简单函数，又不想正式 `def` 一个时用它。

```python
# 普通 def 写法
def square(x):
    return x * x

# lambda 写法 — 等价但更简洁
square = lambda x: x * x

print(square(5))  # 25
```

**语法**：
```python
lambda 参数1, 参数2, ...: 表达式
```

> 💡 **核心要点**：
> - lambda 只能写**一个表达式**，不能写多行语句
> - 表达式的结果就是返回值，不需要 `return`
> - 适合简单逻辑，复杂逻辑还是用 `def`

### 2.2 lambda 的常见用法

```python
# 1. 多个参数
add = lambda a, b: a + b
print(add(3, 5))  # 8

# 2. 带默认参数
greet = lambda name, greeting="你好": f"{greeting}，{name}！"
print(greet("小明"))          # 你好，小明！
print(greet("小明", "早安"))   # 早安，小明！

# 3. 可变参数
total = lambda *nums: sum(nums)
print(total(1, 2, 3, 4))  # 10
```

### 2.3 lambda 最常见的场景 — 作为排序/筛选的钥匙

```python
# 按元组的第二个元素排序
students = [("小明", 85), ("小红", 92), ("小刚", 78)]
students.sort(key=lambda x: x[1])
print(students)  # [('小刚', 78), ('小明', 85), ('小红', 92)]

# 按字典的某个键排序
people = [
    {"name": "小明", "age": 25},
    {"name": "小红", "age": 20},
    {"name": "小刚", "age": 30},
]
people.sort(key=lambda p: p["age"])
print(people)  # 按年龄从小到大
```

> ⚠️ **避坑**：不要滥用 lambda！如果逻辑超过一行，乖乖用 `def`。lambda 的意义是简洁，不是炫技。

---

## 三、高阶函数 — 函数的函数

### 3.1 Python 中函数是"一等公民"

在 Python 中，函数和整数、字符串一样，可以：
- 赋值给变量
- 作为参数传递
- 作为返回值

```python
# 函数可以赋值给变量
def shout(text):
    return text.upper()

yell = shout               # 把函数赋给变量
print(yell("hello"))       # HELLO（调用的是同一个函数）

# 函数可以作为参数
def apply(func, value):
    """把 func 应用到 value 上"""
    return func(value)

print(apply(str.upper, "hello"))   # HELLO
print(apply(len, "hello"))        # 5
```

### 3.2 map() — 批量变换

`map(函数, 可迭代对象)`：把函数依次应用到每个元素上。

```python
# 需求：把列表中每个数字都平方
nums = [1, 2, 3, 4, 5]

# ❌ 笨方法：写循环
squares = []
for n in nums:
    squares.append(n ** 2)

# ✅ 优雅方法：用 map
squares = list(map(lambda x: x ** 2, nums))
print(squares)  # [1, 4, 9, 16, 25]

# 也可以搭配 def 定义的函数
def double(x):
    return x * 2

doubled = list(map(double, nums))
print(doubled)  # [2, 4, 6, 8, 10]
```

**map 处理多个列表**：
```python
# 两个列表对应位置相加
a = [1, 2, 3]
b = [10, 20, 30]
result = list(map(lambda x, y: x + y, a, b))
print(result)  # [11, 22, 33]
```

> 💡 `map()` 返回的是迭代器，需要用 `list()` 转换才能看到结果。

### 3.3 filter() — 批量筛选

`filter(函数, 可迭代对象)`：只保留函数返回 `True` 的元素。

```python
# 需求：从列表中筛选出偶数
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# ❌ 笨方法
evens = []
for n in nums:
    if n % 2 == 0:
        evens.append(n)

# ✅ 优雅方法
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)  # [2, 4, 6, 8, 10]

# 筛选非空字符串
words = ["hello", "", "world", "", "python"]
non_empty = list(filter(None, words))  # None 表示"保留真值"
print(non_empty)  # ['hello', 'world', 'python']
```

> 💡 `filter(None, ...)` 是快捷写法，等价于 `filter(lambda x: bool(x), ...)`。

### 3.4 sorted() — 排序利器

`sorted(可迭代对象, key=函数, reverse=布尔)`：返回排序后的新列表。

```python
# 基础排序
nums = [3, 1, 4, 1, 5, 9, 2, 6]
print(sorted(nums))              # [1, 1, 2, 3, 4, 5, 6, 9]
print(sorted(nums, reverse=True)) # [9, 6, 5, 4, 3, 2, 1, 1]

# 用 key 排序 — 这才是 sorted 强大的地方
words = ["banana", "apple", "cherry", "date"]
print(sorted(words, key=len))           # 按长度排序
# ['date', 'apple', 'banana', 'cherry']

print(sorted(words, key=lambda w: w[-1]))  # 按最后一个字母排序
# ['banana', 'apple', 'date', 'cherry']

# 排序字典列表
students = [
    {"name": "小明", "score": 85},
    {"name": "小红", "score": 92},
    {"name": "小刚", "score": 78},
]
# 按分数从高到低
ranked = sorted(students, key=lambda s: s["score"], reverse=True)
for s in ranked:
    print(f"{s['name']}: {s['score']}分")
# 小红: 92分
# 小明: 85分
# 小刚: 78分
```

### 3.5 map / filter / sorted 速查对比

| 函数 | 作用 | 输入 → 输出 | 原数据是否改变 |
|------|------|-------------|--------------|
| `map(f, seq)` | 变换每个元素 | N个 → N个 | ❌ 不改变 |
| `filter(f, seq)` | 筛选元素 | N个 → ≤N个 | ❌ 不改变 |
| `sorted(seq, key)` | 排序 | N个 → N个（有序） | ❌ 不改变 |

> 💡 三个函数都**不修改原数据**，而是返回新结果。

---

## 四、列表推导式 vs 高阶函数 — 选哪个？

你可能会发现，很多 map/filter 的场景也能用列表推导式实现：

```python
nums = [1, 2, 3, 4, 5]

# map 的等价写法
squares1 = list(map(lambda x: x ** 2, nums))
squares2 = [x ** 2 for x in nums]          # 列表推导式

# filter 的等价写法
evens1 = list(filter(lambda x: x % 2 == 0, nums))
evens2 = [x for x in nums if x % 2 == 0]  # 列表推导式

print(squares2)  # [1, 4, 9, 16, 25]
print(evens2)    # [2, 4]
```

**选择建议**：

| 场景 | 推荐 | 理由 |
|------|------|------|
| 简单变换/筛选 | 列表推导式 | 更 Pythonic，更易读 |
| 逻辑较复杂 | def + map/filter | 可读性更好 |
| 需要排序 | sorted + lambda | 列表推导式做不了排序 |
| 已有现成函数 | map(函数, ...) | 不用再写 lambda |

> 💡 **Python 之禅**：`Simple is better than complex.` 能用列表推导式就别硬上 lambda。

---

## 五、递归入门 — 函数调用自己

### 5.1 什么是递归？

递归 = 函数在自己的内部调用自己。就像两面镜子面对面，会看到无限镜像。

**递归的两个必要条件**：
1. **基线条件（base case）**：什么时候停下来，不再调用自己
2. **递推关系**：把问题拆成更小的同类问题

```python
# 经典例子：计算阶乘 n!
# 5! = 5 × 4 × 3 × 2 × 1 = 120

def factorial(n):
    """计算 n 的阶乘"""
    if n <= 1:           # 基线条件：1! = 1
        return 1
    return n * factorial(n - 1)  # 递推：n! = n × (n-1)!

print(factorial(5))  # 120
# 执行过程：
# factorial(5) → 5 * factorial(4)
#             → 5 * 4 * factorial(3)
#             → 5 * 4 * 3 * factorial(2)
#             → 5 * 4 * 3 * 2 * factorial(1)
#             → 5 * 4 * 3 * 2 * 1 = 120
```

### 5.2 递归的执行过程

理解递归的关键是区分**递推（往下走）**和**回归（往上返）**：

```
factorial(5) 的执行过程：

递推（往下展开）         回归（往上计算）
─────────────────         ─────────────────
factorial(5)              5 * 24 = 120
 = 5 * factorial(4)       4 * 6 = 24
    = 4 * factorial(3)    3 * 2 = 6
       = 3 * factorial(2) 2 * 1 = 2
          = 2 * factorial(1)
             = 1  ← 命中基线条件，开始回归
```

### 5.3 更多递归示例

```python
# 1. 斐波那契数列：1, 1, 2, 3, 5, 8, 13, ...
def fib(n):
    """返回第 n 个斐波那契数"""
    if n <= 2:
        return 1
    return fib(n - 1) + fib(n - 2)

print(fib(7))  # 13

# 2. 求列表中所有数字的和
def list_sum(lst):
    if not lst:           # 空列表 → 0
        return 0
    return lst[0] + list_sum(lst[1:])

print(list_sum([1, 2, 3, 4, 5]))  # 15

# 3. 字符串反转
def reverse_str(s):
    if len(s) <= 1:      # 单字符或空 → 直接返回
        return s
    return reverse_str(s[1:]) + s[0]

print(reverse_str("hello"))  # olleh
```

### 5.4 递归 vs 循环

```python
# 用循环计算阶乘
def factorial_loop(n):
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result

# 用递归计算阶乘
def factorial_rec(n):
    if n <= 1:
        return 1
    return n * factorial_rec(n - 1)

print(factorial_loop(5))  # 120
print(factorial_rec(5))   # 120
```

| 对比 | 循环 | 递归 |
|------|------|------|
| 理解难度 | 较直观 | 需要理解调用栈 |
| 性能 | 通常更快 | 有函数调用开销 |
| 内存 | 省内存 | 每次调用占栈空间 |
| 适用场景 | 大多数场景 | 树形结构、分治问题 |
| 风险 | 几乎无 | 可能栈溢出 |

> ⚠️ **递归避坑**：
> 1. **必须要有基线条件**，否则会无限递归 → `RecursionError`
> 2. Python 默认递归深度上限是 1000（`sys.getrecursionlimit()` 可查看）
> 3. 能用循环解决的，优先用循环（更高效）

```python
# ❌ 忘记基线条件 → 无限递归
def oops(n):
    return n + oops(n - 1)  # 永远停不下来！

# ✅ 正确写法
def correct(n):
    if n <= 0:         # 基线条件
        return 0
    return n + correct(n - 1)
```

---

## 六、实战项目 — 学生成绩分析器

综合运用今天学的 lambda、高阶函数、递归：

```python
# ========== 数据 ==========
students = [
    {"name": "小明", "math": 85, "english": 92, "python": 78},
    {"name": "小红", "math": 90, "english": 88, "python": 95},
    {"name": "小刚", "math": 72, "english": 65, "python": 80},
    {"name": "小美", "math": 95, "english": 70, "python": 88},
    {"name": "小李", "math": 60, "english": 85, "python": 72},
]

# ========== 功能1：计算每个学生的总分 ==========
def get_total(student):
    return student["math"] + student["english"] + student["python"]

# 用 map 批量计算
totals = list(map(lambda s: {"name": s["name"], "total": get_total(s)}, students))
print("📊 学生总分：")
for t in totals:
    print(f"  {t['name']}: {t['total']}分")

# ========== 功能2：按总分排名 ==========
ranked = sorted(students, key=get_total, reverse=True)
print("\n🏆 总分排名：")
for i, s in enumerate(ranked, 1):
    print(f"  第{i}名: {s['name']} - {get_total(s)}分")

# ========== 功能3：筛选Python成绩>=85的学生 ==========
good_python = list(filter(lambda s: s["python"] >= 85, students))
print("\n🐍 Python成绩优秀（≥85）：")
for s in good_python:
    print(f"  {s['name']}: {s['python']}分")

# ========== 功能4：用递归计算班级平均分 ==========
def class_avg(stu_list, subject):
    """递归计算某科平均分"""
    if not stu_list:
        return 0
    total = stu_list[0][subject] + class_avg(stu_list[1:], subject) * (len(stu_list) - 1)
    return total / len(stu_list)

# 更简洁的递归方式
def class_avg_v2(stu_list, subject):
    """递归求某科平均分"""
    n = len(stu_list)
    if n == 1:
        return stu_list[0][subject]
    return (stu_list[0][subject] + class_avg_v2(stu_list[1:], subject) * (n - 1)) / n

print(f"\n📈 班级各科平均分：")
for subj in ["math", "english", "python"]:
    avg = class_avg_v2(students, subj)
    print(f"  {subj}: {avg:.1f}")

# ========== 功能5：找出每科最高分 ==========
subjects = ["math", "english", "python"]
for subj in subjects:
    best = max(students, key=lambda s: s[subj])
    print(f"\n🥇 {subj} 最高分: {best['name']} - {best[subj]}分")
```

**运行结果**：
```
📊 学生总分：
  小明: 255分
  小红: 273分
  小刚: 217分
  小美: 253分
  小李: 217分

🏆 总分排名：
  第1名: 小红 - 273分
  第2名: 小明 - 255分
  第3名: 小美 - 253分
  第4名: 小刚 - 217分
  第5名: 小李 - 217分

🐍 Python成绩优秀（≥85）：
  小红: 95分
  小美: 88分

📈 班级各科平均分：
  math: 80.4
  english: 80.0
  python: 82.6

🥇 math 最高分: 小美 - 95分
🥇 english 最高分: 小明 - 92分
🥇 python 最高分: 小红 - 95分
```

---

## 七、练习题

### 基础题

**1. lambda 练习**
```python
# 用 lambda 完成以下函数：
# (a) 接收一个数，返回它的立方
cube = lambda x: ______

# (b) 接收两个字符串，返回较长的那个
longer = lambda a, b: ______

# (c) 接收一个数字，判断是否为正数（返回 True/False）
is_positive = lambda n: ______
```

**2. map / filter 练习**
```python
nums = [3, -1, 4, -1, 5, -9, 2, -6, 5, 3, 5]

# (a) 用 map 把每个数取绝对值
# (b) 用 filter 只保留正数
# (c) 用 map 把每个数转成字符串
```

### 进阶题

**3. 排序练习**
```python
words = ["banana", "pie", "Washington", "book", "apple"]

# (a) 按单词长度排序
# (b) 按单词最后一个字母排序
# (c) 按单词的小写形式排序（忽略大小写）
```

**4. 递归练习**
```python
# 写一个递归函数 power(base, exp)，计算 base 的 exp 次方
# 例如 power(2, 3) = 8
# 要求：不能使用 ** 运算符
```

### 挑战题

**5. 综合应用**
```python
# 给定一个字符串列表，完成以下任务（尽量用 lambda + 高阶函数）：
words = ["Hello", "world", "Python", "is", "awesome", "AI"]

# (a) 筛选出长度 > 4 的单词
# (b) 把所有单词变成小写
# (c) 按字母顺序排序
# (d) 一行链式调用完成 a+b+c（筛选 → 变小写 → 排序）

# (e) 用递归写一个函数 count_len(lst)，
#     计算列表中所有字符串的总长度
#     count_len(words) 应返回 27
```

---

## 八、常见问题 & 避坑

| 问题 | 原因 | 解决 |
|------|------|------|
| lambda 写了多行 | lambda 只能一个表达式 | 逻辑复杂就用 `def` |
| `map()` 结果是空的 | map 返回迭代器 | 用 `list(map(...))` 转换 |
| sorted 排序不对 | 忘了写 `key` 参数 | 看看是不是在排复杂对象 |
| 递归报 `RecursionError` | 没有基线条件或递归太深 | 检查 base case，考虑改用循环 |
| filter 结果不对 | lambda 返回的不是布尔值 | filter 保留的是返回为 True 的元素 |
| lambda 捕获循环变量 | 闭包延迟绑定 | 用默认参数绑定：`lambda x, i=i: ...` |

### ⚠️ 今日最重要的避坑 — lambda 闭包陷阱

```python
# ❌ 经典错误：lambda 捕获的是变量的引用，不是值
funcs = []
for i in range(5):
    funcs.append(lambda: i)     # 所有 lambda 都引用同一个 i

print([f() for f in funcs])    # [4, 4, 4, 4, 4]  ← 全是4！

# ✅ 正确写法：用默认参数绑定当前值
funcs = []
for i in range(5):
    funcs.append(lambda i=i: i)  # i=i 把当前值绑定到默认参数

print([f() for f in funcs])    # [0, 1, 2, 3, 4]  ✅
```

> 这个坑在写循环创建函数时特别常见，记住：**lambda 中引用循环变量，要用默认参数绑定**。

---

## 九、学习资源

| 资源 | 说明 | 链接 |
|------|------|------|
| Python 官方文档 - lambda | 匿名函数语法 | [docs.python.org](https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions) |
| Python 官方文档 - map/filter | 函数式编程工具 | [docs.python.org](https://docs.python.org/3/howto/functional.html) |
| Real Python - Lambda | 英文，深入讲解 | [realpython.com/python-lambda](https://realpython.com/python-lambda/) |
| 廖雪峰 - 高阶函数 | 中文教程 | [liaoxuefeng.com](https://www.liaoxuefeng.com/wiki/1016959663602400/1017328525009056) |

---

## 十、今日总结

```
📋 Day 11 知识清单
├── lambda 匿名函数
│   ├── 语法：lambda 参数: 表达式
│   ├── 场景：排序key、简单逻辑
│   └── ⚠️ 坑：闭包延迟绑定 → 用 i=i 绑定
├── 高阶函数
│   ├── map(函数, 序列) → 批量变换
│   ├── filter(函数, 序列) → 批量筛选
│   ├── sorted(序列, key=) → 排序
│   └── 列表推导式 vs 高阶函数的选择
└── 递归
    ├── 两个必要条件：基线条件 + 递推关系
    ├── 经典示例：阶乘、斐波那契、列表求和
    ├── 递归 vs 循环的取舍
    └── ⚠️ 坑：无限递归 → RecursionError
```

> 🎯 **Day 12 预告**：文件操作 — 读写文件 / with 语句 / CSV处理 / JSON数据处理
>
> 今天学的 lambda 和高阶函数在数据处理中非常常用，尤其是 `sorted(key=...)` 和 `filter()`。递归不着急精通，先理解概念，后面学树结构时会大量用到。

---

*Day 11 完成！你正在从"会写代码"走向"写优雅的代码" 🚀*
