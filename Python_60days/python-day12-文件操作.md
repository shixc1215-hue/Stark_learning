# Python Day 12：文件操作 —— 让数据"活"起来

> 🎯 今日目标：掌握文件读写、with 语句、CSV 和 JSON 处理，让程序能"记住"数据
> ⏱ 预计用时：60 分钟 | 📅 进度：Day 12/60
> 🏷 阶段：第一阶段 · 基础语法入门（Day 1-14）

---

## 一、为什么学文件操作？

到目前为止，你的程序数据都在**内存**里——程序一关，数据就没了。

文件操作让你能够：
- 💾 **持久化**：把数据存到硬盘，下次还能读回来
- 📊 **处理真实数据**：读取 CSV 报表、JSON 配置文件
- 🔄 **批量处理**：自动处理成百上千个文件

> 一句话：没有文件操作，程序就是"金鱼记忆"——7秒就忘。

---

## 二、文件读写基础（15 分钟）

### 2.1 打开文件的三步曲

```python
# 第一步：打开文件
f = open("hello.txt", "w", encoding="utf-8")   # "w" = write 写入

# 第二步：操作文件
f.write("Hello, Python!\n")
f.write("今天是 Day 12\n")

# 第三步：关闭文件（重要！）
f.close()
```

### 2.2 打开模式速查表

| 模式 | 含义 | 文件不存在时 | 文件存在时 |
|------|------|-------------|-----------|
| `"r"` | 只读（默认） | ❌ 报错 | 从头读 |
| `"w"` | 只写 | ✅ 新建 | **清空**重写 |
| `"a"` | 追加 | ✅ 新建 | 末尾追加 |
| `"x"` | 独占创建 | ✅ 新建 | ❌ 报错 |
| `"r+"` | 读写 | ❌ 报错 | 从头读写 |
| `"rb"` | 二进制读 | ❌ 报错 | — |
| `"wb"` | 二进制写 | ✅ 新建 | 清空重写 |

> ⚠️ **最常见翻车**：用 `"w"` 模式打开已有文件——内容瞬间清空，无法恢复！

### 2.3 读取文件的几种方式

```python
# 先创建一个示例文件
with open("demo.txt", "w", encoding="utf-8") as f:
    f.write("第一行：Python真好玩\n")
    f.write("第二行：文件操作很简单\n")
    f.write("第三行：坚持就是胜利\n")

# 方式1：read() —— 一次性读取全部内容
with open("demo.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)

# 方式2：readline() —— 每次读一行
with open("demo.txt", "r", encoding="utf-8") as f:
    line1 = f.readline()  # "第一行：Python真好玩\n"
    line2 = f.readline()  # "第二行：文件操作很简单\n"
    print(line1, end="")  # end="" 避免多一个换行
    print(line2, end="")

# 方式3：readlines() —— 读取所有行，返回列表
with open("demo.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    print(type(lines))   # <class 'list'>
    print(lines)          # ['第一行...\n', '第二行...\n', '第三行...\n']

# 方式4（⭐推荐）：直接遍历文件对象
with open("demo.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line, end="")  # 最省内存，逐行读取
```

**四种读取方式怎么选？**

| 方式 | 适用场景 | 内存占用 |
|------|---------|---------|
| `read()` | 小文件，需要整体处理 | 高（全部加载） |
| `readline()` | 只需读前几行 | 低 |
| `readlines()` | 需要按行索引访问 | 中（全部行） |
| `for line in f` | **逐行处理大文件** | **最低** |

---

## 三、with 语句 —— 文件操作的正确姿势（10 分钟）

### 3.1 为什么要用 with？

```python
# ❌ 危险写法：忘记 close()
f = open("data.txt", "w")
f.write("重要数据")
# 如果这里出异常，f.close() 永远不会执行！
# → 文件锁住，数据丢失

# ✅ 安全写法：with 自动关闭
with open("data.txt", "w", encoding="utf-8") as f:
    f.write("重要数据")
# 离开 with 块，文件自动关闭，即使出现异常也没问题
```

**with 的三大好处：**
1. **自动关闭**：离开代码块自动调用 `f.close()`
2. **异常安全**：即使中间报错，文件也会正确关闭
3. **代码简洁**：少写一行 `f.close()`

> 📌 **铁律**：凡是操作文件，一律用 `with open(...)` ，不要手动 `f.close()`！

### 3.2 同时操作多个文件

```python
# 同时打开两个文件：一个读，一个写
with open("input.txt", "r", encoding="utf-8") as fin, \
     open("output.txt", "w", encoding="utf-8") as fout:
    for line in fin:
        fout.write(line.upper())  # 把每行转大写写入新文件
```

---

## 四、CSV 文件处理（15 分钟）

CSV（Comma-Separated Values）是最常见的数据交换格式，Excel 可以直接打开。

### 4.1 手动读写 CSV（了解原理）

```python
# 写入 CSV
with open("scores.csv", "w", encoding="utf-8") as f:
    f.write("姓名,语文,数学,英语\n")
    f.write("张三,85,92,78\n")
    f.write("李四,90,88,95\n")
    f.write("王五,76,95,82\n")

# 读取 CSV
with open("scores.csv", "r", encoding="utf-8") as f:
    header = f.readline().strip().split(",")  # 表头
    print(header)  # ['姓名', '语文', '数学', '英语']
    for line in f:
        row = line.strip().split(",")
        print(row)  # ['张三', '85', '92', '78']
```

### 4.2 用 csv 模块读写（⭐推荐）

```python
import csv

# ---------- 写入 CSV ----------
with open("scores.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["姓名", "语文", "数学", "英语"])  # 写表头
    writer.writerow(["张三", 85, 92, 78])
    writer.writerow(["李四", 90, 88, 95])
    writer.writerow(["王五", 76, 95, 82])

# 批量写入
students = [
    ["赵六", 88, 76, 90],
    ["钱七", 92, 85, 88],
]
with open("scores.csv", "a", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(students)  # 注意：writerows（有s）批量写入

# ---------- 读取 CSV ----------
with open("scores.csv", "r", encoding="utf-8") as f:
    reader = csv.reader(f)
    header = next(reader)  # 读取表头
    print(f"表头: {header}")
    for row in reader:
        print(f"{row[0]}: 语文{row[1]} 数学{row[2]} 英语{row[3]}")

# ⭐ 用 DictReader / DictWriter（按列名访问，更直观）
with open("scores.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"{row['姓名']}的数学成绩: {row['数学']}")
        # row 是字典！row = {'姓名': '张三', '语文': '85', ...}
```

> ⚠️ **Windows 避坑**：写 CSV 时必须加 `newline=""`，否则会出现空行！

---

## 五、JSON 文件处理（10 分钟）

JSON 是 Web 世界的数据交换标准，也是配置文件的首选格式。

### 5.1 JSON vs Python 数据类型对照

| JSON | Python |
|------|--------|
| `null` | `None` |
| `true/false` | `True/False` |
| 数字 | `int` / `float` |
| 字符串 | `str` |
| 数组 `[...]` | `list` |
| 对象 `{...}` | `dict` |

### 5.2 json 模块核心操作

```python
import json

# ---------- Python → JSON 字符串（序列化） ----------
data = {
    "name": "史星辰",
    "skills": ["Python", "SQL", "ETL"],
    "experience": 6,
    "is_learning": True,
    "project": None
}

# dumps：转成 JSON 字符串
json_str = json.dumps(data, ensure_ascii=False, indent=2)
print(json_str)
# ensure_ascii=False → 正确显示中文
# indent=2 → 美化缩进

# ---------- JSON 字符串 → Python（反序列化） ----------
parsed = json.loads(json_str)
print(parsed["name"])   # 史星辰
print(parsed["skills"]) # ['Python', 'SQL', 'ETL']

# ---------- 直接读写 JSON 文件 ----------
# 写入
with open("config.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
    # 注意：是 dump（无s），直接写入文件对象

# 读取
with open("config.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)
    # 注意：是 load（无s），直接从文件读取
    print(loaded)
```

### 5.3 dump vs dumps 速记

| 函数 | 作用 | 记忆技巧 |
|------|------|---------|
| `json.dump(obj, f)` | 写入**文件** | dump 到文件 |
| `json.dumps(obj)` | 转成**字符串** | s = string |
| `json.load(f)` | 从**文件**读取 | 从文件 load |
| `json.loads(s)` | 从**字符串**读取 | s = string |

---

## 六、文件路径与 os 模块（5 分钟）

```python
import os

# 获取当前工作目录
print(os.getcwd())

# 拼接路径（跨平台安全，不用手动写 / 或 \）
data_dir = os.path.join("data", "raw")
filepath = os.path.join(data_dir, "scores.csv")
print(filepath)  # Windows: data\raw\scores.csv

# 检查文件/目录是否存在
print(os.path.exists("scores.csv"))      # True/False
print(os.path.isfile("scores.csv"))      # 是否是文件
print(os.path.isdir("data"))             # 是否是目录

# 创建目录
os.makedirs("data/raw", exist_ok=True)   # 已存在不报错

# 列出目录下的文件
files = os.listdir(".")
print([f for f in files if f.endswith(".csv")])
```

> 📌 **路径拼接铁律**：永远用 `os.path.join()`，不要手动拼接 `/` 或 `\`！

---

## 七、实战项目：个人记账本（5 分钟）

把之前学的全部串起来——一个能**持久保存**的记账本：

```python
import json
import os

DATA_FILE = "expenses.json"

def load_expenses():
    """从文件加载记账数据"""
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return []  # 文件不存在，返回空列表

def save_expenses(expenses):
    """保存记账数据到文件"""
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(expenses, f, ensure_ascii=False, indent=2)

def add_expense(expenses):
    """添加一笔支出"""
    item = input("支出项目: ")
    amount = float(input("金额: "))
    category = input("分类（餐饮/交通/购物/其他）: ")
    date = input("日期（如 2026-06-05）: ")

    expense = {
        "item": item,
        "amount": amount,
        "category": category,
        "date": date
    }
    expenses.append(expense)
    save_expenses(expenses)  # 立刻保存！
    print(f"✅ 已记录: {item} ¥{amount}")

def show_expenses(expenses):
    """显示所有支出"""
    if not expenses:
        print("暂无记录")
        return

    print(f"\n{'日期':<12} {'分类':<6} {'项目':<10} {'金额':>8}")
    print("-" * 40)
    total = 0
    for e in expenses:
        print(f"{e['date']:<12} {e['category']:<6} {e['item']:<10} ¥{e['amount']:>7.2f}")
        total += e["amount"]
    print("-" * 40)
    print(f"{'合计':>30} ¥{total:>7.2f}")

def show_by_category(expenses):
    """按分类汇总"""
    summary = {}
    for e in expenses:
        cat = e["category"]
        summary[cat] = summary.get(cat, 0) + e["amount"]

    print("\n📊 分类汇总:")
    for cat, amount in sorted(summary.items(), key=lambda x: x[1], reverse=True):
        print(f"  {cat}: ¥{amount:.2f}")

def main():
    expenses = load_expenses()  # 启动时加载数据

    while True:
        print("\n===== 📒 个人记账本 =====")
        print("1. 记一笔支出")
        print("2. 查看所有记录")
        print("3. 分类汇总")
        print("4. 退出")

        choice = input("请选择 (1-4): ")

        if choice == "1":
            add_expense(expenses)
        elif choice == "2":
            show_expenses(expenses)
        elif choice == "3":
            show_by_category(expenses)
        elif choice == "4":
            print("再见！数据已保存 💾")
            break
        else:
            print("无效选择，请重试")

if __name__ == "__main__":
    main()
```

**运行效果：**
```
===== 📒 个人记账本 =====
1. 记一笔支出
2. 查看所有记录
3. 分类汇总
4. 退出
请选择 (1-4): 1
支出项目: 午餐
金额: 35
分类（餐饮/交通/购物/其他）: 餐饮
日期（如 2026-06-05）: 2026-06-05
✅ 已记录: 午餐 ¥35.0

===== 📒 个人记账本 =====
请选择 (1-4): 2

日期         分类   项目         金额
----------------------------------------
2026-06-05   餐饮   午餐        ¥35.00
----------------------------------------
                             合计 ¥35.00
```

> 🎯 **重点体会**：数据在 `expenses.json` 文件中持久保存，关掉程序再打开，数据还在！

---

## 八、常见问题与避坑（5 分钟）

### ❌ 坑1：忘记 encoding="utf-8"

```python
# ❌ Windows 默认编码可能是 GBK，读中文文件报错
f = open("中文.txt", "r")

# ✅ 始终指定 utf-8
f = open("中文.txt", "r", encoding="utf-8")
```

### ❌ 坑2：用 "w" 模式误清空文件

```python
# ❌ 只是想读文件，却用成了 "w" → 文件清空！
with open("重要数据.csv", "w") as f:  # 灾难！
    pass

# ✅ 读用 "r"，追加用 "a"，确认要用 "w"
with open("重要数据.csv", "r", encoding="utf-8") as f:
    content = f.read()
```

### ❌ 坑3：CSV 写入不加 newline=""

```python
# ❌ Windows 上会出现空行
with open("data.csv", "w", encoding="utf-8") as f:
    writer = csv.writer(f)

# ✅ 加上 newline=""
with open("data.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.writer(f)
```

### ❌ 坑4：JSON 不支持某些 Python 类型

```python
import json
from datetime import datetime

data = {"time": datetime.now()}  # datetime 对象

# ❌ json.dumps 报错：TypeError: Object of type datetime is not JSON serializable
json.dumps(data)

# ✅ 转成字符串再存储
data = {"time": datetime.now().strftime("%Y-%m-%d %H:%M:%S")}
json.dumps(data)  # OK
```

### ❌ 坑5：路径硬编码斜杠

```python
# ❌ Windows 和 Mac/Linux 路径分隔符不同
path = "data\\raw\\scores.csv"  # Windows 可以，Mac 不行

# ✅ 用 os.path.join 跨平台
import os
path = os.path.join("data", "raw", "scores.csv")
```

---

## 九、今日练习

### 练习1：文件内容复制器（基础）

```python
def copy_file(src, dst):
    """将 src 文件内容复制到 dst 文件"""
    # 提示：用 with 同时打开两个文件
    pass
```

<details>
<summary>🔑 参考答案</summary>

```python
def copy_file(src, dst):
    with open(src, "r", encoding="utf-8") as fin, \
         open(dst, "w", encoding="utf-8") as fout:
        for line in fin:
            fout.write(line)

copy_file("demo.txt", "demo_copy.txt")
```

</details>

### 练习2：CSV 成绩分析器（进阶）

读取上面的 `scores.csv`，计算每个学生的总分和平均分，写入新文件 `result.csv`。

```python
def analyze_scores(input_file, output_file):
    """
    读取成绩CSV，计算总分和平均分，写入新CSV
    输出格式：姓名,语文,数学,英语,总分,平均分
    """
    pass
```

<details>
<summary>🔑 参考答案</summary>

```python
import csv

def analyze_scores(input_file, output_file):
    with open(input_file, "r", encoding="utf-8") as fin, \
         open(output_file, "w", encoding="utf-8", newline="") as fout:

        reader = csv.DictReader(fin)

        # 新表头：原列 + 总分 + 平均分
        fieldnames = reader.fieldnames + ["总分", "平均分"]
        writer = csv.DictWriter(fout, fieldnames=fieldnames)
        writer.writeheader()

        for row in reader:
            chinese = int(row["语文"])
            math = int(row["数学"])
            english = int(row["英语"])
            total = chinese + math + english
            avg = round(total / 3, 1)

            row["总分"] = total
            row["平均分"] = avg
            writer.writerow(row)

analyze_scores("scores.csv", "result.csv")
```

</details>

### 练习3：JSON 配置管理器（实战）

```python
def load_config(path):
    """加载JSON配置，文件不存在则返回默认配置"""
    pass

def save_config(path, config):
    """保存配置到JSON文件"""
    pass

def update_config(path, key, value):
    """更新配置中的某个字段"""
    pass
```

<details>
<summary>🔑 参考答案</summary>

```python
import json
import os

DEFAULT_CONFIG = {
    "app_name": "MyApp",
    "version": "1.0",
    "debug": False,
    "max_retries": 3
}

def load_config(path):
    if os.path.exists(path):
        with open(path, "r", encoding="utf-8") as f:
            return json.load(f)
    return DEFAULT_CONFIG.copy()

def save_config(path, config):
    with open(path, "w", encoding="utf-8") as f:
        json.dump(config, f, ensure_ascii=False, indent=2)

def update_config(path, key, value):
    config = load_config(path)
    config[key] = value
    save_config(path, config)
    print(f"✅ 已更新: {key} = {value}")

# 测试
config = load_config("app_config.json")
print(config)
update_config("app_config.json", "debug", True)
update_config("app_config.json", "theme", "dark")
```

</details>

---

## 十、今日知识点速查卡

```
┌─────────────────────────────────────────────────────┐
│              Day 12 · 文件操作速查                      │
├─────────────────────────────────────────────────────┤
│ 打开文件:  with open("file", "r", encoding="utf-8")  │
│ 读取:      read() / readline() / readlines() / for   │
│ 写入:      write() / writelines()                    │
│ 模式:      r(读) w(写) a(追加) x(独占) rb/wb(二进制)   │
│ CSV读:     csv.reader / csv.DictReader               │
│ CSV写:     csv.writer / csv.DictWriter               │
│ JSON→文件:  json.dump(obj, f)                         │
│ 文件→JSON:  json.load(f)                             │
│ JSON→字符串: json.dumps(obj)                          │
│ 字符串→JSON: json.loads(str)                         │
│ 路径拼接:  os.path.join("a", "b")                    │
│ 文件存在:  os.path.exists(path)                      │
├─────────────────────────────────────────────────────┤
│ ⚠️ 铁律: ① 始终 with open  ② 始终 encoding="utf-8"  │
│          ③ CSV写加 newline=""  ④ 路径用 os.path.join │
└─────────────────────────────────────────────────────┘
```

---

## 十一、免费学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| Python 官方教程 | docs.python.org/zh-cn/3/tutorial/inputoutput.html | 文件读写官方文档 |
| 菜鸟教程 CSV | www.runoob.com/python3/python3-csv.html | CSV 模块中文教程 |
| Real Python JSON | realpython.com/python-json/ | JSON 深度讲解（英文） |

---

## 📅 明日预告

**Day 13：异常处理 + 模块导入**
- try/except/else/finally 完整结构
- 常见异常类型与自定义异常
- import/from...import 模块导入
- pip 安装第三方库

> 💪 坚持到 Day 12 了！你已经能读写文件、处理 CSV 和 JSON，程序终于可以"记忆"了！
