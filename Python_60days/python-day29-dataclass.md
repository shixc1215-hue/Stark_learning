# Python Day 29 - dataclass（数据类）

## 一、什么是 dataclass？

在 Python 中，我们经常需要定义一些"只存数据"的类，比如学生信息、商品信息、配置项等。这类类的核心目的是**存储和访问数据**，而不是执行复杂的逻辑。

传统写法下，即使只是存数据，你也得手写一堆重复代码：

```python
class Student:
    def __init__(self, name: str, age: int, score: float):
        self.name = name
        self.age = age
        self.score = score

    def __repr__(self):
        return f"Student(name={self.name!r}, age={self.age}, score={self.score})"

    def __eq__(self, other):
        if not isinstance(other, Student):
            return False
        return (self.name, self.age, self.score) == (other.name, other.age, other.score)
```

上面的代码中，`__init__`、`__repr__`、`__eq__` 这些方法几乎每个数据类都要写一遍，非常繁琐。

**dataclass（数据类）** 是 Python 3.7 引入的特性（`dataclasses` 模块），能让你用**一个装饰器**自动生成这些重复的方法，让代码更简洁、更 Pythonic。

> **核心思想**：把精力放在"有哪些字段"上，样板代码让 Python 帮你生成。

---

## 二、基础用法

### 2.1 定义一个 dataclass

使用 `@dataclass` 装饰器，只需要声明**类变量（类型注解 + 默认值）**即可：

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str       # 姓名
    age: int        # 年龄
    score: float    # 成绩
```

就这么简单！`@dataclass` 会自动帮你生成：

| 自动生成的方法 | 作用 |
|---|---|
| `__init__()` | 构造方法，接收所有字段作为参数 |
| `__repr__()` | 友好的字符串表示，方便调试打印 |
| `__eq__()`   | 判断两个对象的所有字段是否相等 |

```python
# 使用 dataclass
s1 = Student("小明", 18, 95.5)
s2 = Student("小明", 18, 95.5)
s3 = Student("小红", 19, 88.0)

print(s1)            # Student(name='小明', age=18, score=95.5)
print(s1 == s2)       # True —— 所有字段值相同
print(s1 == s3)       # False —— 至少一个字段不同
```

### 2.2 对比：传统写法 vs dataclass

```python
# ===== 传统写法（25+ 行）=====
class Book:
    def __init__(self, title: str, author: str, pages: int, price: float):
        self.title = title
        self.author = author
        self.pages = pages
        self.price = price

    def __repr__(self):
        return f"Book(title={self.title!r}, author={self.author!r}, pages={self.pages}, price={self.price})"

    def __eq__(self, other):
        if not isinstance(other, Book):
            return NotImplemented
        return (self.title, self.author, self.pages, self.price) == \
               (other.title, other.author, other.pages, other.price)

# ===== dataclass 写法（5 行）=====
from dataclasses import dataclass

@dataclass
class Book:
    title: str
    author: str
    pages: int
    price: float
```

两种写法使用方式完全一样，但 dataclass 版本代码量减少 **80%** 以上。

---

## 三、字段默认值

### 3.1 设置默认值

和普通函数参数一样，可以为字段设置默认值：

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    stock: int = 0          # 默认库存为 0
    description: str = ""   # 默认描述为空字符串

# 所有参数都传
p1 = Product("手机", 3999.0, 100, "高端智能手机")
print(p1)  # Product(name='手机', price=3999.0, stock=100, description='高端智能手机')

# 只传必要参数，其余用默认值
p2 = Product("耳机", 199.0)
print(p2)  # Product(name='耳机', price=199.0, stock=0, description='')
```

### 3.2 默认值的顺序规则

**重要**：没有默认值的字段必须放在有默认值的字段**前面**，这和函数参数的规则一致：

```python
# ✅ 正确：无默认值在前，有默认值在后
@dataclass
class Config:
    host: str
    port: int
    timeout: int = 30
    debug: bool = False

# ❌ 错误！有默认值的字段不能在无默认值的字段前面
# @dataclass
# class BadConfig:
#     timeout: int = 30   # 默认值
#     host: str           # 非默认值 —— 会报错！
#     port: int
```

### 3.3 使用 field() 设置高级默认值

当默认值是**可变对象**（列表、字典等）时，不能直接写 `[]` 或 `{}`，否则所有实例会共享同一个对象。这时需要用 `field()` 函数：

```python
from dataclasses import dataclass, field

@dataclass
class Team:
    name: str
    members: list = field(default_factory=list)   # 每个实例创建新的空列表
    scores: dict = field(default_factory=dict)   # 每个实例创建新的空字典

t1 = Team("A组")
t2 = Team("B组")

# 它们各自拥有独立的列表和字典
t1.members.append("小明")
t2.members.append("小红")

print(t1.members)  # ['小明']
print(t2.members)  # ['小红'] —— 互不影响！
```

> **为什么不能用 `members: list = []`？** 因为 `[]` 在类定义时就创建了，所有实例共享这同一个列表——这是 Python 的经典陷阱。`default_factory=list` 意味着"每次创建实例时调用 `list()` 来生成一个新列表"。

---

## 四、field() 的高级功能

`field()` 除了处理可变默认值外，还有其他实用参数：

### 4.1 参数一览

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Employee:
    name: str
    salary: float
    skills: List[str] = field(default_factory=list)
    id: int = field(default=0, init=False)              # 不参与 __init__
    login_count: int = field(default=0, repr=False)    # 不出现在 __repr__ 中
    password: str = field(default="", compare=False)   # 不参与 __eq__ 比较
```

| field() 参数 | 含义 |
|---|---|
| `default` | 设置默认值（不可变类型时使用，如 `default=0`） |
| `default_factory` | 设置默认值工厂函数（可变类型时使用） |
| `init=False` | 该字段**不**作为 `__init__()` 的参数 |
| `repr=False` | 该字段**不**出现在 `__repr__()` 输出中 |
| `compare=False` | 该字段**不**参与 `__eq__()` 等比较 |

### 4.2 init=False：计算字段

`init=False` 常用于"根据其他字段自动计算"的字段：

```python
from dataclasses import dataclass, field

@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)   # 不需要传入，自动计算

    def __post_init__(self):
        """在 __init__ 之后自动执行，用于计算派生字段"""
        self.area = self.width * self.height

# 使用时只需传 width 和 height
r = Rectangle(5.0, 3.0)
print(r)  # Rectangle(width=5.0, height=3.0, area=15.0)
```

### 4.3 __post_init__() 方法

`__post_init__()` 是 dataclass 提供的特殊钩子方法，在 `__init__()` 执行完毕后**自动调用**。它非常适合：

- 数据校验
- 计算派生字段
- 自动转换格式

```python
from dataclasses import dataclass, field

@dataclass
class User:
    username: str
    email: str
    age: int

    def __post_init__(self):
        # 数据校验：用户名不能为空
        if not self.username:
            raise ValueError("用户名不能为空！")

        # 自动转换：邮箱统一小写
        self.email = self.email.lower()

        # 数据校验：年龄范围
        if not (0 < self.age < 150):
            raise ValueError(f"年龄 {self.age} 不合理！")

# 正常创建
u1 = User("xiaoming", "XiaoMing@Gmail.com", 25)
print(u1)  # User(username='xiaoming', email='xiaoming@gmail.com', age=25)

# 异常情况
try:
    u2 = User("", "test@test.com", 25)
except ValueError as e:
    print(e)  # 用户名不能为空！
```

---

## 五、让 dataclass 生成更多方法

`@dataclass` 装饰器有几个参数，控制额外自动生成哪些方法：

### 5.1 参数说明

```python
from dataclasses import dataclass

@dataclass(
    init=True,      # 生成 __init__（默认 True）
    repr=True,      # 生成 __repr__（默认 True）
    eq=True,        # 生成 __eq__（默认 True）
    order=False,    # 生成 < > <= >= 比较方法（默认 False）
    frozen=False,   # 设为不可变（默认 False）
    unsafe_hash=False,  # 控制 __hash__（默认 False）
    slots=False,    # 使用 __slots__ 优化内存（Python 3.10+，默认 False）
)
class Score:
    name: str
    value: int
```

### 5.2 order=True：支持排序

设置 `order=True` 后，dataclass 会自动生成 `__lt__`、`__le__`、`__gt__`、`__ge__` 四个方法，让对象可以比较大小：

```python
from dataclasses import dataclass

@dataclass(order=True)
class Score:
    name: str
    value: int

s1 = Score("小明", 85)
s2 = Score("小红", 92)
s3 = Score("小刚", 78)

# 可以直接比较和排序！
print(s2 > s1)          # True（比较 value 字段）
print(sorted([s1, s2, s3]))  # [Score(name='小刚', value=78), Score(name='小明', value=85), Score(name='小红', value=92)]
```

> **注意**：当 `order=True` 时，默认 `eq=True`。排序会按照字段的**定义顺序**依次比较。

### 5.3 frozen=True：不可变数据类

设置 `frozen=True` 后，创建后就不能修改任何字段（类似于命名元组）：

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(3.0, 4.0)
print(p)  # Point(x=3.0, y=4.0)

try:
    p.x = 5.0  # 尝试修改 —— 会报错！
except dataclasses.FrozenInstanceError as e:
    print(f"错误：{e}")  # 错误：cannot assign to field 'x'
```

**用途**：当你需要数据不可变时（比如配置项、坐标点、金额等），使用 `frozen=True` 可以防止意外修改，也更安全地作为字典的 key 或放入集合。

```python
@dataclass(frozen=True)
class Point:
    x: float
    y: float

# 不可变对象可以作为字典的 key 和集合的元素
point_map = {Point(1, 2): "起点", Point(3, 4): "终点"}
point_set = {Point(1, 2), Point(3, 4), Point(1, 2)}  # 自动去重
print(len(point_set))  # 2
```

---

## 六、嵌套 dataclass

dataclass 可以嵌套使用，构建复杂的数据结构：

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Address:
    province: str
    city: str
    district: str
    detail: str

@dataclass
class Person:
    name: str
    age: int
    address: Address               # 嵌套另一个 dataclass
    hobbies: List[str] = field(default_factory=list)

# 创建嵌套对象
addr = Address("广东", "深圳", "南山区", "科技园路100号")
person = Person("小明", 25, addr, ["编程", "跑步", "阅读"])

print(person)
# Person(name='小明', age=25, address=Address(province='广东', city='深圳', district='南山区', detail='科技园路100号'), hobbies=['编程', '跑步', '阅读'])

# 访问嵌套字段
print(person.address.city)     # 深圳
print(person.hobbies[0])       # 编程
```

---

## 七、dataclass 转换

### 7.1 转换为字典

使用 `dataclasses.asdict()` 将 dataclass 实例转为普通字典：

```python
from dataclasses import dataclass, asdict

@dataclass
class Book:
    title: str
    author: str
    price: float

book = Book("Python入门", "小明", 59.9)

# 转换为字典
book_dict = asdict(book)
print(book_dict)        # {'title': 'Python入门', 'author': '小明', 'price': 59.9}
print(type(book_dict))  # <class 'dict'>

# 方便导出为 JSON
import json
json_str = json.dumps(book_dict, ensure_ascii=False, indent=2)
print(json_str)
# {
#   "title": "Python入门",
#   "author": "小明",
#   "price": 59.9
# }
```

### 7.2 转换为元组

使用 `dataclasses.astuple()` 将 dataclass 实例转为元组：

```python
from dataclasses import dataclass, astuple

@dataclass
class Point:
    x: float
    y: float

p = Point(3.0, 4.0)
point_tuple = astuple(p)
print(point_tuple)       # (3.0, 4.0)
print(type(point_tuple)) # <class 'tuple'>
```

---

## 八、综合实战：待办事项管理系统

结合 dataclass 和之前学过的文件操作、JSON 处理，做一个完整的待办事项管理工具：

```python
from dataclasses import dataclass, field, asdict
from typing import List
from datetime import date
import json
import os

@dataclass
class TodoItem:
    """待办事项"""
    title: str
    done: bool = False
    priority: int = 1         # 1=低, 2=中, 3=高
    created: str = field(default_factory=lambda: date.today().isoformat())

    def mark_done(self):
        """标记为已完成"""
        self.done = True
        print(f"✓ '{self.title}' 已完成！")

    def __str__(self):
        status = "✓" if self.done else "○"
        return f"[{status}] {self.title} (优先级:{self.priority}, 创建:{self.created})"

@dataclass
class TodoList:
    """待办事项列表"""
    name: str
    items: List[TodoItem] = field(default_factory=list)

    def add(self, title: str, priority: int = 1):
        """添加新事项"""
        self.items.append(TodoItem(title, priority=priority))
        print(f"+ 已添加：{title}")

    def show(self):
        """显示所有事项"""
        if not self.items:
            print("（空）暂无待办事项")
            return
        # 未完成的排在前面，按优先级从高到低
        pending = sorted([i for i in self.items if not i.done],
                         key=lambda x: -x.priority)
        done = [i for i in self.items if i.done]
        print(f"\n{'='*40}")
        print(f"  {self.name}（共 {len(self.items)} 项）")
        print(f"{'='*40}")
        print("  待办：")
        for item in pending:
            print(f"    {item}")
        if done:
            print("  已完成：")
            for item in done:
                print(f"    {item}")

    def save(self, filepath: str):
        """保存到 JSON 文件"""
        data = {"name": self.name, "items": [asdict(i) for i in self.items]}
        with open(filepath, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
        print(f"已保存到 {filepath}")

    @classmethod
    def load(cls, filepath: str):
        """从 JSON 文件加载"""
        if not os.path.exists(filepath):
            print("文件不存在，创建新列表")
            return cls("我的待办")
        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)
        items = [TodoItem(**item) for item in data["items"]]
        return cls(data["name"], items)

# ===== 使用示例 =====
todo = TodoList("我的待办")

# 添加事项
todo.add("学习 dataclass", priority=3)
todo.add("完成作业", priority=2)
todo.add("买菜", priority=1)

# 标记完成
todo.items[0].mark_done()

# 显示列表
todo.show()

# 保存到文件
todo.save("my_todos.json")

# 从文件重新加载
loaded = TodoList.load("my_todos.json")
loaded.show()
```

运行效果：

```
+ 已添加：学习 dataclass
+ 已添加：完成作业
+ 已添加：买菜
✓ '学习 dataclass' 已完成！

========================================
  我的待办（共 3 项）
========================================
  待办：
    [○] 完成作业 (优先级:2, 创建:2026-06-29)
    [○] 买菜 (优先级:1, 创建:2026-06-29)
  已完成：
    [✓] 学习 dataclass (优先级:3, 创建:2026-06-29)
```

---

## 九、练习题

### 练习 1：定义商品数据类

定义一个 `Product` dataclass，包含字段：`name`（商品名）、`price`（价格）、`category`（分类，默认"其他"）、`tags`（标签列表，默认空列表）。创建几个实例并测试。

<details>
<summary>参考答案</summary>

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Product:
    name: str
    price: float
    category: str = "其他"
    tags: List[str] = field(default_factory=list)

p1 = Product("iPhone", 7999.0, "电子产品", ["手机", "苹果"])
p2 = Product("鼠标", 79.0)
p3 = Product("键盘", 299.0, tags=["外设"])

print(p1)
print(p2)
print(p3)
# Product(name='iPhone', price=7999.0, category='电子产品', tags=['手机', '苹果'])
# Product(name='鼠标', price=79.0, category='其他', tags=[])
# Product(name='键盘', price=299.0, category='其他', tags=['外设'])
```

</details>

### 练习 2：成绩排名系统

定义 `Student` dataclass，包含 `name`、`chinese`、`math`、`english` 三个成绩，以及 `total`（总分，由 `__post_init__` 自动计算）。支持排序。

<details>
<summary>参考答案</summary>

```python
from dataclasses import dataclass, field

@dataclass(order=True)
class Student:
    name: str
    total: float = field(init=False, compare=True)
    chinese: int
    math: int
    english: int

    def __post_init__(self):
        self.total = self.chinese + self.math + self.english

students = [
    Student("小明", 90, 85, 92),
    Student("小红", 88, 95, 90),
    Student("小刚", 78, 80, 85),
]

# 按总分排序（升序，因为 order=True）
for s in sorted(students):
    print(f"{s.name}: 总分 {s.total}")
# 小刚: 总分 243
# 小明: 总分 267
# 小红: 总分 273

# 降序排序
for s in sorted(students, reverse=True):
    print(f"{s.name}: 总分 {s.total}")
# 小红: 总分 273
# 小明: 总分 267
# 小刚: 总分 243
```

</details>

### 练习 3：不可变配置类

定义一个 `DatabaseConfig` dataclass（`frozen=True`），包含 `host`、`port`、`dbname`、`username`，密码不参与 `repr` 和比较。验证不可变性。

<details>
<summary>参考答案</summary>

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class DatabaseConfig:
    host: str
    port: int
    dbname: str
    username: str
    password: str = field(default="", repr=False, compare=False)

config = DatabaseConfig("localhost", 3306, "mydb", "root", "secret123")

print(config)  # DatabaseConfig(host='localhost', port=3306, dbname='mydb', username='root')
# password 不在 repr 中显示

# 可以比较（不比较 password）
config2 = DatabaseConfig("localhost", 3306, "mydb", "root", "different")
print(config == config2)  # True —— password 不参与比较

# 尝试修改 —— 会报错
try:
    config.port = 5432
except Exception as e:
    print(f"不可变：{e}")
```

</details>

---

## 十、常见问题

### Q1：dataclass 和普通类有什么区别？什么时候用？

| 场景 | 推荐 |
|---|---|
| 主要是存储数据（如 DTO、配置、模型） | ✅ 用 dataclass |
| 有复杂的业务逻辑、继承层次 | ❌ 用普通类 |
| 需要控制属性的读写（property） | ❌ 用普通类 |
| 需要不可变数据对象 | ✅ 用 `frozen=True` 的 dataclass |
| 简单的数据容器、函数返回多值 | ✅ 用 namedtuple 或 dataclass |

**简单原则**：如果你的类主要是 `__init__` + 几个属性 + 少量方法，优先考虑 dataclass。

### Q2：dataclass 可以继承吗？

可以，但有一些细节：

```python
from dataclasses import dataclass

@dataclass
class Animal:
    name: str
    age: int

@dataclass
class Dog(Animal):
    breed: str       # 子类新加的字段

# 子类会继承父类所有字段
d = Dog("旺财", 3, "金毛")
print(d)  # Dog(name='旺财', age=3, breed='金毛')
```

> **注意**：如果父类有默认值字段，子类的非默认值字段不能出现在默认值字段之前，否则会报错。建议子类也使用 `field(default=...)` 来处理。

### Q3：dataclass 和 namedtuple 有什么区别？

```python
# namedtuple：更轻量，但字段固定，不支持默认值和类型注解
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])

# dataclass：功能更丰富，支持默认值、类型注解、方法、继承
from dataclasses import dataclass
@dataclass
class Point:
    x: float = 0.0
    y: float = 0.0

    def distance(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
```

**选择建议**：简单数据容器用 namedtuple，需要更灵活的功能用 dataclass。

### Q4：field(default=[]) 为什么不行？

因为 `[]` 在类定义时只创建一次，所有实例共享同一个列表：

```python
# ❌ 错误示范（但 dataclass 会直接报错阻止你）
@dataclass
class Bad:
    items: list = []  # ValueError: mutable default ... use field(default_factory=list)
```

Python 3.11+ 的 dataclass 直接禁止这种写法。旧版本虽然不报错，但会导致难以调试的 bug。**永远使用 `field(default_factory=list)` 代替**。

### Q5：如何给 dataclass 添加类方法或静态方法？

和普通类完全一样：

```python
from dataclasses import dataclass

@dataclass
class Circle:
    radius: float

    @property
    def area(self):
        return 3.14159 * self.radius ** 2

    @classmethod
    def unit(cls):
        """创建半径为1的单位圆"""
        return cls(1.0)

    @staticmethod
    def is_valid_radius(r):
        return r > 0

c = Circle.unit()
print(c.area)  # 3.14159
print(Circle.is_valid_radius(5))  # True
```

---

## 十一、免费学习资源

- **Python 官方文档 - dataclasses**：https://docs.python.org/zh-cn/3/library/dataclasses.html
- **廖雪峰 Python 教程 - 面向对象高级编程**：https://www.liaoxuefeng.com/wiki/1016959663602400/1017496335944944
- **Real Python - A Guide to Python Dataclasses**：https://realpython.com/python-data-classes/ （英文，非常详细的教程）
- **菜鸟教程 - Python3 面向对象**：https://www.runoob.com/python3/python3-class.html
- **PEP 557（dataclass 原始提案）**：https://peps.python.org/pep-0557/ （了解设计理念）

---

> **下一篇预告**：Day 30 将学习**单元测试**——如何用 `unittest` 模块为你的代码写自动化测试，确保代码正确性，重构时不再心虚。
