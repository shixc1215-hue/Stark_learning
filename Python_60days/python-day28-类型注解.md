# Python Day 28 - 类型注解（Type Annotations / Type Hints）

## 一、什么是类型注解？

在 Python 中，变量和函数参数不需要声明类型——Python 是**动态类型**语言，变量的类型在运行时才确定。这虽然灵活，但当项目变大、多人协作时，容易因为类型问题出 bug。

**类型注解（Type Hints）** 是 Python 3.5+ 引入的语法，让你可以在代码中**标注**变量、参数和返回值的类型。它不会改变程序的运行行为，但能帮助：
- IDE（如 VS Code、PyCharm）提供更智能的**代码补全和错误提示**
- 团队协作时**明确接口约定**
- 使用工具（如 mypy）做**静态类型检查**，提前发现类型错误

> **核心概念**：类型注解只是"标注"，不是"限制"。Python 解释器不会因为你标注了 `int` 就阻止你传入 `str`——它只是一种约定和提示。

---

## 二、基础语法

### 2.1 变量类型注解

用 `变量: 类型` 的语法标注变量：

```python
# 基本类型
name: str = "小明"
age: int = 25
height: float = 1.75
is_student: bool = True

# 列表、字典（需要从 typing 导入）
from typing import List, Dict

scores: List[int] = [90, 85, 92]
student_info: Dict[str, str] = {"name": "小明", "school": "清华"}

# 也可以不赋初值，仅标注类型
address: str  # 地址尚未赋值，但明确它应该是字符串
```

### 2.2 函数参数和返回值注解

```python
def greet(name: str) -> str:
    """向指定的人打招呼"""
    return f"你好，{name}！"

def add(a: int, b: int) -> int:
    """两数相加"""
    return a + b

def calculate_circle(radius: float) -> tuple:
    """计算圆的周长和面积"""
    circumference = 2 * 3.14159 * radius
    area = 3.14159 * radius ** 2
    return (circumference, area)
```

语法说明：
- `参数: 类型` —— 标注参数类型
- `-> 类型` —— 标注返回值类型
- 没有返回值时用 `-> None`

---

## 三、typing 模块常用类型

Python 内置的 `typing` 模块提供了丰富的类型标注工具。

### 3.1 容器类型

```python
from typing import List, Dict, Tuple, Set

# List: 列表，标注元素类型
numbers: List[int] = [1, 2, 3]
names: List[str] = ["Alice", "Bob"]

# Dict: 字典，标注键类型和值类型
config: Dict[str, str] = {"host": "localhost", "port": "8080"}
grades: Dict[str, int] = {"语文": 95, "数学": 88}

# Tuple: 元组，标注每个位置的类型
point: Tuple[float, float] = (3.5, 7.2)        # (x, y) 坐标
record: Tuple[str, int, float] = ("小明", 25, 95.5)  # (姓名, 年龄, 分数)

# Set: 集合
tags: Set[str] = {"python", "编程", "学习"}
```

### 3.2 Optional 与 Union

```python
from typing import Optional, Union

# Optional[X] = Union[X, None]，表示"可能是 X，也可能是 None"
def find_user(user_id: int) -> Optional[str]:
    """根据用户ID查找用户名，找不到返回 None"""
    if user_id == 1:
        return "小明"
    return None

# Union: 表示多种可能的类型
def process_data(data: Union[int, str, float]) -> str:
    """处理不同类型的数据"""
    return f"收到数据: {data}"

# 简写形式（Python 3.10+）
# Optional[str] 可以写成 str | None
# Union[int, str] 可以写成 int | str
```

### 3.3 Any 与类型别名

```python
from typing import Any, TypeAlias

# Any: 任意类型（相当于不做类型检查）
def print_anything(value: Any) -> None:
    print(value)

# 类型别名: 给复杂类型起个简短的名字
Point = tuple[float, float]          # Python 3.9+ 写法
UserInfo = dict[str, str | int]      # 用户信息字典

user: UserInfo = {"name": "小明", "age": 25, "score": 95}
```

---

## 四、高级类型注解

### 4.1 Callable —— 函数类型

当参数或返回值本身是函数时，用 `Callable` 标注：

```python
from typing import Callable

# Callable[[参数类型1, 参数类型2], 返回值类型]
def apply_operation(a: int, b: int, operation: Callable[[int, int], int]) -> int:
    """将 operation 函数应用于 a 和 b"""
    return operation(a, b)

def multiply(x: int, y: int) -> int:
    return x * y

result = apply_operation(3, 4, multiply)  # result = 12
```

### 4.2 字面量类型 Literal

限制变量只能是特定的几个值（Python 3.8+）：

```python
from typing import Literal

# 状态只能是这三种之一
Status = Literal["pending", "success", "failed"]

def handle_order(status: Status) -> None:
    if status == "success":
        print("订单处理成功！")
    elif status == "failed":
        print("订单处理失败！")
    else:
        print("订单等待处理中...")

handle_order("success")   # ✅ 正确
# handle_order("error")    # ❌ 类型检查器会报错
```

### 4.3 TypeGuard 与类型窄化

运行时判断类型后，告诉类型检查器变量的具体类型：

```python
from typing import TypeGuard

def is_string_list(value: list) -> TypeGuard[list[str]]:
    """判断列表中的元素是否都是字符串"""
    return all(isinstance(item, str) for item in value)

data: list = ["hello", "world"]
if is_string_list(data):
    # 在这个分支中，类型检查器知道 data 是 list[str]
    print(data[0].upper())  # IDE 不会报错
```

### 4.4 泛型（Generics）

泛型让你写出适用于多种类型的通用代码（进阶内容，了解即可）：

```python
from typing import TypeVar, Generic

T = TypeVar("T")  # 定义一个类型变量

class Box(Generic[T]):
    """一个可以装任何类型东西的盒子"""
    def __init__(self, content: T) -> None:
        self.content = content

    def get(self) -> T:
        return self.content

int_box: Box[int] = Box(42)
str_box: Box[str] = Box("Hello")

print(int_box.get())  # 42
print(str_box.get())  # Hello
```

---

## 五、Python 3.10+ 简化语法

Python 3.10 对类型注解做了大幅简化，可以不再依赖 `typing` 模块：

```python
# === 旧写法（Python 3.9 以下）===
from typing import List, Dict, Optional, Union, Tuple

def old_style(
    items: List[int],
    config: Dict[str, str],
    maybe: Optional[str] = None
) -> Union[int, str]:
    ...

# === 新写法（Python 3.10+）===
def new_style(
    items: list[int],          # 直接用小写 list
    config: dict[str, str],    # 直接用小写 dict
    maybe: str | None = None   # 用 | 代替 Union
) -> int | str:
    ...
```

| 旧写法 | 新写法 (3.10+) |
|--------|----------------|
| `List[int]` | `list[int]` |
| `Dict[str, int]` | `dict[str, int]` |
| `Tuple[int, str]` | `tuple[int, str]` |
| `Set[str]` | `set[str]` |
| `Optional[str]` | `str \| None` |
| `Union[int, str]` | `int \| str` |

---

## 六、静态类型检查工具 mypy

### 6.1 安装 mypy

```bash
pip install mypy
```

### 6.2 使用方法

假设有一个文件 `example.py`：

```python
def add(a: int, b: int) -> int:
    return a + b

# 下面这行，mypy 会报错！传入了字符串
result: int = add("hello", "world")
```

运行检查：

```bash
mypy example.py
```

输出：
```
example.py:6: error: Argument 1 to "add" has incompatible type "str"; expected "int"
example.py:6: error: Argument 2 to "add" has incompatible type "str"; expected "int"
Found 2 errors in 1 file (checked 1 source file)
```

### 6.3 配置文件 `mypy.ini`

```ini
[mypy]
python_version = 3.11
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True   # 要求所有函数都有类型注解
```

---

## 七、综合实战：学生成绩管理系统（带类型注解）

```python
from typing import Optional, Dict, List

# === 类型别名，提高可读性 ===
StudentName = str
StudentID = str
SubjectName = str
Score = float
Grade = Dict[SubjectName, Score]

class StudentManager:
    """带完整类型注解的学生成绩管理系统"""

    def __init__(self) -> None:
        self._students: Dict[StudentID, dict] = {}

    def add_student(self, student_id: StudentID, name: StudentName) -> None:
        """添加学生"""
        if student_id in self._students:
            raise ValueError(f"学生ID {student_id} 已存在")
        self._students[student_id] = {"name": name, "grades": {}}

    def add_grade(
        self, student_id: StudentID, subject: SubjectName, score: Score
    ) -> None:
        """添加成绩"""
        if student_id not in self._students:
            raise KeyError(f"学生ID {student_id} 不存在")
        if not 0 <= score <= 100:
            raise ValueError("成绩必须在 0-100 之间")
        self._students[student_id]["grades"][subject] = score

    def get_student(self, student_id: StudentID) -> Optional[dict]:
        """查询学生信息"""
        return self._students.get(student_id)

    def get_average(self, student_id: StudentID) -> Optional[float]:
        """计算平均分"""
        student = self.get_student(student_id)
        if student is None or not student["grades"]:
            return None
        scores: List[Score] = list(student["grades"].values())
        return round(sum(scores) / len(scores), 2)

    def get_top_student(self) -> Optional[tuple[str, float]]:
        """获取平均分最高的学生"""
        best_id: Optional[str] = None
        best_avg: float = 0.0

        for sid in self._students:
            avg = self.get_average(sid)
            if avg is not None and avg > best_avg:
                best_avg = avg
                best_id = sid

        if best_id is None:
            return None
        return (self._students[best_id]["name"], best_avg)

    def list_all(self) -> List[tuple[str, str, float | None]]:
        """列出所有学生及其平均分"""
        result: List[tuple[str, str, float | None]] = []
        for sid, info in self._students.items():
            avg = self.get_average(sid)
            result.append((sid, info["name"], avg))
        return result


# === 使用示例 ===
if __name__ == "__main__":
    manager = StudentManager()

    # 添加学生
    manager.add_student("001", "小明")
    manager.add_student("002", "小红")
    manager.add_student("003", "小刚")

    # 添加成绩
    for sid in ["001", "002", "003"]:
        manager.add_grade(sid, "语文", 90 + hash(sid) % 10)
        manager.add_grade(sid, "数学", 85 + hash(sid) % 15)
        manager.add_grade(sid, "英语", 80 + hash(sid) % 20)

    # 查询结果
    for sid, name, avg in manager.list_all():
        print(f"[{sid}] {name}: 平均分 {avg}")

    top = manager.get_top_student()
    if top:
        print(f"\n🏆 最高平均分: {top[0]} ({top[1]}分)")
```

---

## 八、练习题

### 练习 1：为函数添加类型注解

为下面的函数补全类型注解：

```python
def create_user(username, age, active, tags):
    return {
        "username": username,
        "age": age,
        "active": active,
        "tags": tags
    }
```

<details>
<summary>参考答案</summary>

```python
from typing import Dict, List, Any

def create_user(
    username: str,
    age: int,
    active: bool,
    tags: List[str]
) -> Dict[str, Any]:
    return {
        "username": username,
        "age": age,
        "active": active,
        "tags": tags
    }
```

</details>

### 练习 2：使用 Optional 和 Union

编写一个函数 `search_item`，在字典列表中搜索指定键值的项，找到返回该项（字典），找不到返回 `None`：

```python
# items 是一个字典列表，key 是要搜索的键，value 是要匹配的值
def search_item(items, key, value):
    ...
```

<details>
<summary>参考答案</summary>

```python
from typing import Optional, List, Dict, Any

def search_item(
    items: List[Dict[str, Any]],
    key: str,
    value: Any
) -> Optional[Dict[str, Any]]:
    """在字典列表中搜索指定键值的项"""
    for item in items:
        if item.get(key) == value:
            return item
    return None

# 测试
products = [
    {"name": "手机", "price": 3999},
    {"name": "电脑", "price": 7999},
    {"name": "平板", "price": 2999},
]
result = search_item(products, "name", "电脑")
print(result)  # {'name': '电脑', 'price': 7999}
```

</details>

### 练习 3：类型别名实战

为一个"图书管理系统"定义类型别名，并实现一个 `add_book` 函数：

```python
# 提示：
# ISBN = str
# BookInfo = dict，包含 title(str), author(str), price(float), tags(list[str])
```

<details>
<summary>参考答案</summary>

```python
from typing import TypeAlias

ISBN: TypeAlias = str
BookInfo: TypeAlias = dict[str, str | float | list[str]]

def add_book(
    library: dict[ISBN, BookInfo],
    isbn: ISBN,
    title: str,
    author: str,
    price: float,
    tags: list[str]
) -> None:
    """添加一本书到图书馆"""
    if isbn in library:
        raise ValueError(f"ISBN {isbn} 已存在")
    library[isbn] = {
        "title": title,
        "author": author,
        "price": price,
        "tags": tags
    }

# 测试
my_books: dict[ISBN, BookInfo] = {}
add_book(my_books, "978-7-0001", "Python编程", "小明", 59.9, ["编程", "入门"])
print(my_books)
```

</details>

---

## 九、常见问题 FAQ

### Q1：类型注解会降低程序性能吗？

**不会。** 类型注解在运行时几乎不影响性能。Python 解释器基本忽略它们（注解存储在函数的 `__annotations__` 属性中），只在导入模块时解析一次。真正做检查的是 mypy 这类外部工具。

---

### Q2：必须给每个变量都加类型注解吗？

**不需要。** 遵循实用原则：
- **公开 API / 函数签名** —— 推荐加，帮助调用者理解
- **复杂的数据结构** —— 推荐加，避免混淆
- **简单的局部变量**（如循环中的 `i`）—— 可以不加

---

### Q3：类型注解和运行时类型检查有什么区别？

```python
x: int = "hello"  # 类型注解：标注 x 应该是 int，但 Python 不会报错
x = int("hello")   # 运行时类型检查：这里会抛出 ValueError
```

类型注解是**开发时**的提示，运行时类型检查是**执行时**的验证。如果需要运行时检查，用 `isinstance()` 或第三方库 `pydantic`。

---

### Q4：mypy 报 "too many arguments" 或 "unexpected keyword" 怎么办？

这通常是因为 mypy 检查的代码版本和你实际运行的 Python 版本不一致。在 `mypy.ini` 中设置正确的 `python_version` 即可。

---

### Q5：什么时候用 Any，什么时候不用？

尽量**避免**使用 `Any`——它等于放弃了类型检查。只有在以下情况可以接受：
- 处理完全动态的数据（如 JSON 反序列化前）
- 与没有类型注解的旧代码交互
- 临时过渡阶段

优先考虑 `Union` 或更精确的类型来替代 `Any`。

---

## 十、免费学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| 廖雪峰 Python 教程 | https://www.liaoxuefeng.com/wiki/1016959663602400 | 中文经典，通俗易懂 |
| 菜鸟教程 - Python | https://www.runoob.com/python3/python3-tutorial.html | 查阅方便，适合入门 |
| Python 官方 typing 文档 | https://docs.python.org/zh-cn/3/library/typing.html | 类型注解权威参考 |
| mypy 官方文档 | https://mypy.readthedocs.io/ | 静态类型检查工具 |
| Real Python - Type Hinting | https://realpython.com/python-type-checking/ | 英文进阶教程 |

---

## 十一、今日小结

| 知识点 | 说明 |
|--------|------|
| 变量注解 | `name: str = "小明"` |
| 函数注解 | `def add(a: int, b: int) -> int` |
| 容器类型 | `List[int]`、`Dict[str, float]`、`Tuple[int, str]` |
| Optional / Union | `Optional[str]` = `str \| None` |
| 类型别名 | `Point = tuple[float, float]` |
| Callable | 标注函数类型 |
| Literal | 限制为固定几个值 |
| Python 3.10+ | `list[int]` 代替 `List[int]`，`\|` 代替 `Union` |
| mypy | 静态类型检查工具 |

> **明日预告：Day 29 - dataclass（数据类）**，用更优雅的方式定义数据容器类，减少样板代码。
