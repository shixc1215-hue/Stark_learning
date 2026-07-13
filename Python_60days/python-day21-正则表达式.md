# Python Day 21：正则表达式

> **学习目标**：掌握正则表达式的基本语法和 Python `re` 模块的常用操作，能够用正则进行文本匹配、搜索、替换和提取。

---

## 一、什么是正则表达式？

正则表达式（Regular Expression，简称 regex 或 re）是一种**描述字符串匹配规则**的工具。你可以把它理解为一个"超级通配符"——

- 你平时用 `*.txt` 找文件，只支持简单的 `*` 和 `?`
- 正则表达式可以表达**任意复杂的匹配规则**

### 现实场景

| 场景 | 需求 |
|------|------|
| 表单验证 | 判断用户输入的是否是合法手机号、邮箱 |
| 日志分析 | 从大量日志中提取 IP 地址、错误码 |
| 数据清洗 | 从混乱文本中抓取价格、日期、姓名 |
| 文本替换 | 批量将日期格式从 `2026/06/20` 改为 `2026-06-20` |

正则表达式虽然看起来像"乱码"，但学会后你会发现它极其强大。

---

## 二、基础语法

### 2.1 普通字符

普通字符就是它们自己，直接匹配：

```python
import re

text = "hello world"
result = re.findall("hello", text)   # ['hello'] — 找到 "hello"
result = re.findall("xyz", text)      # [] — 没找到
```

### 2.2 元字符（特殊字符）

以下是正则中最核心的元字符：

| 元字符 | 含义 | 示例 |
|--------|------|------|
| `.` | 匹配任意**单个**字符（除换行符） | `a.c` 匹配 `abc`、`a1c`、`a c` |
| `\d` | 数字 `[0-9]` | `\d\d\d` 匹配 `123` |
| `\D` | 非数字 | `\D+` 匹配 `abc` |
| `\w` | 字母/数字/下划线 `[a-zA-Z0-9_]` | `\w+` 匹配 `hello_123` |
| `\W` | 非字母/数字/下划线 | 匹配空格、标点等 |
| `\s` | 空白字符（空格/制表/换行） | `\s+` 匹配连续空格 |
| `\S` | 非空白字符 | |
| `^` | 匹配**字符串开头** | `^Hello` 匹配以 Hello 开头 |
| `$` | 匹配**字符串结尾** | `world$` 匹配以 world 结尾 |
| `\b` | 单词边界 | `\bcat\b` 匹配独立单词 cat |

### 2.3 量词（控制出现次数）

| 量词 | 含义 | 示例 |
|------|------|------|
| `*` | 出现 **0 次或多次** | `ab*c` 匹配 `ac`、`abc`、`abbc` |
| `+` | 出现 **1 次或多次** | `ab+c` 匹配 `abc`、`abbc`，不匹配 `ac` |
| `?` | 出现 **0 次或 1 次** | `colou?r` 匹配 `color` 和 `colour` |
| `{n}` | 恰好出现 **n 次** | `\d{4}` 匹配 4 位数字 |
| `{n,m}` | 出现 **n 到 m 次** | `\d{1,3}` 匹配 1-3 位数字 |
| `{n,}` | 至少出现 **n 次** | `\d{2,}` 匹配 2 位及以上的数字 |

```python
import re

# 匹配手机号（11位数字）
phone = "我的手机号是13812345678，他的号码是13987654321"
result = re.findall(r'\d{11}', phone)
print(result)  # ['13812345678', '13987654321']

# 匹配英文单词（连续字母）
text = "Hello World Python"
result = re.findall(r'\w+', text)
print(result)  # ['Hello', 'World', 'Python']
```

### 2.4 字符类 `[ ]` 和取反 `[^ ]`

用方括号定义一组字符，匹配其中任意一个：

```python
import re

# 匹配元音字母
text = "hello world"
result = re.findall(r'[aeiou]', text)
print(result)  # ['e', 'o', 'o']

# 匹配数字或字母
result = re.findall(r'[a-zA-Z0-9]', "abc123!@#")
print(result)  # ['a', 'b', 'c', '1', '2', '3']

# 取反：匹配非数字
result = re.findall(r'[^0-9]', "abc123!@#")
print(result)  # ['a', 'b', 'c', '!', '@', '#']

# 范围：匹配 a-g 之间的字母
result = re.findall(r'[a-g]', "hello world")
print(result)  # ['e', 'd']
```

### 2.5 分组 `( )` 和选择 `|`

```python
import re

# 选择：匹配 "cat" 或 "dog"
text = "I have a cat and a dog"
result = re.findall(r'cat|dog', text)
print(result)  # ['cat', 'dog']

# 分组：将整体作为一个单元
text = "2026-06-20"
result = re.findall(r'(\d{4})-(\d{2})-(\d{2})', text)
print(result)  # [('2026', '06', '20')] — 返回元组

year, month, day = result[0]
print(f"年份：{year}, 月份：{month}, 日期：{day}")

# 用 | 实现多规则匹配：手机号或座机号
text = "手机13812345678，座机010-87654321"
result = re.findall(r'(\d{11}|0\d{2,3}-\d{7,8})', text)
print(result)  # ['13812345678', '010-87654321']
```

### 2.6 转义字符 `\`

如果需要匹配元字符本身，用反斜杠转义：

```python
import re

# 匹配小数点（. 是元字符，需要转义）
text = "价格是 3.14 元"
result = re.findall(r'\d+\.\d+', text)
print(result)  # ['3.14']

# 匹配括号
text = "test(123)"
result = re.findall(r'\(.*?\)', text)
print(result)  # ['(123)']
```

---

## 三、Python re 模块核心函数

### 3.1 re.match() vs re.search() vs re.findall()

这是最常用的三个函数，区别如下：

| 函数 | 作用 | 匹配位置 | 返回值 |
|------|------|----------|--------|
| `re.match()` | 从**字符串开头**匹配 | 只从头开始 | 匹配对象或 None |
| `re.search()` | **扫描整个字符串**找第一个匹配 | 任意位置 | 匹配对象或 None |
| `re.findall()` | 找到**所有**匹配 | 任意位置 | 列表 |

```python
import re

text = "cat says meow, dog says woof"

# match() 只从开头匹配
result = re.match(r'cat', text)      # ✅ 匹配到
print(result.group())                 # 'cat'

result = re.match(r'dog', text)      # ❌ None（dog 不在开头）
print(result)                         # None

# search() 扫描整个字符串
result = re.search(r'dog', text)     # ✅ 匹配到
print(result.group())                 # 'dog'

# findall() 找所有
result = re.findall(r'\b\w+says\w*\b', text)
print(result)                         # ['catsaysmeow', 'dogsayswoof']
```

### 3.2 re.sub() — 替换

```python
import re

# 基本替换：把所有数字替换为 *
text = "订单号：12345，金额：99.9 元"
result = re.sub(r'\d+', '***', text)
print(result)  # '订单号：***，金额：***.*** 元'

# 使用函数替换：将手机号中间4位替换为 ****
def mask_phone(match):
    phone = match.group()
    return phone[:3] + "****" + phone[7:]

text = "联系人：13812345678，客服：13987654321"
result = re.sub(r'\d{11}', mask_phone, text)
print(result)  # '联系人：138****5678，客服：139****4321'

# 限制替换次数（count 参数）
text = "aaa bbb aaa ccc"
result = re.sub(r'aaa', 'XXX', text, count=1)
print(result)  # 'XXX bbb aaa ccc'
```

### 3.3 re.split() — 分割

```python
import re

# 按多种空白字符分割
text = "hello   world\tPython\r\nGood"
result = re.split(r'\s+', text)
print(result)  # ['hello', 'world', 'Python', 'Good']

# 保留分隔符（用分组）
text = "2026-06-20"
result = re.split(r'(-)', text)
print(result)  # ['2026', '-', '06', '-', '20']
```

### 3.4 re.compile() — 预编译正则

当一个正则表达式需要**多次使用**时，预编译能提高效率：

```python
import re

# 预编译（推荐：一次编译，多次使用）
phone_pattern = re.compile(r'^1[3-9]\d{9}$')

# 判断是否是手机号
print(phone_pattern.match("13812345678"))   # <re.Match object>
print(phone_pattern.match("12345678901"))    # None（不以 13-19 开头）
print(phone_pattern.match("1381234"))         # None（位数不够）

# 预编译后也能用 findall、sub 等方法
email_pattern = re.compile(r'\b[\w.]+@[\w.]+\.\w+\b')
text = "发邮件给 alice@qq.com 和 bob@163.com"
print(email_pattern.findall(text))
# ['alice@qq.com', 'bob@163.com']
```

---

## 四、常用正则表达式模板

下面是实际开发中**最常用**的正则模板，建议收藏：

```python
import re

# ========== 1. 手机号（中国大陆） ==========
phone_re = re.compile(r'^1[3-9]\d{9}$')
print(phone_re.match("13812345678"))    # ✅
print(phone_re.match("12345678901"))   # ❌

# ========== 2. 邮箱地址 ==========
email_re = re.compile(r'^[\w.+-]+@[\w-]+\.[\w.]+$')
print(email_re.match("test@example.com"))     # ✅
print(email_re.match("user.name+tag@qq.com"))  # ✅
print(email_re.match("invalid@"))              # ❌

# ========== 3. 身份证号（18位） ==========
id_re = re.compile(r'^\d{17}[\dXx]$')
print(id_re.match("110101199001011234"))  # ✅
print(id_re.match("11010119900101123X"))  # ✅

# ========== 4. 日期（YYYY-MM-DD） ==========
date_re = re.compile(r'^\d{4}-\d{2}-\d{2}$')
print(date_re.match("2026-06-20"))   # ✅
print(date_re.match("2026/06/20"))  # ❌

# ========== 5. IP 地址 ==========
ip_re = re.compile(
    r'^((25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)\.){3}'
    r'(25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)$'
)
print(ip_re.match("192.168.1.1"))     # ✅
print(ip_re.match("255.255.255.255"))  # ✅
print(ip_re.match("256.1.1.1"))       # ❌

# ========== 6. URL ==========
url_re = re.compile(r'^https?://[\w\-]+(\.[\w\-]+)+[/\w\-]*$')
print(url_re.match("https://www.example.com"))       # ✅
print(url_re.match("http://blog.example.com/path"))  # ✅

# ========== 7. 中文字符 ==========
chinese_re = re.compile(r'^[\u4e00-\u9fa5]+$')
print(chinese_re.match("你好世界"))   # ✅
print(chinese_re.match("Hello"))     # ❌
```

---

## 五、贪婪与非贪婪匹配

这是一个**非常重要**的概念，也是新手最容易踩坑的地方。

### 贪婪模式（默认）

量词会**尽可能多地**匹配字符：

```python
import re

text = '<div>hello</div><div>world</div>'

# 贪婪匹配：.* 会尽可能多地匹配
result = re.findall(r'<div>(.*)</div>', text)
print(result)  # ['hello</div><div>world']  — 不是我们想要的！
```

### 非贪婪模式（加 `?`）

在量词后面加 `?` 变成**非贪婪**，**尽可能少**地匹配：

```python
# 非贪婪匹配：.*? 尽可能少匹配
result = re.findall(r'<div>(.*?)</div>', text)
print(result)  # ['hello', 'world']  — 正确！
```

**记忆口诀**：贪婪是"多吃"，非贪婪是"少吃"，在量词后加 `?` 切换。

```python
import re

text = "价格：100-200-300 元"

# 贪婪：匹配到最后
result = re.search(r'(\d+).*?(\d+)', text)
print(result.groups())  # ('100', '200')

# 常见对比
text = 'aabbcc'
print(re.findall(r'a.*b', text))   # ['aabb'] — 贪婪，匹配到最后一个 b
print(re.findall(r'a.*?b', text))  # ['ab']    — 非贪婪，匹配到第一个 b 就停
```

---

## 六、Match 对象的常用方法

`re.match()` 和 `re.search()` 返回的是 Match 对象，它有很多实用方法：

```python
import re

text = "订单号：A20260620，日期2026-06-20"
match = re.search(r'([A-Z])(\d{8})', text)

print(match.group())       # 'A20260620' — 整个匹配内容
print(match.group(0))      # 'A20260620' — 同 group()
print(match.group(1))      # 'A'         — 第 1 个分组
print(match.group(2))      # '20260620'  — 第 2 个分组
print(match.start())       # 4           — 匹配起始位置
print(match.end())         # 13          — 匹配结束位置
print(match.span())        # (4, 13)     — 匹配范围元组
print(match.groups())      # ('A', '20260620') — 所有分组的元组
```

---

## 七、综合实战：从文本中提取信息

假设你有一段混乱的文本，需要从中提取结构化信息：

```python
import re

text = """
客户信息汇总
================================
姓名：张三，手机：13812345678，邮箱：zhangsan@qq.com
姓名：李四，手机：13987654321，邮箱：lisi@163.com
姓名：王五，手机：15011112222，邮箱：wangwu@gmail.com
--------------------------------
日期：2026-06-20，地址：北京市朝阳区
订单编号：ORD-20260620-001，金额：1599.00 元
订单编号：ORD-20260620-002，金额：259.50 元
================================
"""

# 1. 提取所有手机号
phones = re.findall(r'1[3-9]\d{9}', text)
print("手机号：", phones)

# 2. 提取所有邮箱
emails = re.findall(r'[\w.+-]+@[\w-]+\.[\w.]+', text)
print("邮箱：", emails)

# 3. 提取所有金额
amounts = re.findall(r'(\d+\.\d+)\s*元', text)
print("金额：", amounts)

# 4. 提取订单编号和金额（分组匹配）
orders = re.findall(r'(ORD-\d{8}-\d{3}).*?(\d+\.\d+)\s*元', text)
print("订单：", orders)

# 5. 将日期格式从 YYYY-MM-DD 转为 MM/DD/YYYY
def convert_date(match):
    y, m, d = match.groups()
    return f"{m}/{d}/{y}"

new_text = re.sub(r'(\d{4})-(\d{2})-(\d{2})', convert_date, text)
print("\n日期格式转换后：")
print(new_text)
```

输出：
```
手机号： ['13812345678', '13987654321', '15011112222']
邮箱： ['zhangsan@qq.com', 'lisi@163.com', 'wangwu@gmail.com']
金额： ['1599.00', '259.50']
订单： [('ORD-20260620-001', '1599.00'), ('ORD-20260620-002', '259.50')]

日期格式转换后：
...
日期：06/20/2026，地址：北京市朝阳区
...
```

---

## 八、flags 参数（匹配标志）

re 模块的函数支持 flags 参数来改变匹配行为：

```python
import re

text = "Hello World\nhello python"

# re.IGNORECASE（或 re.I）— 忽略大小写
result = re.findall(r'hello', text, re.IGNORECASE)
print(result)  # ['Hello', 'hello']

# re.MULTILINE（或 re.M）— 多行模式，^ 和 $ 匹配每行的开头和结尾
result = re.findall(r'^hello', text, re.MULTILINE)
print(result)  # ['hello']

# re.DOTALL（或 re.S）— 让 . 匹配换行符
html = "<div>\nhello\n</div>"
result = re.findall(r'<div>(.*?)</div>', html, re.DOTALL)
print(result)  # ['\nhello\n']

# 组合使用多个 flags
result = re.findall(r'^hello', text, re.IGNORECASE | re.MULTILINE)
print(result)  # ['Hello', 'hello']

# 在正则表达式中内嵌标志（写在开头）
result = re.findall(r'(?i)hello', text)
print(result)  # ['Hello', 'hello']
```

---

## 九、练习题

### 练习 1：验证密码强度

写一个函数，验证密码是否满足以下要求：
- 至少 8 个字符
- 包含大写字母、小写字母、数字、特殊字符各至少一个

```python
import re

def check_password(password):
    """验证密码强度，返回 (是否通过, 提示信息)"""
    if len(password) < 8:
        return False, "密码长度至少 8 个字符"
    if not re.search(r'[A-Z]', password):
        return False, "需要至少一个大写字母"
    if not re.search(r'[a-z]', password):
        return False, "需要至少一个小写字母"
    if not re.search(r'\d', password):
        return False, "需要至少一个数字"
    if not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
        return False, "需要至少一个特殊字符"
    return True, "密码强度合格"

# 测试
tests = ["abc", "Abc12345", "Abcdefgh", "Abc12345!", "abc12345!"]
for pwd in tests:
    ok, msg = check_password(pwd)
    print(f"{pwd:15} → {'✅' if ok else '❌'} {msg}")
```

### 练习 2：提取 URL 中的域名

从文本中提取所有 URL，并从中分离出域名：

```python
import re

text = """
访问 https://www.baidu.com 获取搜索结果，
查看 https://github.com/python/cpython 源代码，
学习资料在 http://docs.python.org/3/ 官方文档。
"""

# TODO: 用正则提取所有域名（如 www.baidu.com, github.com, docs.python.org）
```

### 练习 3：文本清洗器

写一个函数，将一段文本中的：
- 连续多个空格压缩为一个
- HTML 标签去除
- 特殊字符替换为空格

```python
import re

def clean_text(text):
    """清洗文本"""
    # 1. 去除 HTML 标签
    text = re.sub(r'<[^>]+>', '', text)
    # 2. 压缩连续空格
    text = re.sub(r' +', ' ', text)
    # 3. 替换特殊字符
    text = re.sub(r'[^\w\s\u4e00-\u9fa5]', ' ', text)
    # 4. 去除首尾空格
    text = text.strip()
    return text

raw = '<p>Hello   World!!!</p>  <div>Python 正则</div>'
print(clean_text(raw))  # 预期: 'Hello World Python 正则'
```

---

## 十、常见问题

### Q1：什么时候用正则，什么时候用字符串方法？

**能用字符串方法就用字符串方法**，正则更强大但有性能开销：

```python
text = "hello world"

# ✅ 简单判断 → 用字符串方法
if "hello" in text:
    print("找到了")

# ✅ 简单替换 → 用字符串方法
text.replace("hello", "hi")

# ✅ 简单分割 → 用字符串方法
text.split(" ")

# ✅ 复杂匹配规则 → 用正则
re.findall(r'\d{4}-\d{2}-\d{2}', text)
```

**判断标准**：需要模式匹配（如"3位数字+连字符+4位数字"）→ 正则；简单的查找/替换/分割 → 字符串方法。

### Q2：r 前缀是什么意思？

在正则表达式的字符串前加 `r` 表示"原始字符串"（raw string），避免反斜杠被 Python 转义：

```python
# 不加 r：\b 被 Python 解释为退格符，正则引擎收到的是 ASCII 退格
re.findall('\bcat\b', text)   # 可能不工作！

# 加 r：\b 作为字面量传给正则引擎，表示单词边界
re.findall(r'\bcat\b', text)  # ✅ 正确

# 建议：写正则永远加 r 前缀
```

### Q3：正则匹配太慢怎么办？

1. **尽量使用具体的字符**：`[0-9]` 比 `\d` 快（Python 3.11+ 已优化）
2. **使用非贪婪匹配** `.*?`，避免大量回溯
3. **使用 `re.compile()`** 预编译，避免重复编译正则
4. **避免嵌套量词**：`(a+)+` 这类写法会导致灾难性回溯

### Q4：如何调试正则？

推荐在线工具：
- **regex101.com**：支持 Python 语法高亮，实时解释每个符号含义
- 写正则时，先在工具上测试确认，再复制到代码中

---

## 十一、本节小结

| 知识点 | 关键内容 |
|--------|----------|
| 基本语法 | `. \d \w \s ^ $ \b` 和量词 `* + ? {n,m}` |
| 字符类 | `[abc]` 匹配一组、`[^abc]` 取反 |
| 分组 | `( )` 提取子匹配、`\1` 反向引用 |
| 核心函数 | `match()` `search()` `findall()` `sub()` `split()` |
| 预编译 | `re.compile()` 提高性能 |
| 贪婪/非贪婪 | 默认贪婪，加 `?` 变非贪婪 |
| flags | `re.I` 忽略大小写、`re.M` 多行、`re.S` 跨行匹配 |
| 最佳实践 | 写正则加 `r` 前缀，能用字符串方法就不用正则 |

---

## 推荐学习资源

1. **廖雪峰 Python 教程 — 正则表达式**：https://www.liaoxuefeng.com/wiki/1016959663602400/1017639890281664
2. **菜鸟教程 — Python 正则表达式**：https://www.runoob.com/python/python-reg-expressions.html
3. **regex101.com**：https://regex101.com/（在线调试正则，选 Python 模式）
4. **Python 官方文档 — re 模块**：https://docs.python.org/zh-cn/3/library/re.html

---

> **下一节预告**：Day 22 将学习 **datetime 时间处理** — 掌握 Python 中日期和时间的各种操作，包括格式化、计算时间差、时区处理等实用技能。
