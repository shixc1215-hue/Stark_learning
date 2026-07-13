# Python Day 30 - 单元测试

> **学习目标**：掌握 Python 单元测试的核心概念与 `unittest` 框架的使用，学会编写可靠的测试用例。

---

## 一、什么是单元测试？

**单元测试（Unit Test）** 是对程序中最小的可测试单元进行验证的测试方式。在 Python 中，一个"单元"通常是一个**函数**或一个**方法**。

### 为什么需要单元测试？

| 好处 | 说明 |
|------|------|
| **尽早发现 Bug** | 在代码合并前就捕获问题，修复成本最低 |
| **放心重构** | 有测试保护，改代码不怕引入新问题 |
| **文档作用** | 测试用例本身就是函数用法的最佳示例 |
| **设计改善** | 难以测试的代码往往设计有问题 |

### 一个直观的比喻

把写代码比作盖房子：
- **单元测试** = 检查每一块砖是否合格
- **集成测试** = 检查一面墙是否稳固
- **端到端测试** = 检查整栋房子能否住人

如果砖头本身就有裂缝，墙和房子注定出问题。所以**单元测试是一切测试的基础**。

---

## 二、unittest 框架入门

Python 内置了 `unittest` 模块，无需安装任何第三方库即可使用。它是 Python 标准库中最常用的测试框架。

### 2.1 最简单的测试

先写一个要测试的函数，创建文件 `calculator.py`：

```python
# calculator.py - 待测试的模块

def add(a, b):
    """两数相加"""
    return a + b

def divide(a, b):
    """两数相除"""
    if b == 0:
        raise ValueError("除数不能为零")
    return a / b

def is_even(n):
    """判断是否为偶数"""
    return n % 2 == 0
```

再创建测试文件 `test_calculator.py`：

```python
# test_calculator.py - 测试文件
import unittest
from calculator import add, divide, is_even


class TestCalculator(unittest.TestCase):
    """测试 Calculator 模块"""

    def test_add(self):
        """测试加法"""
        self.assertEqual(add(1, 2), 3)       # 1 + 2 应该等于 3
        self.assertEqual(add(-1, 1), 0)      # -1 + 1 应该等于 0
        self.assertEqual(add(0, 0), 0)       # 0 + 0 应该等于 0

    def test_divide(self):
        """测试除法"""
        self.assertEqual(divide(10, 2), 5.0)    # 10 / 2 = 5.0
        self.assertEqual(divide(7, 2), 3.5)     # 7 / 2 = 3.5

    def test_divide_by_zero(self):
        """测试除以零异常"""
        with self.assertRaises(ValueError):
            divide(10, 0)  # 应该抛出 ValueError

    def test_is_even(self):
        """测试偶数判断"""
        self.assertTrue(is_even(4))    # 4 是偶数
        self.assertFalse(is_even(3))   # 3 不是偶数


if __name__ == "__main__":
    unittest.main()
```

运行测试：

```bash
python -m unittest test_calculator
```

输出结果：

```
....
----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
```

> 每个 `.` 代表一个通过的测试，`F` 代表失败，`E` 代表错误。看到 `OK` 就表示全部通过！

### 2.2 测试文件的基本结构

```
project/
├── calculator.py          # 被测试的代码（源代码）
└── test_calculator.py     # 测试代码
```

**关键规则**：
1. 测试文件名以 `test_` 开头（这是约定俗成的规范）
2. 测试类继承 `unittest.TestCase`
3. 测试方法名以 `test_` 开头
4. 使用 `self.assertXxx()` 方法做断言

---

## 三、断言方法大全

断言（Assertion）是测试的核心。它像一个小裁判，判断实际结果是否符合预期。

### 3.1 常用断言方法

```python
class TestAssertions(unittest.TestCase):

    # ---- 相等性判断 ----
    def test_equal(self):
        self.assertEqual(1 + 1, 2)           # a == b
        self.assertNotEqual(1 + 1, 3)       # a != b

    # ---- 真假判断 ----
    def test_boolean(self):
        self.assertTrue(10 > 5)              # 值为 True
        self.assertFalse(10 < 5)            # 值为 False

    # ---- 是否为 None ----
    def test_none(self):
        result = None
        self.assertIsNone(result)            # 值是 None
        self.assertIsNotNone(42)             # 值不是 None

    # ---- 包含关系 ----
    def test_in(self):
        self.assertIn("a", "abc")            # "a" 在 "abc" 中
        self.assertNotIn("d", "abc")        # "d" 不在 "abc" 中

    # ---- 类型判断 ----
    def test_type(self):
        self.assertIsInstance(42, int)       # 42 是 int 类型
        self.assertNotIsInstance("hi", int) # "hi" 不是 int 类型

    # ---- 近似相等（浮点数比较） ----
    def test_almost_equal(self):
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=7)
        # 浮点数精度问题，不能直接用 ==

    # ---- 异常判断 ----
    def test_exception(self):
        with self.assertRaises(ValueError):
            int("abc")  # 应该抛出 ValueError

    # ---- 警告判断 ----
    def test_warning(self):
        import warnings
        with self.assertWarns(DeprecationWarning):
            warnings.warn("old", DeprecationWarning)
```

### 3.2 断言方法速查表

| 方法 | 含义 | 示例 |
|------|------|------|
| `assertEqual(a, b)` | a == b | `self.assertEqual(1+1, 2)` |
| `assertNotEqual(a, b)` | a != b | `self.assertNotEqual(1, 2)` |
| `assertTrue(x)` | x 为 True | `self.assertTrue(10 > 5)` |
| `assertFalse(x)` | x 为 False | `self.assertFalse(10 < 5)` |
| `assertIsNone(x)` | x 是 None | `self.assertIsNone(None)` |
| `assertIn(a, b)` | a 在 b 中 | `self.assertIn(1, [1,2,3])` |
| `assertIsInstance(a, b)` | a 是 b 的实例 | `self.assertIsInstance(1, int)` |
| `assertRaises(E)` | 抛出异常 E | `with self.assertRaises(ValueError):` |
| `assertAlmostEqual(a,b)` | 近似相等 | 浮点数比较 |

---

## 四、测试固件（Fixture）

测试固件是在测试**之前**和**之后**自动执行的代码，用于准备测试环境和清理资源。

### 4.1 setUp 和 tearDown

```python
import unittest

class TestListOperations(unittest.TestCase):
    """演示 setUp 和 tearDown 的用法"""

    def setUp(self):
        """每个测试方法执行前自动调用 - 准备测试数据"""
        self.fruits = ["apple", "banana", "cherry"]
        print(f"\n[setUp] 准备数据: {self.fruits}")

    def tearDown(self):
        """每个测试方法执行后自动调用 - 清理资源"""
        self.fruits.clear()
        print(f"[tearDown] 清理数据")

    def test_append(self):
        """测试追加元素"""
        self.fruits.append("durian")
        self.assertEqual(len(self.fruits), 4)
        self.assertIn("durian", self.fruits)

    def test_remove(self):
        """测试删除元素"""
        self.fruits.remove("banana")
        self.assertEqual(len(self.fruits), 2)
        self.assertNotIn("banana", self.fruits)

    def test_sort(self):
        """测试排序"""
        self.fruits.sort()
        self.assertEqual(self.fruits, ["apple", "banana", "cherry"])


if __name__ == "__main__":
    unittest.main()
```

**执行顺序**：

```
setUp → test_append → tearDown
setUp → test_remove → tearDown
setUp → test_sort  → tearDown
```

### 4.2 类级别固件

如果多个测试共享同一个昂贵的资源（如数据库连接），用类级别的固件更高效：

```python
class TestDatabase(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        """所有测试方法执行前只调用一次"""
        cls.db_connection = "模拟数据库连接"
        print("\n[setUpClass] 建立数据库连接")

    @classmethod
    def tearDownClass(cls):
        """所有测试方法执行后只调用一次"""
        cls.db_connection = None
        print("[tearDownClass] 关闭数据库连接")

    def test_query(self):
        """测试查询"""
        self.assertIsNotNone(self.db_connection)

    def test_insert(self):
        """测试插入"""
        self.assertIsNotNone(self.db_connection)
```

### 4.3 固件对比

| 固件 | 调用时机 | 调用次数 | 典型用途 |
|------|----------|----------|----------|
| `setUp` | 每个测试前 | N 次（N = 测试数） | 准备独立的测试数据 |
| `tearDown` | 每个测试后 | N 次 | 清理临时文件/变量 |
| `setUpClass` | 所有测试前 | 1 次 | 建立数据库连接 |
| `tearDownClass` | 所有测试后 | 1 次 | 关闭数据库连接 |

---

## 五、跳过测试与预期失败

有时某些测试暂时无法运行（比如依赖外部服务），可以用装饰器跳过：

```python
class TestAdvanced(unittest.TestCase):

    @unittest.skip("暂时跳过，功能未完成")
    def test_unfinished_feature(self):
        self.fail("这个功能还没写完")

    @unittest.skipIf(sys.version_info < (3, 10), "需要 Python 3.10+")
    def test_new_syntax(self):
        # 使用 3.10+ 的新语法
        pass

    @unittest.skipUnless(has_internet(), "需要网络连接")
    def test_api_call(self):
        # 调用外部 API
        pass

    @unittest.expectedFailure
    def test_known_bug(self):
        """已知 Bug，预期会失败"""
        result = 1 + 1
        self.assertEqual(result, 3)  # 故意写错，标记为已知问题
```

运行结果中会显示跳过/预期的数量：

```
....s..x
----------------------------------------------------------------------
Ran 7 tests in 0.002s

OK (skipped=1, expected failures=1)
```

---

## 六、综合实战：测试一个用户管理模块

### 被测试代码 `user_manager.py`

```python
# user_manager.py

class User:
    """用户类"""
    def __init__(self, username, email, age=0):
        self.username = username
        self.email = email
        self.age = age

    def __repr__(self):
        return f"User({self.username}, {self.email}, {self.age})"


class UserManager:
    """用户管理器"""

    def __init__(self):
        self._users = {}  # username -> User

    def add_user(self, user):
        """添加用户，用户名不能重复"""
        if not user.username:
            raise ValueError("用户名不能为空")
        if "@" not in user.email:
            raise ValueError("邮箱格式不正确")
        if user.username in self._users:
            raise ValueError(f"用户名 {user.username} 已存在")
        self._users[user.username] = user
        return True

    def get_user(self, username):
        """根据用户名查找用户"""
        return self._users.get(username)

    def remove_user(self, username):
        """删除用户"""
        if username not in self._users:
            raise KeyError(f"用户 {username} 不存在")
        del self._users[username]
        return True

    def list_users(self):
        """列出所有用户"""
        return list(self._users.values())

    def get_adult_users(self):
        """获取所有成年用户（age >= 18）"""
        return [u for u in self._users.values() if u.age >= 18]

    def count(self):
        """用户总数"""
        return len(self._users)
```

### 测试代码 `test_user_manager.py`

```python
# test_user_manager.py
import unittest
from user_manager import User, UserManager


class TestUser(unittest.TestCase):
    """测试 User 类"""

    def test_create_user(self):
        """测试创建用户"""
        user = User("alice", "alice@example.com", 25)
        self.assertEqual(user.username, "alice")
        self.assertEqual(user.email, "alice@example.com")
        self.assertEqual(user.age, 25)

    def test_default_age(self):
        """测试默认年龄"""
        user = User("bob", "bob@test.com")
        self.assertEqual(user.age, 0)


class TestUserManager(unittest.TestCase):
    """测试 UserManager 类"""

    def setUp(self):
        """每个测试前创建一个干净的管理器"""
        self.manager = UserManager()
        self.alice = User("alice", "alice@example.com", 25)
        self.bob = User("bob", "bob@example.com", 16)

    def test_add_user_success(self):
        """测试成功添加用户"""
        result = self.manager.add_user(self.alice)
        self.assertTrue(result)
        self.assertEqual(self.manager.count(), 1)

    def test_add_duplicate_user(self):
        """测试添加重复用户应报错"""
        self.manager.add_user(self.alice)
        with self.assertRaises(ValueError) as context:
            self.manager.add_user(self.alice)
        self.assertIn("已存在", str(context.exception))

    def test_add_empty_username(self):
        """测试空用户名应报错"""
        bad_user = User("", "test@test.com")
        with self.assertRaises(ValueError):
            self.manager.add_user(bad_user)

    def test_add_invalid_email(self):
        """测试无效邮箱应报错"""
        bad_user = User("charlie", "not-an-email")
        with self.assertRaises(ValueError):
            self.manager.add_user(bad_user)

    def test_get_user(self):
        """测试查找用户"""
        self.manager.add_user(self.alice)
        found = self.manager.get_user("alice")
        self.assertIsNotNone(found)
        self.assertEqual(found.email, "alice@example.com")

    def test_get_nonexistent_user(self):
        """测试查找不存在的用户"""
        found = self.manager.get_user("nobody")
        self.assertIsNone(found)

    def test_remove_user(self):
        """测试删除用户"""
        self.manager.add_user(self.alice)
        self.manager.remove_user("alice")
        self.assertEqual(self.manager.count(), 0)

    def test_remove_nonexistent_user(self):
        """测试删除不存在的用户应报错"""
        with self.assertRaises(KeyError):
            self.manager.remove_user("nobody")

    def test_list_users(self):
        """测试列出所有用户"""
        self.manager.add_user(self.alice)
        self.manager.add_user(self.bob)
        users = self.manager.list_users()
        self.assertEqual(len(users), 2)

    def test_get_adult_users(self):
        """测试筛选成年用户"""
        self.manager.add_user(self.alice)  # 25岁
        self.manager.add_user(self.bob)    # 16岁
        adults = self.manager.get_adult_users()
        self.assertEqual(len(adults), 1)
        self.assertEqual(adults[0].username, "alice")

    def test_count(self):
        """测试用户计数"""
        self.assertEqual(self.manager.count(), 0)
        self.manager.add_user(self.alice)
        self.assertEqual(self.manager.count(), 1)
        self.manager.add_user(self.bob)
        self.assertEqual(self.manager.count(), 2)


if __name__ == "__main__":
    unittest.main()
```

---

## 七、mock 模拟对象（进阶）

有时测试需要调用外部服务（网络请求、数据库等），我们不想在测试时真正去调用它们。这时用 `unittest.mock` 来**模拟**这些依赖。

```python
from unittest.mock import Mock, patch
import unittest


def fetch_user_info(user_id):
    """模拟一个网络请求函数（真实场景中会调 API）"""
    # 假设这里会发送 HTTP 请求
    return {"id": user_id, "name": "Alice"}


def get_user_greeting(user_id):
    """使用 fetch_user_info 的业务函数"""
    info = fetch_user_info(user_id)
    return f"Hello, {info['name']}!"


class TestMock(unittest.TestCase):

    def test_mock_basic(self):
        """Mock 基本用法"""
        # 创建一个 Mock 对象
        mock_func = Mock(return_value={"id": 1, "name": "Alice"})

        # 调用它，返回预设值
        result = mock_func(1)
        self.assertEqual(result["name"], "Alice")

        # 验证它被调用过
        mock_func.assert_called_once_with(1)

    @patch("__main__.fetch_user_info")
    def test_patch(self, mock_fetch):
        """使用 patch 替换函数"""
        # 让 fetch_user_info 返回假数据
        mock_fetch.return_value = {"id": 42, "name": "Bob"}

        result = get_user_greeting(42)
        self.assertEqual(result, "Hello, Bob!")

        # 验证函数被正确调用
        mock_fetch.assert_called_once_with(42)
```

> `patch` 是最常用的 mock 手段。它会**临时替换**某个函数或对象，测试结束后自动恢复。

---

## 八、运行测试的多种方式

```bash
# 方式1：运行单个测试文件
python -m unittest test_calculator

# 方式2：运行单个测试类
python -m unittest test_user_manager.TestUserManager

# 方式3：运行单个测试方法
python -m unittest test_user_manager.TestUserManager.test_add_user_success

# 方式4：自动发现当前目录所有测试
python -m unittest discover

# 方式5：指定发现路径和模式
python -m unittest discover -s tests -p "test_*.py"

# 方式6：显示详细输出（推荐开发时使用）
python -m unittest -v test_calculator
```

`-v`（verbose）模式会显示每个测试的名称和结果，比默认的 `.` 更直观：

```
test_add (test_calculator.TestCalculator) ... ok
test_divide (test_calculator.TestCalculator) ... ok
test_divide_by_zero (test_calculator.TestCalculator) ... ok
test_is_even (test_calculator.TestCalculator) ... ok
----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
```

---

## 九、测试命名与组织最佳实践

### 9.1 测试方法命名规范

好的测试名应该**描述预期行为**：

```python
class TestUserManager(unittest.TestCase):

    # 好的命名 - 一眼看出测什么
    def test_add_user_success(self): ...               # 成功添加
    def test_add_duplicate_user_raises_error(self): ... # 重复添加报错
    def test_get_nonexistent_user_returns_none(self): ...# 查不到返回 None

    # 不好的命名 - 不知道测什么
    def test_user1(self): ...
    def test_add(self): ...
    def test_123(self): ...
```

### 9.2 一个测试只验证一件事

```python
# 好的 - 每个测试只关注一个场景

def test_add_user_with_valid_data(self):
    """正常数据添加用户"""
    ...

def test_add_user_with_empty_name(self):
    """空用户名添加用户"""
    ...

def test_add_user_with_invalid_email(self):
    """无效邮箱添加用户"""
    ...

# 不好的 - 一个测试测太多东西
def test_user_operations(self):
    # 添加、删除、查询、统计全塞一起
    ...
```

### 9.3 AAA 模式

每个测试方法遵循 **Arrange-Act-Assert** 三步走：

```python
def test_user_age_filter(self):
    # Arrange（准备）- 设置测试数据
    manager = UserManager()
    manager.add_user(User("alice", "a@b.com", 25))
    manager.add_user(User("bob", "b@b.com", 16))

    # Act（执行）- 调用被测方法
    adults = manager.get_adult_users()

    # Assert（断言）- 验证结果
    self.assertEqual(len(adults), 1)
    self.assertEqual(adults[0].username, "alice")
```

---

## 十、练习题

### 练习1：基础断言

为以下函数编写完整的测试用例：

```python
# string_utils.py
def reverse_string(s):
    """反转字符串"""
    return s[::-1]

def is_palindrome(s):
    """判断是否为回文"""
    s = s.lower().replace(" ", "")
    return s == s[::-1]

def count_vowels(s):
    """统计元音字母数量"""
    vowels = "aeiouAEIOU"
    return sum(1 for c in s if c in vowels)
```

**要求**：至少为每个函数写 3 个测试用例，包括正常情况和边界情况。

### 练习2：测试异常

为以下函数编写测试，重点测试异常情况：

```python
def validate_password(password):
    """验证密码强度
    要求：
    1. 长度 >= 8
    2. 包含大写字母
    3. 包含小写字母
    4. 包含数字
    不满足时抛出 ValueError
    """
    if len(password) < 8:
        raise ValueError("密码长度至少8位")
    if not any(c.isupper() for c in password):
        raise ValueError("密码需包含大写字母")
    if not any(c.islower() for c in password):
        raise ValueError("密码需包含小写字母")
    if not any(c.isdigit() for c in password):
        raise ValueError("密码需包含数字")
    return True
```

### 练习3：综合实战

编写一个 `ShoppingCart`（购物车）类并为其编写完整测试：

```python
class ShoppingCart:
    def __init__(self):
        self._items = {}  # {name: {"price": float, "quantity": int}}

    def add_item(self, name, price, quantity=1): ...
    def remove_item(self, name): ...
    def get_total(self) -> float: ...           # 总价
    def get_item_count(self) -> int: ...        # 商品种类数
    def clear(self): ...
    def apply_discount(self, percent) -> float: # 打折后总价
```

**测试要求**：至少 8 个测试方法，覆盖正常流程和异常情况。

---

## 十一、常见问题

### Q1：测试文件和源文件放在一起还是分开？

**推荐分开**，创建一个 `tests/` 目录：

```
myproject/
├── src/
│   ├── calculator.py
│   └── user_manager.py
└── tests/
    ├── __init__.py
    ├── test_calculator.py
    └── test_user_manager.py
```

小项目放一起也完全可以，不必纠结。

### Q2：测试覆盖率要达到 100% 吗？

**不需要**。追求 100% 覆盖率往往得不偿失。优先测试：
1. 核心业务逻辑
2. 边界条件和异常路径
3. 容易出错的复杂逻辑

简单的 getter/setter、配置代码不需要测试。

### Q3：`assertEqual` 和直接用 `assert` 有什么区别？

```python
# 不要这样做！
assert add(1, 2) == 3

# 应该这样做
self.assertEqual(add(1, 2), 3)
```

直接用 `assert` 时，如果运行 `python -O`（优化模式），所有 `assert` 语句会被**直接跳过**，测试就失效了。而 `self.assertEqual` 是 `unittest` 框架的方法，不受优化模式影响。

### Q4：如何测试有随机性的代码？

使用 `mock` 固定随机结果，或者在断言时只验证范围而非精确值：

```python
import random
from unittest.mock import patch

def roll_dice():
    return random.randint(1, 6)

class TestDice(unittest.TestCase):
    @patch("random.randint", return_value=4)
    def test_roll_dice(self, mock_randint):
        self.assertEqual(roll_dice(), 4)
```

### Q5：pytest 和 unittest 该用哪个？

- **unittest**：Python 内置，零安装，适合入门
- **pytest**：第三方库，语法更简洁（直接写 `assert`），插件生态丰富

建议：先学好 unittest 的核心概念（断言、固件、mock），这些概念在 pytest 中同样适用。后续可以平滑切换到 pytest。

---

## 十二、免费学习资源

- [廖雪峰 - 单元测试](https://www.liaoxuefeng.com/wiki/1016959663602400/1017604217253792) - 讲解 unittest 和 pytest
- [菜鸟教程 - Python 单元测试](https://www.runoob.com/python3/python3-unittest.html) - 基础语法和示例
- [Python 官方文档 - unittest](https://docs.python.org/zh-cn/3/library/unittest.html) - 最权威的参考
- [Real Python - Unit Testing](https://realpython.com/python-testing/) - 英文，深入浅出

---

## 总结

| 知识点 | 内容 |
|--------|------|
| **unittest 基础** | TestCase 类、test_ 方法、assertXxx 断言 |
| **测试固件** | setUp / tearDown（方法级）、setUpClass / tearDownClass（类级） |
| **跳过测试** | @skip / @skipIf / @skipUnless / @expectedFailure |
| **mock 模拟** | Mock 对象、@patch 装饰器替换依赖 |
| **运行方式** | `python -m unittest`、discover 自动发现、-v 详细模式 |
| **最佳实践** | AAA 模式、清晰命名、一个测试一件事 |

> **恭喜你完成了第二阶段的最后一个主题！** 接下来将进入第三阶段，学习项目工程化相关的知识：pip 与虚拟环境、项目结构规范、日志系统等。这些是成为一名合格 Python 开发者必备的技能。一名合格 Python 开发者必备的技能。一名合格 Python 开发者必备的技能。