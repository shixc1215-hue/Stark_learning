# Python Day 15：面向对象编程基础（类与对象）

> **进度：Day 15/60 | 阶段：第二阶段 · 面向对象 | 预计用时：1~1.5 小时**

---

## 一、什么是面向对象？（生活类比）

之前 14 天你一直在用"面向过程"的思维方式写代码——**按步骤来，一步一步执行**。

面向对象（OOP）换了一种思路：**把数据和操作数据的方法打包在一起，做成一个"模板"，然后从这个模板批量生产"产品"。**

```
面向过程（你之前的方式）：
  "把苹果洗干净 → 切苹果 → 把苹果放进榨汁机 → 打开开关 → 倒出果汁"

面向对象（今天开始学）：
  "有一个【榨汁机】对象，它有一个方法叫【榨汁】，
   你只需调用 榨汁机.榨汁(苹果)，它自己知道怎么做"
```

### 四个核心概念（先混个脸熟）

| 概念 | 一句话解释 | 生活类比 |
|------|-----------|---------|
| **类（Class）** | 创建对象的蓝图/模板 | 汽车设计图纸 |
| **对象（Object）** | 根据蓝图造出来的具体实例 | 一辆红色的特斯拉 |
| **属性（Attribute）** | 对象身上的数据特征 | 颜色=红色、速度=200 |
| **方法（Method）** | 对象能做的事 | 启动()、刹车()、加速() |

> **你其实已经用过类了！** Day 14 的 `Student` 就是类，`s = Student("001", "张三")` 就是创建对象。今天把这块彻底搞懂。

---

## 二、类的定义与使用

### 2.1 最简单的类

```python
# 定义一个类（类名用大驼峰：每个单词首字母大写）
class Dog:
    """这是一只狗"""

# 用类创建对象（实例化）
dog1 = Dog()   # 第一只狗
dog2 = Dog()   # 第二只狗

print(dog1)    # <__main__.Dog object at 0x...>
print(dog2)    # 地址不同，说明是两个独立对象
print(dog1 is dog2)  # False，不同的对象
```

> **类比**：`Dog` 是"狗"这个概念，`dog1` 和 `dog2` 是具体的两只狗（比如旺财和来福）。

### 2.2 添加属性和方法

```python
class Dog:
    """狗类"""

    def __init__(self, name, breed, age):
        """
        __init__ 是初始化方法（构造函数）
        创建对象时自动调用，用来设置初始属性
        """
        self.name = name      # 名字
        self.breed = breed    # 品种
        self.age = age        # 年龄
        self.energy = 100     # 精力值（默认100）

    def bark(self):
        """狗叫的方法"""
        print(f"{self.name}：汪汪汪！")

    def eat(self, food):
        """吃东西恢复精力"""
        self.energy = min(100, self.energy + 20)
        print(f"{self.name} 吃了 {food}，精力恢复到 {self.energy}")

    def run(self):
        """跑步消耗精力"""
        if self.energy >= 10:
            self.energy -= 10
            print(f"{self.name} 跑了一圈！剩余精力: {self.energy}")
        else:
            print(f"{self.name} 太累了，跑不动了...")

    def __str__(self):
        """定义 print(对象) 时显示的内容"""
        return f"🐾 {self.name}（{self.breed}，{self.age}岁，精力:{self.energy}）"


# 创建对象
wangcai = Dog("旺财", "柴犬", 3)
laifu = Dog("来福", "金毛", 2)

# 调用方法
wangcai.bark()        # 旺财：汪汪汪！
laifu.bark()          # 来福：汪汪汪！

# 属性是每个对象独立的
wangcai.run()         # 旺财跑了一圈！剩余精力: 90
laifu.run()           # 来福跑了一圈！剩余精力: 90
laifu.run()           # 来福跑了一圈！剩余精力: 80

print(wangcai)        # 🐾 旺财（柴犬，3岁，精力:90）
print(laifu)          # 🐾 来福（金毛，2岁，精力:80）
```

### 2.3 self 是什么？（新手最容易困惑的点）

```python
class Cat:
    def __init__(self, name):
        self.name = name    # self.name = 对象的属性

    def introduce(self):
        # self 指的是"调用这个方法的对象本身"
        print(f"我是 {self.name}")

c1 = Cat("小花")
c2 = Cat("小黑")

c1.introduce()   # 此时 self = c1，所以打印"我是 小花"
c2.introduce()   # 此时 self = c2，所以打印"我是 小黑"
```

> **记忆口诀**：`self` 就是"我自己"。`self.name` 就是"我自己的名字"。调用 `c1.introduce()` 时，Python 自动把 `c1` 作为 `self` 传进去。

---

## 三、类属性 vs 实例属性

这是 OOP 的重要区分：

```python
class BankAccount:
    """银行账户类"""

    # 类属性 —— 所有账户共享的数据（属于类，不属于某个具体对象）
    bank_name = "AMC银行"
    interest_rate = 0.03     # 所有账户利率一样

    def __init__(self, owner, balance=0):
        # 实例属性 —— 每个账户各自独立的数据
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        """存款"""
        if amount > 0:
            self.balance += amount
            print(f"{self.owner} 存入 ¥{amount}，余额: ¥{self.balance:.2f}")
        else:
            print("存款金额必须大于0！")

    def withdraw(self, amount):
        """取款"""
        if amount > self.balance:
            print(f"{self.owner} 余额不足！当前余额: ¥{self.balance:.2f}")
        else:
            self.balance -= amount
            print(f"{self.owner} 取出 ¥{amount}，余额: ¥{self.balance:.2f}")

    def show_info(self):
        """显示账户信息"""
        print(f"银行: {BankAccount.bank_name}")         # 通过类名访问类属性
        print(f"利率: {BankAccount.interest_rate * 100}%")
        print(f"户主: {self.owner}")
        print(f"余额: ¥{self.balance:.2f}")


# 创建两个不同账户
acc1 = BankAccount("张三", 1000)
acc2 = BankAccount("李四", 5000)

# 实例属性各自独立
acc1.deposit(500)     # 张三的余额变成 1500
acc2.withdraw(1000)   # 李四的余额变成 4000

# 类属性共享
print(f"银行名: {BankAccount.bank_name}")  # AMC银行

# 修改类属性（影响所有对象）
BankAccount.interest_rate = 0.035
print(f"新利率: {acc1.interest_rate * 100}%")  # 3.5%
print(f"新利率: {acc2.interest_rate * 100}%")  # 3.5%
```

### 对比总结

```
类属性（写在方法外面）          实例属性（写在 __init__ 里面）
─────────────────────         ─────────────────────
bank_name = "AMC银行"         self.owner = owner
interest_rate = 0.03          self.balance = balance

✅ 所有对象共享                  ✅ 每个对象各自独立
✅ 用 类名.属性 访问            ✅ 用 self.属性 访问
✅ 适合存固定配置、计数器         ✅ 适合存每个对象独有的数据
```

---

## 四、实战练习：AMC 资产管理类

结合你的 AMC 工作背景，写一个更贴近实际的类：

```python
class AssetRecord:
    """AMC 不良资产记录类"""

    # 类属性：资产类型映射
    TYPE_MAP = {
        "1": "贷款",
        "2": "信用卡",
        "3": "票据",
        "4": "抵债资产",
        "5": "其他",
    }

    # 类属性：计数器（记录创建了多少条）
    _count = 0

    def __init__(self, debtor_name, principal, overdue_days, asset_type="1"):
        """
        debtor_name:   债务人名称
        principal:      本金金额（万元）
        overdue_days:   逾期天数
        asset_type:     资产类型代码
        """
        AssetRecord._count += 1    # 每创建一个对象，计数+1
        self.id = f"NPL-{AssetRecord._count:04d}"  # 自动生成编号
        self.debtor_name = debtor_name
        self.principal = principal
        self.overdue_days = overdue_days
        self.asset_type = asset_type
        self.settled = False       # 是否已处置

    @property
    def type_name(self):
        """将类型代码转为中文名（属性装饰器，像属性一样使用）"""
        return self.TYPE_MAP.get(self.asset_type, "未知")

    @property
    def risk_level(self):
        """根据逾期天数判断风险等级"""
        if self.overdue_days <= 90:
            return "低风险"
        elif self.overdue_days <= 365:
            return "中风险"
        elif self.overdue_days <= 730:
            return "高风险"
        else:
            return "严重风险"

    def settle(self, recovery_amount):
        """处置资产"""
        self.settled = True
        self.recovery_amount = recovery_amount
        recovery_rate = recovery_amount / self.principal * 100
        print(f"资产 {self.id} 已处置，回收 ¥{recovery_amount:.2f}万")
        print(f"回收率: {recovery_rate:.1f}%")

    def __str__(self):
        status = "✅ 已处置" if self.settled else "🔴 未处置"
        return (
            f"[{self.id}] {self.debtor_name} | "
            f"本金: ¥{self.principal:.2f}万 | "
            f"逾期: {self.overdue_days}天 | "
            f"类型: {self.type_name} | "
            f"风险: {self.risk_level} | "
            f"{status}"
        )


# 创建资产记录
a1 = AssetRecord("XX贸易有限公司", 500, 120)
a2 = AssetRecord("YY实业有限公司", 300, 45)
a3 = AssetRecord("ZZ房地产开发公司", 2000, 800)

print(a1)
# [NPL-0001] XX贸易有限公司 | 本金: ¥500.00万 | 逾期: 120天 | 类型: 贷款 | 风险: 中风险 | 🔴 未处置

print(a2)
# [NPL-0002] YY实业有限公司 | 本金: ¥300.00万 | 逾期: 45天 | 类型: 贷款 | 风险: 低风险 | 🔴 未处置

print(a3)
# [NPL-0003] ZZ房地产开发公司 | 本金: ¥2000.00万 | 逾期: 800天 | 类型: 贷款 | 风险: 严重风险 | 🔴 未处置

# 处置资产
a1.settle(350)
# 资产 NPL-0001 已处置，回收 ¥350.00万
# 回收率: 70.0%

print(a1)
# [NPL-0001] XX贸易有限公司 | 本金: ¥500.00万 | 逾期: 120天 | 类型: 贷款 | 风险: 中风险 | ✅ 已处置
```

> **重点理解 `@property`**：它让方法像属性一样使用。调用时写 `a1.risk_level` 而不是 `a1.risk_level()`，更直观。

---

## 五、三个"魔术方法"（双下划线方法）

Python 用 `__xxx__` 这种命名方式定义了很多内置行为，你只要写好对应方法，Python 会自动在合适的时候调用它。

### 5.1 最常用的魔术方法

```python
class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    # 1️⃣ __str__：定义 print(对象) 和 str(对象) 的显示内容
    def __str__(self):
        return f"{self.name} - ¥{self.price}"

    # 2️⃣ __repr__：定义在交互式环境或列表中显示的内容
    def __repr__(self):
        return f"Product('{self.name}', {self.price}, {self.stock})"

    # 3️⃣ __len__：定义 len(对象) 的行为
    def __len__(self):
        return self.stock

    # 4️⃣ __bool__：定义 if 对象 的判断条件
    def __bool__(self):
        return self.stock > 0

    # 5️⃣ __eq__：定义 == 比较的行为
    def __eq__(self, other):
        if not isinstance(other, Product):
            return False
        return self.name == other.name and self.price == other.price


p1 = Product("iPhone", 7999, 100)
p2 = Product("iPad", 3999, 50)
p3 = Product("iPhone", 7999, 0)

print(p1)              # iPhone - ¥7999  （调用了 __str__）
print([p1, p2])        # [Product('iPhone', 7999, 100), Product('iPad', 3999, 50)]  （调用了 __repr__）
print(len(p1))         # 100  （调用了 __len__）

if p1:                 # True  （调用了 __bool__，stock > 0）
    print(f"{p1.name} 有货")

if p3:
    print(f"{p3.name} 有货")
else:
    print(f"{p3.name} 没货")  # stock = 0

print(p1 == p3)        # True  （名字和价格相同）
```

### 魔术方法速查表

| 方法 | 触发时机 | 示例 |
|------|---------|------|
| `__init__` | 创建对象时 | `p = Product()` |
| `__str__` | `print()` 或 `str()` | `print(p)` |
| `__repr__` | 交互式环境 / 列表显示 | `[p1, p2]` |
| `__len__` | `len()` | `len(p)` |
| `__bool__` | `if` 判断 | `if p:` |
| `__eq__` | `==` 比较 | `p1 == p2` |
| `__add__` | `+` 运算 | `p1 + p2` |
| `__gt__` | `>` 比较 | `p1 > p2` |

> **实际开发中，`__init__` 和 `__str__` 用得最多**，其他按需了解即可。

---

## 六、类与类的关系（组合）

两个类之间最常用的关系是"组合"——**一个类里面包含另一个类的对象作为属性**。

```python
class Debtor:
    """债务人类"""
    def __init__(self, name, id_number, industry):
        self.name = name
        self.id_number = id_number
        self.industry = industry

    def __str__(self):
        return f"{self.name}（{self.industry}行业）"


class AssetPackage:
    """资产包类 —— 包含多个资产记录和一个债务人"""
    def __init__(self, package_id, debtor):
        self.package_id = package_id
        self.debtor = debtor          # 组合：AssetPackage 包含 Debtor 对象
        self.assets = []              # 组合：包含多个 AssetRecord 对象
        self.total_principal = 0

    def add_asset(self, asset):
        """添加资产到资产包"""
        self.assets.append(asset)
        self.total_principal += asset.principal
        print(f"已添加资产 {asset.id}（¥{asset.principal}万）到包 {self.package_id}")

    def summary(self):
        """输出资产包摘要"""
        settled = [a for a in self.assets if a.settled]
        unsettled = [a for a in self.assets if not a.settled]

        print(f"\n{'='*50}")
        print(f"  资产包 {self.package_id} 摘要")
        print(f"{'='*50}")
        print(f"  债务人: {self.debtor}")
        print(f"  资产总数: {len(self.assets)} 笔")
        print(f"  本金总额: ¥{self.total_principal:.2f}万")
        print(f"  已处置: {len(settled)} 笔")
        print(f"  未处置: {len(unsettled)} 笔")
        print(f"{'='*50}")

        for asset in self.assets:
            print(f"    {asset}")


# 使用
debtor = Debtor("XX贸易有限公司", "91330100XXX", "房地产")

pkg = AssetPackage("PKG-2026-001", debtor)
pkg.add_asset(AssetRecord("XX贸易", 500, 120))
pkg.add_asset(AssetRecord("XX贸易", 800, 365))
pkg.add_asset(AssetRecord("XX贸易", 300, 45))

pkg.summary()
```

> **组合的好处**：资产包管理资产、债务人有自己的信息——各自独立又互相协作，代码更清晰。

---

## 七、今日练习（3选2必做）

### 练习 1：记账本类（推荐 ⭐）

```python
"""
实现一个简单的记账本类：
- 可以记录每笔支出（名称、金额、分类）
- 可以查看所有记录
- 可以按分类统计总支出
- 可以查看总支出

分类预设：餐饮、交通、购物、娱乐、其他
"""

# 你的代码写在这里
```

**参考步骤**：
1. 定义 `Ledger` 类，`__init__` 中初始化一个空列表 `self.records`
2. `add_record(name, amount, category)` 方法添加记录
3. `show_all()` 方法遍历打印所有记录
4. `summary_by_category()` 方法用字典统计各类总金额
5. `total()` 方法返回总支出

<details>
<summary>点击查看参考答案</summary>

```python
class Ledger:
    CATEGORIES = ["餐饮", "交通", "购物", "娱乐", "其他"]

    def __init__(self):
        self.records = []

    def add_record(self, name, amount, category):
        record = {
            "name": name,
            "amount": amount,
            "category": category,
        }
        self.records.append(record)
        print(f"✅ 已记录: {name} ¥{amount} ({category})")

    def show_all(self):
        if not self.records:
            print("还没有记录")
            return
        print(f"\n{'名称':<12} {'金额':>8} {'分类':<6}")
        print("-" * 30)
        for r in self.records:
            print(f"{r['name']:<12} ¥{r['amount']:>6.2f} {r['category']:<6}")

    def summary_by_category(self):
        stats = {}
        for r in self.records:
            cat = r["category"]
            stats[cat] = stats.get(cat, 0) + r["amount"]

        print("\n📊 分类统计:")
        for cat, total in sorted(stats.items(), key=lambda x: x[1], reverse=True):
            print(f"  {cat}: ¥{total:.2f}")

    def total(self):
        return sum(r["amount"] for r in self.records)


# 使用
book = Ledger()
book.add_record("午饭", 25, "餐饮")
book.add_record("地铁", 4, "交通")
book.add_record("电影", 45, "娱乐")
book.add_record("午饭", 30, "餐饮")
book.add_record("衣服", 299, "购物")

book.show_all()
book.summary_by_category()
print(f"\n总支出: ¥{book.total():.2f}")
```
</details>

### 练习 2：升级学生管理系统

回看 Day 14 的 `Student` 类，给它添加以下功能：

- [ ] `@property grade_level`：根据 `average` 返回 "优秀/良好/中等/及格/不及格"
- [ ] `__eq__`：两个学生学号相同就认为相等
- [ ] `__lt__`：按平均分排序用

<details>
<summary>点击查看参考答案</summary>

```python
# 在 Student 类中添加以下方法

@property
def grade_level(self):
    """根据均分返回等级"""
    avg = self.average
    if avg >= 90:
        return "优秀"
    elif avg >= 80:
        return "良好"
    elif avg >= 70:
        return "中等"
    elif avg >= 60:
        return "及格"
    else:
        return "不及格"

def __eq__(self, other):
    """学号相同就是同一个学生"""
    if not isinstance(other, Student):
        return False
    return self.student_id == other.student_id

def __lt__(self, other):
    """按平均分比较大小（用于排序）"""
    return self.average < other.average

# 测试
s1 = Student("001", "张三", 20, [85, 92])
s2 = Student("001", "张三", 20, [85, 92])
s3 = Student("002", "李四", 21, [95, 98])

print(s1.grade_level)   # 良好
print(s1 == s2)          # True（学号相同）
print(s1 < s3)           # True（张三均分 < 李四均分）

# 排序
students = [s3, s1]
print(sorted(students))   # 自动按平均分升序
```
</details>

### 练习 3：对比题

```python
"""
下面两种写法效果相同，但设计思路不同。
判断哪个是"面向过程"，哪个是"面向对象"，各自优缺点是什么？

方式A（面向过程）：
"""
students_data = [
    {"name": "张三", "scores": [85, 92, 78]},
    {"name": "李四", "scores": [90, 88, 95]},
]

def calc_avg(student):
    scores = student["scores"]
    return sum(scores) / len(scores)

for s in students_data:
    avg = calc_avg(s)
    print(f"{s['name']} 均分: {avg:.1f}")

"""
方式B（面向对象）：
"""
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    @property
    def average(self):
        return sum(self.scores) / len(self.scores)

    def __str__(self):
        return f"{self.name} 均分: {self.average:.1f}"

for s in [Student("张三", [85, 92, 78]), Student("李四", [90, 88, 95])]:
    print(s)
```

<details>
<summary>点击查看分析</summary>

| 维度 | 方式A（面向过程） | 方式B（面向对象） |
|------|-----------------|-----------------|
| 数据与操作 | 分离（数据在字典里，逻辑在函数里） | 绑定（数据和操作在同一个类里） |
| 扩展性 | 加功能要写新函数，传字典参数 | 加功能就在类里加方法 |
| 代码量 | 简单场景更少 | 简单场景略多 |
| 可维护性 | 数据多了容易乱 | 结构清晰，易于维护 |
| 适用场景 | 小脚本、简单计算 | 大项目、多人协作 |

**结论**：数据简单、逻辑少 → 面向过程更直接；数据复杂、功能多 → 面向对象更清晰。两者不矛盾，实际开发中混着用。
</details>

---

## 八、今日避坑指南

1. **类名用大驼峰**（`BankAccount`），函数名用小写加下划线（`calc_average`），这是 Python 约定
2. **`self` 不能省** — 定义方法时第一个参数必须是 `self`，但调用时不需要传
3. **`__init__` 不是构造函数** — 它是初始化函数。真正的构造是 `__new__`（一般不需要自己写）
4. **不要直接操作属性，用方法** — 比如 `account.deposit(100)` 比 `account.balance += 100` 更安全（可以在方法里加校验）
5. **类属性和实例属性同名时** — 实例属性会"遮蔽"类属性，写 `self.x = 5` 不会修改 `Class.x`

---

## 九、今日知识脑图

```
┌─────────────────────────────────────────────────┐
│                面向对象编程 OOP                     │
├─────────────┬───────────────┬─────────────────────┤
│   核心概念    │   魔术方法      │   设计关系            │
├─────────────┼───────────────┼─────────────────────┤
│ • 类 Class   │ • __init__    │ • 组合（has-a）      │
│ • 对象 Object│ • __str__     │   一个类包含另一个类  │
│ • 属性 attr  │ • __repr__    │                     │
│ • 方法 method│ • __len__     │ • 继承（is-a）      │
│ • self       │ • __eq__      │   子类继承父类        │
│ • @property  │ • __bool__    │   （明天讲）         │
├─────────────┼───────────────┼─────────────────────┤
│ 类属性 vs     │               │                     │
│ 实例属性      │               │                     │
│ • 共享 vs 独立│               │                     │
│ • 类名.属性   │               │                     │
│ • self.属性   │               │                     │
└─────────────┴───────────────┴─────────────────────┘
```

---

## 十、推荐资源

| 资源 | 说明 |
|------|------|
| [Python OOP 官方教程](https://docs.python.org/zh-cn/3/tutorial/classes.html) | 最权威的类与继承讲解 |
| [廖雪峰 - 面向对象编程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017496085248352) | 中文讲解，配合实例 |
| [菜鸟教程 Python3 面向对象](https://www.runoob.com/python3/python3-class.html) | 快速查阅 |

---

## 十一、明日预告（Day 16）

明天继续面向对象 —— **继承与多态**：

- 子类继承父类的属性和方法
- 方法重写（Override）
- `super()` 的使用
- 多态的概念
- 用继承重构 AMC 资产类（不同资产类型子类化）

---

> **今日金句**：面向对象的本质不是让你写更少的代码，而是让代码更有条理。
> 好比一个仓库——东西多了不可怕，可怕的是没有分类。
