# Python Day 17：类的高级用法

> **进度：Day 17/60 | 阶段：第二阶段 · 面向对象进阶 | 预计用时：1.5~2 小时**

---

## 一、回顾与引入

Day 15 你学会了创建类，Day 16 学会了继承与多态。今天我们继续深入，学习类的**高级特性**——这些是让代码更专业、更安全的"内功心法"。

今天要掌握的知识点：

| 知识点 | 解决什么问题 |
|--------|-------------|
| **类属性 vs 实例属性** | 数据应该放在类上还是实例上？ |
| **@classmethod 类方法** | 不创建对象也能调用类功能 |
| **@staticmethod 静态方法** | 和类无关的工具函数放哪里？ |
| **@property 属性装饰器** | 像访问变量一样调用方法 |
| **私有属性与封装** | 保护数据不被外部随意修改 |
| **魔术方法（dunder）** | 让自定义类支持 `+`、`len()`、`str()` 等操作 |

---

## 二、类属性 vs 实例属性

### 2.1 它们的区别

```python
class Dog:
    # 类属性 —— 写在类里面、方法外面
    # 所有实例共享同一份数据
    species = "犬科动物"

    def __init__(self, name, age):
        # 实例属性 —— 写在 __init__ 里
        # 每个对象各自拥有不同的值
        self.name = name
        self.age = age


# 创建两只狗
dog1 = Dog("旺财", 3)
dog2 = Dog("来福", 5)

# 类属性：所有狗的物种相同
print(dog1.species)   # 输出: 犬科动物
print(dog2.species)   # 输出: 犬科动物
print(Dog.species)    # 输出: 犬科动物（也可以通过类名访问）

# 实例属性：每只狗的名字和年龄不同
print(dog1.name)      # 输出: 旺财
print(dog2.name)      # 输出: 来福
```

### 2.2 修改类属性 vs 实例属性

```python
# 修改类属性 —— 影响所有实例
Dog.species = "家犬"
print(dog1.species)   # 输出: 家犬
print(dog2.species)   # 输出: 家犬

# 修改实例属性 —— 只影响当前实例
dog1.name = "富贵"
print(dog1.name)      # 输出: 富贵
print(dog2.name)      # 输出: 来福（不受影响）
```

> **注意**：如果某个实例对类属性进行了赋值，Python 会为该实例**新建一个同名实例属性**，从此这个实例就用自己的了：

```python
dog1.species = "猫科动物"  # 这不会改变 Dog.species
print(dog1.species)   # 输出: 猫科动物（dog1 自己的）
print(dog2.species)   # 输出: 家犬（还是类属性的值）
print(Dog.species)    # 输出: 家犬（类属性没变）
```

### 2.3 什么时候用类属性？

```python
class Circle:
    """圆类"""

    # PI 是不变的常量，所有圆共享 → 用类属性
    PI = 3.14159

    # 用类属性统计创建了多少个圆
    count = 0

    def __init__(self, radius):
        self.radius = radius            # 实例属性：每个圆半径不同
        Circle.count += 1               # 每创建一个圆，计数器 +1

    def area(self):
        return Circle.PI * self.radius ** 2


c1 = Circle(5)
c2 = Circle(10)
print(f"已创建 {Circle.count} 个圆")   # 输出: 已创建 2 个圆
print(f"圆1面积: {c1.area()}")          # 输出: 圆1面积: 78.53975
```

**使用原则**：
- 所有对象共享的常量/计数器 → **类属性**
- 每个对象各自不同的数据 → **实例属性**

---

## 三、类方法 @classmethod

类方法的第一个参数是 `cls`（代表类本身），可以通过它访问和修改类属性。

```python
class Student:
    school = "Python学院"
    students = []   # 用列表存储所有学生

    def __init__(self, name, score):
        self.name = name
        self.score = score
        Student.students.append(self)

    def __str__(self):
        return f"{self.name}（{self.score}分）"

    @classmethod
    def change_school(cls, new_name):
        """修改学校名称 —— 用 cls 操作类属性"""
        cls.school = new_name

    @classmethod
    def get_top_student(cls):
        """获取最高分学生 —— 用 cls 访问类属性"""
        if not cls.students:
            return None
        return max(cls.students, key=lambda s: s.score)

    @classmethod
    def from_string(cls, info_str):
        """从字符串创建学生对象 —— 这是一个'工厂方法'"""
        name, score = info_str.split(",")
        return cls(name.strip(), int(score.strip()))


# 不需要创建对象，直接用类名调用类方法
Student.change_school("大数据学院")
print(Student.school)   # 输出: 大数据学院

# 工厂方法：提供多种创建对象的方式
s1 = Student("小明", 95)
s2 = Student("小红", 88)
s3 = Student.from_string("小刚, 92")   # 从字符串创建

print(f"全校最高分: {Student.get_top_student()}")   # 输出: 全校最高分: 小明（95分）
```

> **@classmethod 最常见的用途**：
> 1. **工厂方法**：提供多种方式创建对象（如从字符串、字典、文件行创建）
> 2. **操作类属性**：修改或查询所有对象共享的数据

---

## 四、静态方法 @staticmethod

静态方法**既不需要 `self`，也不需要 `cls`**。它就像一个"寄居"在类里面的普通函数，和类本身没有直接关系。

```python
class MathHelper:
    """数学工具类 —— 所有方法都是静态方法"""

    @staticmethod
    def is_prime(n):
        """判断一个数是否为质数"""
        if n < 2:
            return False
        for i in range(2, int(n ** 0.5) + 1):
            if n % i == 0:
                return False
        return True

    @staticmethod
    def factorial(n):
        """计算阶乘 n!"""
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result


# 调用静态方法 —— 不需要创建对象
print(MathHelper.is_prime(7))    # 输出: True
print(MathHelper.is_prime(10))   # 输出: False
print(MathHelper.factorial(5))   # 输出: 120

# 也可以通过对象调用（但不推荐，没意义）
helper = MathHelper()
print(helper.is_prime(11))       # 输出: True
```

### @classmethod vs @staticmethod vs 普通方法

| 类型 | 参数 | 调用方式 | 用途 |
|------|------|---------|------|
| **普通方法** | `self`（实例） | 对象调用 | 操作实例数据 |
| **@classmethod** | `cls`（类） | 类名或对象调用 | 操作类数据 / 工厂方法 |
| **@staticmethod** | 无 | 类名或对象调用 | 工具函数，与类无关 |

```python
class Demo:
    count = 0                       # 类属性

    def __init__(self):
        Demo.count += 1
        self.id = Demo.count        # 实例属性

    def normal_method(self):
        """普通方法：需要 self，操作实例数据"""
        return f"我是第 {self.id} 个对象"

    @classmethod
    def class_method(cls):
        """类方法：需要 cls，操作类数据"""
        return f"总共创建了 {cls.count} 个对象"

    @staticmethod
    def static_method():
        """静态方法：不需要 self 也不需要 cls"""
        return "我是一个工具函数，和类的数据无关"
```

---

## 五、@property 属性装饰器

`@property` 让你**像访问属性一样调用方法**，常用于在获取或设置值时加入验证逻辑。

### 5.1 没有 @property 的问题

```python
class Product:
    def __init__(self, price):
        self.price = price

    def get_price(self):
        return self.price

    def set_price(self, value):
        if value < 0:
            raise ValueError("价格不能为负数")
        self.price = value


p = Product(100)
print(p.get_price())   # 要写 get_price()，不够直观
p.set_price(150)       # 要写 set_price()，不像操作属性
```

### 5.2 用 @property 优雅解决

```python
class Product:
    def __init__(self, price):
        self.price = price  # 这里会调用下面的 setter

    @property
    def price(self):
        """获取价格 —— 像属性一样访问"""
        return self._price

    @price.setter
    def price(self, value):
        """设置价格 —— 自动验证"""
        if value < 0:
            raise ValueError("价格不能为负数！")
        if value > 100000:
            print("⚠️ 警告：价格超过10万，请确认")
        self._price = value

    @price.deleter
    def price(self):
        """删除价格"""
        print("价格已清除")
        del self._price


# 使用起来就像操作普通属性一样！
p = Product(99.9)        # 自动调用 setter
print(p.price)           # 自动调用 getter，输出: 99.9

p.price = 199.9          # 自动调用 setter，简洁直观
print(p.price)           # 输出: 199.9

# p.price = -50          # 自动验证，报错: ValueError

del p.price              # 自动调用 deleter
```

### 5.3 只读属性（只有 getter，没有 setter）

```python
class Person:
    def __init__(self, birth_year):
        self._birth_year = birth_year

    @property
    def age(self):
        """年龄是只读的，无法通过赋值修改"""
        import datetime
        return datetime.datetime.now().year - self._birth_year


p = Person(2000)
print(p.age)      # 输出: 26（根据当前年份）
# p.age = 30      # 报错！没有 setter，无法赋值
```

> **@property 的核心价值**：在保持简洁的属性访问语法的同时，加入**数据验证**和**计算逻辑**。

---

## 六、私有属性与封装

### 6.1 命名约定：_（单下划线）和 __（双下划线）

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner          # 公有属性：谁都能访问
        self._balance = balance     # 保护属性：约定不直接访问（但技术上可以）
        self.__pin = "123456"       # 私有属性：Python 会做名称改写

    def get_balance(self):
        """提供公开方法来访问私有数据"""
        return self._balance


account = BankAccount("小明", 1000)

print(account.owner)         # 公有 → 正常访问: 小明
print(account._balance)      # 保护 → 技术上可以，但不推荐
# print(account.__pin)      # 私有 → 直接访问报错！AttributeError

# 但实际上 Python 只是把 __pin 改名为 _BankAccount__pin（名称改写）
print(account._BankAccount__pin)  # 能访问，但不推荐这样做！
```

### 6.2 为什么要封装？

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.__score = score     # 私有：分数不能被随意篡改

    @property
    def score(self):
        """只读属性：外部只能查看分数"""
        return self.__score

    def set_score(self, new_score, teacher_password):
        """修改分数需要密码验证"""
        if teacher_password != "teacher123":
            raise PermissionError("只有老师才能修改分数！")
        if not (0 <= new_score <= 100):
            raise ValueError("分数必须在0-100之间")
        self.__score = new_score
        print(f"{self.name} 的分数已更新为 {new_score}")


s = Student("小明", 85)
print(s.score)          # 正常查看: 85
# s.score = 100         # 只读，无法直接修改

s.set_score(95, "teacher123")  # 通过方法修改，有权限验证
s.set_score(95, "wrong")       # 报错: PermissionError
```

**封装原则**：
- `_`（单下划线）：意思是"请不要直接用"，但不会阻止你
- `__`（双下划线）：意思是"不应该直接访问"，Python 会做名称改写来增加难度
- 配合 `@property` 和方法来提供**受控的访问方式**

---

## 七、魔术方法（Dunder Methods）

"dunder" = **double underscore**（双下划线）。这些以 `__` 开头和结尾的方法，让自定义类能支持 Python 的内置操作。

### 7.1 最常用的魔术方法

```python
class Vector:
    """二维向量类 —— 演示魔术方法的威力"""

    def __init__(self, x, y):
        self.x = x
        self.y = y

    # __str__：print(obj) 时调用
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

    # __repr__：在交互环境或调试时显示
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    # __len__：len(obj) 时调用
    def __len__(self):
        return int((self.x ** 2 + self.y ** 2) ** 0.5)

    # __add__：obj1 + obj2 时调用
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    # __sub__：obj1 - obj2 时调用
    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    # __eq__：obj1 == obj2 时调用
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    # __mul__：obj * number 时调用
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

    # __bool__：if obj 时调用
    def __bool__(self):
        return self.x != 0 or self.y != 0


# 演示各种魔术方法的效果
v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(v1)                # __str__ → Vector(3, 4)
print(len(v1))           # __len__ → 5（向量长度）
print(v1 + v2)           # __add__ → Vector(4, 6)
print(v1 - v2)           # __sub__ → Vector(2, 2)
print(v1 * 3)            # __mul__ → Vector(9, 12)
print(v1 == v2)          # __eq__ → False
print(bool(v1))          # __bool__ → True

# 组合使用
v3 = v1 + v2
v3_scaled = v3 * 2
print(f"缩放后的向量: {v3_scaled}")   # Vector(8, 12)
```

### 7.2 魔术方法速查表

| 魔术方法 | 触发时机 | 例子 |
|---------|---------|------|
| `__init__(self, ...)` | 创建对象 | `obj = MyClass()` |
| `__str__(self)` | `print(obj)` | 友好显示 |
| `__repr__(self)` | 交互环境/调试 | 精确显示 |
| `__len__(self)` | `len(obj)` | `len("hello")` → 5 |
| `__add__(self, other)` | `obj + other` | 字符串拼接、数字加法 |
| `__sub__(self, other)` | `obj - other` | |
| `__mul__(self, other)` | `obj * other` | |
| `__eq__(self, other)` | `obj == other` | |
| `__lt__(self, other)` | `obj < other` | |
| `__bool__(self)` | `if obj` | |
| `__getitem__(self, key)` | `obj[key]` | |
| `__setitem__(self, key, val)` | `obj[key] = val` | |
| `__contains__(self, item)` | `item in obj` | |
| `__call__(self, ...)` | `obj()` | 让对象"可调用" |

### 7.3 让对象支持 `in` 和 `[key]` 操作

```python
class Team:
    """团队类 —— 演示 __getitem__ 和 __contains__"""

    def __init__(self, name):
        self.name = name
        self._members = []

    def add(self, member):
        self._members.append(member)

    def __len__(self):
        return len(self._members)

    def __contains__(self, member):
        """支持: member in team"""
        return member in self._members

    def __getitem__(self, index):
        """支持: team[0]、team[-1]、for m in team"""
        return self._members[index]


t = Team("研发组")
t.add("小明")
t.add("小红")
t.add("小刚")

print(len(t))          # __len__ → 3
print("小明" in t)     # __contains__ → True
print(t[0])            # __getitem__ → 小明
print(t[-1])           # __getitem__ → 小刚

# __getitem__ 还让对象支持 for 循环！
for member in t:
    print(f"- {member}")
# 输出:
# - 小明
# - 小红
# - 小刚
```

---

## 八、综合实战：智能商品库存系统

把今天学的所有知识用起来：

```python
class Product:
    """商品类 —— 使用 @property、私有属性、魔术方法"""

    count = 0  # 类属性：记录商品总数

    def __init__(self, name, price, stock=0):
        self.name = name
        self._price = 0       # 先初始化，再通过 setter 赋值
        self._stock = 0
        self.price = price     # 走 setter 验证
        self.stock = stock     # 走 setter 验证
        Product.count += 1

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value <= 0:
            raise ValueError("价格必须大于0")
        self._price = value

    @property
    def stock(self):
        return self._stock

    @stock.setter
    def stock(self, value):
        if value < 0:
            raise ValueError("库存不能为负数")
        self._stock = value

    @property
    def total_value(self):
        """计算属性：库存总价值"""
        return self._price * self._stock

    def __str__(self):
        return f"{self.name}（¥{self._price}，库存{self._stock}）"

    def __eq__(self, other):
        return self.name == other.name and self._price == other._price

    def __gt__(self, other):
        """支持: product1 > product2（按价格比较）"""
        return self._price > other._price

    @classmethod
    def from_dict(cls, data):
        """工厂方法：从字典创建商品"""
        return cls(
            name=data["name"],
            price=data["price"],
            stock=data.get("stock", 0)
        )

    @staticmethod
    def format_price(price):
        """工具方法：格式化价格显示"""
        if price >= 10000:
            return f"¥{price / 10000:.1f}万"
        return f"¥{price:.2f}"


# === 使用示例 ===

# 工厂方法创建
p1 = Product.from_dict({"name": "MacBook Pro", "price": 14999, "stock": 10})
p2 = Product.from_dict({"name": "iPad Air", "price": 4799, "stock": 25})
p3 = Product("AirPods", 1299, 50)

# 查看信息
print(p1)                      # MacBook Pro（¥14999，库存10）
print(f"库存总价值: {p1.total_value}")  # 149990

# 使用魔术方法比较
print(p1 > p2)                 # True（MacBook 比 iPad 贵）

# 类属性统计
print(f"商品总数: {Product.count}")  # 3

# 格式化工具
print(Product.format_price(14999))   # ¥1.5万
```

---

## 九、练习题

### 练习1：温度转换器

使用 `@property` 实现一个 Temperature 类，内部用摄氏度存储，但可以像属性一样获取华氏度：

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def fahrenheit(self):
        """摄氏 → 华氏: F = C × 1.8 + 32"""
        # 请补充代码

    @fahrenheit.setter
    def fahrenheit(self, value):
        """华氏 → 摄氏: C = (F - 32) / 1.8"""
        # 请补充代码

# 测试
t = Temperature(100)
print(t.fahrenheit)   # 应输出 212.0
t.fahrenheit = 32
print(t.celsius)       # 应输出 0.0
```

### 练习2：分数类 Fraction

实现一个 Fraction 类，支持加减乘除和比较：

```python
class Fraction:
    def __init__(self, numerator, denominator):
        # 请补充代码

    def __str__(self):
        return f"{self.numerator}/{self.denominator}"

    def __add__(self, other):
        """实现分数加法"""
        # 请补充代码

    def __eq__(self, other):
        """判断两个分数是否相等"""
        # 请补充代码

# 测试
f1 = Fraction(1, 2)
f2 = Fraction(1, 3)
print(f1 + f2)        # 应输出 5/6
print(f1 == Fraction(2, 4))   # 应输出 True
```

### 练习3：图书管理系统

创建 `Library` 类，使用类方法统计、静态方法格式化、属性控制：

```python
class Library:
    """图书管理类"""
    # 要求：
    # 1. 用类属性 total_books 统计所有图书馆的总藏书量
    # 2. @classmethod add_library() 工厂方法
    # 3. @property books_count 返回当前图书馆的藏书数
    # 4. @staticmethod format_isbn() 格式化ISBN号
    # 5. 实现 __len__、__contains__ 魔术方法
```

---

## 十、常见问题

**Q1：`@property` 和直接定义一个方法有什么区别？**

```python
# 方法方式（不推荐）
product.get_price()

# @property 方式（推荐）
product.price    # 更简洁，像操作属性一样
```

对使用者来说，`@property` 让代码更简洁。更重要的好处是：如果以后需要在获取价格时加入额外逻辑（如日志、缓存），使用方**完全不需要改代码**。

**Q2：Python 的"私有"属性真的私有吗？**

不是。Python 的 `__name` 只是做了**名称改写**（变为 `_ClassName__name`），不是真正的访问控制。Python 信奉"我们是成年人"，用约定而不是强制来保护数据。如果真的需要严格私有，可以考虑使用 `__slots__` 或第三方库。

**Q3：什么时候用 `@classmethod`，什么时候用 `@staticmethod`？**

- 如果需要访问/修改**类属性**或**创建对象** → `@classmethod`
- 如果只是一个**工具函数**，恰好和这个类相关 → `@staticmethod`
- 如果需要操作**实例数据** → 普通方法

**Q4：需要记住所有魔术方法吗？**

不需要！最常用的几个先记住：

| 优先级 | 魔术方法 | 说明 |
|--------|---------|------|
| ⭐⭐⭐ | `__init__` | 初始化，天天用 |
| ⭐⭐⭐ | `__str__` | print 时友好显示 |
| ⭐⭐ | `__repr__` | 调试时精确显示 |
| ⭐⭐ | `__len__` | 支持 len() |
| ⭐⭐ | `__eq__` | 支持 == |
| ⭐ | 其他 | 需要时再查 |

---

## 十一、今日总结

```
┌─────────────────────────────────────────────────┐
│              Day 17 知识地图                      │
├─────────────────────────────────────────────────┤
│                                                  │
│   类属性 ── 共享数据（常量/计数器）                │
│   实例属性 ── 独立数据（每个对象不同）              │
│                                                  │
│   @classmethod ── 工厂方法 / 操作类属性           │
│   @staticmethod ── 工具函数                        │
│   @property ── 像属性一样访问方法                  │
│                                                  │
│   _  保护属性（约定）                             │
│   __ 私有属性（名称改写）                         │
│                                                  │
│   魔术方法 ── 让自定义类支持 +, len, print 等     │
│                                                  │
└─────────────────────────────────────────────────┘
```

**今天掌握的技能让你能写出：**
- 更安全的数据保护代码（封装）
- 更优雅的 API（@property）
- 更灵活的对象创建（@classmethod 工厂方法）
- 更"Pythonic"的自定义类（魔术方法）

---

## 十二、免费学习资源

- [廖雪峰 Python 教程 - 面向对象高级编程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017496095445440) — 详尽的 @property 和多重继承讲解
- [菜鸟教程 Python 面向对象](https://www.runoob.com/python3/python3-class.html) — 基础概念和实例
- [Python 官方文档 - data model](https://docs.python.org/zh-cn/3/reference/datamodel.html) — 所有魔术方法的权威参考

---

> **Day 18 预告**：模块与包 —— 学会组织代码，把相关功能打包成模块，像乐高积木一样灵活组合。

> 💡 **学习小贴士**：今天的内容比较抽象，建议把综合实战的"智能商品库存系统"自己敲一遍。如果你能独立写出来，说明今天的知识点已经掌握了！
