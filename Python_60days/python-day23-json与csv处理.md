# Python Day 23：json 与 csv 处理

> **学习目标**：掌握 Python 中 `json` 和 `csv` 模块的核心用法，能够进行数据的序列化/反序列化、CSV 文件读写，并结合实际业务场景做数据交换与持久化。

---

## 一、什么是数据序列化？

在编程中，我们经常需要在不同的地方"搬运"数据：

| 场景 | 说明 |
|------|------|
| 保存到文件 | 程序运行的结果存下来，下次启动还能用 |
| 网络传输 | 把 Python 数据发给其他系统（API 调用） |
| 配置文件 | 程序的设置用文件存储，而不是写死在代码里 |

**序列化（Serialization）**：把 Python 对象（字典、列表等）转成字符串/字节流，方便存储或传输。

**反序列化（Deserialization）**：把字符串/字节流转回 Python 对象。

```
Python 对象 ←——反序列化——→ 字符串（json / csv）
   {'name': 'Tom'}          '{"name": "Tom"}'
```

今天学两种最常用的数据格式：**JSON** 和 **CSV**。

---

## 二、JSON 处理

### 2.1 JSON 是什么？

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，几乎所有的编程语言都支持。

```json
{
    "name": "张三",
    "age": 25,
    "skills": ["Python", "SQL"],
    "address": {
        "city": "北京",
        "district": "海淀"
    }
}
```

JSON 和 Python 数据类型的对照关系：

| JSON | Python |
|------|--------|
| `object` `{}` | `dict` |
| `array` `[]` | `list` |
| `"string"` | `str` |
| `123` | `int` |
| `3.14` | `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

> ⚠️ 注意：JSON 的布尔值是小写的 `true/false`，Python 是大写的 `True/False`。JSON 的空值是 `null`，Python 是 `None`。

### 2.2 json 模块核心函数

Python 内置的 `json` 模块提供了四个最常用的函数：

```python
import json

# ====== 1. json.dumps() —— Python 对象 → JSON 字符串 ======
student = {
    "name": "张三",
    "age": 25,
    "skills": ["Python", "SQL"],
    "address": {"city": "北京", "district": "海淀"}
}

# 转成 JSON 字符串
json_str = json.dumps(student, ensure_ascii=False, indent=4)
print(json_str)
print(type(json_str))  # <class 'str'>

# ensure_ascii=False：允许中文正常显示（否则会被转义成 \uXXXX）
# indent=4：缩进4个空格，让输出更易读


# ====== 2. json.loads() —— JSON 字符串 → Python 对象 ======
json_data = '{"name": "李四", "age": 30, "is_student": false}'
result = json.loads(json_data)
print(result)          # {'name': '李四', 'age': 30, 'is_student': False}
print(type(result))    # <class 'dict'>


# ====== 3. json.dump() —— Python 对象 → 写入 JSON 文件 ======
data = {
    "project": "学生管理系统",
    "version": "1.0",
    "students": [
        {"name": "张三", "score": 95},
        {"name": "李四", "score": 88},
        {"name": "王五", "score": 92}
    ]
}

with open("students.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=4)
# 文件写入完成，可以去查看 students.json


# ====== 4. json.load() —— 读取 JSON 文件 → Python 对象 ======
with open("students.json", "r", encoding="utf-8") as f:
    loaded_data = json.load(f)

print(loaded_data["project"])           # 学生管理系统
print(loaded_data["students"][0]["name"])  # 张三
```

> 💡 **记忆技巧**：带 `s` 的操作字符串（dumps/loads），不带 `s` 的操作文件（dump/load）。

### 2.3 JSON 高级用法

```python
import json

# ----- 按键排序输出 -----
data = {"c": 3, "a": 1, "b": 2}
print(json.dumps(data, sort_keys=True))  # {"a": 1, "b": 2, "c": 3}


# ----- 自定义分隔符（压缩输出，减少文件体积） -----
data = {"name": "test", "value": 123}
compact = json.dumps(data, separators=(",", ":"))
print(compact)  # {"name":"test","value":123}


# ----- 处理不支持的 Python 类型 -----
# JSON 原生不支持 set、datetime 等类型
from datetime import datetime, date

data = {
    "name": "test",
    "created_at": datetime(2026, 6, 22, 10, 30, 0),
    "tags": {"Python", "JSON"}  # set 类型
}

# 直接 dumps 会报错！TypeError
# 解决方法1：自定义 default 函数
def my_converter(obj):
    """处理 JSON 不支持的 Python 类型"""
    if isinstance(obj, datetime):
        return obj.strftime("%Y-%m-%d %H:%M:%S")
    if isinstance(obj, date):
        return obj.strftime("%Y-%m-%d")
    if isinstance(obj, set):
        return list(obj)
    raise TypeError(f"Type {type(obj)} not serializable")

json_str = json.dumps(data, default=my_converter, ensure_ascii=False)
print(json_str)
# {"name": "test", "created_at": "2026-06-22 10:30:00", "tags": ["Python", "JSON"]}
```

### 2.4 JSON 与中文编码的坑

```python
import json

data = {"name": "中文测试", "city": "上海"}

# 不加 ensure_ascii=False 时，中文会被转义
print(json.dumps(data))
# 输出: {"name": "\u4e2d\u6587\u6d4b\u8bd5", "city": "\u4e0a\u6d77"}

# 加上 ensure_ascii=False 才能正常显示中文
print(json.dumps(data, ensure_ascii=False))
# 输出: {"name": "中文测试", "city": "上海"}

# 💡 建议：凡是涉及中文，都加上 ensure_ascii=False
```

---

## 三、CSV 处理

### 3.1 CSV 是什么？

CSV（Comma-Separated Values，逗号分隔值）是最简单的表格数据格式，Excel 可以直接打开。

```
姓名,年龄,城市,成绩
张三,25,北京,95
李四,30,上海,88
王五,28,广州,92
```

每个字段用逗号分隔，第一行通常是表头。

> 💡 CSV 文件的本质就是纯文本，用记事本就能打开和编辑。

### 3.2 csv 模块基础读写

```python
import csv

# ====== 写入 CSV ======
headers = ["姓名", "年龄", "城市", "成绩"]
rows = [
    ["张三", 25, "北京", 95],
    ["李四", 30, "上海", 88],
    ["王五", 28, "广州", 92],
]

with open("students.csv", "w", newline="", encoding="utf-8-sig") as f:
    writer = csv.writer(f)
    writer.writerow(headers)   # 写入表头
    writer.writerows(rows)      # 批量写入数据行

# ⚠️ newline="" 是必须的！否则 Windows 上会出现空行
# ⚠️ encoding="utf-8-sig" 带 BOM，Excel 打开不会乱码


# ====== 读取 CSV ======
with open("students.csv", "r", encoding="utf-8-sig") as f:
    reader = csv.reader(f)
    headers = next(reader)  # 先读取表头
    print(f"表头: {headers}")

    for row in reader:       # 逐行读取
        print(row)
        # ['张三', '25', '北京', '95']
        # ['李四', '30', '上海', '88']
        # ...

    # 也可以一次性读取所有行
    # all_rows = list(reader)
```

> 💡 **编码小知识**：`utf-8-sig` 在文件开头加了 BOM（字节顺序标记），用 Excel 打开 CSV 时能正确识别编码。如果用 `utf-8`，Excel 打开中文 CSV 可能会乱码。

### 3.3 DictReader 和 DictWriter（推荐使用）

`csv.DictReader` 每行返回一个字典，按列名访问数据，比按索引访问更安全：

```python
import csv

# ====== 写入（字典模式）======
with open("students.csv", "w", newline="", encoding="utf-8-sig") as f:
    fieldnames = ["姓名", "年龄", "城市", "成绩"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()  # 自动写入表头

    writer.writerow({"姓名": "张三", "年龄": 25, "城市": "北京", "成绩": 95})
    writer.writerow({"姓名": "李四", "年龄": 30, "城市": "上海", "成绩": 88})
    writer.writerows([
        {"姓名": "王五", "年龄": 28, "城市": "广州", "成绩": 92},
        {"姓名": "赵六", "年龄": 27, "城市": "深圳", "成绩": 85}
    ])


# ====== 读取（字典模式）======
with open("students.csv", "r", encoding="utf-8-sig") as f:
    reader = csv.DictReader(f)

    # reader.fieldnames 可以获取表头
    print(f"列名: {reader.fieldnames}")
    # 列名: ['姓名', '年龄', '城市', '成绩']

    for row in reader:
        print(f"{row['姓名']} 来自 {row['城市']}，成绩 {row['成绩']}")
        # 用列名访问，代码可读性更好
        # 不用记住第几列是什么
```

### 3.4 处理 CSV 中的特殊情况

```python
import csv

# ----- 字段包含逗号 -----
# CSV 会自动用引号包裹含逗号的字段
data = [
    ["张三", "北京市,海淀区", "Python, SQL, Java"],
    ["李四", "上海市,浦东新区", "C++, Go"]
]

with open("complex.csv", "w", newline="", encoding="utf-8-sig") as f:
    writer = csv.writer(f)
    writer.writerow(["姓名", "地址", "技能"])
    writer.writerows(data)

# complex.csv 内容：
# 姓名,地址,技能
# 张三,"北京市,海淀区","Python, SQL, Java"
# 李四,"上海市,浦东新区","C++, Go"


# ----- 自定义分隔符 -----
# 有些 CSV 用分号、制表符分隔（常见于欧洲地区）
with open("tab_separated.tsv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f, delimiter="\t")  # 制表符分隔
    writer.writerow(["姓名", "年龄"])
    writer.writerow(["张三", 25])


# ----- 跳过特定行 -----
with open("students.csv", "r", encoding="utf-8-sig") as f:
    reader = csv.DictReader(f)
    for row in reader:
        if row["姓名"] == "李四":
            continue  # 跳过李四
        print(row)
```

---

## 四、JSON 与 CSV 的互转

实际工作中，经常需要在 JSON 和 CSV 之间转换数据：

```python
import json
import csv

# ====== 场景1：JSON → CSV ======
json_data = '''
[
    {"name": "张三", "age": 25, "city": "北京"},
    {"name": "李四", "age": 30, "city": "上海"},
    {"name": "王五", "age": 28, "city": "广州"}
]
'''

students = json.loads(json_data)

with open("output.csv", "w", newline="", encoding="utf-8-sig") as f:
    # 从第一条数据自动推断列名
    writer = csv.DictWriter(f, fieldnames=students[0].keys())
    writer.writeheader()
    writer.writerows(students)

print("JSON → CSV 完成！")


# ====== 场景2：CSV → JSON ======
result = []

with open("output.csv", "r", encoding="utf-8-sig") as f:
    reader = csv.DictReader(f)
    for row in reader:
        # 注意：CSV 读取的都是字符串，需要类型转换
        row["age"] = int(row["age"])  # 字符串 → 整数
        result.append(row)

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(result, f, ensure_ascii=False, indent=4)

print("CSV → JSON 完成！")
print(result)
# [{'name': '张三', 'age': 25, 'city': '北京'}, ...]
```

> ⚠️ **CSV 数据类型陷阱**：CSV 中所有数据都是字符串！`"25"` 读出来是字符串 `"25"`，不是整数 `25`。做数值计算时记得转换类型。

---

## 五、实战：员工信息管理系统

把今天学的 JSON 和 CSV 结合起来，做一个完整的员工信息管理小工具：

```python
import json
import csv
import os

DATA_FILE = "employees.json"
EXPORT_FILE = "employees.csv"


def load_employees():
    """从 JSON 文件加载员工数据"""
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, "r", encoding="utf-8") as f:
        return json.load(f)


def save_employees(employees):
    """保存员工数据到 JSON 文件"""
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(employees, f, ensure_ascii=False, indent=4)
    print(f"✅ 已保存 {len(employees)} 条数据到 {DATA_FILE}")


def add_employee(employees):
    """添加员工信息"""
    emp = {
        "id": input("工号: ").strip(),
        "name": input("姓名: ").strip(),
        "department": input("部门: ").strip(),
        "position": input("职位: ").strip(),
        "salary": int(input("薪资: ").strip()),
        "join_date": input("入职日期(YYYY-MM-DD): ").strip()
    }
    employees.append(emp)
    save_employees(employees)
    print(f"✅ 员工 {emp['name']} 添加成功！")


def list_employees(employees):
    """展示所有员工信息"""
    if not employees:
        print("📋 暂无员工数据")
        return

    print(f"\n{'工号':<8}{'姓名':<8}{'部门':<10}{'职位':<12}{'薪资':<10}{'入职日期'}")
    print("-" * 65)
    for emp in employees:
        print(f"{emp['id']:<8}{emp['name']:<8}{emp['department']:<10}"
              f"{emp['position']:<12}{emp['salary']:<10}{emp['join_date']}")
    print(f"\n共 {len(employees)} 名员工")


def export_to_csv(employees):
    """导出员工数据为 CSV（方便用 Excel 查看）"""
    if not employees:
        print("⚠️ 没有数据可导出")
        return

    fieldnames = ["id", "name", "department", "position", "salary", "join_date"]
    with open(EXPORT_FILE, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(employees)

    print(f"✅ 已导出 {len(employees)} 条数据到 {EXPORT_FILE}（可用 Excel 打开）")


def search_employee(employees):
    """按姓名搜索员工"""
    keyword = input("请输入搜索姓名: ").strip()
    results = [emp for emp in employees if keyword in emp["name"]]

    if results:
        print(f"🔍 找到 {len(results)} 条匹配结果:")
        for emp in results:
            print(f"  - {emp['name']} | {emp['department']} | {emp['position']}")
    else:
        print("🔍 未找到匹配的员工")


def main():
    """主程序"""
    employees = load_employees()

    while True:
        print("\n" + "=" * 40)
        print("   📋 员工信息管理系统")
        print("=" * 40)
        print("  1. 添加员工")
        print("  2. 查看员工列表")
        print("  3. 搜索员工")
        print("  4. 导出为 CSV")
        print("  0. 退出")
        print("=" * 40)

        choice = input("请选择操作 (0-4): ").strip()

        if choice == "1":
            add_employee(employees)
        elif choice == "2":
            list_employees(employees)
        elif choice == "3":
            search_employee(employees)
        elif choice == "4":
            export_to_csv(employees)
        elif choice == "0":
            print("👋 再见！")
            break
        else:
            print("⚠️ 无效选项，请重新选择")


if __name__ == "__main__":
    main()
```

运行流程示例：
```
请选择操作 (0-4): 1
工号: 001
姓名: 张三
部门: 技术部
职位: Python工程师
薪资: 15000
入职日期(YYYY-MM-DD): 2025-03-15
✅ 员工 张三 添加成功！
✅ 已保存 1 条数据到 employees.json

请选择操作 (0-4): 4
✅ 已导出 1 条数据到 employees.csv（可用 Excel 打开）
```

---

## 六、常见问题与排错

### Q1：`json.dumps()` 中文变成 `\uXXXX` 怎么办？

```python
# ❌ 错误写法
json.dumps({"name": "张三"})
# 输出: {"name": "\u5f20\u4e09"}

# ✅ 正确写法
json.dumps({"name": "张三"}, ensure_ascii=False)
# 输出: {"name": "张三"}
```

### Q2：CSV 文件在 Excel 中中文乱码？

```python
# ❌ 用 utf-8 写入，Excel 可能乱码
open("data.csv", "w", encoding="utf-8")

# ✅ 用 utf-8-sig（带 BOM），Excel 能正确识别中文
open("data.csv", "w", encoding="utf-8-sig")
```

### Q3：CSV 写入后每行之间有空行？

```python
# ❌ 不加 newline 参数
open("data.csv", "w", encoding="utf-8")

# ✅ 加 newline=""（Windows 必须加！）
open("data.csv", "w", newline="", encoding="utf-8")
```

### Q4：`json.load()` 报 `JSONDecodeError`？

常见原因：
- JSON 文件内容为空
- JSON 格式不正确（如多余的逗号、单引号代替双引号）

```python
# ❌ Python 用单引号，但 JSON 必须用双引号
# '{"name": "Tom"}'  ✅ 正确的 JSON
# "{'name': 'Tom'}"  ❌ 不是合法的 JSON

# 排查方法：先读出文件内容看看
with open("data.json", "r") as f:
    content = f.read()
    print(repr(content))  # 查看原始内容
```

### Q5：大数据文件怎么处理？

```python
import json

# JSON 文件太大，一次性加载内存不够用？
# 解决方案：使用 ijson 库（第三方）流式解析
# pip install ijson
# import ijson
# with open("big_data.json") as f:
#     for item in ijson.items(f, "item"):
#         process(item)  # 逐条处理，不全部加载到内存
```

---

## 七、今日练习题

### 练习1：JSON 配置文件管理器

编写一个程序，实现以下功能：
1. 创建一个 `config.json` 配置文件，包含数据库连接信息、日志级别、超时时间等
2. 提供修改配置值的功能（通过命令行输入键名和新值）
3. 提供读取配置的功能
4. 修改后自动保存回文件

```python
# 预期效果：
# 当前配置: {'db_host': 'localhost', 'db_port': 3306, 'log_level': 'INFO'}
# 请输入要修改的键: db_port
# 请输入新值: 5432
# ✅ 配置已更新并保存
```

### 练习2：CSV 成绩分析工具

给定以下 CSV 数据（自己创建文件），完成统计任务：

```
姓名,语文,数学,英语
张三,90,85,92
李四,78,95,88
王五,92,80,76
赵六,85,90,95
钱七,70,65,80
```

要求：
1. 读取 CSV 文件
2. 计算每个学生的总分和平均分
3. 找出每科的最高分和最低分学生
4. 将统计结果导出为新的 CSV 文件

### 练习3：JSON 数据清洗

给定一个嵌套较深的 JSON 数据，提取并整理关键信息：

```python
raw_data = '''
{
    "company": "XX科技",
    "departments": [
        {
            "name": "技术部",
            "manager": "张三",
            "employees": [
                {"name": "员工A", "salary": 15000, "level": "P6"},
                {"name": "员工B", "salary": 20000, "level": "P7"}
            ]
        },
        {
            "name": "市场部",
            "manager": "李四",
            "employees": [
                {"name": "员工C", "salary": 12000, "level": "P5"}
            ]
        }
    ]
}
'''
```

要求：
1. 解析 JSON 数据
2. 统计每个部门的人数和平均薪资
3. 找出薪资最高的员工
4. 将结果整理为一个新的字典并保存为 JSON

---

## 八、总结

今天学习了两大数据处理利器：

```
json 模块                          csv 模块
├── dumps()  → Python→字符串        ├── csv.writer()     → 按列表写入
├── loads()  → 字符串→Python        ├── csv.DictWriter() → 按字典写入（推荐）
├── dump()   → Python→文件          ├── csv.reader()     → 按列表读取
└── load()   → 文件→Python          └── csv.DictReader() → 按字典读取（推荐）

关键注意事项：
• JSON 中文字符串 → ensure_ascii=False
• CSV 写入 → newline=""（Windows）
• CSV 用 Excel 打开 → encoding="utf-8-sig"
• CSV 读出的全是字符串 → 需要类型转换
```

| 格式 | 优势 | 劣势 | 适用场景 |
|------|------|------|----------|
| JSON | 支持嵌套结构、类型丰富 | 文件体积较大 | 配置文件、API 交互、复杂数据 |
| CSV | 简单直观、Excel 直接打开 | 只能表示二维表格 | 表格数据导出、数据报表 |

---

## 九、免费学习资源

- **廖雪峰 Python 教程 - JSON**：https://www.liaoxuefeng.com/wiki/1016959663602400/1017651167731552
- **菜鸟教程 - JSON**：https://www.runoob.com/python/python-json.html
- **菜鸟教程 - CSV**：https://www.runoob.com/python/python-csv.html
- **Python 官方文档 - json 模块**：https://docs.python.org/zh-cn/3/library/json.html
- **Python 官方文档 - csv 模块**：https://docs.python.org/zh-cn/3/library/csv.html

---

> 📅 **下一篇预告**：Day 24 将学习 **os 与 pathlib** —— 掌握文件与目录操作，自动化管理你的文件系统。
