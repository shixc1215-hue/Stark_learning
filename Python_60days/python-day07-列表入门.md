# 🐍 Python 60天学习计划 — Day 7：列表（list）入门

> **当前进度**：Day 7 / 60 | **阶段**：第一阶段 - 基础语法入门（Day 1-14）
> **今日投入**：约 1 小时 | **上节课**：while 循环、猜数字游戏

---

## 一、知识回顾（3分钟）

到目前为止你已经掌握了 Python 的核心骨架：

| 已学内容 | 核心能力 |
|----------|----------|
| 变量 + 数据类型 | 存一个值：`name = "小明"` |
| if/elif/else | 做判断 |
| for 循环 | 已知次数重复做 |
| while 循环 | 条件满足就重复做 |

**问题来了**：如果我想存 100 个学生的名字怎么办？写 100 个变量？

```python
# 😱 这样太蠢了
name1 = "小明"
name2 = "小红"
name3 = "小华"
# ... 写到 name100？
```

今天学的 **列表（list）** 就是用来解决"存一堆东西"这个问题的。

---

## 二、什么是列表？（5分钟）

列表就是**一个变量里装多个值**，用方括号 `[]` 包裹，逗号分隔：

```python
# 创建列表
fruits = ["苹果", "香蕉", "橘子", "葡萄"]
scores = [90, 85, 78, 92, 88]
mixed = [1, "hello", True, 3.14]      # 列表可以混合类型
empty = []                              # 空列表
```

**关键概念**：列表里的每一个值叫 **元素**，每个元素有个 **编号**（从 0 开始）。

```
fruits = ["苹果", "香蕉", "橘子", "葡萄"]
            ↑       ↑       ↑       ↑
          索引 0   索引 1   索引 2   索引 3

            ↑                            ↑
          第一个                        最后一个
```

> 💡 Python 的索引从 **0** 开始！这是编程界的通用规则，新手最容易犯的错就是以为第一个是索引 1。

---

## 三、访问元素：索引（10分钟）

### 3.1 正向索引（从左往右，从 0 开始）

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄"]

print(fruits[0])     # 苹果   ← 第一个元素
print(fruits[1])     # 香蕉
print(fruits[2])     # 橘子
print(fruits[3])     # 葡萄
# print(fruits[4])   # ❌ IndexError! 越界了，一共只有 0~3
```

### 3.2 负向索引（从右往左，从 -1 开始）

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄"]

print(fruits[-1])    # 葡萄   ← 最后一个元素
print(fruits[-2])    # 橘子
print(fruits[-3])    # 香蕉
print(fruits[-4])    # 苹果
```

**记忆技巧**：`-1` 就是"倒数第一个"，`-2` 就是"倒数第二个"。

### 3.3 获取列表长度

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄"]

print(len(fruits))   # 4

# 所以最后一个元素可以这样取：
print(fruits[len(fruits) - 1])   # 葡萄（和 fruits[-1] 一样）
```

> ⚠️ **新手坑**：列表有 4 个元素，索引范围是 `0~3`，不是 `1~4`！`fruits[4]` 会报 `IndexError`。

---

## 四、列表的增删改查（15分钟）

### 4.1 修改元素

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄"]
fruits[1] = "草莓"              # 把香蕉改成草莓
print(fruits)                    # ["苹果", "草莓", "橘子", "葡萄"]
```

### 4.2 添加元素

```python
fruits = ["苹果", "香蕉"]

# append：在末尾添加一个
fruits.append("橘子")
print(fruits)                    # ["苹果", "香蕉", "橘子"]

# insert：在指定位置插入
fruits.insert(1, "草莓")         # 在索引 1 的位置插入
print(fruits)                    # ["苹果", "草莓", "香蕉", "橘子"]

# extend：把另一个列表的元素加进来
more_fruits = ["葡萄", "西瓜"]
fruits.extend(more_fruits)
print(fruits)                    # ["苹果", "草莓", "香蕉", "橘子", "葡萄", "西瓜"]
```

### 4.3 删除元素

```python
fruits = ["苹果", "草莓", "香蕉", "橘子"]

# remove：按值删除（删第一个匹配的）
fruits.remove("草莓")
print(fruits)                    # ["苹果", "香蕉", "橘子"]

# pop：按索引删除，并返回被删的值
deleted = fruits.pop(1)          # 删索引 1（香蕉）
print(fruits)                    # ["苹果", "橘子"]
print(f"被删的是：{deleted}")      # 被删的是：香蕉

# pop() 不传参数 → 删最后一个
fruits.pop()
print(fruits)                    # ["苹果"]
```

### 4.4 查找元素

```python
fruits = ["苹果", "香蕉", "橘子", "香蕉", "葡萄"]

# in：判断元素是否存在
print("苹果" in fruits)          # True
print("西瓜" in fruits)          # False

# index：找到元素的位置（第一个匹配）
print(fruits.index("香蕉"))      # 1
# fruits.index("西瓜")           # ❌ ValueError: 不存在

# count：统计出现次数
print(fruits.count("香蕉"))      # 2
```

### 📋 方法速查表

| 方法 | 作用 | 示例 |
|------|------|------|
| `append(x)` | 末尾添加一个 | `fruits.append("西瓜")` |
| `insert(i, x)` | 指定位置插入 | `fruits.insert(0, "西瓜")` |
| `extend(list)` | 合并另一个列表 | `fruits.extend(other)` |
| `remove(x)` | 按值删除 | `fruits.remove("苹果")` |
| `pop(i)` | 按索引删除 | `fruits.pop(0)` |
| `index(x)` | 查找索引 | `fruits.index("苹果")` |
| `count(x)` | 计数 | `fruits.count("苹果")` |
| `sort()` | 排序（原地） | `scores.sort()` |
| `reverse()` | 反转（原地） | `fruits.reverse()` |
| `clear()` | 清空列表 | `fruits.clear()` |

---

## 五、列表切片（8分钟）

切片 = **取列表的一部分**，语法是 `[start:stop:step]`。

```python
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# [start:stop] — 取从 start 到 stop-1（⚠️ 不含 stop！）
print(nums[2:5])       # [2, 3, 4]
print(nums[0:3])       # [0, 1, 2]
print(nums[:3])        # [0, 1, 2]    ← start 省略 = 从头
print(nums[7:])        # [7, 8, 9]    ← stop 省略 = 到尾
print(nums[:])         # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]  ← 全部（复制一份）

# step：步长（间隔取）
print(nums[::2])       # [0, 2, 4, 6, 8]    ← 每隔 2 个取一个
print(nums[1::2])      # [1, 3, 5, 7, 9]    ← 奇数位
print(nums[::-1])      # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]  ← 反转！
```

**记忆公式**：`[起点:终点:步长]`，**起点包含，终点不包含**。

> 💡 切片不会报错——越界了就取到边界为止，这比索引安全多了。

---

## 六、用 for 循环遍历列表（5分钟）

这是最常用的操作：

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄"]

# 方式 1：直接遍历元素
for fruit in fruits:
    print(f"我喜欢吃{fruit}")

# 方式 2：需要索引时用 enumerate
for i, fruit in enumerate(fruits):
    print(f"第 {i+1} 个水果是{fruit}")

# 方式 3：用 range + 索引（旧写法，不推荐）
for i in range(len(fruits)):
    print(fruits[i])
```

> 💡 **推荐用 `enumerate()`**，既能拿到索引又能拿到元素，代码更清晰。

---

## 七、列表排序（5分钟）

```python
scores = [88, 72, 95, 60, 83]

# sort()：原地排序（改变原列表）
scores.sort()                    # 默认从小到大
print(scores)                    # [60, 72, 83, 88, 95]

scores.sort(reverse=True)        # 从大到小
print(scores)                    # [95, 88, 83, 72, 60]

# sorted()：返回新列表（原列表不变）
original = [3, 1, 2]
new_list = sorted(original)
print(original)                  # [3, 1, 2]   ← 没变
print(new_list)                  # [1, 2, 3]   ← 新的

# 字符串排序（按字母顺序）
fruits = ["grape", "apple", "cherry", "banana"]
fruits.sort()
print(fruits)                    # ['apple', 'banana', 'cherry', 'grape']
```

---

## 八、实战项目：待办事项管理器（15分钟）🎯

把今天学的知识全部用起来：

```python
tasks = []          # 存储所有待办事项的列表

print("=== 📋 待办事项管理器 ===")
print("命令：add(添加) / show(查看) / done(完成) / quit(退出)")

while True:
    print()
    cmd = input("请输入命令：").strip().lower()

    if cmd == "add":
        task = input("输入待办事项：").strip()
        if task:                     # 不允许空内容
            tasks.append(task)
            print(f"✅ 已添加：{task}（共 {len(tasks)} 项）")
        else:
            print("❌ 内容不能为空")

    elif cmd == "show":
        if not tasks:                # 列表为空时
            print("📭 暂无待办事项")
        else:
            print(f"\n📋 待办列表（共 {len(tasks)} 项）：")
            print("-" * 30)
            for i, task in enumerate(tasks):
                print(f"  {i + 1}. {task}")
            print("-" * 30)

    elif cmd == "done":
        if not tasks:
            print("📭 暂无待办事项")
        else:
            # 先展示列表
            for i, task in enumerate(tasks):
                print(f"  {i + 1}. {task}")
            # 让用户选择
            try:
                num = int(input("输入要完成的事项编号："))
                if 1 <= num <= len(tasks):
                    finished = tasks.pop(num - 1)   # 注意索引要 -1
                    print(f"🎉 完成：{finished}（剩余 {len(tasks)} 项）")
                else:
                    print(f"❌ 请输入 1~{len(tasks)} 之间的数字")
            except ValueError:
                print("❌ 请输入有效的数字")

    elif cmd == "quit":
        print(f"\n👋 再见！你还有 {len(tasks)} 项待办事项")
        break

    else:
        print("❓ 未知命令，请用 add / show / done / quit")
```

**运行效果**：
```
=== 📋 待办事项管理器 ===
命令：add(添加) / show(查看) / done(完成) / quit(退出)

请输入命令：add
输入待办事项：学 Python
✅ 已添加：学 Python（共 1 项）

请输入命令：add
输入待办事项：跑步 30 分钟
✅ 已添加：跑步 30 分钟（共 2 项）

请输入命令：show

📋 待办列表（共 2 项）：
------------------------------
  1. 学 Python
  2. 跑步 30 分钟
------------------------------

请输入命令：done
  1. 学 Python
  2. 跑步 30 分钟
输入要完成的事项编号：1
🎉 完成：学 Python（剩余 1 项）
```

---

## 九、今日练习（12分钟）✏️

### 练习 1：学生成绩管理（必做）

创建一个成绩列表，实现以下功能：

```python
scores = [85, 92, 78, 95, 88, 72, 90]

# 1. 输出所有成绩
# 2. 输出最高分、最低分
# 3. 计算平均分
# 4. 找出所有不及格（<60）的成绩（用 if + for）
# 5. 在末尾添加一个新成绩 96
```

<details>
<summary>✅ 参考答案</summary>

```python
scores = [85, 92, 78, 95, 88, 72, 90]

# 1. 输出所有成绩
for i, score in enumerate(scores):
    print(f"第{i+1}位同学：{score}分")

# 2. 最高分、最低分
print(f"\n最高分：{max(scores)}")
print(f"最低分：{min(scores)}")

# 3. 平均分
average = sum(scores) / len(scores)
print(f"平均分：{average:.1f}")

# 4. 找出不及格的
failed = []
for score in scores:
    if score < 60:
        failed.append(score)

if failed:
    print(f"不及格成绩：{failed}")
else:
    print("没有不及格的同学")

# 5. 添加新成绩
scores.append(96)
print(f"\n添加后成绩：{scores}")
```
</details>

### 练习 2：列表去重（必做）

去掉列表中的重复元素。

```python
nums = [1, 3, 5, 3, 7, 1, 9, 5, 3]
# 要求：不用 set，用 for 循环 + if 判断实现去重
```

<details>
<summary>✅ 参考答案</summary>

```python
nums = [1, 3, 5, 3, 7, 1, 9, 5, 3]
unique = []

for num in nums:
    if num not in unique:      # 如果 unique 里还没有这个数
        unique.append(num)     # 才加进去

print(f"原列表：{nums}")
print(f"去重后：{unique}")
# 输出：去重后：[1, 3, 5, 7, 9]
```
</details>

### 练习 3：合并两个有序列表（选做）

```python
list1 = [1, 3, 5, 7]
list2 = [2, 4, 6, 8]
# 将两个列表合并后排序，输出 [1, 2, 3, 4, 5, 6, 7, 8]
```

<details>
<summary>✅ 参考答案</summary>

```python
list1 = [1, 3, 5, 7]
list2 = [2, 4, 6, 8]

# 方法 1：extend + sort
merged = list1 + list2          # 或 list1.copy() 然后 extend(list2)
merged.sort()
print(merged)                    # [1, 2, 3, 4, 5, 6, 7, 8]

# 方法 2：用 sorted（不改变原列表）
merged2 = sorted(list1 + list2)
print(merged2)
```
</details>

---

## 十、常见坑 & 避坑指南（5分钟）⚠️

| 问题 | 原因 | 解决办法 |
|------|------|----------|
| **索引越界 `IndexError`** | 列表 4 个元素却访问 `fruits[4]` | 最大索引是 `len(list)-1`，用 `-1` 取最后一个 |
| **索引从 0 开始** | `fruits[1]` 不是第一个！ | 时刻记住：**第一个是 `[0]`** |
| **`sort()` 没返回值** | `sorted_list = scores.sort()` → `None` | `sort()` 改变原列表不返回；要新列表用 `sorted()` |
| **`==` vs `is`** | `[1,2] == [1,2]` 为 True（值相等），`is` 不一定 | 判断内容相等用 `==`，判断是否同一个对象用 `is` |
| **修改列表时循环出错** | 遍历时删除元素会跳过 | 遍历副本 `for x in list[:]`，或用新列表收集 |
| **append vs extend** | `a.append([1,2])` 变成 `[[1,2]]` 不是 `[1,2]` | 添加一个元素用 `append`，合并列表用 `extend` |

---

## 十一、免费资源推荐 📚

| 资源 | 说明 | 链接 |
|------|------|------|
| **Python官方教程 - 数据结构** | 列表、元组、字典、集合官方指南 | https://docs.python.org/zh-cn/3/tutorial/datastructures.html |
| **菜鸟教程 - 列表** | 带在线编辑器，可边看边练 | https://www.runoob.com/python/python-lists.html |
| **廖雪峰 - 列表和元组** | 中文讲解，风格简洁 | https://www.liaoxuefeng.com/wiki/1016959663602400/1017092875646304 |
| **Python Tutor** | 可视化代码执行过程，看列表怎么变化 | https://pythontutor.com/ |

> 🌟 **强烈推荐 Python Tutor**：把代码粘贴进去，可以一步一步看列表的每个操作是怎么变化的，对理解索引、切片特别有帮助！

---

## 十二、今日小结 ✅

```
✅ 学会了                 ⚠️ 注意
─────────────────────────────────────────────────
列表创建 [1, 2, 3]        索引从 0 开始！
索引访问（正向+负向）      最大索引是 len-1
增删改查（6大方法）        append 加一个，extend 加一组
切片 [start:stop:step]    stop 不包含！
for + enumerate 遍历      推荐enumerate，别用range(len)
排序 sort() vs sorted()   sort 改原列表，sorted 返回新的
待办事项管理器             remove 删值，pop 删索引
```

---

## 📅 明日预告：Day 8 — 元组（tuple）+ 字符串进阶

你已经掌握了列表，明天将学习它的"不可变兄弟"——元组，以及字符串的更多高级玩法：

- 元组的创建与特性（不可变意味着什么）
- 元组 vs 列表的选择
- 字符串的 split()、join()、replace()
- 字符串常用方法大全
- 实战：简易文本分析器

> **第 7 天了，你已经学会了如何组织和操作数据！** 列表是 Python 中最常用的数据结构，掌握它就像掌握了一把万能钥匙，后面学字典、集合都会很顺畅。💪
