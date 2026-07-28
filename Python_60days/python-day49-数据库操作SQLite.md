# Python Day 49 - 数据库操作 SQLite

> 欢迎来到第 49 天！今天我们将学习 Python 内置的数据库模块 `sqlite3`，它是 Python 标准库的一部分，**无需安装任何额外包**就能直接使用。SQLite 是世界上最广泛部署的数据库引擎，几乎所有智能手机、浏览器和操作系统都在用它。

---

## 一、什么是 SQLite？

### 1.1 概念介绍

SQLite 是一个**轻量级的嵌入式关系型数据库**，和 MySQL、PostgreSQL 不同，它不需要单独安装数据库服务器，数据直接存在一个 `.db` 文件里。

| 特性 | SQLite | MySQL / PostgreSQL |
|------|--------|-------------------|
| 安装 | 零配置，无需安装 | 需要安装数据库服务 |
| 运行方式 | 嵌入到程序中 | 独立的服务进程 |
| 适用场景 | 小型应用、开发测试、移动端 | 大型生产环境 |
| 并发能力 | 适合中低并发 | 适合高并发 |
| Python 支持 | 标准库内置 `sqlite3` | 需安装第三方库 |

### 1.2 为什么学 SQLite？

- **Python 标准库自带**，`import sqlite3` 就能用
- **零配置**，不需要安装任何数据库软件
- 数据存在一个文件中，方便备份和迁移
- 完整支持 SQL 语法（增删改查、事务、索引等）
- 学会了 SQLite 的 SQL，切换到 MySQL/PostgreSQL 非常容易

---

## 二、基本操作：连接、建表、增删改查

### 2.1 连接数据库

```python
import sqlite3

# 连接数据库（如果文件不存在会自动创建）
conn = sqlite3.connect("test.db")

# 关闭连接
conn.close()
```

> **提示**：用 `with` 语句可以自动关闭连接，更安全：

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    # 在这里操作数据库
    # 退出 with 块后连接自动关闭
    pass
```

### 2.2 创建游标

游标（Cursor）是用来执行 SQL 语句和获取结果的对象：

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()  # 创建游标
    # 用游标执行 SQL...
```

### 2.3 创建表

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    # 创建一个学生信息表
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS students (
            id        INTEGER PRIMARY KEY AUTOINCREMENT,  -- 主键，自增
            name      TEXT    NOT NULL,                  -- 姓名，不能为空
            age       INTEGER,                           -- 年龄
            gender    TEXT     DEFAULT '未知',            -- 性别，默认值
            major     TEXT,                              -- 专业
            gpa       REAL                               -- 绩点（浮点数）
        )
    """)

    conn.commit()  # 提交事务，保存更改
```

### 2.4 插入数据

**方式一：直接传入值**

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    # 插入一条数据（用 ? 作为占位符，防止 SQL 注入）
    cursor.execute(
        "INSERT INTO students (name, age, gender, major, gpa) VALUES (?, ?, ?, ?, ?)",
        ("张三", 20, "男", "计算机科学", 3.8)
    )

    conn.commit()
```

> **重要**：永远使用 `?` 占位符，不要用字符串拼接！这样可以**防止 SQL 注入攻击**：
>
> ```python
> # 危险！千万别这样写！
> cursor.execute(f"INSERT INTO students (name) VALUES ('{user_input}')")
>
> # 安全：使用占位符
> cursor.execute("INSERT INTO students (name) VALUES (?)", (user_input,))
> ```

**方式二：批量插入（executemany）**

```python
import sqlite3

students_data = [
    ("李四", 21, "女", "数据科学", 3.9),
    ("王五", 22, "男", "人工智能", 3.5),
    ("赵六", 20, "女", "软件工程", 3.7),
    ("钱七", 23, "男", "信息安全", 3.2),
    ("孙八", 21, "女", "计算机科学", 3.6),
]

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()
    cursor.executemany(
        "INSERT INTO students (name, age, gender, major, gpa) VALUES (?, ?, ?, ?, ?)",
        students_data
    )
    conn.commit()
    print(f"插入了 {cursor.rowcount} 条数据")
```

### 2.5 查询数据

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    # 查询所有学生
    cursor.execute("SELECT * FROM students")
    results = cursor.fetchall()  # 获取所有结果，返回列表
    for row in results:
        print(row)
```

输出类似：
```
(1, '张三', 20, '男', '计算机科学', 3.8)
(2, '李四', 21, '女', '数据科学', 3.9)
...
```

**常用查询方法对比：**

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `fetchall()` | 获取所有行 | 列表的列表 |
| `fetchone()` | 获取下一行 | 单个元组，没有则返回 None |
| `fetchmany(n)` | 获取 n 行 | 列表的列表 |

```python
# fetchone：只取一条
cursor.execute("SELECT * FROM students WHERE name = ?", ("张三",))
row = cursor.fetchone()
if row:
    print(f"找到了：{row}")

# fetchmany：取前 3 条
cursor.execute("SELECT * FROM students")
rows = cursor.fetchmany(3)
print(f"前 3 条：{rows}")
```

### 2.6 条件查询与排序

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    # 条件查询：查找绩点大于 3.5 的学生
    cursor.execute("SELECT name, gpa FROM students WHERE gpa > ?", (3.5,))
    high_gpa = cursor.fetchall()
    print("高绩点学生：", high_gpa)

    # 模糊查询：查找姓'张'的学生
    cursor.execute("SELECT * FROM students WHERE name LIKE ?", ("张%",))
    print("姓张的学生：", cursor.fetchall())

    # 排序：按绩点从高到低
    cursor.execute("SELECT name, gpa FROM students ORDER BY gpa DESC")
    print("按绩点排序：", cursor.fetchall())

    # 聚合查询：统计人数和平均绩点
    cursor.execute("SELECT COUNT(*), AVG(gpa) FROM students")
    count, avg_gpa = cursor.fetchone()
    print(f"共 {count} 人，平均绩点 {avg_gpa:.2f}")
```

### 2.7 更新和删除数据

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    # 更新：把张三的绩点改为 4.0
    cursor.execute("UPDATE students SET gpa = ? WHERE name = ?", (4.0, "张三"))
    print(f"更新了 {cursor.rowcount} 行")

    # 删除：删除绩点低于 3.3 的学生
    cursor.execute("DELETE FROM students WHERE gpa < ?", (3.3,))
    print(f"删除了 {cursor.rowcount} 行")

    conn.commit()
```

---

## 三、让查询结果更友好

默认 `fetchall()` 返回的是元组，我们可以用以下两种方式获得更友好的结果。

### 3.1 用 `row_factory` 获取字典结果

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    # 设置行工厂，让结果以字典形式返回
    conn.row_factory = sqlite3.Row

    cursor = conn.cursor()
    cursor.execute("SELECT name, age, major FROM students")

    for row in cursor.fetchall():
        # 现在可以用列名访问
        print(f"姓名: {row['name']}, 年龄: {row['age']}, 专业: {row['major']}")
```

### 3.2 手动转换为字典列表

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT name, age, major, gpa FROM students")

    # 获取列名
    columns = [desc[0] for desc in cursor.description]

    # 将结果转为字典列表
    students_list = [dict(zip(columns, row)) for row in cursor.fetchall()]

    for s in students_list:
        print(f"{s['name']} - {s['major']} - GPA: {s['gpa']}")
```

---

## 四、事务（Transaction）

事务是数据库操作的基石，它保证一组操作**要么全部成功，要么全部失败**。

### 4.1 什么是事务？

想象你在银行转账：从 A 账户扣 100 元，给 B 账户加 100 元。如果扣款成功但加钱失败，钱就丢了。事务保证这两个操作作为一个整体，要么都成功，要么都不发生。

### 4.2 在 Python 中使用事务

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    try:
        # 开启事务（Python 的 sqlite3 默认自动开启）
        cursor.execute("UPDATE students SET gpa = gpa - 0.1 WHERE name = ?", ("王五",))
        cursor.execute("UPDATE students SET gpa = gpa + 0.1 WHERE name = ?", ("李四",))

        conn.commit()  # 全部成功，提交
        print("事务提交成功")

    except Exception as e:
        conn.rollback()  # 出错则回滚，撤销所有更改
        print(f"事务回滚：{e}")
```

> **说明**：`with sqlite3.connect()` 默认会在成功退出时自动 commit，异常退出时自动 rollback。但显式控制更清晰。

---

## 五、创建索引提高查询速度

当数据量大时，索引可以大幅提高查询效率：

```python
import sqlite3

with sqlite3.connect("test.db") as conn:
    cursor = conn.cursor()

    # 在 name 列上创建索引
    cursor.execute("CREATE INDEX IF NOT EXISTS idx_students_name ON students(name)")

    # 在 major + gpa 上创建复合索引
    cursor.execute(
        "CREATE INDEX IF NOT EXISTS idx_students_major_gpa ON students(major, gpa)"
    )

    conn.commit()

    # 有索引后，按姓名查询会更快
    cursor.execute("SELECT * FROM students WHERE name = ?", ("张三",))
    print(cursor.fetchone())
```

---

## 六、实战：学生管理系统

把今天学的知识综合起来，做一个简单但完整的"学生管理系统"：

```python
import sqlite3

DB_FILE = "school.db"


def init_db():
    """初始化数据库和表结构"""
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS students (
                id      INTEGER PRIMARY KEY AUTOINCREMENT,
                name    TEXT NOT NULL,
                age     INTEGER,
                gender  TEXT DEFAULT '未知',
                major   TEXT,
                gpa     REAL DEFAULT 0.0
            )
        """)
        conn.commit()


def add_student(name, age, gender, major, gpa):
    """添加学生"""
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute(
            "INSERT INTO students (name, age, gender, major, gpa) VALUES (?, ?, ?, ?, ?)",
            (name, age, gender, major, gpa)
        )
        conn.commit()
        print(f"添加学生成功：{name} (ID: {cursor.lastrowid})")


def query_all_students():
    """查询所有学生"""
    with sqlite3.connect(DB_FILE) as conn:
        conn.row_factory = sqlite3.Row
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM students ORDER BY id")
        students = cursor.fetchall()

        if not students:
            print("暂无学生数据")
            return

        print(f"{'ID':<4} {'姓名':<8} {'年龄':<4} {'性别':<4} {'专业':<12} {'GPA':<5}")
        print("-" * 45)
        for s in students:
            print(f"{s['id']:<4} {s['name']:<8} {s['age']:<4} {s['gender']:<4} "
                  f"{s['major']:<12} {s['gpa']:<5.1f}")


def search_by_major(major):
    """按专业查询"""
    with sqlite3.connect(DB_FILE) as conn:
        conn.row_factory = sqlite3.Row
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM students WHERE major = ? ORDER BY gpa DESC", (major,))
        results = cursor.fetchall()

        print(f"\n【{major}】专业的学生：")
        for s in results:
            print(f"  {s['name']} - GPA: {s['gpa']}")
        return results


def update_gpa(student_id, new_gpa):
    """更新学生绩点"""
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("UPDATE students SET gpa = ? WHERE id = ?", (new_gpa, student_id))
        conn.commit()
        if cursor.rowcount > 0:
            print(f"更新成功：ID={student_id} 的新绩点为 {new_gpa}")
        else:
            print(f"未找到 ID={student_id} 的学生")


def delete_student(student_id):
    """删除学生"""
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("DELETE FROM students WHERE id = ?", (student_id,))
        conn.commit()
        if cursor.rowcount > 0:
            print(f"已删除 ID={student_id} 的学生")
        else:
            print(f"未找到 ID={student_id} 的学生")


def get_statistics():
    """获取统计信息"""
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            SELECT
                COUNT(*) as total,
                AVG(gpa) as avg_gpa,
                MAX(gpa) as max_gpa,
                MIN(gpa) as min_gpa
            FROM students
        """)
        total, avg_gpa, max_gpa, min_gpa = cursor.fetchone()

        print("\n===== 学生统计 =====")
        print(f"总人数：{total}")
        print(f"平均绩点：{avg_gpa:.2f}")
        print(f"最高绩点：{max_gpa}")
        print(f"最低绩点：{min_gpa}")

        # 按专业分组统计
        cursor.execute("""
            SELECT major, COUNT(*) as cnt, AVG(gpa) as avg_gpa
            FROM students
            GROUP BY major
            ORDER BY cnt DESC
        """)
        print("\n各专业统计：")
        for row in cursor.fetchall():
            print(f"  {row[0]}：{row[1]} 人，平均 GPA {row[2]:.2f}")


# ===== 主程序 =====
if __name__ == "__main__":
    init_db()

    # 插入测试数据
    test_data = [
        ("张三", 20, "男", "计算机科学", 3.8),
        ("李四", 21, "女", "计算机科学", 3.6),
        ("王五", 22, "男", "人工智能", 3.9),
        ("赵六", 20, "女", "数据科学", 3.7),
        ("钱七", 23, "男", "人工智能", 3.4),
        ("孙八", 21, "女", "软件工程", 3.5),
    ]

    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.executemany(
            "INSERT OR IGNORE INTO students (name, age, gender, major, gpa) VALUES (?, ?, ?, ?, ?)",
            test_data
        )
        conn.commit()

    # 演示各功能
    print("\n===== 所有学生 =====")
    query_all_students()

    search_by_major("计算机科学")
    search_by_major("人工智能")

    print()
    update_gpa(1, 3.9)  # 把张三的绩点改为 3.9

    print()
    get_statistics()

    # print("\n删除测试...")
    # delete_student(6)
    # query_all_students()
```

---

## 七、常见问题

### Q1：`sqlite3.connect()` 和 `sqlite3.connect(":memory:")` 有什么区别？

- `sqlite3.connect("test.db")`：数据保存到磁盘文件 `test.db` 中，程序关闭后数据还在
- `sqlite3.connect(":memory:")`：数据保存在内存中，程序关闭后数据就丢失了，适合测试

```python
# 内存数据库示例（适合写单元测试）
conn = sqlite3.connect(":memory:")
cursor = conn.cursor()
cursor.execute("CREATE TABLE test (id INTEGER, name TEXT)")
cursor.execute("INSERT INTO test VALUES (1, 'hello')")
print(cursor.fetchone())  # (1, 'hello')
conn.close()  # 数据消失
```

### Q2：为什么 `fetchall()` 返回的是元组而不是字典？

因为默认的 `row_factory` 是 None，返回元组。你可以通过设置 `conn.row_factory = sqlite3.Row` 来获得支持列名访问的结果（见 3.1 节）。

### Q3：占位符 `?` 和字符串拼接有什么区别？

```python
# 字符串拼接 — 有 SQL 注入风险！
# 如果 user_input = "'; DROP TABLE students; --"
# 最终执行的 SQL 会是：SELECT * FROM students WHERE name = ''; DROP TABLE students; --'
cursor.execute(f"SELECT * FROM students WHERE name = '{user_input}'")

# 使用占位符 — 安全！参数会被正确转义
cursor.execute("SELECT * FROM students WHERE name = ?", (user_input,))
```

### Q4：`conn.commit()` 一定要写吗？

如果你使用 `with` 语句块，Python 会在正常退出 `with` 块时**自动提交**（commit），异常退出时**自动回滚**（rollback）。但显式调用 `commit()` 是好习惯，代码更清晰。

### Q5：SQLite 支持哪些数据类型？

SQLite 使用**动态类型系统**（类型亲和性），但推荐使用以下标准类型：

| 类型 | 说明 | 示例 |
|------|------|------|
| `INTEGER` | 整数 | 1, 42, -7 |
| `REAL` | 浮点数 | 3.14, -0.5 |
| `TEXT` | 文本字符串 | "hello", "你好" |
| `BLOB` | 二进制数据 | 图片、文件等 |
| `NULL` | 空值 | NULL |

---

## 八、练习题

### 练习 1：创建成绩表（基础）

创建一个 `courses` 表，包含字段：`id`（主键自增）、`course_name`（课程名）、`teacher`（教师）、`credit`（学分）。

插入 3-5 门课程数据，然后查询所有课程并打印。

### 练习 2：关联查询（进阶）

在 `students` 表基础上，创建 `scores` 表（`student_id`, `course_name`, `score`）。

插入一些成绩数据，然后用 SQL 的 **JOIN** 语法查询每个学生各科成绩：

```python
cursor.execute("""
    SELECT s.name, sc.course_name, sc.score
    FROM students s
    JOIN scores sc ON s.id = sc.student_id
    ORDER BY s.name, sc.course_name
""")
```

### 练习 3：批量导入导出（挑战）

编写一个程序：
1. 从 CSV 文件读取学生数据（可以用 Python 内置的 `csv` 模块，Day 23 学过）
2. 批量插入到 SQLite 数据库
3. 再从数据库查询数据，写入另一个 CSV 文件

这其实就是最简单的"数据导入导出"功能。

---

## 九、推荐学习资源

- **SQLite 官方文档**：https://www.sqlite.org/docs.html
- **Python sqlite3 官方文档**：https://docs.python.org/zh-cn/3/library/sqlite3.html
- **廖雪峰 Python 教程 - 数据库**：https://www.liaoxuefeng.com/wiki/1016959663602400/1017806484193792
- **菜鸟教程 - SQLite**：https://www.runoob.com/sqlite/sqlite-tutorial.html
- **SQL 在线练习**：https://www.sqlzoo.net/ （学好 SQL 语法很重要）

---

## 十、总结

| 知识点 | 要点 |
|--------|------|
| 连接数据库 | `sqlite3.connect("文件名.db")` |
| 创建游标 | `conn.cursor()` |
| 执行 SQL | `cursor.execute(sql, params)` |
| 批量操作 | `cursor.executemany(sql, data_list)` |
| 获取结果 | `fetchall()` / `fetchone()` / `fetchmany(n)` |
| 字典结果 | `conn.row_factory = sqlite3.Row` |
| 提交事务 | `conn.commit()` |
| 回滚事务 | `conn.rollback()` |
| 安全查询 | 始终使用 `?` 占位符 |

SQLite 是轻量但功能完整的数据库，非常适合小型项目和开发学习。掌握了它之后，明天我们将学习 **ORM 与 SQLAlchemy**，用更 Pythonic 的方式操作数据库。

> **明天预告**：Day 50 - ORM 与 SQLAlchemy，用 Python 类来操作数据库，告别手写 SQL！
