# 🐍 Python 60天学习计划 · Day 9

> **阶段**：第一阶段 · 基础语法入门（Day 1-14）
> **今日主题**：字典（dict）— Python 最强大的数据结构之一
> **预计时间**：约 60 分钟
> **进度**：Day 9 / 60 ▓▓▓▓░░░░░░░░░░░░░░░░ 15%

---

## 一、今日目标

- ✅ 理解字典的结构：键值对（key-value）
- ✅ 掌握字典的增删改查（get / update / pop / del）
- ✅ 熟练遍历字典（keys / values / items）
- ✅ 学会字典推导式
- ✅ 完成实战项目：词频统计器

---

## 二、什么是字典？

### 2.1 字典 vs 列表

| 特性 | 列表 list | 字典 dict |
|------|-----------|-----------|
| 括号 | `[]` | `{}` |
| 索引方式 | 数字下标 `[0]` | 任意键 `["name"]` |
| 适合存储 | 有序的同类数据 | 有明确"标签"的数据 |
| 查找速度 | 慢（逐个找） | 极快（直接定位） |

**直觉理解**：列表是"排队叫号"，字典是"查字典"——直接按关键词找答案。

```python
# 列表：靠位置找数据（不直观）
student_list = ["Alice", 20, "female", 90]
print(student_list[3])   # 90 ← 这是成绩？还是什么？不清楚

# 字典：靠名字找数据（直观清晰）
student_dict = {
    "name": "Alice",
    "age": 20,
    "gender": "female",
    "score": 90
}
print(student_dict["score"])   # 90 ← 一目了然
```

---

### 2.2 创建字典

```python
# 方式1：直接用 {} 创建
person = {
    "name": "Alice",
    "age": 25,
    "city": "北京"
}

# 方式2：dict() 函数
config = dict(host="localhost", port=3306, debug=True)

# 方式3：空字典（后面逐步填充）
scores = {}

# 方式4：用两个列表合并成字典
keys = ["name", "age", "city"]
values = ["Bob", 30, "上海"]
person2 = dict(zip(keys, values))
print(person2)   # {'name': 'Bob', 'age': 30, 'city': '上海'}
```

**字典的键（key）规则**：
- ✅ 可以是：字符串、数字、元组（不可变类型）
- ❌ 不可以是：列表（可变类型会报错）
- 键必须唯一，同一个键只能出现一次

---

## 三、字典的增删改查

### 3.1 查（读取数据）

```python
student = {
    "name": "Alice",
    "age": 20,
    "score": 90
}

# 方法1：直接用键访问（键不存在会报错❌）
print(student["name"])   # Alice
# print(student["email"])  # ❌ KeyError：不存在的键直接崩溃

# 方法2：get() 方法（推荐✅，键不存在返回None而不报错）
print(student.get("name"))    # Alice
print(student.get("email"))   # None ← 不报错
print(student.get("email", "未填写"))  # 未填写 ← 可设置默认值

# 判断键是否存在
if "score" in student:
    print(f"成绩：{student['score']}")
```

> 💡 **黄金法则**：读取字典时，**始终用 `get()` 而不是 `[]`**，除非你100%确定键存在。

---

### 3.2 增和改（写入数据）

```python
student = {"name": "Alice", "age": 20}

# 新增键值对（键不存在 → 新增）
student["email"] = "alice@email.com"
student["score"] = 90
print(student)
# {'name': 'Alice', 'age': 20, 'email': 'alice@email.com', 'score': 90}

# 修改已有键（键已存在 → 覆盖）
student["age"] = 21
student["score"] = 95
print(student["age"])    # 21
print(student["score"])  # 95

# update() 批量更新（可新增也可修改）
student.update({"city": "上海", "age": 22})
print(student)
# {'name': 'Alice', 'age': 22, ..., 'city': '上海'}
```

---

### 3.3 删（删除数据）

```python
student = {"name": "Alice", "age": 20, "score": 90, "temp": "临时数据"}

# pop()：删除指定键并返回其值（常用✅）
removed = student.pop("temp")
print(removed)    # 临时数据
print(student)    # temp 已被删除

# pop() 带默认值：键不存在时不报错
val = student.pop("email", "不存在")
print(val)   # 不存在

# del：删除指定键（不返回值）
del student["age"]
print(student)   # age 被删除

# clear()：清空整个字典
# student.clear()  # 慎用！清空后字典变为 {}

# 注意：pop 和 del 对不存在的键都会报 KeyError，
# 所以 pop(key, default) 是最安全的删除方式
```

---

## 四、遍历字典

### 4.1 三种遍历方式

```python
scores = {
    "Alice": 95,
    "Bob": 88,
    "Charlie": 72,
    "Diana": 91
}

# 方式1：只遍历键（默认）
print("--- 遍历键 ---")
for name in scores:           # 等价于 for name in scores.keys()
    print(name)

# 方式2：只遍历值
print("--- 遍历值 ---")
for score in scores.values():
    print(score)

# 方式3：同时遍历键和值（最常用✅）
print("--- 遍历键值对 ---")
for name, score in scores.items():
    print(f"{name}: {score}分")
```

**输出：**
```
--- 遍历键值对 ---
Alice: 95分
Bob: 88分
Charlie: 72分
Diana: 91分
```

---

### 4.2 遍历时的实用操作

```python
scores = {"Alice": 95, "Bob": 88, "Charlie": 72, "Diana": 91}

# 找最高分
max_name = max(scores, key=scores.get)
print(f"最高分：{max_name} → {scores[max_name]}分")

# 筛选出优秀学生（≥90分）
excellent = {name: s for name, s in scores.items() if s >= 90}
print(f"优秀学生：{excellent}")

# 按分数降序排列
sorted_scores = sorted(scores.items(), key=lambda x: x[1], reverse=True)
for rank, (name, score) in enumerate(sorted_scores, 1):
    print(f"第{rank}名：{name} {score}分")
```

---

## 五、字典推导式

> 类似列表推导式，用一行代码生成字典。

```python
# 语法：{key表达式: value表达式 for 变量 in 可迭代对象 if 条件}

# 示例1：生成数字的平方字典
squares = {n: n**2 for n in range(1, 6)}
print(squares)   # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 示例2：从列表生成字典
students = ["Alice", "Bob", "Charlie"]
default_score = {name: 0 for name in students}
print(default_score)   # {'Alice': 0, 'Bob': 0, 'Charlie': 0}

# 示例3：过滤 + 转换
raw = {"Alice": "95", "Bob": "88", "Charlie": "abc", "Diana": "91"}
valid_scores = {
    name: int(score)
    for name, score in raw.items()
    if score.isdigit()    # 只保留有效数字
}
print(valid_scores)   # {'Alice': 95, 'Bob': 88, 'Diana': 91}

# 示例4：键值互换
original = {"one": 1, "two": 2, "three": 3}
flipped = {v: k for k, v in original.items()}
print(flipped)   # {1: 'one', 2: 'two', 3: 'three'}
```

---

## 六、字典的常见使用场景

```python
# 场景1：计数器（词频统计的核心逻辑）
fruits = ["apple", "banana", "apple", "cherry", "banana", "apple"]
count = {}
for fruit in fruits:
    # 如果键存在+1，不存在则从0开始
    count[fruit] = count.get(fruit, 0) + 1
print(count)   # {'apple': 3, 'banana': 2, 'cherry': 1}

# 场景2：分组（把相同类别的数据归类）
students = [
    ("Alice", "A班"), ("Bob", "B班"),
    ("Charlie", "A班"), ("Diana", "B班"), ("Eve", "A班")
]
groups = {}
for name, cls in students:
    if cls not in groups:
        groups[cls] = []        # 先创建空列表
    groups[cls].append(name)    # 再追加

print(groups)
# {'A班': ['Alice', 'Charlie', 'Eve'], 'B班': ['Bob', 'Diana']}

# 场景3：配置管理（替代一堆零散变量）
# ❌ 糟糕的做法：
host = "localhost"
port = 3306
user = "root"
password = "123456"

# ✅ 好的做法：
db_config = {
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "123456"
}
print(f"连接 {db_config['host']}:{db_config['port']}")
```

---

## 七、综合实战：词频统计器

> 统计一段文章中，每个单词出现了多少次，并输出 Top 5 高频词。

```python
# 一段英文（或中文拆词后的词列表）
article = """
Python is a versatile programming language Python is used for
web development data science machine learning and automation
Python has a simple and readable syntax Python is popular
among beginners and professionals Python community is large
"""

print("=" * 45)
print("        📊 词频统计器")
print("=" * 45)

# 第一步：预处理文本
# 转小写，按空格分割成单词列表
words = article.lower().split()

# 第二步：统计词频
word_count = {}
for word in words:
    # 去掉标点（简单处理）
    word = word.strip(".,!?;:'\"")
    if word:   # 非空才统计
        word_count[word] = word_count.get(word, 0) + 1

print(f"文章共 {len(words)} 个词，{len(word_count)} 个不重复词\n")

# 第三步：按频率排序
sorted_words = sorted(word_count.items(), key=lambda x: x[1], reverse=True)

# 第四步：展示 Top 10
print(f"{'排名':<5}{'单词':<15}{'出现次数':>8}  {'频率':>6}")
print("-" * 38)
for rank, (word, count) in enumerate(sorted_words[:10], 1):
    freq = count / len(words) * 100
    bar = "█" * count   # 简单的横向条形图
    print(f"{rank:<5}{word:<15}{count:>8}   {freq:>5.1f}%  {bar}")

print("=" * 45)
print(f"\n🏆 最高频词：'{sorted_words[0][0]}' 出现了 {sorted_words[0][1]} 次")

# 第五步（扩展）：统计停用词（the/is/a 这类没意义的词）
stop_words = {"is", "a", "and", "for", "has", "the", "of", "in"}
content_words = {w: c for w, c in word_count.items() if w not in stop_words}
sorted_content = sorted(content_words.items(), key=lambda x: x[1], reverse=True)

print("\n📌 去除停用词后 Top 5 关键词：")
for word, count in sorted_content[:5]:
    print(f"  {word}: {count}次")
```

**预期输出（部分）：**
```
=============================================
        📊 词频统计器
=============================================
文章共 35 个词，20 个不重复词

排名  单词              出现次数    频率
--------------------------------------
1     python               5   14.3%  █████
2     is                   4   11.4%  ████
3     and                  3    8.6%  ███
...

📌 去除停用词后 Top 5 关键词：
  python: 5次
  simple: 1次
  ...
```

---

## 八、今日练习题

**练习1（5分钟）：基本操作**
```python
# 创建一个存储个人信息的字典，包含 name/age/city/hobby
# 1. 打印所有键
# 2. 安全地获取 "email" 键（不报错，显示"未填写"）
# 3. 新增 "email" 键
# 4. 删除 "hobby" 键并打印被删除的值
person = {"name": "你的名字", "age": 0, "city": "你的城市", "hobby": "编程"}
# 你的代码写在这里 ↓
```

**练习2（10分钟）：成绩统计**
```python
# 给定一批成绩，用字典完成以下分析：
# 1. 统计各分数段人数：优秀(90+) / 良好(75-89) / 及格(60-74) / 不及格(<60)
# 2. 输出每个分数段的人数

scores = [95, 88, 72, 60, 45, 91, 78, 83, 55, 98, 67, 74, 89, 52, 94]
# 你的代码写在这里 ↓
```

**练习3（15分钟）：通讯录管理**
```python
# 用字典实现一个简单通讯录（支持以下操作）：
# 1. 添加联系人  add <姓名> <电话>
# 2. 查找联系人  find <姓名>
# 3. 删除联系人  delete <姓名>
# 4. 显示所有    list
# 5. 退出        quit

contacts = {}

while True:
    cmd = input("\n请输入命令 (add/find/delete/list/quit): ").strip().split()
    # 你的代码写在这里 ↓
```

---

## 九、参考答案

<details>
<summary>📖 点击查看答案（建议先自己做）</summary>

**练习1答案：**
```python
person = {"name": "你的名字", "age": 0, "city": "你的城市", "hobby": "编程"}

# 1. 打印所有键
print("所有键：", list(person.keys()))

# 2. 安全获取 email
email = person.get("email", "未填写")
print(f"邮箱：{email}")

# 3. 新增 email
person["email"] = "example@email.com"
print(f"新增后：{person}")

# 4. 删除 hobby
deleted = person.pop("hobby")
print(f"删除了：{deleted}")
print(f"剩余：{person}")
```

**练习2答案：**
```python
scores = [95, 88, 72, 60, 45, 91, 78, 83, 55, 98, 67, 74, 89, 52, 94]

distribution = {"优秀(90+)": 0, "良好(75-89)": 0, "及格(60-74)": 0, "不及格(<60)": 0}

for s in scores:
    if s >= 90:
        distribution["优秀(90+)"] += 1
    elif s >= 75:
        distribution["良好(75-89)"] += 1
    elif s >= 60:
        distribution["及格(60-74)"] += 1
    else:
        distribution["不及格(<60)"] += 1

print(f"共 {len(scores)} 人：")
for level, count in distribution.items():
    print(f"  {level}: {count}人")
```

**练习3答案：**
```python
contacts = {}

while True:
    raw = input("\n请输入命令 (add/find/delete/list/quit): ").strip()
    if not raw:
        continue
    parts = raw.split()
    action = parts[0].lower()

    if action == "add" and len(parts) == 3:
        name, phone = parts[1], parts[2]
        contacts[name] = phone
        print(f"✅ 已添加：{name} → {phone}")

    elif action == "find" and len(parts) == 2:
        name = parts[1]
        phone = contacts.get(name)
        if phone:
            print(f"📞 {name}：{phone}")
        else:
            print(f"❌ 未找到联系人：{name}")

    elif action == "delete" and len(parts) == 2:
        name = parts[1]
        removed = contacts.pop(name, None)
        if removed:
            print(f"🗑️ 已删除：{name}")
        else:
            print(f"❌ 联系人不存在：{name}")

    elif action == "list":
        if contacts:
            print(f"📋 通讯录（{len(contacts)}人）：")
            for name, phone in contacts.items():
                print(f"  {name}: {phone}")
        else:
            print("通讯录为空")

    elif action == "quit":
        print("👋 再见！")
        break
    else:
        print("⚠️ 命令格式：add <姓名> <电话> | find <姓名> | delete <姓名> | list | quit")
```
</details>

---

## 十、今日避坑提醒

**坑1：用 `[]` 读取不存在的键，直接报错**
```python
d = {"name": "Alice"}

# ❌ 键不存在直接崩溃
# print(d["age"])  → KeyError: 'age'

# ✅ 用 get()，安全读取
print(d.get("age"))         # None
print(d.get("age", 0))      # 0 ← 默认值
```

**坑2：遍历字典时修改字典会报错**
```python
d = {"a": 1, "b": 2, "c": 3}

# ❌ 遍历中删除会报错
# for k in d:
#     if d[k] < 2:
#         del d[k]   # RuntimeError: dictionary changed size during iteration

# ✅ 先用列表收集要删除的键，再统一删除
to_delete = [k for k, v in d.items() if v < 2]
for k in to_delete:
    del d[k]
print(d)   # {'b': 2, 'c': 3}
```

**坑3：计数时忘记初始值，直接 +1 报错**
```python
count = {}
words = ["apple", "banana", "apple"]

# ❌ 首次出现时 count["apple"] 不存在，+1 报 KeyError
# for w in words:
#     count[w] += 1

# ✅ 方案1：用 get() 设默认值
for w in words:
    count[w] = count.get(w, 0) + 1

# ✅ 方案2：用 setdefault（了解即可）
count2 = {}
for w in words:
    count2.setdefault(w, 0)
    count2[w] += 1

print(count)   # {'apple': 2, 'banana': 1}
```

**坑4：字典是无序的吗？**
```python
# Python 3.7+ 开始，字典会保持插入顺序（有序）
d = {"c": 3, "a": 1, "b": 2}
for k in d:
    print(k)   # 输出 c, a, b（按插入顺序）

# 但如果你需要按键排序，要手动 sorted()
for k in sorted(d):
    print(k)   # 输出 a, b, c（按字母顺序）
```

---

## 十一、学习资源推荐

| 资源 | 链接 | 说明 |
|------|------|------|
| Python官方文档（字典） | https://docs.python.org/zh-cn/3/library/stdtypes.html#mapping-types-dict | 权威完整 |
| 菜鸟教程 Python 字典 | https://www.runoob.com/python3/python3-dictionary.html | 中文，适合入门 |
| 练习题：两数之和 | https://leetcode.cn/problems/two-sum/ | 经典字典应用，值得一做 |
| Python Tutor | https://pythontutor.com | 可视化理解字典结构 |

---

## 十二、字典方法速查表

| 方法 | 作用 | 示例 |
|------|------|------|
| `d[key]` | 取值（键不存在报错） | `d["name"]` |
| `d.get(key, default)` | 安全取值 | `d.get("age", 0)` |
| `d[key] = val` | 新增或修改 | `d["score"] = 90` |
| `d.update(dict2)` | 批量更新 | `d.update({"a":1})` |
| `d.pop(key, default)` | 删除并返回值 | `d.pop("temp", None)` |
| `del d[key]` | 删除键 | `del d["age"]` |
| `d.keys()` | 所有键 | `list(d.keys())` |
| `d.values()` | 所有值 | `list(d.values())` |
| `d.items()` | 键值对 | `for k,v in d.items()` |
| `key in d` | 判断键是否存在 | `"name" in d` |
| `d.clear()` | 清空字典 | `d.clear()` |
| `len(d)` | 键的数量 | `len(d)` |

---

## 十三、明日预告 · Day 10

**主题：集合（set）+ 函数入门（def）**

```
📦 Day 10 学习内容
├── 集合（set）：自动去重的容器
│   ├── 创建集合 {1, 2, 3}
│   ├── 集合运算：交集/并集/差集
│   └── 实际应用：快速去重
└── 函数基础（def）
    ├── 定义函数 + 调用函数
    ├── 参数与返回值
    └── 用函数重构之前的代码
```

---

## 今日总结

| 知识点 | 重要程度 | 掌握建议 |
|--------|----------|----------|
| 字典基本结构与创建 | ⭐⭐⭐⭐ | 理解键值对的概念 |
| get() 安全读取 | ⭐⭐⭐⭐⭐ | 形成习惯，优先用 get() |
| 增删改查 | ⭐⭐⭐⭐⭐ | 必须熟练，每天都会用 |
| items() 遍历 | ⭐⭐⭐⭐⭐ | for k,v in d.items() 要背下来 |
| 字典推导式 | ⭐⭐⭐⭐ | 写出来代码更优雅 |
| 词频统计器 | ⭐⭐⭐⭐ | 经典算法题模板，务必自己敲一遍 |

> 💡 **今日行动**：
> 1. 把"词频统计器"代码完整敲一遍，理解每一行
> 2. 完成通讯录练习3，做出一个可以实际使用的程序
> 3. 挑战：去 LeetCode 做「两数之和」，用字典解决！

---

*Day 9 · 2026-05-30 · Python 60天学习计划*
