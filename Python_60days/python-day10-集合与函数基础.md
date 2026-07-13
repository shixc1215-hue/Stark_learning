# 🐍 Python 60天学习计划 · Day 10

> **阶段**：第一阶段 · 基础语法入门（Day 1-14）
> **今日主题**：集合（set）去重利器 + 函数基础（def）— 写出可复用的代码
> **预计时间**：约 60 分钟
> **进度**：Day 10 / 60 ▓▓▓▓░░░░░░░░░░░░░░░░ 17%

---

## 一、今日目标

- ✅ 理解集合（set）的特点和用途
- ✅ 掌握集合的创建、增删、运算（交集/并集/差集）
- ✅ 学会用集合高效去重
- ✅ 理解函数的意义：封装、复用、组织代码
- ✅ 掌握函数定义（def）、参数、返回值
- ✅ 完成实战练习

---

## 二、集合（set）— 去重利器

### 2.1 为什么需要集合？

```python
# 场景：统计一个列表中有多少种不同的水果
fruits = ["苹果", "香蕉", "苹果", "橙子", "香蕉", "苹果"]

# 方法1：用列表手动去重（很麻烦）
unique = []
for f in fruits:
    if f not in unique:
        unique.append(f)
print(unique)  # ['苹果', '香蕉', '橙子']

# 方法2：用集合，一行搞定！
unique = list(set(fruits))
print(unique)  # ['苹果', '香蕉', '橙子']（顺序可能不同）
```

### 2.2 集合的创建

```python
# 方式1：花括号
colors = {"红", "绿", "蓝"}
print(type(colors))  # <class 'set'>

# 方式2：set() 函数
nums = set([1, 2, 3, 3, 3])
print(nums)  # {1, 2, 3}

# ⚠️ 空集合只能用 set()，不能用 {}
empty = set()       # ✅ 这是集合
wrong = {}          # ❌ 这是字典！
print(type(wrong))  # <class 'dict'>
```

### 2.3 集合的核心特点

| 特点 | 说明 | 示例 |
|------|------|------|
| **去重** | 自动去除重复元素 | `{1,1,2}` → `{1,2}` |
| **无序** | 没有索引，不能通过下标访问 | `s[0]` ❌ 报错 |
| **可变** | 可以增删元素 | `add()` / `remove()` |
| **元素必须可哈希** | 不能放列表、字典 | `{[1,2]}` ❌ 报错 |

```python
s = {1, 2, 3}
# s[0]       # ❌ TypeError: 'set' object is not subscriptable
# s = {[1]}  # ❌ TypeError: unhashable type: 'list'
s = {(1,2)}  # ✅ 元组可以，因为元组不可变
```

### 2.4 集合的增删操作

```python
fruits = {"苹果", "香蕉"}

# 添加单个元素
fruits.add("橙子")
print(fruits)  # {'苹果', '香蕉', '橙子'}

# 添加多个元素
fruits.update(["葡萄", "西瓜"])
print(fruits)  # {'苹果', '香蕉', '橙子', '葡萄', '西瓜'}

# 删除元素
fruits.remove("香蕉")    # ✅ 元素不存在会报错 KeyError
fruits.discard("芒果")    # ✅ 元素不存在也不报错（推荐用这个）
popped = fruits.pop()     # 随机移除一个元素并返回
print(f"移除了: {popped}")

# 清空
fruits.clear()
print(fruits)  # set()
```

> 💡 **经验**：`discard()` 比 `remove()` 更安全，不确定元素是否存在时用 `discard()`。

### 2.5 集合运算 — 最强大的功能

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# 并集：两个集合的所有元素（去重）
print(a | b)              # {1, 2, 3, 4, 5, 6}
print(a.union(b))         # 同上

# 交集：两个集合共有的元素
print(a & b)              # {3, 4}
print(a.intersection(b))  # 同上

# 差集：a 有但 b 没有的
print(a - b)                  # {1, 2}
print(a.difference(b))        # 同上

# 对称差集：只在其中一个集合的元素（去掉交集）
print(a ^ b)                       # {1, 2, 5, 6}
print(a.symmetric_difference(b))   # 同上

# 子集判断
c = {1, 2}
print(c <= a)           # True，c 是 a 的子集
print(c.issubset(a))    # 同上
print(a >= c)           # True，a 是 c 的超集
```

**记忆口诀**：

| 运算 | 符号 | 方法 | 记忆 |
|------|------|------|------|
| 并集 | `\|` | union | "或"的关系，合在一起 |
| 交集 | `&` | intersection | "且"的关系，共同拥有 |
| 差集 | `-` | difference | 减掉共有的 |
| 对称差 | `^` | symmetric_difference | 不同时拥有 |

### 2.6 frozenset — 不可变集合

```python
# 普通集合可变，不能作为字典的键或集合的元素
# frozenset 不可变，可以

fs = frozenset([1, 2, 3])
# fs.add(4)  # ❌ AttributeError

# 用途：作为字典的键
d = {frozenset([1, 2]): "值"}
```

---

## 三、函数基础（def）— 代码复用的起点

### 3.1 为什么需要函数？

```python
# ❌ 没有函数：重复代码
print("你好，张三！欢迎学习Python！")
print("你好，李四！欢迎学习Python！")
print("你好，王五！欢迎学习Python！")

# ✅ 用函数：写一次，用多次
def greet(name):
    print(f"你好，{name}！欢迎学习Python！")

greet("张三")
greet("李四")
greet("王五")
```

**函数的好处**：
1. **复用** — 写一次，调用无数次
2. **简洁** — 消除重复代码
3. **易改** — 改一处，处处生效
4. **可读** — 一个好函数名就是最好的注释

### 3.2 定义函数的基本语法

```python
def 函数名(参数1, 参数2, ...):
    """文档字符串（docstring）：说明函数的作用"""
    函数体
    return 返回值
```

```python
def add(a, b):
    """返回两个数的和"""
    result = a + b
    return result

# 调用函数
total = add(3, 5)
print(total)  # 8
```

> 💡 **函数定义的4个要素**：
> - `def` 关键字
> - 函数名（遵守变量命名规则，通常用小写+下划线）
> - 参数（可以没有）
> - 返回值（没有 return 则返回 None）

### 3.3 参数的多种形式

```python
# 1. 位置参数 — 按顺序传
def introduce(name, age):
    print(f"我叫{name}，今年{age}岁")

introduce("小明", 25)  # 位置参数

# 2. 关键字参数 — 按名字传，不依赖顺序
introduce(age=25, name="小明")  # 关键字参数

# 3. 默认参数 — 不传就使用默认值
def greet(name, greeting="你好"):
    print(f"{greeting}，{name}！")

greet("小明")              # 你好，小明！
greet("小明", "早上好")    # 早上好，小明！

# 4. 可变位置参数 *args — 接收任意数量的位置参数
def total(*numbers):
    print(f"接收到的参数: {numbers}")  # 元组形式
    return sum(numbers)

print(total(1, 2, 3, 4, 5))  # 接收到的参数: (1, 2, 3, 4, 5) → 15

# 5. 可变关键字参数 **kwargs — 接收任意数量的关键字参数
def print_info(**info):
    for key, value in info.items():
        print(f"{key}: {value}")

print_info(name="小明", age=25, city="北京")
# name: 小明
# age: 25
# city: 北京
```

**参数顺序规则**（必须按此顺序定义）：

```
位置参数 → 默认参数 → *args → **kwargs
```

```python
def func(a, b, c=10, *args, **kwargs):
    pass
```

### 3.4 返回值

```python
# 返回单个值
def square(n):
    return n * n

result = square(5)  # 25

# 返回多个值（本质是返回元组）
def min_max(lst):
    return min(lst), max(lst)

lo, hi = min_max([3, 1, 4, 1, 5, 9])  # 元组解包
print(f"最小={lo}, 最大={hi}")  # 最小=1, 最大=9

# 没有 return → 返回 None
def say_hi():
    print("Hi!")

result = say_hi()  # 打印 "Hi!"
print(result)       # None
```

### 3.5 变量作用域 — 哪里能访问哪里

```python
x = "全局变量"         # 全局作用域

def my_func():
    x = "局部变量"     # 局部作用域，和外面的 x 无关！
    print(x)          # 局部变量

my_func()
print(x)              # 全局变量

# ⚠️ 函数内修改全局变量需要 global 关键字
count = 0

def increment():
    global count      # 声明我要用全局的 count
    count += 1

increment()
print(count)  # 1
```

> ⚠️ **避坑**：尽量少用 `global`，函数应该通过参数接收数据、通过返回值输出结果，这样更安全、更易理解。

---

## 四、集合 + 函数 实战演练

### 实战1：列表去重工具

```python
def remove_duplicates(lst, keep_order=True):
    """去重函数
    参数:
        lst: 需要去重的列表
        keep_order: 是否保持原始顺序（默认True）
    返回:
        去重后的列表
    """
    if keep_order:
        seen = set()
        result = []
        for item in lst:
            if item not in seen:
                seen.add(item)
                result.append(item)
        return result
    else:
        return list(set(lst))

# 测试
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
print(remove_duplicates(data))              # [3, 1, 4, 5, 9, 2, 6] 保持顺序
print(remove_duplicates(data, False))       # {1, 2, 3, 4, 5, 6, 9} 顺序不定
```

### 实战2：共同好友查找器

```python
def find_mutual_friends(my_friends, their_friends):
    """查找共同好友"""
    mutual = set(my_friends) & set(their_friends)
    return mutual

def find_suggested_friends(my_friends, their_friends):
    """推荐好友：对方有但我没有的"""
    suggested = set(their_friends) - set(my_friends)
    return suggested

# 测试
my = ["小明", "小红", "小刚", "小美"]
their = ["小红", "小刚", "小丽", "小强"]

mutual = find_mutual_friends(my, their)
suggested = find_suggested_friends(my, their)

print(f"共同好友: {mutual}")       # {'小红', '小刚'}
print(f"推荐好友: {suggested}")     # {'小丽', '小强'}
```

### 实战3：简易通讯录（函数版）

```python
# 用字典存储通讯录
contacts = {}

def add_contact(name, phone):
    """添加联系人"""
    if name in contacts:
        print(f"⚠️ {name} 已存在，号码为 {contacts[name]}")
        return False
    contacts[name] = phone
    print(f"✅ 已添加: {name} - {phone}")
    return True

def find_contact(name):
    """查找联系人"""
    phone = contacts.get(name)
    if phone:
        print(f"📞 {name}: {phone}")
    else:
        print(f"❌ 未找到 {name}")
    return phone

def delete_contact(name):
    """删除联系人"""
    if name in contacts:
        phone = contacts.pop(name)
        print(f"🗑️ 已删除: {name} - {phone}")
    else:
        print(f"❌ 未找到 {name}")

def list_contacts():
    """列出所有联系人"""
    if not contacts:
        print("📭 通讯录为空")
        return
    print(f"📋 共 {len(contacts)} 位联系人:")
    for name, phone in contacts.items():
        print(f"  {name}: {phone}")

# === 测试 ===
add_contact("小明", "13800138000")
add_contact("小红", "13900139000")
add_contact("小明", "13700137000")  # 已存在

find_contact("小明")
find_contact("小刚")  # 不存在

delete_contact("小红")
delete_contact("小刚")  # 不存在

list_contacts()
```

---

## 五、练习题

### 基础题

**1. 集合操作**
```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}
# 求：并集、交集、A-B、B-A、对称差
```

**2. 列表去重并排序**
```python
nums = [5, 3, 5, 2, 1, 3, 5, 2, 4, 1]
# 期望输出：[1, 2, 3, 4, 5]
```

### 进阶题

**3. 写一个函数 `is_anagram(word1, word2)`**
判断两个单词是否是字母异位词（字母相同，顺序不同）。
> 提示：把单词转成集合比较
> 进阶：考虑字母频率（如 "aab" 和 "abb" 不是异位词）

**4. 写一个函数 `merge_lists(list1, list2)`**
合并两个列表，去重，保持顺序（先出现的优先）。

### 挑战题

**5. 写一个函数 `analyze_text(text)`**
输入一段文字，返回：
- 不同字符的数量
- 最常见的3个字符
- 只出现1次的字符集合

```python
def analyze_text(text):
    # 你的代码
    pass

result = analyze_text("hello world")
# 期望类似: 不同字符=8, 最常见=[('l',3), ('o',2), ('h',1)], 只出现1次={'h','e','w','r','d'}
```

---

## 六、常见问题 & 避坑

| 问题 | 原因 | 解决 |
|------|------|------|
| `set()` vs `{}` 创建空集合 | `{}` 创建的是字典 | 空集合只能用 `set()` |
| 集合不能放列表 | 列表是可变对象，不可哈希 | 用元组替代：`{(1,2)}` |
| `remove()` 报 KeyError | 元素不存在 | 改用 `discard()` |
| 函数修改了外部变量 | 局部变量名和全局变量名相同 | 用参数传入，用 return 传出 |
| 函数返回 None | 忘记写 `return` | 检查是否写了 `return` 语句 |
| `*args` 和 `**kwargs` 搞混 | 位置参数和关键字参数不分 | `*args` 收集多余位置参数为元组，`**kwargs` 收集多余关键字参数为字典 |

### ⚠️ 今日最重要的避坑

```python
# ❌ 经典错误：函数内修改可变默认参数
def append_item(item, lst=[]):    # 默认参数是可变对象！
    lst.append(item)
    return lst

print(append_item(1))  # [1]
print(append_item(2))  # [1, 2] ← 不是 [2]！默认列表被共享了！

# ✅ 正确写法
def append_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst

print(append_item(1))  # [1]
print(append_item(2))  # [2] ✅
```

> 这是 Python 最经典的坑之一：**永远不要用可变对象（列表、字典、集合）作为默认参数！** 用 `None` 代替。

---

## 七、学习资源

| 资源 | 说明 | 链接 |
|------|------|------|
| Python 官方文档 - set | 集合完整参考 | [docs.python.org/3/library/stdtypes.html#set](https://docs.python.org/3/library/stdtypes.html#set) |
| Python 官方文档 - 函数 | 函数定义详解 | [docs.python.org/3/tutorial/controlflow.html#defining-functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) |
| 廖雪峰 - 函数 | 中文教程，简洁易懂 | [liaoxuefeng.com](https://www.liaoxuefeng.com/wiki/1016959663602400/1017105145133280) |
| Real Python - Sets | 英文，图文并茂 | [realpython.com/python-sets](https://realpython.com/python-sets/) |

---

## 八、今日总结

```
📋 Day 10 知识清单
├── 集合 set
│   ├── 创建：{1,2,3} 或 set()
│   ├── 去重：list(set(lst))
│   ├── 运算：| 并集 & 交集 - 差集 ^ 对称差
│   ├── 方法：add / discard / update / pop / clear
│   └── frozenset：不可变集合
└── 函数 def
    ├── 定义：def 名(参): return 值
    ├── 参数：位置 / 关键字 / 默认 / *args / **kwargs
    ├── 返回值：return / 多值返回（元组解包）
    ├── 作用域：局部 vs 全局 / global
    └── ⚠️ 坑：可变默认参数 → 用 None
```

> 🎯 **Day 11 预告**：函数进阶 — lambda匿名函数 / 高阶函数（map/filter/sorted）/ 递归入门
>
> 今天的内容比较多，集合和函数都是核心基础。函数是 Python 编程最重要的概念之一，务必多练！如果时间不够，优先把函数的参数和返回值搞清楚。

---

*坚持就是胜利！你已经完成 10/60，基础阶段快过半了 🎉*
