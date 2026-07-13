# 🐍 Python 60天学习计划 · Day 8

> **阶段**：第一阶段 · 基础语法入门（Day 1-14）
> **今日主题**：元组（tuple）+ 字符串进阶方法
> **预计时间**：约 60 分钟
> **进度**：Day 8 / 60 ▓▓▓░░░░░░░░░░░░░░░░░ 13%

---

## 一、今日目标

- ✅ 理解元组（tuple）的特性与使用场景
- ✅ 掌握元组的解包技巧
- ✅ 熟练使用字符串的常用方法（split / join / replace / strip / find）
- ✅ 完成字符串处理小练习

---

## 二、元组（Tuple）

### 2.1 什么是元组？

元组和列表很像，都能存多个数据，但有一个核心区别：

| 特性 | 列表 list | 元组 tuple |
|------|-----------|------------|
| 括号 | `[]` | `()` |
| 可修改 | ✅ 是 | ❌ 否（不可变） |
| 适用场景 | 动态数据 | 固定数据 |

```python
# 创建元组
colors = ("red", "green", "blue")
point = (10, 20)
single = (42,)   # ⚠️ 单元素元组必须加逗号！

# 不可修改
# colors[0] = "yellow"   # ❌ 报错：TypeError

# 访问元素（和列表一样）
print(colors[0])    # red
print(colors[-1])   # blue
print(colors[1:])   # ('green', 'blue')
```

---

### 2.2 元组解包（超实用技巧）

```python
# 基本解包
x, y = (10, 20)
print(x, y)   # 10 20

# 多值交换（Python 独有的优雅写法）
a, b = 1, 2
a, b = b, a
print(a, b)   # 2 1

# 忽略不需要的值（用 _ 占位）
name, _, age = ("Alice", "female", 25)
print(name, age)   # Alice 25

# 星号收集剩余元素
first, *rest = (1, 2, 3, 4, 5)
print(first)   # 1
print(rest)    # [2, 3, 4, 5]

head, *middle, last = (1, 2, 3, 4, 5)
print(head, last)    # 1 5
print(middle)        # [2, 3, 4]
```

---

### 2.3 元组的常见用途

```python
# 1. 函数返回多个值（本质是返回元组）
def get_size():
    return 1920, 1080   # 其实是 return (1920, 1080)

width, height = get_size()
print(f"屏幕分辨率：{width}x{height}")

# 2. 固定配置（不希望被修改的常量）
DB_CONFIG = ("localhost", 3306, "admin")

# 3. 字典的键（列表不能当键，元组可以）
locations = {(39.9, 116.4): "北京", (31.2, 121.5): "上海"}
print(locations[(39.9, 116.4)])   # 北京
```

---

## 三、字符串进阶方法

> 字符串是 Python 中使用最频繁的数据类型，这些方法要熟记！

### 3.1 大小写与空格处理

```python
s = "  Hello, Python!  "

# 去除空格
print(s.strip())      # "Hello, Python!"  ← 去两端
print(s.lstrip())     # "Hello, Python!  " ← 去左边
print(s.rstrip())     # "  Hello, Python!" ← 去右边

# 大小写转换
text = "hello WORLD"
print(text.upper())       # HELLO WORLD
print(text.lower())       # hello world
print(text.capitalize())  # Hello world  ← 首字母大写
print(text.title())       # Hello World  ← 每词首字母大写
```

---

### 3.2 查找与替换

```python
sentence = "Python is easy, Python is powerful"

# 查找（返回第一次出现的下标，找不到返回 -1）
print(sentence.find("Python"))    # 0
print(sentence.find("Java"))      # -1  ← 不会报错

# 统计出现次数
print(sentence.count("Python"))   # 2

# 替换（返回新字符串，原字符串不变）
new_s = sentence.replace("Python", "Go")
print(new_s)   # Go is easy, Go is powerful
print(sentence)  # 原字符串不变

# 只替换第一个
new_s2 = sentence.replace("Python", "Go", 1)
print(new_s2)   # Go is easy, Python is powerful

# 判断是否包含
print("Python" in sentence)   # True
print("Java" in sentence)     # False
```

---

### 3.3 分割与拼接（split / join）

```python
# split：字符串 → 列表
csv_line = "Alice,25,Engineer,Beijing"
parts = csv_line.split(",")
print(parts)   # ['Alice', '25', 'Engineer', 'Beijing']

# 按空格分割（默认）
words = "hello world python".split()
print(words)   # ['hello', 'world', 'python']

# 限制分割次数
result = "a-b-c-d".split("-", 2)
print(result)   # ['a', 'b', 'c-d']

# join：列表 → 字符串（split 的反向操作）
fruits = ["apple", "banana", "cherry"]
print(", ".join(fruits))    # apple, banana, cherry
print(" | ".join(fruits))   # apple | banana | cherry
print("".join(fruits))      # applebananacherry

# 常见组合：先split再处理再join
text = "  hello   world  python  "
clean = " ".join(text.split())   # 清理多余空格
print(clean)   # hello world python
```

---

### 3.4 判断类方法

```python
# 判断字符串内容（返回 True/False）
print("123".isdigit())        # True  ← 全是数字
print("abc".isalpha())        # True  ← 全是字母
print("abc123".isalnum())     # True  ← 全是字母或数字
print("hello world".isspace()) # False ← 全是空白字符？
print("  ".isspace())         # True

# 判断开头结尾
url = "https://www.python.org"
print(url.startswith("https"))   # True
print(url.endswith(".org"))      # True

# 实际使用场景：过滤非数字输入
user_input = "123"
if user_input.isdigit():
    number = int(user_input)
    print(f"有效数字：{number}")
else:
    print("请输入纯数字！")
```

---

### 3.5 格式化补充：center / ljust / rjust

```python
# 居中对齐（常用于打印表格/标题）
title = "成绩单"
print(title.center(20, "="))    # ========成绩单=========
print("Alice".ljust(10, "."))   # Alice.....
print("99".rjust(5, "0"))       # 00099  ← 数字补零
```

---

## 四、综合实战：CSV 数据处理器

> 把今天学的都用上！

```python
# 模拟读取一份学生成绩 CSV 数据
csv_data = """姓名,语文,数学,英语
  Alice ,95,88,92
  bob ,78,85,90
  CHARLIE ,60,72,68
  Diana ,88,91,95
"""

print("=" * 40)
print("       学生成绩分析报告")
print("=" * 40)

# 分割成行
lines = csv_data.strip().split("\n")

# 第一行是表头，跳过
header = lines[0]
student_lines = lines[1:]

results = []

for line in student_lines:
    # 分割字段并清理空格
    parts = line.split(",")
    name = parts[0].strip().capitalize()  # 去空格 + 首字母大写
    chinese = int(parts[1].strip())
    math = int(parts[2].strip())
    english = int(parts[3].strip())

    total = chinese + math + english
    avg = total / 3

    # 等级判断
    if avg >= 90:
        grade = "优秀"
    elif avg >= 75:
        grade = "良好"
    elif avg >= 60:
        grade = "及格"
    else:
        grade = "不及格"

    results.append((name, chinese, math, english, avg, grade))

# 按平均分排序（用元组索引4）
results.sort(key=lambda x: x[4], reverse=True)

# 输出表格
print(f"{'姓名':<10}{'语文':>6}{'数学':>6}{'英语':>6}{'平均分':>8}{'等级':>6}")
print("-" * 46)
for name, ch, ma, en, avg, grade in results:
    print(f"{name:<10}{ch:>6}{ma:>6}{en:>6}{avg:>8.1f}{grade:>6}")

print("=" * 40)
print(f"共 {len(results)} 名学生，优秀率：{sum(1 for r in results if r[5]=='优秀')/len(results)*100:.0f}%")
```

**预期输出：**
```
========================================
       学生成绩分析报告
========================================
姓名          语文    数学    英语     平均分    等级
----------------------------------------------
Diana         88      91      95      91.3    优秀
Alice         95      88      92      91.7    优秀
Bob           78      85      90      84.3    良好
Charlie       60      72      68      66.7    及格
========================================
共 4 名学生，优秀率：50%
```

---

## 五、今日练习题

**练习1（5分钟）：元组解包练习**
```python
# 给定一个描述矩形的元组 rect = (10, 5)
# 用解包得到宽度和高度，计算并打印面积和周长
rect = (10, 5)
# 你的代码写在这里 ↓
```

**练习2（10分钟）：字符串处理**
```python
# 有一段包含多余空格的文本，要求：
# 1. 去除首尾空格
# 2. 把所有单词首字母大写
# 3. 统计单词数量
# 4. 判断是否以 "Dear" 开头

text = "  dear friends, welcome to python programming world  "
# 你的代码写在这里 ↓
```

**练习3（15分钟）：手机号格式化**
```python
# 给定一批手机号（格式不统一），要求：
# 1. 去除空格和横线
# 2. 判断是否是11位数字
# 3. 格式化输出为 138-0013-8000 格式

phones = ["138 0013 8000", "139-1234-5678", "12345", "13abc456789", "15900001234"]

for phone in phones:
    # 你的代码写在这里 ↓
    pass
```

---

## 六、参考答案

<details>
<summary>📖 点击查看答案（建议先自己做）</summary>

**练习1答案：**
```python
rect = (10, 5)
width, height = rect
area = width * height
perimeter = 2 * (width + height)
print(f"矩形 {width}x{height}：面积={area}，周长={perimeter}")
```

**练习2答案：**
```python
text = "  dear friends, welcome to python programming world  "
cleaned = text.strip()
titled = cleaned.title()
words = cleaned.split()
word_count = len(words)
starts_with_dear = cleaned.lower().startswith("dear")

print(f"处理后：{titled}")
print(f"单词数：{word_count}")
print(f"以Dear开头：{starts_with_dear}")
```

**练习3答案：**
```python
phones = ["138 0013 8000", "139-1234-5678", "12345", "13abc456789", "15900001234"]

for phone in phones:
    # 去除空格和横线
    cleaned = phone.replace(" ", "").replace("-", "")
    
    # 验证
    if len(cleaned) == 11 and cleaned.isdigit():
        # 格式化为 138-0013-8000
        formatted = f"{cleaned[:3]}-{cleaned[3:7]}-{cleaned[7:]}"
        print(f"✅ {phone} → {formatted}")
    else:
        print(f"❌ {phone} → 格式无效")
```
</details>

---

## 七、今日避坑提醒

> ⚠️ **新手常犯的错误，今天就避开！**

**坑1：单元素元组忘加逗号**
```python
# ❌ 错误：这是整数，不是元组
x = (42)
print(type(x))   # <class 'int'>

# ✅ 正确：加逗号才是元组
x = (42,)
print(type(x))   # <class 'tuple'>
```

**坑2：字符串方法不会修改原字符串**
```python
s = "hello"
s.upper()          # ❌ 这行没有用！结果被丢弃了
print(s)           # 还是 hello

s = s.upper()      # ✅ 要赋值给变量
print(s)           # HELLO
```

**坑3：split() 后忘记strip()**
```python
# CSV数据中经常有多余空格
line = "Alice , 25 , Engineer"
parts = line.split(",")
print(parts)   # ['Alice ', ' 25 ', ' Engineer']  ← 有多余空格

# ✅ 记得 strip
parts = [p.strip() for p in line.split(",")]
print(parts)   # ['Alice', '25', 'Engineer']  ← 干净了
```

**坑4：find() vs index() 的区别**
```python
s = "hello"
print(s.find("z"))    # -1     ← 找不到返回-1，不报错
# print(s.index("z")) # ❌ 报错 ValueError：找不到直接崩溃

# 建议：优先用 find()，更安全
```

---

## 八、学习资源推荐

| 资源 | 链接 | 说明 |
|------|------|------|
| Python官方文档（字符串方法） | https://docs.python.org/zh-cn/3/library/stdtypes.html#string-methods | 最权威，可当字典用 |
| 菜鸟教程 Python 字符串 | https://www.runoob.com/python3/python3-string.html | 中文，例子多 |
| Python Tutor | https://pythontutor.com | 可视化执行代码，理解变量变化 |
| 练习平台 LeetCode（简单题） | https://leetcode.cn/problems/reverse-string/ | 字符串翻转，适合今天练手 |

---

## 九、明日预告 · Day 9

**主题：字典（dict）**

```
📦 字典入门
├── 创建字典 {key: value}
├── 增删改查（get / update / pop / keys / values / items）
├── 字典遍历（for k, v in d.items()）
├── 字典推导式
└── 实战：词频统计器（统计文章中每个词出现多少次）
```

---

## 今日总结

| 知识点 | 重要程度 | 掌握建议 |
|--------|----------|----------|
| 元组创建与访问 | ⭐⭐⭐ | 理解"不可变"的含义 |
| 元组解包 | ⭐⭐⭐⭐⭐ | 必须熟练，后面大量使用 |
| strip / split / join | ⭐⭐⭐⭐⭐ | 字符串处理三件套，天天用 |
| replace / find / count | ⭐⭐⭐⭐ | 高频方法，多练 |
| isdigit / startswith / endswith | ⭐⭐⭐ | 输入验证常用 |

> 💡 **今日行动**：打开 Python，把"CSV数据处理器"代码自己敲一遍，不要复制粘贴！
> 手动敲代码比看100遍更有效果。

---
*Day 8 · 2026-05-29 · Python 60天学习计划*
