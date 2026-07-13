# Python Day 16：继承与多态

> **进度：Day 16/60 | 阶段：第二阶段 · 面向对象进阶 | 预计用时：1~1.5 小时**

---

## 一、什么是继承？（生活类比）

Day 15 你学会了创建类，今天学**让一个类"继承"另一个类的全部能力**。

```
生活类比：
  "车" 是一个类，有 品牌、颜色、启动() 这些属性和方法

  "汽车" 继承 "车" → 自动拥有 品牌、颜色、启动()
  同时可以加自己独有的：燃料类型、加油()

  "电动车" 继承 "车" → 自动拥有 品牌、颜色、启动()
  同时可以加自己独有的：电池容量、充电()
```

> **核心思想**：不重复造轮子。父类写好的东西，子类直接拿来用，只需关注自己独特的部分。

### 继承的三个好处

| 好处 | 说明 | 例子 |
|------|------|------|
| **代码复用** | 子类直接拥有父类的属性和方法 | `ElectricCar` 不用重写 `start()` |
| **统一接口** | 所有子类都遵循父类的"契约" | 所有车都有 `start()` 方法 |
| **扩展性** | 新增子类不影响已有代码 | 加一个 `HybridCar` 无需改动 `Car` |

---

## 二、继承的基本语法

### 2.1 最简单的继承

```python
# 父类（基类）
class Animal:
    """动物类"""

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print(f"{self.name} 正在吃东西...")

    def sleep(self):
        print(f"{self.name} 正在睡觉...")

    def __str__(self):
        return f"{self.name}（{self.age}岁）"


# 子类 — 在类名后面的括号里写父类名
class Dog(Animal):
    """狗类，继承 Animal"""

    def bark(self):
        """狗独有的方法"""
        print(f"{self.name}：汪汪汪！")


class Cat(Animal):
    """猫类，继承 Animal"""

    def meow(self):
        """猫独有的方法"""
        print(f"{self.name}：喵~")


# 使用
dog = Dog("旺财", 3)
cat = Cat("小花", 2)

# ✅ 子类继承了父类的所有方法
dog.eat()      # 旺财 正在吃东西...  （来自 Animal）
dog.sleep()    # 旺财 正在睡觉...    （来自 Animal）
dog.bark()     # 旺财：汪汪汪！      （Dog 自己的）

cat.eat()      # 小花 正在吃东西...  （来自 Animal）
cat.meow()     # 小花：喵~           （Cat 自己的）

# ✅ 子类继承了父类的 __init__ 属性
print(dog.name)   # 旺财
print(cat.age)    # 2
```

> **关键**：`class Dog(Animal)` 括号里的 `Animal` 就是父类。子类自动拥有父类的全部属性和方法。

### 2.2 用 `issubclass()` 和 `isinstance()` 检查继承关系

```python
print(issubclass(Dog, Animal))   # True  —— Dog 是 Animal 的子类
print(issubclass(Cat, Animal))   # True  —— Cat 是 Animal 的子类
print(issubclass(Dog, Cat))      # False —— Dog 不是 Cat 的子类

print(isinstance(dog, Dog))      # True  —— dog 是 Dog 的对象
print(isinstance(dog, Animal))   # True  —— dog 也是 Animal 的对象（因为继承）
print(isinstance(dog, Cat))      # False
```

---

## 三、方法重写（Override）

子类可以**重写**（覆盖）父类的方法，实现不同的行为。

### 3.1 基本重写

```python
class Animal:
    def speak(self):
        print("...（动物发出声音）")


class Dog(Animal):
    def speak(self):
        # 完全覆盖父类的 speak()
        print("汪汪汪！")


class Cat(Animal):
    def speak(self):
        print("喵~")


class Duck(Animal):
    def speak(self):
        print("嘎嘎嘎！")


# 同一个方法，不同对象有不同的表现
animals = [Dog("旺财", 3), Cat("小花", 2), Duck("唐老鸭", 5)]

for animal in animals:
    print(f"{animal.name} 说:", end=" ")
    animal.speak()

# 旺财 说: 汪汪汪！
# 小花 说: 喵~
# 唐老鸭 说: 嘎嘎嘎！
```

> **这就是多态！** 同一个方法名 `speak()`，不同对象调用时表现出不同行为。

### 3.2 重写 `__init__` + `super()`

当子类需要额外的属性时，要重写 `__init__`，但**不要忘记调用父类的 `__init__`**。

```python
class Animal:
    def __init__(self, name, age):
        self.name = name
        self.age = age


class Dog(Animal):
    def __init__(self, name, age, breed):
        # ⭐ super().__init__() 调用父类的初始化，设置 name 和 age
        super().__init__(name, age)
        # 然后设置子类独有的属性
        self.breed = breed   # 品种
        self.tricks = []     # 会学的技能

    def learn_trick(self, trick):
        self.tricks.append(trick)
        print(f"{self.name} 学会了: {trick}")

    def show_tricks(self):
        if self.tricks:
            print(f"{self.name} 的技能: {', '.join(self.tricks)}")
        else:
            print(f"{self.name} 还没学任何技能")

    def __str__(self):
        return f"{self.name}（{self.breed}，{self.age}岁）"


dog = Dog("旺财", 3, "柴犬")
dog.learn_trick("握手")
dog.learn_trick("装死")
dog.show_tricks()
# 旺财 的技能: 握手, 装死
```

### `super()` 是什么？

```
super().__init__(name, age)

翻译成人话：
"调用我父类的 __init__ 方法，帮我把 name 和 age 设置好"

等价写法（不推荐）：
Animal.__init__(self, name, age)   # 硬编码父类名，不好

推荐写法（推荐）：
super().__init__(name, age)         # 自动找到父类，改父类名也不怕
```

> **记忆口诀**：子类重写 `__init__` 时，第一行写 `super().__init__(...)` 把父类的初始化做了，再写自己的。

---

## 四、多态详解

### 4.1 什么是多态？

**多态 = 同一个接口，不同的实现。**

你不需要关心对象具体是什么类型，只要它有某个方法，就直接调用。Python 的多态天然存在——鸭子类型（Duck Typing）。

```python
class PDFReport:
    """PDF 格式报告"""
    def generate(self, data):
        print(f"[PDF] 生成报告: {len(data)} 条数据")
        # 真实场景：用 reportlab 生成 PDF

class ExcelReport:
    """Excel 格式报告"""
    def generate(self, data):
        print(f"[Excel] 生成报告: {len(data)} 条数据")
        # 真实场景：用 openpyxl 生成 Excel

class HTMLReport:
    """HTML 格式报告"""
    def generate(self, data):
        print(f"[HTML] 生成报告: {len(data)} 条数据")
        # 真实场景：生成 HTML 页面


# 报告生成器 —— 不关心具体是什么报告，只要有 generate() 方法就行
class ReportGenerator:
    def __init__(self):
        self.reports = []

    def add_report(self, report):
        self.reports.append(report)

    def generate_all(self, data):
        """遍历所有报告并生成"""
        for report in self.reports:
            report.generate(data)


# 使用
data = [{"id": 1, "name": "张三"}, {"id": 2, "name": "李四"}]

gen = ReportGenerator()
gen.add_report(PDFReport())
gen.add_report(ExcelReport())
gen.add_report(HTMLReport())

gen.generate_all(data)
# [PDF] 生成报告: 2 条数据
# [Excel] 生成报告: 2 条数据
# [HTML] 生成报告: 2 条数据
```

> **多态的核心**：`report.generate(data)` 这行代码不需要知道 `report` 是 PDF、Excel 还是 HTML——它只管调用 `generate()`。这就是"面向接口编程"。

### 4.2 Python 的鸭子类型

```
"如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子。"
```

Python 不要求类之间有继承关系，只要有相同的方法名就能多态：

```python
# 完全没有继承关系！但都有 process() 方法

class CSVProcessor:
    def process(self, filepath):
        print(f"处理 CSV 文件: {filepath}")

class JSONProcessor:
    def process(self, filepath):
        print(f"处理 JSON 文件: {filepath}")

class XMLProcessor:
    def process(self, filepath):
        print(f"处理 XML 文件: {filepath}")


# 同一个函数，传入不同对象
def process_file(processor, filepath):
    processor.process(filepath)   # 不关心 processor 是什么类型


# 全都能用！
process_file(CSVProcessor(), "data.csv")
process_file(JSONProcessor(), "data.json")
process_file(XMLProcessor(), "data.xml")
```

> **对比 Java/C++**：Java/C++ 需要定义接口或父类才能多态；Python 只要有相同方法名就够了。更灵活，但也更需要自律。

---

## 五、多重继承与 MRO（了解即可）

Python 支持一个类继承多个父类，但这块容易出问题，初学阶段了解即可。

### 5.1 基本语法

```python
class A:
    def show(self):
        print("A 的 show")


class B:
    def show(self):
        print("B 的 show")


class C(A, B):   # C 同时继承 A 和 B（A 在前面，优先级更高）
    pass


c = C()
c.show()        # A 的 show（因为 A 写在前面）

# 查看方法查找顺序（MRO）
print(C.__mro__)
# (<class 'C'>, <class 'A'>, <class 'B'>, <class 'object'>)
```

### 5.2 什么时候用多重继承？

| 场景 | 推荐做法 | 说明 |
|------|---------|------|
| 真正的"是A也是B" | ✅ 可以用 | 如 `FlyingCar(Car, Airplane)` |
| 只是为了混入功能 | ✅ 用组合替代 | 把功能做成独立类，在 `__init__` 中组合 |
| 模棱两可的关系 | ❌ 避免使用 | 会导致 MRO 复杂，难以调试 |

> **实际建议**：90% 的场景用**单继承 + 组合**就够了。多重继承是高级话题，遇到真正需要时再学不迟。

---

## 六、实战练习：用继承重构 AMC 资产类体系

Day 15 写了一个 `AssetRecord` 类，所有资产都塞在一起。现在用继承拆分成不同资产类型：

```python
# ==================== 父类 ====================
class AssetRecord:
    """资产记录基类"""

    _count = 0   # 类属性：计数器

    def __init__(self, debtor_name, principal, overdue_days):
        AssetRecord._count += 1
        self.id = f"NPL-{AssetRecord._count:04d}"
        self.debtor_name = debtor_name
        self.principal = principal        # 本金（万元）
        self.overdue_days = overdue_days  # 逾期天数
        self.settled = False
        self.recovery_amount = 0

    @property
    def risk_level(self):
        if self.overdue_days <= 90:
            return "低风险"
        elif self.overdue_days <= 365:
            return "中风险"
        elif self.overdue_days <= 730:
            return "高风险"
        else:
            return "严重风险"

    def settle(self, recovery_amount):
        self.settled = True
        self.recovery_amount = recovery_amount
        rate = recovery_amount / self.principal * 100
        print(f"资产 {self.id} 已处置，回收率: {rate:.1f}%")

    def calc_risk_score(self):
        """计算风险评分（子类重写此方法）"""
        return 0

    def __str__(self):
        status = "✅ 已处置" if self.settled else "🔴 未处置"
        return (
            f"[{self.id}] {self.debtor_name} | "
            f"本金: ¥{self.principal:.2f}万 | "
            f"逾期: {self.overdue_days}天 | "
            f"风险: {self.risk_level} | {status}"
        )


# ==================== 子类1：贷款类资产 ====================
class LoanAsset(AssetRecord):
    """贷款类不良资产"""

    def __init__(self, debtor_name, principal, overdue_days, loan_type="信用贷款"):
        super().__init__(debtor_name, principal, overdue_days)
        self.loan_type = loan_type    # 贷款类型
        self.guarantee = None         # 担保方式

    def set_guarantee(self, method):
        """设置担保方式"""
        self.guarantee = method
        print(f"资产 {self.id} 担保方式: {method}")

    def calc_risk_score(self):
        """贷款类风险评估：逾期天数 + 担保情况"""
        base_score = self.overdue_days / 10
        if self.guarantee and "抵押" in self.guarantee:
            base_score *= 0.7   # 有抵押担保，风险降低
        return round(base_score, 1)

    def __str__(self):
        base = super().__str__()
        return f"{base} | 类型: {self.loan_type}"


# ==================== 子类2：信用卡类资产 ====================
class CreditCardAsset(AssetRecord):
    """信用卡类不良资产"""

    def __init__(self, debtor_name, principal, overdue_days, credit_limit=0):
        super().__init__(debtor_name, principal, overdue_days)
        self.credit_limit = credit_limit  # 授信额度
        self.over_limit = principal > credit_limit  # 是否超限

    def calc_risk_score(self):
        """信用卡风险评估：逾期天数 + 超限情况"""
        base_score = self.overdue_days / 10
        if self.over_limit:
            base_score *= 1.3   # 超限加重风险
        return round(base_score, 1)

    def __str__(self):
        base = super().__str__()
        status = "⚠️ 超限" if self.over_limit else "✓ 正常"
        return f"{base} | 额度: ¥{self.credit_limit}万 | {status}"


# ==================== 子类3：抵债资产 ====================
class CollateralAsset(AssetRecord):
    """抵债资产（以物抵债）"""

    def __init__(self, debtor_name, principal, overdue_days, asset_desc, estimate_value):
        super().__init__(debtor_name, principal, overdue_days)
        self.asset_desc = asset_desc      # 抵债资产描述
        self.estimate_value = estimate_value  # 评估价值

    @property
    def discount_rate(self):
        """折扣率 = 评估价 / 本金"""
        return self.estimate_value / self.principal

    def calc_risk_score(self):
        """抵债资产风险：看折扣率"""
        if self.discount_rate >= 0.8:
            return 5.0   # 低风险
        elif self.discount_rate >= 0.5:
            return 15.0  # 中风险
        else:
            return 30.0  # 高风险

    def __str__(self):
        base = super().__str__()
        return f"{base} | 抵债物: {self.asset_desc} | 评估: ¥{self.estimate_value}万"


# ==================== 使用 ====================
# 创建不同类型的资产
assets = [
    LoanAsset("XX贸易有限公司", 500, 120, "抵押贷款"),
    LoanAsset("YY实业有限公司", 300, 45, "信用贷款"),
    CreditCardAsset("王某", 15, 200, 10),
    CreditCardAsset("李某", 8, 90, 10),
    CollateralAsset("ZZ房地产", 2000, 800, "杭州市房产一套", 1500),
]

# 设置担保方式
assets[0].set_guarantee("房产抵押")

# 统一操作：多态的体现
print("=" * 60)
print("  AMC 不良资产风险报告")
print("=" * 60)

total_principal = 0
for asset in assets:
    print(f"\n{asset}")
    score = asset.calc_risk_score()   # 同一个方法名，不同实现
    print(f"  风险评分: {score}")
    total_principal += asset.principal

print(f"\n{'='*60}")
print(f"  资产总数: {len(assets)} 笔")
print(f"  本金总额: ¥{total_principal:.2f}万")
print(f"{'='*60}")
```

### 运行效果

```
资产 NPL-0001 担保方式: 房产抵押

============================================================
  AMC 不良资产风险报告
============================================================

[NPL-0001] XX贸易有限公司 | 本金: ¥500.00万 | 逾期: 120天 | 风险: 中风险 | 🔴 未处置 | 类型: 抵押贷款
  风险评分: 8.4

[NPL-0002] YY实业有限公司 | 本金: ¥300.00万 | 逾期: 45天 | 风险: 低风险 | 🔴 未处置 | 类型: 信用贷款
  风险评分: 4.5

[NPL-0003] 王某 | 本金: ¥15.00万 | 逾期: 200天 | 风险: 中风险 | 🔴 未处置 | 额度: ¥10万 | ⚠️ 超限
  风险评分: 26.0

[NPL-0004] 李某 | 本金: ¥8.00万 | 逾期: 90天 | 风险: 低风险 | 🔴 未处置 | 额度: ¥10万 | ✓ 正常
  风险评分: 9.0

[NPL-0005] ZZ房地产 | 本金: ¥2000.00万 | 逾期: 800天 | 风险: 严重风险 | 🔴 未处置 | 抵债物: 杭州市房产一套 | 评估: ¥1500万
  风险评分: 30.0

============================================================
  资产总数: 5 笔
  本金总额: ¥2823.00万
============================================================
```

> **注意 `super().__str__()` 的用法**：子类的 `__str__` 先调用 `super().__str__()` 获取父类的字符串，再追加自己的信息，避免重复写代码。

---

## 七、`type()`、`isinstance()`、`issubclass()` 速查

这三个函数用于判断类型和继承关系：

```python
class Animal:
    pass

class Dog(Animal):
    pass

d = Dog()

print(type(d))                  # <class '__main__.Dog'>
print(type(d).__name__)         # Dog

print(isinstance(d, Dog))       # True
print(isinstance(d, Animal))   # True（继承关系）
print(isinstance(d, object))    # True（所有类都继承 object）

print(issubclass(Dog, Animal))  # True
print(issubclass(Dog, object))  # True
```

| 函数 | 用途 | 示例 |
|------|------|------|
| `type(obj)` | 查看对象的确切类型 | `type(dog)` → `Dog` |
| `isinstance(obj, Class)` | 判断对象是否是某个类（或子类）的实例 | `isinstance(dog, Animal)` → `True` |
| `issubclass(A, B)` | 判断 A 是否是 B 的子类 | `issubclass(Dog, Animal)` → `True` |

> **最佳实践**：做类型判断时优先用 `isinstance()` 而不是 `type() ==`，因为 `isinstance` 能识别子类。

---

## 八、今日练习（3选2必做）

### 练习 1：员工薪资系统（推荐 ⭐⭐）

```python
"""
设计一个员工薪资系统：
- 父类 Employee：姓名、工号、基本工资
- 子类 FullTimeEmployee（全职）：基本工资 + 绩效奖金
- 子类 PartTimeEmployee（兼职）：按小时计费
- 子类 Manager（经理）：基本工资 + 管理津贴 + 部门

所有子类都要实现 calc_salary() 方法返回月薪。
"""
```

**参考步骤**：
1. 父类 `Employee` 定义 `__init__` 和抽象方法 `calc_salary()`
2. 子类各自实现 `calc_salary()`
3. 写一个 `Payroll` 类，接收员工列表，调用 `calc_salary()` 计算总薪资
4. 测试多态：不同类型员工放同一个列表

<details>
<summary>点击查看参考答案</summary>

```python
class Employee:
    """员工基类"""

    def __init__(self, name, emp_id, base_salary):
        self.name = name
        self.emp_id = emp_id
        self.base_salary = base_salary

    def calc_salary(self):
        """子类必须实现此方法"""
        raise NotImplementedError("子类必须实现 calc_salary()")

    def __str__(self):
        return f"[{self.emp_id}] {self.name} | 月薪: ¥{self.calc_salary():.2f}"


class FullTimeEmployee(Employee):
    """全职员工"""

    def __init__(self, name, emp_id, base_salary, bonus=0):
        super().__init__(name, emp_id, base_salary)
        self.bonus = bonus

    def calc_salary(self):
        return self.base_salary + self.bonus


class PartTimeEmployee(Employee):
    """兼职员工"""

    def __init__(self, name, emp_id, hourly_rate, hours_per_month):
        super().__init__(name, emp_id, 0)  # 兼职没有基本工资
        self.hourly_rate = hourly_rate
        self.hours_per_month = hours_per_month

    def calc_salary(self):
        return self.hourly_rate * self.hours_per_month


class Manager(Employee):
    """经理"""

    def __init__(self, name, emp_id, base_salary, allowance, department):
        super().__init__(name, emp_id, base_salary)
        self.allowance = allowance
        self.department = department

    def calc_salary(self):
        return self.base_salary + self.allowance

    def __str__(self):
        base = super().__str__()
        return f"{base} | 部门: {self.department}"


# 使用
employees = [
    FullTimeEmployee("张三", "E001", 8000, 2000),
    FullTimeEmployee("李四", "E002", 10000, 3000),
    PartTimeEmployee("王五", "P001", 80, 96),     # 80元/小时，96小时/月
    PartTimeEmployee("赵六", "P002", 60, 80),
    Manager("钱七", "M001", 15000, 5000, "数据部"),
]

# 多态：统一调用 calc_salary()
total = 0
print("========== 薪资明细 ==========")
for emp in employees:
    print(emp)
    total += emp.calc_salary()

print("=" * 30)
print(f"  总薪资支出: ¥{total:.2f}")
print(f"  员工人数: {len(employees)} 人")
```
</details>

### 练习 2：实现 `len()` 和 `+` 运算

给 `AssetRecord` 添加魔术方法：
- `__len__`：返回资产包中的资产数量（用列表存多笔资产时）
- `__add__`：两笔资产的本金相加
- `__gt__`：按本金比较大小

<details>
<summary>点击查看参考答案</summary>

```python
class SimpleAsset:
    def __init__(self, name, principal):
        self.name = name
        self.principal = principal

    def __add__(self, other):
        """两笔资产本金相加"""
        if isinstance(other, SimpleAsset):
            return SimpleAsset(
                f"{self.name}+{other.name}",
                self.principal + other.principal
            )
        return NotImplemented

    def __gt__(self, other):
        """按本金比较大小"""
        if isinstance(other, SimpleAsset):
            return self.principal > other.principal
        return NotImplemented

    def __str__(self):
        return f"{self.name}: ¥{self.principal}万"


a1 = SimpleAsset("资产A", 500)
a2 = SimpleAsset("资产B", 300)
a3 = SimpleAsset("资产C", 800)

print(a1 + a2)       # 资产A+资产B: ¥800万
print(a3 > a1)       # True（800 > 500）
print(sorted([a2, a3, a1]))  # 按本金升序排列
```
</details>

### 练习 3：思考题

```python
"""
以下代码有什么问题？如何修正？

class Animal:
    def __init__(self, name):
        self.name = name

class Bird(Animal):
    def __init__(self, name, wingspan):
        # 忘记调用 super().__init__()
        self.wingspan = wingspan

b = Bird("鹦鹉", 0.3)
print(b.name)  # ???  这里会怎样？
"""
```

<details>
<summary>点击查看分析</summary>

会报 `AttributeError: 'Bird' object has no attribute 'name'`！

因为 `Bird.__init__` 重写了父类的初始化，但没有调用 `super().__init__(name)`，
导致 `name` 属性从未被设置。

修正：
```python
class Bird(Animal):
    def __init__(self, name, wingspan):
        super().__init__(name)  # ⭐ 加上这行
        self.wingspan = wingspan
```
</details>

---

## 九、今日避坑指南

1. **子类重写 `__init__` 时别忘了 `super().__init__()`** — 否则父类的属性不会被初始化
2. **`super()` 只能在子类方法内部使用** — 不能在类外面单独调用
3. **`isinstance` vs `type()`** — 类型判断优先用 `isinstance`，它考虑继承关系
4. **避免多重继承** — 初学阶段坚持单继承 + 组合，减少心智负担
5. **方法重写不是"删掉重写"** — 可以先 `super().方法名()` 调用父类的实现，再追加自己的逻辑

---

## 十、今日知识脑图

```
┌───────────────────────────────────────────────────────┐
│                  继承与多态                               │
├───────────────┬───────────────┬─────────────────────────┤
│    继承基础     │    方法重写     │    多态                 │
├───────────────┼───────────────┼─────────────────────────┤
│ class 子(父)  │ • 覆盖父类方法  │ • 同一方法，不同实现      │
│ super().__init│ • super().方法()│ • isinstance 判断       │
│ isinstance()  │ • 扩展而非替换  │ • 鸭子类型（Duck Typing）│
│ issubclass()  │               │ • 面向接口编程           │
├───────────────┼───────────────┼─────────────────────────┤
│ 代码复用       │ 灵活扩展       │ 统一接口                 │
│ 不重复造轮子   │ 保留父类能力   │ 不关心具体类型           │
└───────────────┴───────────────┴─────────────────────────┘
```

---

## 十一、推荐资源

| 资源 | 说明 |
|------|------|
| [Python 继承官方教程](https://docs.python.org/zh-cn/3/tutorial/classes.html#inheritance) | 权威的继承讲解 |
| [廖雪峰 - 继承与多态](https://www.liaoxuefeng.com/wiki/1016959663602400/1017496039359296) | 中文讲解 + 实例 |
| [Python MRO 详解](https://pythoninternal.wordpress.com/2014/08/04/the-super-method/) | 进阶：方法解析顺序 |

---

## 十二、明日预告（Day 17）

明天继续面向对象进阶 —— **类方法、静态方法与封装**：

- `@classmethod` 类方法与 `@staticmethod` 静态方法
- `_` 和 `__` 开头的属性（封装与访问控制）
- `@property` 深入使用（ setter / deleter）
- Python 的"约定大于强制"哲学
- 用封装改造 AMC 资产类的属性保护

---

> **今日金句**：继承不是"复制粘贴"，而是"站在父类的肩膀上"。
> 好的继承设计让代码像搭积木一样，一层一层往上叠。
