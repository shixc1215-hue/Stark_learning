# Python Day 50 - ORM 与 SQLAlchemy

> 欢迎来到第 50 天！昨天我们学习了用 `sqlite3` 模块直接写 SQL 语句来操作数据库。今天我们要学习一种更现代、更 Pythonic 的方式——**ORM（对象关系映射）**，以及 Python 中最强大的 ORM 框架 **SQLAlchemy**。学完今天，你就掌握了从"手写 SQL"到"用对象操作数据库"的关键跨越，这是实际工作中最常用的数据库操作方式。

---

## 一、什么是 ORM？

### 1.1 从 SQL 到 ORM

回顾昨天我们用 `sqlite3` 插入一条数据的代码：

```python
import sqlite3

conn = sqlite3.connect("school.db")
cursor = conn.cursor()
cursor.execute(
    "INSERT INTO students (name, age, grade) VALUES (?, ?, ?)",
    ("张三", 20, "A")
)
conn.commit()
```

这段代码有几个问题：
- **SQL 语句是字符串**，写错了编译器不会报错，只有运行时才崩溃
- **表结构和 Python 对象脱节**，改了表结构要到处改 SQL
- **SQL 注入风险**，虽然用了占位符，但稍不留神就可能写错

ORM 的核心思想是：**用 Python 类来表示数据库表，用类的实例（对象）来表示一行数据**。你操作对象，ORM 帮你翻译成 SQL。

```python
from sqlalchemy.orm import Session

# 用 ORM 做同样的事
student = Student(name="张三", age=20, grade="A")
session.add(student)
session.commit()
```

是不是干净很多？ORM 替你生成了 `INSERT INTO students (name, age, grade) VALUES (?, ?, ?)`，你完全不用碰 SQL。

### 1.2 ORM 术语对照

| 数据库概念 | ORM（Python）概念 | 说明 |
|-----------|-------------------|------|
| 表（Table） | 类（Class） | 一个类对应一张表 |
| 行（Row） | 对象（Object/实例） | 一个对象对应一行数据 |
| 列（Column） | 属性（Attribute） | 一个属性对应一列 |
| 外键（FK） | 关系（Relationship） | 用对象引用代替外键 ID |
| 查询语句 | 方法调用 | `query.filter()` 代替 `WHERE` |

### 1.3 ORM 的优缺点

**优点：**
- 面向对象操作，代码可读性高
- 自动防 SQL 注入
- 换数据库（如从 SQLite 换成 MySQL）几乎不用改代码
- 支持 Migration（数据库迁移）

**缺点：**
- 有性能开销（对象转换）
- 复杂查询（如多表 JOIN + 聚合）写起来比原生 SQL 啰嗦
- 需要额外学习成本

> 💡 **实际工作中的选择**：日常 CRUD 用 ORM，复杂报表查询用原生 SQL。SQLAlchemy 两种都支持。

---

## 二、SQLAlchemy 安装与核心组件

### 2.1 安装

```bash
pip install sqlalchemy
```

SQLAlchemy 2.0 是当前主流版本（2023 年发布），本教程基于 2.0 语法。

### 2.2 SQLAlchemy 的两层架构

SQLAlchemy 分为两层，理解这个架构非常重要：

```
┌─────────────────────────────┐
│   ORM 层（高层 API）         │  ← 用类和对象操作（今天重点）
│   Declarative / Session      │
├─────────────────────────────┤
│   Core 层（底层 API）        │  ← 直接操作 SQL 表达式
│   Engine / Connection        │
└─────────────────────────────┘
```

- **Core 层**：负责连接数据库、执行 SQL，是底层引擎
- **ORM 层**：在 Core 之上，提供面向对象的接口

我们今天主要学习 **ORM 层**，这也是实际开发中最常用的。

### 2.3 核心组件速览

| 组件 | 作用 | 类比 |
|------|------|------|
| `Engine` | 数据库引擎，管理连接池 | `sqlite3.connect()` |
| `DeclarativeBase` | 所有模型的基类 | 就像一个"表注册中心" |
| `Mapped` / `mapped_column` | 定义列（2.0 新语法） | `CREATE TABLE` 的列定义 |
| `Session` | 会话，管理对象和数据库的交互 | `cursor` + `commit` |
| `select()` | 查询语句（2.0 新语法） | `SELECT * FROM` |

---

## 三、定义模型（映射数据库表）

### 3.1 创建基类与引擎

```python
from sqlalchemy import create_engine, String, Integer, Float, ForeignKey, select
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session

# 1. 创建基类 —— 所有模型都继承它
class Base(DeclarativeBase):
    pass

# 2. 创建引擎 —— 连接数据库
# SQLite 文件数据库（推荐开发使用）
engine = create_engine("sqlite:///school.db", echo=False)
# echo=True 会打印生成的 SQL，方便学习时调试
# 内存数据库（测试用，程序关闭就消失）：
# engine = create_engine("sqlite:///:memory:")

# 其他数据库连接字符串格式：
# MySQL:    "mysql+pymysql://user:password@localhost:3306/dbname"
# PostgreSQL: "postgresql+psycopg://user:password@localhost:5432/dbname"
```

### 3.2 定义第一个模型

```python
class Student(Base):
    __tablename__ = "students"  # 指定表名

    # SQLAlchemy 2.0 新语法：Mapped + mapped_column
    id: Mapped[int] = mapped_column(primary_key=True)  # 主键，自动自增
    name: Mapped[str] = mapped_column(String(50))       # 姓名，VARCHAR(50)
    age: Mapped[int] = mapped_column(Integer)           # 年龄，INTEGER
    grade: Mapped[str] = mapped_column(String(2), default="B")  # 默认值
    score: Mapped[float | None] = mapped_column(Float, nullable=True)  # 可为空

    def __repr__(self):
        return f"<Student(id={self.id}, name='{self.name}', age={self.age}, grade='{self.grade}')>"

# 创建所有表（如果不存在的话）
Base.metadata.create_all(engine)
```

> 💡 **类型注解的力量**：`Mapped[int]` 不只是给 IDE 看的——SQLAlchemy 会根据它推断数据库列类型（`int` → `INTEGER`），你的 IDE 也能提供智能补全。

### 3.3 映射类型对照表

| Python 类型 | SQLAlchemy 类型 | 数据库类型 | 说明 |
|-------------|-----------------|-----------|------|
| `int` | `Integer` | INTEGER | 整数 |
| `float` | `Float` | REAL/FLOAT | 浮点数 |
| `str` | `String(n)` | VARCHAR(n) | 定长字符串 |
| `str` | `Text` | TEXT | 长文本 |
| `bool` | `Boolean` | BOOLEAN | 布尔值 |
| `datetime.datetime` | `DateTime` | DATETIME | 日期时间 |
| `datetime.date` | `Date` | DATE | 日期 |
| `bytes` | `LargeBinary` | BLOB | 二进制数据 |
| `Decimal` | `Numeric` | DECIMAL | 精确小数（金额） |

### 3.4 常用列选项

```python
class Example(Base):
    __tablename__ = "examples"

    id: Mapped[int] = mapped_column(primary_key=True)
    # primary_key=True   → 主键
    username: Mapped[str] = mapped_column(String(30), unique=True)
    # unique=True        → 唯一约束
    email: Mapped[str] = mapped_column(String(100), nullable=False)
    # nullable=False     → 不允许 NULL
    created_at: Mapped[str] = mapped_column(default="2024-01-01")
    # default=值         → 插入时的默认值
    is_active: Mapped[bool] = mapped_column(default=True)
    balance: Mapped[float] = mapped_column(default=0.0, server_default="0.0")
    # server_default     → 数据库层面的默认值（写在 DDL 里）
```

---

## 四、CRUD 操作（增删改查）

### 4.1 新增（Create）

```python
# 创建 Session
with Session(engine) as session:
    # 1. 创建单个对象
    s1 = Student(name="张三", age=20, grade="A")
    s2 = Student(name="李四", age=22, grade="B")
    s3 = Student(name="王五", age=19, grade="A")

    # 2. 添加到会话
    session.add(s1)          # 添加单个
    session.add_all([s2, s3])  # 批量添加

    # 3. 提交到数据库
    session.commit()

    # 提交后，对象会自动获得数据库分配的 id
    print(s1.id)  # 1
    print(s2.id)  # 2
```

### 4.2 查询（Read）—— SQLAlchemy 2.0 语法

```python
from sqlalchemy import select

with Session(engine) as session:
    # --- 基本查询 ---

    # 查询全部
    stmt = select(Student)
    students = session.scalars(stmt).all()  # 返回列表 [Student, Student, ...]
    for s in students:
        print(s)

    # 按主键查询单个
    student = session.get(Student, 1)  # 等价于 WHERE id=1
    print(student.name)  # 张三

    # --- 条件查询 ---

    # WHERE age > 20
    stmt = select(Student).where(Student.age > 20)
    result = session.scalars(stmt).all()

    # WHERE grade = 'A' AND age < 21
    stmt = select(Student).where(
        Student.grade == "A",
        Student.age < 21
    )
    result = session.scalars(stmt).all()

    # WHERE name LIKE '张%'
    stmt = select(Student).where(Student.name.like("张%"))
    result = session.scalars(stmt).all()

    # WHERE age IN (19, 20, 21)
    stmt = select(Student).where(Student.age.in_([19, 20, 21]))
    result = session.scalars(stmt).all()

    # --- 排序与限制 ---

    # ORDER BY age DESC LIMIT 3
    stmt = select(Student).order_by(Student.age.desc()).limit(3)
    result = session.scalars(stmt).all()

    # --- 只查询特定列 ---

    # SELECT name, age FROM students
    stmt = select(Student.name, Student.age)
    rows = session.execute(stmt).all()
    for name, age in rows:
        print(f"{name}: {age}")

    # --- 聚合查询 ---

    from sqlalchemy import func, count

    # COUNT(*)
    stmt = select(func.count()).select_from(Student)
    total = session.scalar(stmt)  # 返回单个值
    print(f"总人数: {total}")

    # AVG(score) GROUP BY grade
    stmt = (
        select(Student.grade, func.avg(Student.score).label("avg_score"))
        .group_by(Student.grade)
    )
    for grade, avg_score in session.execute(stmt):
        print(f"等级 {grade}: 平均分 {avg_score:.1f}")
```

> ⚠️ **2.0 vs 1.4 语法区别**：旧版用 `session.query(Student).filter(...)`，2.0 推荐用 `select(Student).where(...)` + `session.scalars()`。两种都能用，但新项目建议用 2.0 语法。

### 4.3 修改（Update）

```python
with Session(engine) as session:
    # 方式一：查出对象 → 修改属性 → 提交
    student = session.get(Student, 1)
    student.age = 21        # 修改属性
    student.grade = "A+"    # 再改一个
    session.commit()        # 提交后自动生成 UPDATE 语句

    # 方式二：批量更新（不加载对象，性能更好）
    from sqlalchemy import update
    stmt = (
        update(Student)
        .where(Student.grade == "C")
        .values(grade="B", score=75.0)
    )
    session.execute(stmt)
    session.commit()
```

### 4.4 删除（Delete）

```python
from sqlalchemy import delete

with Session(engine) as session:
    # 方式一：查出对象再删除
    student = session.get(Student, 3)
    if student:
        session.delete(student)
        session.commit()

    # 方式二：批量删除（不加载对象）
    stmt = delete(Student).where(Student.age < 18)
    session.execute(stmt)
    session.commit()
```

---

## 五、表间关系（Relationship）

真实项目中，表和表之间往往有关联。SQLAlchemy 的 `relationship()` 让你用对象引用代替外键 ID。

### 5.1 一对多关系

一个班级（Class）有多个学生（Student）：

```python
class ClassRoom(Base):
    __tablename__ = "classes"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))  # 班级名称，如"计算机一班"

    # 关系：一个班级有多个学生
    # back_populates 双向关联，cascade 级联删除
    students: Mapped[list["Student"]] = relationship(
        back_populates="classroom",
        cascade="all, delete-orphan"
    )

    def __repr__(self):
        return f"<ClassRoom(id={self.id}, name='{self.name}')>"


class Student(Base):
    __tablename__ = "students"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
    age: Mapped[int] = mapped_column(Integer)
    grade: Mapped[str] = mapped_column(String(2), default="B")

    # 外键：指向 classes 表的 id
    class_id: Mapped[int | None] = mapped_column(
        ForeignKey("classes.id"), nullable=True
    )

    # 关系：一个学生属于一个班级
    classroom: Mapped["ClassRoom | None"] = relationship(back_populates="students")

    def __repr__(self):
        return f"<Student(name='{self.name}', age={self.age})>"


# 重建表
Base.metadata.create_all(engine)
```

### 5.2 操作关联数据

```python
with Session(engine) as session:
    # 创建班级和学生，自动关联外键
    cls = ClassRoom(name="计算机一班")
    cls.students = [
        Student(name="张三", age=20),
        Student(name="李四", age=21),
        Student(name="王五", age=19),
    ]
    session.add(cls)  # 只需 add 班级，学生会自动级联添加
    session.commit()

    # 查询班级，自动带出所有学生（懒加载）
    cls = session.get(ClassRoom, 1)
    print(f"班级: {cls.name}")
    for s in cls.students:
        print(f"  - {s.name}, {s.age}岁")

    # 从学生查班级（反向）
    student = session.get(Student, 1)
    print(f"{student.name} 的班级是: {student.classroom.name}")
```

### 5.3 关系类型总结

| 关系类型 | 说明 | 示例 |
|---------|------|------|
| 一对多 | 一个 A 有多个 B | 班级 → 学生 |
| 多对一 | 多个 B 属于一个 A | 学生 → 班级 |
| 一对一 | 一个 A 对应一个 B | 用户 → 用户详情 |
| 多对多 | A 和 B 互相多对多 | 学生 ↔ 课程（需中间表） |

多对多示例（了解即可，进阶内容）：

```python
from sqlalchemy import Table

# 中间关联表
student_course = Table(
    "student_course",
    Base.metadata,
    mapped_column("student_id", ForeignKey("students.id"), primary_key=True),
    mapped_column("course_id", ForeignKey("courses.id"), primary_key=True),
)

class Course(Base):
    __tablename__ = "courses"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))

    # secondary 指定中间表
    students: Mapped[list["Student"]] = relationship(
        secondary=student_course, back_populates="courses"
    )
```

---

## 六、Session 深入理解

### 6.1 Session 是什么

Session 是 ORM 与数据库之间的"中间人"：

```
你的代码  →  Session（内存中跟踪对象状态）  →  数据库
              ↑ commit() 时才真正写入
```

### 6.2 对象的四种状态

```python
with Session(engine) as session:
    # 1. transient（临时态）：刚创建，未加入 Session
    s = Student(name="赵六", age=20)

    # 2. pending（待定态）：加入 Session 但未 commit
    session.add(s)
    # 此时数据库中还没有这条记录

    # 3. persistent（持久态）：commit 后，对象与数据库同步
    session.commit()
    print(s.id)  # 此时才拿到自增 id

    # 4. detached（游离态）：Session 关闭后，对象脱离管理
    # session.close() 之后 s 就是 detached 状态
```

### 6.3 Session 最佳实践

```python
# ✅ 推荐用 with 上下文管理器，自动关闭
with Session(engine) as session:
    student = session.get(Student, 1)
    student.age = 22
    session.commit()  # with 块结束自动 close

# ✅ 多个操作放一个 Session
with Session(engine) as session:
    s1 = Student(name="A", age=20)
    s2 = Student(name="B", age=21)
    session.add_all([s1, s2])
    session.commit()

# ❌ 不要每个操作开一个 Session
# session1 = Session(engine)
# do_something(session1)
# session1.close()
# session2 = Session(engine)  # 太频繁了
```

---

## 七、综合实战：图书管理系统

```python
"""
图书管理系统 —— 演示 SQLAlchemy 完整 CRUD + 关系映射
"""
from datetime import datetime
from sqlalchemy import create_engine, String, Integer, Float, ForeignKey, select, func
from sqlalchemy.orm import (
    DeclarativeBase, Mapped, mapped_column, relationship, Session
)

# ========== 1. 定义模型 ==========

class Base(DeclarativeBase):
    pass


class Author(Base):
    """作者表（一对多：一个作者有多本书）"""
    __tablename__ = "authors"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50), unique=True)
    nationality: Mapped[str] = mapped_column(String(30), default="中国")

    books: Mapped[list["Book"]] = relationship(
        back_populates="author", cascade="all, delete-orphan"
    )

    def __repr__(self):
        return f"<Author(name='{self.name}')>"


class Book(Base):
    """图书表"""
    __tablename__ = "books"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(100))
    price: Mapped[float] = mapped_column(Float, default=0.0)
    stock: Mapped[int] = mapped_column(Integer, default=0)
    author_id: Mapped[int] = mapped_column(ForeignKey("authors.id"))

    author: Mapped["Author"] = relationship(back_populates="books")

    def __repr__(self):
        return f"<Book(title='{self.title}', price={self.price})>"


# ========== 2. 初始化 ==========

engine = create_engine("sqlite:///library.db", echo=False)
Base.metadata.create_all(engine)


# ========== 3. 业务函数 ==========

def add_author(name: str, nationality: str = "中国") -> Author:
    """添加作者"""
    with Session(engine) as session:
        author = Author(name=name, nationality=nationality)
        session.add(author)
        session.commit()
        session.refresh(author)  # 刷新，拿到 id
        return author


def add_book(title: str, price: float, stock: int, author_id: int):
    """添加图书"""
    with Session(engine) as session:
        book = Book(title=title, price=price, stock=stock, author_id=author_id)
        session.add(book)
        session.commit()


def list_books():
    """列出所有图书（带作者名）"""
    with Session(engine) as session:
        stmt = select(Book)
        books = session.scalars(stmt).all()
        print("\n📚 图书列表:")
        print(f"{'ID':<5} {'书名':<20} {'价格':<8} {'库存':<6} {'作者':<10}")
        print("-" * 55)
        for b in books:
            print(f"{b.id:<5} {b.title:<20} {b.price:<8.1f} {b.stock:<6} {b.author.name:<10}")


def search_books(keyword: str):
    """按书名模糊搜索"""
    with Session(engine) as session:
        stmt = select(Book).where(Book.title.like(f"%{keyword}%"))
        books = session.scalars(stmt).all()
        print(f"\n🔍 搜索 '{keyword}' 的结果:")
        for b in books:
            print(f"  - {b.title}（{b.author.name}著）¥{b.price}")
        if not books:
            print("  未找到匹配图书")


def update_price(book_id: int, new_price: float):
    """更新价格"""
    with Session(engine) as session:
        book = session.get(Book, book_id)
        if book:
            book.price = new_price
            session.commit()
            print(f"✅ 已更新《{book.title}》价格为 ¥{new_price}")
        else:
            print("❌ 图书不存在")


def delete_book(book_id: int):
    """删除图书"""
    with Session(engine) as session:
        book = session.get(Book, book_id)
        if book:
            session.delete(book)
            session.commit()
            print(f"✅ 已删除《{book.title}》")
        else:
            print("❌ 图书不存在")


def stats():
    """统计信息"""
    with Session(engine) as session:
        # 作者数量
        author_count = session.scalar(select(func.count()).select_from(Author))
        # 图书数量
        book_count = session.scalar(select(func.count()).select_from(Book))
        # 总库存价值
        total_value = session.scalar(
            select(func.sum(Book.price * Book.stock))
        )

        print(f"\n📊 统计信息:")
        print(f"  作者总数: {author_count}")
        print(f"  图书总数: {book_count}")
        print(f"  库存总价值: ¥{total_value or 0:.2f}")

        # 每位作者的图书数量
        stmt = (
            select(Author.name, func.count(Book.id).label("book_count"))
            .join(Book)
            .group_by(Author.id)
        )
        print("\n  各作者图书数:")
        for name, count in session.execute(stmt):
            print(f"    {name}: {count} 本")


# ========== 4. 运行演示 ==========

if __name__ == "__main__":
    # 添加作者
    a1 = add_author("鲁迅", "中国")
    a2 = add_author("余华", "中国")
    a3 = add_author("J.K.罗琳", "英国")

    # 添加图书
    add_book("狂人日记", 29.0, 100, a1.id)
    add_book("呐喊", 35.0, 80, a1.id)
    add_book("活着", 42.0, 150, a2.id)
    add_book("兄弟", 58.0, 60, a2.id)
    add_book("哈利波特与魔法石", 68.0, 200, a3.id)
    add_book("哈利波特与密室", 68.0, 180, a3.id)

    # 列出所有图书
    list_books()

    # 搜索
    search_books("哈利")

    # 更新价格
    update_price(1, 39.0)

    # 删除
    delete_book(4)

    # 统计
    stats()
```

运行结果：
```
📚 图书列表:
ID    书名                  价格     库存   作者
-------------------------------------------------------
1     狂人日记              29.0    100    鲁迅
2     呐喊                  35.0    80     鲁迅
3     活着                  42.0    150    余华
4     兄弟                  58.0    60     余华
5     哈利波特与魔法石       68.0    200    J.K.罗琳
6     哈利波特与密室         68.0    180    J.K.罗琳

🔍 搜索 '哈利' 的结果:
  - 哈利波特与魔法石（J.K.罗琳著）¥68.0
  - 哈利波特与密室（J.K.罗琳著）¥68.0

✅ 已更新《狂人日记》价格为 ¥39.0
✅ 已删除《兄弟》

📊 统计信息:
  作者总数: 3
  图书总数: 5
  库存总价值: ¥24930.00

  各作者图书数:
    鲁迅: 2 本
    余华: 1 本
    J.K.罗琳: 2 本
```

---

## 八、进阶技巧

### 8.1 懒加载与预加载（N+1 问题）

```python
# ❌ N+1 查询问题：每个作者单独查一次 books
with Session(engine) as session:
    authors = session.scalars(select(Author)).all()
    for a in authors:
        print(a.name, len(a.books))  # 每次访问 a.books 都触发一次查询！

# ✅ 使用 joinedload 一次 JOIN 搞定
from sqlalchemy.orm import joinedload
with Session(engine) as session:
    stmt = select(Author).options(joinedload(Author.books))
    authors = session.scalars(stmt).unique().all()
    for a in authors:
        print(a.name, len(a.books))  # 不再触发额外查询
```

### 8.2 原生 SQL（SQLAlchemy 也支持）

```python
from sqlalchemy import text

with Session(engine) as session:
    # 执行原生 SQL
    result = session.execute(
        text("SELECT name, price FROM books WHERE price > :min_price"),
        {"min_price": 40}
    )
    for name, price in result:
        print(f"{name}: ¥{price}")
```

### 8.3 事务回滚

```python
with Session(engine) as session:
    try:
        book1 = session.get(Book, 1)
        book1.stock -= 5  # 扣库存

        # 假设这里出了错
        if book1.stock < 0:
            raise ValueError("库存不足！")

        session.commit()
    except Exception as e:
        session.rollback()  # 回滚，数据恢复原状
        print(f"操作失败，已回滚: {e}")
```

### 8.4 连接池配置

```python
# 生产环境配置连接池
engine = create_engine(
    "sqlite:///library.db",
    pool_size=5,          # 连接池大小
    max_overflow=10,      # 超出 pool_size 的最大连接数
    pool_timeout=30,      # 获取连接超时（秒）
    pool_recycle=3600,    # 连接回收时间（秒）
    echo=False,           # 是否打印 SQL
)
# 注意：SQLite 不支持连接池参数，以上仅适用于 MySQL/PostgreSQL
```

---

## 九、练习题

### 练习 1（基础）：学生选课系统
创建 `Student` 和 `Course` 两个模型，实现**多对多关系**（一个学生可以选多门课，一门课也可以被多个学生选）。
- 创建中间表 `student_course`
- 添加 3 个学生和 4 门课程
- 给学生选课
- 查询某个学生选了哪些课
- 查询某门课有哪些学生

### 练习 2（进阶）：电商订单系统
设计以下模型并实现完整 CRUD：
- `User`（用户）：id, username, email, created_at
- `Product`（商品）：id, name, price, stock
- `Order`（订单）：id, user_id, total, status, created_at
- `OrderItem`（订单明细）：id, order_id, product_id, quantity, price

要求：
- 一个用户有多个订单（一对多）
- 一个订单有多个订单明细（一对多）
- 创建订单时自动扣减商品库存
- 查询某用户的所有订单及明细

### 练习 3（挑战）：带缓存的图书查询
在综合实战的图书管理系统基础上：
- 用 `functools.lru_cache` 缓存热门查询结果
- 当图书信息修改时自动清除缓存
- 实现分页查询（`limit` + `offset`）
- 添加 `created_at` 时间字段并支持按时间范围查询

---

## 十、常见问题（FAQ）

### Q1：`session.scalars()` 和 `session.execute()` 有什么区别？

`session.execute(stmt)` 返回 `Result` 对象，包含**行（Row）**。当你查询整行对象时需要再取一次：

```python
# execute 返回 Row，每个元素是元组
result = session.execute(select(Student))
for row in result:
    print(row[0])  # row 是一个元组 (Student,)

# scalars 直接返回对象本身
students = session.scalars(select(Student))
for s in students:
    print(s)  # s 直接是 Student 对象
```

**记忆口诀**：查对象用 `scalars`，查多列用 `execute`。

### Q2：`echo=True` 打印的 SQL 怎么看不懂？

`echo=True` 打印的是参数化 SQL，参数值在下一行单独显示：
```
INSERT INTO students (name, age) VALUES (?, ?)
[generated in 0.001s] ('张三', 20)
```
`?` 是占位符，`('张三', 20)` 是实际参数。这是防注入的正确做法。

### Q3：修改了模型后怎么更新数据库表结构？

SQLAlchemy 的 `create_all()` **只创建不存在的表，不会修改已有表的结构**。修改表结构需要用 **Alembic**（SQLAlchemy 的迁移工具）：

```bash
pip install alembic
alembic init migrations    # 初始化
alembic revision --autogenerate -m "add email column"  # 生成迁移
alembic upgrade head       # 执行迁移
```

开发阶段最简单的办法是删掉 `.db` 文件重新 `create_all()`。

### Q4：SQLite 报 "database is locked" 怎么办？

这是 SQLite 并发写入的限制。解决方法：
- 确保用 `with Session(engine) as session:` 正确关闭会话
- 设置 `connect_args={"timeout": 30}` 增加等待时间
- 生产环境换用 MySQL 或 PostgreSQL

```python
engine = create_engine(
    "sqlite:///school.db",
    connect_args={"timeout": 30}
)
```

### Q5：ORM 性能比原生 SQL 差很多吗？

对于**大多数 Web 应用**（CRUD 为主），ORM 的性能损耗可以忽略不计。只有在以下场景才需要优化：
- 批量插入上万条数据 → 用 `session.bulk_save_objects()` 或 Core 的 `insert()`
- 复杂多表 JOIN 查询 → 用原生 SQL `text()`
- 高频实时查询 → 考虑加缓存

```python
# 批量插入优化
with Session(engine) as session:
    session.bulk_save_objects([
        Student(name=f"学生{i}", age=20) for i in range(10000)
    ])
    session.commit()
```

---

## 学习资源推荐

| 资源 | 链接 | 说明 |
|------|------|------|
| SQLAlchemy 官方文档 | https://docs.sqlalchemy.org/ | 最权威，2.0 教程很完整 |
| 廖雪峰 Python 教程 | https://www.liaoxuefeng.com/wiki/1016959663602400 | Python 基础部分 |
| 菜鸟教程 SQLAlchemy | https://www.runoob.com/sqlalchemy/ | 中文快速入门 |
| Real Python SQLAlchemy | https://realpython.com/python-sqlite-sqlalchemy/ | 实战导向 |
| Alembic 官方文档 | https://alembic.sqlalchemy.org/ | 数据库迁移工具 |

---

## 本节小结

今天我们学习了 SQLAlchemy ORM 的核心知识：

1. **ORM 概念**：用 Python 类映射数据库表，对象映射行，属性映射列
2. **模型定义**：继承 `DeclarativeBase`，用 `Mapped` + `mapped_column` 定义列
3. **CRUD 操作**：`session.add()` 增、`select()` 查、改属性更新、`session.delete()` 删
4. **表间关系**：`relationship()` + `ForeignKey` 实现一对多、多对多
5. **Session 机制**：对象状态管理、事务提交与回滚
6. **进阶技巧**：N+1 问题、原生 SQL、事务、连接池
7. **综合实战**：完整的图书管理系统

> 🎯 **关键转变**：从现在起，操作数据库时先想"我要操作什么对象"，而不是"我要写什么 SQL"。这是 ORM 思维的核心。

**下一站 Day 51**：我们将进入 **Streamlit** 的世界——用纯 Python 就能快速搭建交互式 Web 应用，把你的数据分析结果变成漂亮的网页！

---

> 📅 **Day 50 完成！** 你已经走完了 Python 60 天计划的 5/6，基础语法 → 工程实践 → 数据处理 → Web 开发，下一阶段将进入**前端与全栈实战**。坚持就是胜利！💪
