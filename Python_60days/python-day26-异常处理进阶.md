# Python Day 26：异常处理进阶

> **学习目标**：在 Day 13 的基础上，深入掌握自定义异常、异常链、`else`/`finally` 子句、异常层级体系、`warnings` 模块以及异常处理的最佳实践，写出更健壮的 Python 程序。

---

## 一、回顾：Day 13 我们学了什么？

在 Day 13 中，我们学习了异常处理的基础知识：

```python
# 基本的 try-except 结构
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"出错了：{e}")
```

我们已经掌握了：
- `try` / `except` / `else` / `finally` 的基本结构
- 捕获特定异常类型
- `Exception` 基类的使用
- 主动抛出异常 `raise`

今天我们要在这个基础上大幅升级，让异常处理从"能写"变成"写得专业"。

---

## 二、完整的 try 语句结构

很多同学只知道 `try-except`，其实 `try` 语句有 **四个可选子句**。让我们一次性搞清楚：

```python
try:
    # 1. 尝试执行的代码（可能出错的代码）
    result = int("abc")
except ValueError as e:
    # 2. 发生异常时的处理
    print(f"值错误：{e}")
else:
    # 3. 没有发生异常时执行（try 成功了才走这里）
    print(f"结果是：{result}")
finally:
    # 4. 无论是否发生异常，都会执行
    print("这段代码无论如何都会执行")
```

**执行流程图**：

```
try 块开始
    │
    ├── 成功 ──→ else 块 ──→ finally 块 ──→ 结束
    │
    └── 异常 ──→ 匹配的 except 块 ──→ finally 块 ──→ 结束
                                   │
                              不匹配？──→ 程序崩溃
```

### 什么时候用 else？

`else` 的作用是**把"不出错时的逻辑"和"可能出错的代码"分开**，让代码更清晰：

```python
# ❌ 不推荐：成功逻辑放在 try 里
try:
    data = read_file("config.txt")
    process(data)          # 这行本身不会出错，但放在 try 里容易误导
    save_to_db(data)
except FileNotFoundError:
    print("文件不存在")

# ✅ 推荐：成功逻辑放在 else 里
try:
    data = read_file("config.txt")   # 只放可能出错的代码
except FileNotFoundError:
    print("文件不存在")
else:
    process(data)          # 只在 try 成功时执行
    save_to_db(data)       # 逻辑更清晰
```

### finally 的典型场景

`finally` 最常见的用途是**释放资源**，确保即使出错也不会泄露：

```python
def read_config():
    f = None
    try:
        f = open("config.json", "r", encoding="utf-8")
        return f.read()
    except FileNotFoundError:
        print("配置文件不存在，使用默认配置")
        return "{}"
    finally:
        # 无论成功还是失败，都要关闭文件
        if f is not None:
            f.close()
            print("文件已关闭")
```

> **注意**：用 `with` 语句（上下文管理器）可以自动处理资源释放，比手动 `finally` 更优雅。Day 27 会专门讲。

---

## 三、捕获多个异常

在实际开发中，一段代码可能抛出多种异常。Python 提供了几种灵活的捕获方式：

### 方式一：多个 except 块

```python
def safe_divide(a, b):
    try:
        result = float(a) / float(b)
        return result
    except ValueError:
        print("错误：请输入有效的数字")
    except ZeroDivisionError:
        print("错误：除数不能为零")
    except TypeError:
        print("错误：不支持的数据类型")
```

### 方式二：元组方式（多种异常同一处理）

当几种异常的处理方式相同时：

```python
def parse_number(text):
    try:
        return int(text)
    except (ValueError, TypeError) as e:
        # ValueError: 文本不是数字，如 int("abc")
        # TypeError: 传入了不支持类型，如 int([1,2])
        print(f"无法将 '{text}' 转换为数字：{e}")
        return 0
```

### 方式三：捕获所有异常（慎用！）

```python
def do_something():
    try:
        # 各种操作...
        result = 10 / 0
    except Exception as e:
        # Exception 是所有普通异常的基类
        # 但不捕获 KeyboardInterrupt、SystemExit 等系统异常
        print(f"发生了异常：{type(e).__name__}: {e}")
```

> ⚠️ **千万别写裸的 `except:`**（不带任何异常类型），它会捕获所有异常，包括你想用 Ctrl+C 中断程序的情况！

```python
# ❌ 绝对不要这样写
try:
    do_something()
except:
    pass  # 吞掉所有错误，连你自己都不知道出了什么问题

# ✅ 至少捕获 Exception 并记录
try:
    do_something()
except Exception as e:
    print(f"发生错误：{e}")  # 记录日志，方便调试
```

### as 关键字：保存异常对象

```python
try:
    numbers = [1, 2, 3]
    print(numbers[10])
except IndexError as e:
    # e 就是异常对象，包含错误信息
    print(f"索引越界：{e}")
    print(f"异常类型：{type(e).__name__}")   # IndexError
    print(f"异常参数：{e.args}")               # ('list index out of range',)
```

---

## 四、自定义异常类

当 Python 内置异常不能准确描述你的业务错误时，就需要**自定义异常**。

### 基本语法

自定义异常很简单——继承 `Exception` 类即可：

```python
# 自定义年龄验证异常
class AgeError(Exception):
    """当年龄不合法时抛出"""
    def __init__(self, age, message="年龄不合法"):
        self.age = age
        self.message = message
        super().__init__(self.message)  # 调用父类的 __init__

    def __str__(self):
        return f"{self.message}（传入的年龄：{self.age}）"


# 使用自定义异常
def verify_age(age):
    if age < 0:
        raise AgeError(age, "年龄不能为负数")
    if age > 150:
        raise AgeError(age, "年龄超出合理范围")
    return f"年龄 {age} 验证通过"

# 测试
try:
    print(verify_age(25))    # ✅ 年龄 25 验证通过
    print(verify_age(-5))   # ❌ 抛出 AgeError
except AgeError as e:
    print(f"年龄验证失败：{e}")  # 年龄验证失败：年龄不能为负数（传入的年龄：-5）
```

### 构建异常层级体系

在大型项目中，通常会为不同模块定义不同的异常，形成**异常层级体系**：

```python
# 基础异常
class AppError(Exception):
    """应用基础异常"""
    pass

# 用户相关异常
class UserError(AppError):
    """用户相关异常"""
    pass

class UserNotFoundError(UserError):
    """用户不存在"""
    pass

class DuplicateUserError(UserError):
    """用户已存在"""
    pass

# 订单相关异常
class OrderError(AppError):
    """订单相关异常"""
    pass

class InsufficientStockError(OrderError):
    """库存不足"""
    pass

class PaymentError(OrderError):
    """支付失败"""
    pass


# 实际使用
class UserService:
    def __init__(self):
        self.users = {"张三": {"age": 25, "email": "zhangsan@example.com"}}

    def get_user(self, name):
        if name not in self.users:
            raise UserNotFoundError(f"用户 '{name}' 不存在")
        return self.users[name]

    def register(self, name, info):
        if name in self.users:
            raise DuplicateUserError(f"用户 '{name}' 已存在")
        self.users[name] = info
        return f"用户 '{name}' 注册成功"


# 捕获时可以按层级捕获
try:
    service = UserService()
    service.register("张三", {"age": 30, "email": "new@example.com"})
except UserNotFoundError as e:
    print(f"查无此人：{e}")
except DuplicateUserError as e:
    print(f"注册失败：{e}")
except UserError as e:
    # 能捕获所有用户相关异常
    print(f"用户操作出错：{e}")
except AppError as e:
    # 能捕获所有应用异常
    print(f"应用错误：{e}")
```

> **设计原则**：异常类应该是"轻量"的，只存储错误信息，不要在异常中塞入大量业务逻辑。

---

## 五、异常链（Exception Chaining）

当你在处理一个异常时，又引发了新的异常，可以用 `raise ... from ...` 保留原始异常信息。这在调试时非常关键！

### raise from：保留异常原因

```python
def read_config_file(path):
    """读取配置文件"""
    try:
        with open(path, "r", encoding="utf-8") as f:
            return f.read()
    except FileNotFoundError as e:
        # 将底层异常包装成业务异常，保留原因链
        raise ConfigNotFoundError(f"配置文件缺失：{path}") from e


class ConfigNotFoundError(Exception):
    """配置文件不存在"""
    pass


# 测试
try:
    read_config_file("not_exist.json")
except ConfigNotFoundError as e:
    print(f"业务异常：{e}")
    print(f"原始异常：{e.__cause__}")  # FileNotFoundError 的信息
    print(f"完整链：")
    import traceback
    traceback.print_exc()
```

输出示例：
```
业务异常：配置文件缺失：not_exist.json
原始异常：[Errno 2] No such file or directory: 'not_exist.json'
完整链：
Traceback (most recent call last):
  ...
  File "not_exist.json" 通过 raise from 关联
ConfigNotFoundError: 配置文件缺失：not_exist.json
```

### 默认异常链

如果你在 `except` 块中直接 `raise` 一个新异常，Python 会**自动**建立异常链：

```python
def parse_json(text):
    try:
        import json
        return json.loads(text)
    except json.JSONDecodeError:
        raise ValueError("JSON 格式无效")
    # ↑ Python 自动设置 __cause__ 为原始的 JSONDecodeError
```

### 抑制异常链

如果你不想显示原始异常，使用 `raise ... from None`：

```python
try:
    result = int("hello")
except ValueError:
    raise RuntimeError("输入必须是数字") from None
    # 输出中不会显示原始的 ValueError
```

---

## 六、traceback 模块——调试利器

`traceback` 模块提供了获取和格式化异常信息的高级工具：

### 获取异常信息（不中断程序）

```python
import traceback
import sys

def risky_operation():
    return 1 / 0

def main():
    try:
        risky_operation()
    except Exception:
        # 方式一：打印完整堆栈（不终止程序）
        traceback.print_exc()

        # 方式二：获取堆栈为字符串（可用于写入日志）
        error_msg = traceback.format_exc()
        print(f"\n捕获到的错误信息：\n{error_msg}")

        # 方式三：获取当前异常的类型和值
        exc_type, exc_value, exc_tb = sys.exc_info()
        print(f"类型：{exc_type.__name__}")
        print(f"信息：{exc_value}")
        print(f"行号：{exc_tb.tb_lineno}")

main()
```

### 输出美化

```python
import traceback

def format_error(func, *args, **kwargs):
    """统一格式化异常输出"""
    try:
        return func(*args, **kwargs)
    except Exception:
        # 获取格式化的异常字符串
        tb_str = traceback.format_exc()
        # 只显示最后 3 层
        tb_lines = tb_str.strip().split('\n')
        short_tb = '\n'.join(tb_lines[-6:])  # 最后6行
        return f"❌ 执行失败:\n{short_tb}"

# 测试
result = format_error(int, "not a number")
print(result)
```

---

## 七、warnings 模块——非致命警告

有些情况"不算错误，但值得注意"，比如使用了过时的函数。这时用 `warnings` 比 `raise` 更合适：

```python
import warnings

# 发出警告
def old_function():
    warnings.warn(
        "old_function() 已过时，请使用 new_function() 代替",
        DeprecationWarning,
        stacklevel=2
    )
    return "这是旧版本的实现"

# 调用时会显示警告
print(old_function())
# 输出：xxx.py:xx: DeprecationWarning: old_function() 已过时，请使用 new_function() 代替
```

### 警告类型

| 警告类型 | 说明 | 使用场景 |
|---------|------|---------|
| `DeprecationWarning` | 弃用警告 | 旧函数/旧接口即将移除 |
| `UserWarning` | 用户警告 | 自定义的业务警告 |
| `RuntimeWarning` | 运行时警告 | 可疑的运行时行为 |
| `FutureWarning` | 未来变更 | 未来版本行为会改变 |
| `ImportWarning` | 导入警告 | 导入有问题 |

### 控制警告显示

```python
import warnings

# 忽略特定类型的警告
warnings.filterwarnings("ignore", category=DeprecationWarning)

# 将警告转为异常（测试时很有用）
warnings.filterwarnings("error", category=UserWarning)

# 只显示一次
warnings.filterwarnings("once", category=UserWarning)
```

---

## 八、异常处理最佳实践

### ✅ DO：应该做的

**1. 只捕获预期的异常**

```python
# ✅ 只捕获你预期会发生的异常
try:
    value = int(user_input)
except ValueError:
    print("请输入数字")

# ❌ 裸 except 会捕获一切，包括 KeyboardInterrupt
try:
    value = int(user_input)
except:
    pass
```

**2. 异常信息要具体有用**

```python
# ✅ 错误信息包含上下文
try:
    with open(filepath, "r") as f:
        content = f.read()
except FileNotFoundError:
    print(f"配置文件 {filepath} 不存在，请检查路径")

# ❌ 错误信息含糊不清
try:
    with open(filepath, "r") as f:
        content = f.read()
except Exception:
    print("出错了")
```

**3. 尽早抛出，高层捕获**

```python
# ✅ 底层函数抛出异常，上层函数决定如何处理
def load_data(filename):
    if not os.path.exists(filename):
        raise FileNotFoundError(f"文件不存在: {filename}")
    # ... 读取逻辑

def main():
    for filename in config["data_files"]:
        try:
            data = load_data(filename)
        except FileNotFoundError:
            log_warning(f"跳过缺失文件: {filename}")
            continue
        process(data)
```

**4. 使用自定义异常区分业务错误**

```python
# ✅ 业务代码用自定义异常，而非内置类型
class InsufficientBalanceError(Exception):
    pass

def withdraw(account, amount):
    if account.balance < amount:
        raise InsufficientBalanceError(
            f"余额不足：当前 {account.balance}，需要 {amount}"
        )
    account.balance -= amount
```

### ❌ DON'T：不该做的

**1. 不要吞掉异常**

```python
# ❌ 完全吞掉，出问题都不知道
try:
    process_data()
except:
    pass  # 这里什么都没做！

# ✅ 至少记录日志
try:
    process_data()
except Exception as e:
    logging.error(f"数据处理失败: {e}")
```

**2. 不要用异常控制正常流程**

```python
# ❌ 用异常来控制流程，性能差且难以阅读
def get_first(items):
    try:
        return items[0]
    except IndexError:
        return None

# ✅ 用正常的条件判断
def get_first(items):
    return items[0] if items else None
```

**3. 不要在 finally 中 return**

```python
# ❌ finally 中的 return 会覆盖 try 中的 return
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return float('inf')
    finally:
        return 0  # ← 永远返回 0，上面的 return 全被覆盖！

# ✅ finally 只做清理工作
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return float('inf')
```

---

## 九、综合实战：带完整异常处理的简易银行系统

```python
"""
简易银行系统 - 异常处理进阶实战
演示：自定义异常、异常链、异常层级、最佳实践
"""


# ========== 自定义异常体系 ==========
class BankError(Exception):
    """银行系统基础异常"""
    pass


class InsufficientBalanceError(BankError):
    """余额不足"""
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(
            f"余额不足：当前余额 ¥{balance:.2f}，"
            f"取款金额 ¥{amount:.2f}"
        )


class NegativeAmountError(BankError):
    """金额为负数"""
    pass


class AccountNotFoundError(BankError):
    """账户不存在"""
    pass


class AccountLockedError(BankError):
    """账户已锁定"""
    pass


# ========== 银行账户类 ==========
class BankAccount:
    """银行账户"""

    def __init__(self, account_id, name, balance=0.0):
        self.account_id = account_id
        self.name = name
        self.balance = balance
        self.is_locked = False
        self.transaction_history = []

    def deposit(self, amount):
        """存款"""
        if self.is_locked:
            raise AccountLockedError(
                f"账户 {self.account_id} 已锁定，无法操作"
            )
        if amount <= 0:
            raise NegativeAmountError(
                f"存款金额必须为正数，传入值：{amount}"
            )
        self.balance += amount
        self.transaction_history.append(
            f"存入 ¥{amount:.2f}，余额 ¥{self.balance:.2f}"
        )
        return self.balance

    def withdraw(self, amount):
        """取款"""
        if self.is_locked:
            raise AccountLockedError(
                f"账户 {self.account_id} 已锁定，无法操作"
            )
        if amount <= 0:
            raise NegativeAmountError(
                f"取款金额必须为正数，传入值：{amount}"
            )
        if self.balance < amount:
            raise InsufficientBalanceError(self.balance, amount)
        self.balance -= amount
        self.transaction_history.append(
            f"取出 ¥{amount:.2f}，余额 ¥{self.balance:.2f}"
        )
        return self.balance

    def transfer(self, target, amount):
        """转账"""
        try:
            self.withdraw(amount)
        except BankError as e:
            # 用 raise from 保留原始异常
            raise BankError(
                f"从 {self.name} 转账失败"
            ) from e
        try:
            target.deposit(amount)
        except BankError as e:
            # 转入失败，把钱退回来
            self.balance += amount
            raise BankError(
                f"向 {target.name} 入账失败，已退回"
            ) from e
        self.transaction_history.append(
            f"转给 {target.name} ¥{amount:.2f}"
        )

    def __str__(self):
        status = "🔒 已锁定" if self.is_locked else "✅ 正常"
        return f"[{status}] {self.name}({self.account_id}) 余额: ¥{self.balance:.2f}"


# ========== 银行管理类 ==========
class Bank:
    """银行管理系统"""

    def __init__(self):
        self.accounts = {}

    def create_account(self, account_id, name, initial_balance=0.0):
        """创建账户"""
        if account_id in self.accounts:
            raise BankError(f"账户 {account_id} 已存在")
        account = BankAccount(account_id, name, initial_balance)
        self.accounts[account_id] = account
        return account

    def get_account(self, account_id):
        """获取账户"""
        if account_id not in self.accounts:
            raise AccountNotFoundError(
                f"账户 {account_id} 不存在"
            )
        return self.accounts[account_id]

    def lock_account(self, account_id):
        """锁定账户"""
        account = self.get_account(account_id)
        account.is_locked = True
        return f"账户 {account_id} 已锁定"

    def show_summary(self):
        """显示所有账户摘要"""
        total = sum(a.balance for a in self.accounts.values())
        print(f"\n{'='*50}")
        print(f"  银行总览：共 {len(self.accounts)} 个账户，"
              f"总余额 ¥{total:.2f}")
        print(f"{'='*50}")
        for acc in self.accounts.values():
            print(f"  {acc}")
        print(f"{'='*50}\n")


# ========== 主程序 ==========
def main():
    bank = Bank()

    try:
        # 创建账户
        bank.create_account("001", "张三", 1000.0)
        bank.create_account("002", "李四", 500.0)
        print("✅ 账户创建成功")

        # 存款
        zhang = bank.get_account("001")
        zhang.deposit(500)
        print("✅ 存款成功")

        # 取款
        zhang.withdraw(200)
        print("✅ 取款成功")

        # 转账
        li = bank.get_account("002")
        zhang.transfer(li, 300)
        print("✅ 转账成功")

        # 查看摘要
        bank.show_summary()

    except InsufficientBalanceError as e:
        print(f"💸 {e}")
    except NegativeAmountError as e:
        print(f"🔢 {e}")
    except AccountNotFoundError as e:
        print(f"❓ {e}")
    except AccountLockedError as e:
        print(f"🔒 {e}")
    except BankError as e:
        print(f"🏦 银行系统错误：{e}")
        if e.__cause__:
            print(f"   原因：{e.__cause__}")
    except Exception as e:
        print(f"💥 未知错误：{type(e).__name__}: {e}")
    else:
        print("\n🎉 所有操作成功完成！")
    finally:
        print("📋 操作结束，感谢使用银行系统")


if __name__ == "__main__":
    main()
```

---

## 十、练习题

### 练习 1：完善异常处理

下面代码有多个潜在的错误，请添加完善的异常处理：

```python
def read_scores(filepath):
    """从文件读取学生成绩，计算平均分"""
    f = open(filepath, "r")
    lines = f.readlines()
    scores = []
    for line in lines:
        score = int(line.strip())
        scores.append(score)
    avg = sum(scores) / len(scores)
    f.close()
    return avg
```

要求：
- 处理文件不存在的错误
- 处理文件内容不是数字的错误
- 处理文件为空的错误
- 确保文件一定会被关闭

<details>
<summary>参考答案</summary>

```python
def read_scores(filepath):
    """从文件读取学生成绩，计算平均分"""
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            lines = f.readlines()
    except FileNotFoundError:
        raise FileNotFoundError(f"成绩文件 {filepath} 不存在")

    scores = []
    for i, line in enumerate(lines, 1):
        try:
            score = int(line.strip())
            scores.append(score)
        except ValueError:
            print(f"警告：第 {i} 行 '{line.strip()}' 不是有效数字，已跳过")

    if not scores:
        raise ValueError(f"文件中没有有效的成绩数据")

    avg = sum(scores) / len(scores)
    return avg
```

</details>

### 练习 2：自定义异常体系

为一个电商系统设计异常体系：
- 基础异常 `EcommerceError`
- 商品相关：`ProductNotFoundError`、`OutOfStockError`
- 订单相关：`OrderError`、`InvalidOrderStatusError`
- 支付相关：`PaymentError`、`PaymentTimeoutError`

然后实现一个 `place_order(product_id, quantity)` 函数，在合适的位置抛出这些异常。

<details>
<summary>参考答案</summary>

```python
# 异常体系
class EcommerceError(Exception):
    """电商系统基础异常"""
    pass

class ProductError(EcommerceError):
    """商品相关异常"""
    pass

class ProductNotFoundError(ProductError):
    pass

class OutOfStockError(ProductError):
    def __init__(self, product_id, requested, available):
        self.product_id = product_id
        self.requested = requested
        self.available = available
        super().__init__(
            f"商品 {product_id} 库存不足：需要 {requested}，"
            f"仅剩 {available}"
        )

class OrderError(EcommerceError):
    """订单相关异常"""
    pass

class InvalidOrderStatusError(OrderError):
    pass

class PaymentError(EcommerceError):
    """支付相关异常"""
    pass

class PaymentTimeoutError(PaymentError):
    pass


# 业务逻辑
class ProductStore:
    def __init__(self):
        self.products = {
            "P001": {"name": "Python 入门书", "price": 59.9, "stock": 10},
            "P002": {"name": "机械键盘", "price": 299.0, "stock": 3},
        }

    def check_product(self, product_id):
        if product_id not in self.products:
            raise ProductNotFoundError(f"商品 {product_id} 不存在")
        return self.products[product_id]

    def check_stock(self, product_id, quantity):
        product = self.check_product(product_id)
        if product["stock"] < quantity:
            raise OutOfStockError(
                product_id, quantity, product["stock"]
            )

    def place_order(self, product_id, quantity):
        self.check_stock(product_id, quantity)
        product = self.products[product_id]
        product["stock"] -= quantity
        total = product["price"] * quantity
        print(f"下单成功：{product['name']} x{quantity}，"
              f"合计 ¥{total:.2f}")
        return total


# 使用
store = ProductStore()
try:
    store.place_order("P001", 2)    # ✅
    store.place_order("P999", 1)    # ❌ ProductNotFoundError
except OutOfStockError as e:
    print(f"库存不足：{e}")
except ProductNotFoundError as e:
    print(f"商品不存在：{e}")
except EcommerceError as e:
    print(f"系统错误：{e}")
```

</details>

### 练习 3：异常链追踪

写一个函数 `load_and_parse_config(filepath)`：
1. 读取 JSON 配置文件
2. 验证配置中的必要字段
3. 如果任何一步出错，抛出带完整异常链的 `ConfigError`

<details>
<summary>参考答案</summary>

```python
import json

class ConfigError(Exception):
    """配置相关异常"""
    pass

REQUIRED_FIELDS = ["host", "port", "database"]

def load_and_parse_config(filepath):
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            content = f.read()
    except FileNotFoundError as e:
        raise ConfigError(f"配置文件 {filepath} 不存在") from e
    except PermissionError as e:
        raise ConfigError(f"无权限读取 {filepath}") from e

    try:
        config = json.loads(content)
    except json.JSONDecodeError as e:
        raise ConfigError(f"配置文件格式无效") from e

    if not isinstance(config, dict):
        raise ConfigError("配置文件根元素必须是 JSON 对象")

    missing = [f for f in REQUIRED_FIELDS if f not in config]
    if missing:
        raise ConfigError(
            f"缺少必要配置项：{', '.join(missing)}"
        )

    return config


# 测试
try:
    config = load_and_parse_config("missing_file.json")
except ConfigError as e:
    print(f"配置加载失败：{e}")
    if e.__cause__:
        print(f"根本原因：{e.__cause__}")
```

</details>

---

## 十一、常见问题（FAQ）

**Q1：`Exception` 和 `BaseException` 有什么区别？**

`BaseException` 是所有异常的根类，包括 `KeyboardInterrupt`（Ctrl+C）、`SystemExit` 等系统级异常。`Exception` 是 `BaseException` 的子类，只包含"普通"的程序错误。日常开发中应该捕获 `Exception`，不要捕获 `BaseException`，否则连 Ctrl+C 都会被拦截。

**Q2：什么时候用 `raise`，什么时候用 `print` 提示错误？**

- `raise`：用于**函数内部**，告诉调用者"出问题了，你来决定怎么处理"
- `print`：用于**程序入口/交互界面**，给用户展示友好的错误提示
- 原则：底层抛异常，顶层捕获并展示

**Q3：`except Exception as e` 和 `except Exception`（不带 as）有什么区别？**

带 `as e` 可以在 except 块中引用异常对象（查看错误信息、获取类型等）。如果你只需要"出错就做某事"，不需要看错误细节，可以不带 `as`。

**Q4：为什么说"不要用异常控制流程"？**

因为异常处理的性能比正常的 if/else 低很多（Python 创建异常对象需要分配内存、记录堆栈等）。只在"真正的异常情况"下才用 try-except，正常的业务判断用 if/else。

---

## 十二、今日总结

| 知识点 | 要点 |
|-------|------|
| `else` 子句 | 只在 try 成功时执行，保持代码清晰 |
| `finally` 子句 | 无论如何都执行，用于资源清理 |
| 捕获多个异常 | 元组方式 `(A, B)` 或多个 except 块 |
| 自定义异常 | 继承 `Exception`，构建异常层级 |
| 异常链 `raise from` | 保留原始异常，方便调试 |
| `raise from None` | 切断异常链，隐藏底层细节 |
| `traceback` 模块 | 获取/格式化堆栈信息 |
| `warnings` 模块 | 发出非致命警告 |
| 最佳实践 | 只捕获预期异常、不吞掉异常、不用异常控制流程 |

> **下节预告**：Day 27 将学习**上下文管理器**（Context Manager），也就是 `with` 语句背后的原理。我们会学到如何自己实现一个上下文管理器，让资源管理变得更优雅。

---

## 免费学习资源

- [廖雪峰 Python 教程 - 错误处理](https://www.liaoxuefeng.com/wiki/1016959663602400/1017571665842464)
- [菜鸟教程 - Python 异常处理](https://www.runoob.com/python3/python3-errors.html)
- [Python 官方文档 - 内置异常](https://docs.python.org/zh-cn/3/library/exceptions.html)
- [Real Python - Python Exceptions](https://realpython.com/python-exceptions/)
