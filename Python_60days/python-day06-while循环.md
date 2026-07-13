# 🐍 Python 60天学习计划 — Day 6：while 循环 + 猜数字游戏

> **当前进度**：Day 6 / 60 | **阶段**：第一阶段 - 基础语法入门（Day 1-14）
> **今日投入**：约 1 小时 | **上节课**：for 循环、range()、break/continue

---

## 一、知识回顾（3分钟）

前面你已经掌握了两大控制流工具：

| 工具 | 用途 | 关键语法 |
|------|------|------|
| `if/elif/else` | **条件判断** | 满足条件才执行 |
| `for` 循环 | **已知次数**的遍历 | `for i in range(10)` |

今天的 `while` 循环解决另一种场景：**不知道要循环多少次，只知道什么时候停**。

---

## 二、while 循环基础（10分钟）

### 2.1 语法结构

```python
while 条件:
    循环体（条件为 True 时反复执行）
```

**执行逻辑**：

```
进入 while → 检查条件 → True? 执行循环体 → 再次检查条件 → True? 再执行 → ... → False? 退出
```

### 2.2 第一个例子：数到 5

```python
count = 1
while count <= 5:
    print(f"第 {count} 次循环")
    count += 1          # ⚠️ 容易忘！不加这行会死循环
print("循环结束！")
```

**输出**：
```
第 1 次循环
第 2 次循环
第 3 次循环
第 4 次循环
第 5 次循环
循环结束！
```

### 2.3 while 的核心三要素（缺一不可）

```python
# ① 初始条件（在 while 之前）
i = 0

# ② 循环条件（while 后面）
while i < 10:

    # ③ 条件变化（循环体内部）
    i += 1           # 让 i 慢慢逼近 10，最终退出
```

> ⚠️ **头号坑**：忘了写条件变化 → 死循环 → 按 `Ctrl+C` 强制停止！

---

## 三、while vs for — 什么时候用谁？（8分钟）

### 对比表

| 场景 | 用 for | 用 while |
|------|--------|----------|
| 遍历列表/字符串 | ✅ `for item in items` | ❌ 别扭 |
| 固定次数循环 | ✅ `for i in range(n)` | ⚠️ 可以但不简洁 |
| **不确定次数，等条件满足** | ❌ | ✅ `while 条件` |
| 用户输入验证 | ❌ | ✅ |
| 游戏主循环 | ❌ | ✅ |
| 读取文件到末尾 | ✅ 也可以 | ✅ `while True + break` |

### 同一个任务，两种写法

**【任务】打印 0 到 9**

```python
# for 版本（知道次数）
for i in range(10):
    print(i)

# while 版本
i = 0
while i < 10:
    print(i)
    i += 1
```

**【任务】让用户一直输入，直到输入 "quit"**

```python
# while 版本（自然）
while True:
    cmd = input("请输入命令（quit退出）：")
    if cmd == "quit":
        break
    print(f"执行：{cmd}")

# for 做不到，因为你不知道用户会输入多少次
```

> 💡 **一句话总结**：知道循环次数用 `for`，不知道用 `while`。

---

## 四、break / continue 在 while 中（5分钟）

和 for 循环完全一样：

```python
# break：立即退出整个循环
while True:
    user_input = input("输入 q 退出：")
    if user_input == "q":
        print("拜拜！")
        break                    # 直接跳出 while
    print(f"你输入了：{user_input}")

# continue：跳过本轮，进入下一次判断
num = 0
while num < 10:
    num += 1
    if num % 3 == 0:
        continue                 # 跳过 3 的倍数
    print(num)                   # 输出 1 2 4 5 7 8 10
```

---

## 五、while...else 结构（3分钟）

```python
count = 0
while count < 3:
    print(count)
    count += 1
else:
    print("循环正常结束！")   # 条件变为 False 时执行

# 输出：
# 0
# 1
# 2
# 循环正常结束！
```

**关键区别**：如果是 `break` 跳出，`else` **不会**执行：

```python
count = 0
while count < 10:
    print(count)
    if count == 3:
        break          # break 退出
    count += 1
else:
    print("这句不会执行！")   # ← 被 break 跳过了
```

---

## 六、实战项目：猜数字游戏（20分钟）🎯

### 6.1 基础版：猜一次

```python
import random

answer = random.randint(1, 100)    # 生成 1~100 的随机数
guess = int(input("猜一个 1~100 的数字："))

if guess == answer:
    print("🎉 一次就猜中了！")
elif guess > answer:
    print(f"大了！答案是 {answer}")
else:
    print(f"小了！答案是 {answer}")
```

### 6.2 进阶版：猜到对为止（用 while）

```python
import random

answer = random.randint(1, 100)
guess = 0           # 初始化一个不可能的值
count = 0           # 计数器

while guess != answer:
    guess = int(input("猜一个 1~100 的数字："))
    count += 1

    if guess > answer:
        print("📉 大了，再试试！")
    elif guess < answer:
        print("📈 小了，再试试！")
    else:
        print(f"🎉 恭喜！你用了 {count} 次猜中了答案 {answer}！")
```

### 6.3 完整版：加入重玩功能

```python
import random

while True:                              # 外层循环：重玩
    answer = random.randint(1, 100)
    guess = 0
    count = 0

    print("\n=== 🎯 猜数字游戏 ===")
    print("我已经想好了一个 1~100 之间的数字")

    while guess != answer:               # 内层循环：猜中为止
        guess = int(input("你猜是多少？"))
        count += 1

        if guess > answer:
            print("📉 大了！")
        elif guess < answer:
            print("📈 小了！")
        else:
            print(f"\n🎉 猜中了！答案是 {answer}")
            if count == 1:
                print("天选之人，一发入魂！")
            elif count <= 5:
                print(f"用了 {count} 次，运气不错！")
            elif count <= 10:
                print(f"用了 {count} 次，还可以哦～")
            else:
                print(f"用了 {count} 次，下次加油！")

    # 重玩判断
    replay = input("\n再来一局？(y/n)：").lower()
    if replay != "y":
        print("游戏结束，再见！👋")
        break                            # 退出外层循环

print("程序退出。")
```

---

## 七、今日练习（15分钟）✏️

### 练习 1：密码验证（必做）

要求用户输入密码，密码是 `"python123"`，最多给 3 次机会。

```python
# 你的代码写在这里
correct_password = "python123"
# ...
```

<details>
<summary>✅ 参考答案</summary>

```python
correct_password = "python123"
max_attempts = 3
attempts = 0

while attempts < max_attempts:
    password = input("请输入密码：")
    attempts += 1

    if password == correct_password:
        print("✅ 密码正确，欢迎！")
        break
    else:
        remaining = max_attempts - attempts
        if remaining > 0:
            print(f"❌ 密码错误，还剩 {remaining} 次机会")
        else:
            print("❌ 密码错误，账户已锁定！")
```
</details>

### 练习 2：数字反转（必做）

输入一个正整数，输出反转后的数字。例如输入 `1234` 输出 `4321`。

```python
# 提示：用 while + 取余 %
num = int(input("请输入一个正整数："))
# ...
```

<details>
<summary>✅ 参考答案</summary>

```python
num = int(input("请输入一个正整数："))
reversed_num = 0

while num > 0:
    last_digit = num % 10         # 取最后一位
    reversed_num = reversed_num * 10 + last_digit  # 拼到结果后面
    num = num // 10               # 去掉最后一位

print(f"反转后：{reversed_num}")

# 执行过程示例（输入 1234）：
# num=1234 → last=4 → rev=4     → num=123
# num=123  → last=3 → rev=43    → num=12
# num=12   → last=2 → rev=432   → num=1
# num=1    → last=1 → rev=4321  → num=0 → 退出
```
</details>

### 练习 3：累加到目标（选做）

从 1 开始累加：1+2+3+...，问加到第几个数时总和超过 1000？

```python
# 用 while 循环
sum = 0
n = 0

while sum <= 1000:
    n += 1
    sum += n

print(f"1 加到 {n} 时总和为 {sum}，首次超过 1000")
```
> 答案应该是：1 加到 45 时总和为 1035

---

## 八、常见坑 & 避坑指南（5分钟）⚠️

| 问题 | 原因 | 解决办法 |
|------|------|----------|
| **死循环** | 忘了更新循环变量 | 检查循环体内是否有 `i += 1` 或等价操作 |
| **条件永远为 True** | 条件写错，如 `while i = 5`（赋值） | 比较用 `==`，赋值为 `=` |
| **少循环一次** | 边界条件搞反，如 `while i < 10` vs `while i <= 10` | 用纸笔模拟前几次循环 |
| **break 后 else 不执行** | 这是 Python 的**设计行为**！ | 需要"正常结束才执行"的逻辑时才用 `else` |
| **input() 返回的是字符串** | `guess = input()` 得到 `"50"` 而非 `50` | 用 `int()` 转换：`guess = int(input())` |

---

## 九、免费资源推荐 📚

| 资源 | 说明 | 链接 |
|------|------|------|
| **Python官方教程 - 流程控制** | 官方 while 教程，最权威 | https://docs.python.org/zh-cn/3/tutorial/controlflow.html |
| **廖雪峰 Python - 循环** | 中文讲解，适合初学者 | https://www.liaoxuefeng.com/wiki/1016959663602400/1017100774566304 |
| **菜鸟教程 - while 循环** | 带在线编辑器，边学边练 | https://www.runoob.com/python/python-while-loop.html |

---

## 十、今日小结 ✅

```
✅ 学会了             ⚠️ 注意
─────────────────────────────────────────────
while 循环语法        死循环陷阱（必加条件变化）
while vs for 选择     边界条件（< vs <=）
break/continue        input() 返回值类型
while...else          用 Ctrl+C 中断死循环
完整猜数字游戏         break 会跳过 else
```

---

## 📅 明日预告：Day 7 — 列表（list）入门

列表是 Python 最强大的数据结构之一。明天你将学到：

- 列表的创建和索引（`[ ]`、`[0]`、`[-1]`）
- 列表的增删改查（`append`、`remove`、`pop`）
- 用 for 循环遍历列表
- 实战：待办事项管理器

> **坚持 6 天了，很棒！** 变量 → 条件 → 循环 → 现在你已掌握了编程的三大控制结构，明天开始学习怎么**组织和存储数据**。💪
