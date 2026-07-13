# Python Day 14：第一阶段总结 + 综合实战（学生管理系统）

> **进度：Day 14/60 | 阶段：第一阶段收尾 | 预计用时：1~1.5 小时**

---

## 一、第一阶段知识地图（14天回顾）

过去两周，你从零基础走到了能写完整小项目的水平。来回顾一下你掌握了什么：

```
┌─────────────────────────────────────────────────────────┐
│                  Python 第一阶段知识体系                   │
├───────────────┬───────────────┬─────────────────────────┤
│   Day 1-3     │   Day 4-6     │     Day 7-9              │
│   基础入门     │   流程控制     │     数据结构              │
├───────────────┼───────────────┼─────────────────────────┤
│ • 变量与赋值   │ • if/elif/else│ • 列表(list)增删改查切片  │
│ • 数据类型     │ • 比较/逻辑运算│ • 元组(tuple)解包        │
│   int/float   │ • for 循环    │ • 字典(dict)键值操作     │
│   str/bool    │ • while 循环  │ • 字符串方法全家桶        │
│ • print/input │ • break       │ • enumerate/zip          │
│ • f-string    │ • continue    │ • 推导式入门             │
│ • 注释习惯     │ • range()     │                          │
├───────────────┼───────────────┼─────────────────────────┤
│   Day 10-11   │   Day 12-13   │      Day 14              │
│   函数世界     │   工程能力     │      整合实战             │
├───────────────┼───────────────┼─────────────────────────┤
│ • 函数def定义  │ • 文件读写    │ • 综合项目：学生管理系统   │
│ • 参数5种形式  │ • with语句    │ • 模块化拆分             │
│ • 返回值return │ • CSV/JSON   │ • 异常全覆盖             │
│ • 作用域/闭包  │ • os/path操作 │ • 数据持久化             │
│ • lambda匿名  │ • try/except  │ • 代码规范总结           │
│ • map/filter   │ • raise自定义 │                          │
│ • sorted/key   │ • pip/venv   │                          │
│ • 递归入门     │ • if __name__ │                          │
└───────────────┴───────────────┴─────────────────────────┘
```

### 你已具备的核心能力

| 能力 | 关键词 | 掌握程度 |
|------|--------|---------|
| 变量与运算 | 命名、类型转换、f-string | ✅ 熟练 |
| 流程控制 | if/for/while、break/continue | ✅ 熟练 |
| 数据结构 | list/tuple/dict/set | ✅ 会用 |
| 函数封装 | def/参数/返回值/作用域 | ✅ 能写 |
| 高阶函数 | lambda/map/filter/sorted | ✅ 理解 |
| 文件操作 | 读写/CSV/JSON/with | ✅ 能写 |
| 异常处理 | try/except/raise | ✅ 基础 |
| 模块组织 | import/pip/venv | ✅ 理解 |

---

## 二、综合实战：学生管理系统（完整版）

> **为什么选这个项目？** 它几乎用到了第一阶段的全部知识点：列表、字典、函数、文件读写、JSON持久化、异常处理、模块拆分。写完这个你就是真正的入门者了！

### 功能需求

```
========== 学生管理系统 ==========
1. 添加学生
2. 查看所有学生
3. 查询学生（按学号/姓名）
4. 修改学生信息
5. 删除学生
6. 成绩统计（均分/排名）
7. 保存数据到文件
8. 从文件加载数据
0. 退出系统
==================================
```

### 项目结构（模块化拆分）

```
student_system/
├── main.py          # 主程序入口（菜单 + 调度）
├── models.py        # 数据模型（学生类）
├── service.py       # 业务逻辑（增删改查）
├── storage.py       # 数据持久化（JSON读写）
└── utils.py         # 工具函数（输入验证等）
```

---

## 三、逐步实现

### 步骤 1：数据模型 `models.py`

```python
"""
models.py - 学生数据模型
"""

class Student:
    """学生类"""

    def __init__(self, student_id, name, age, scores=None):
        self.student_id = student_id      # 学号（字符串，唯一标识）
        self.name = name                  # 姓名
        self.age = age                    # 年龄
        self.scores = scores if scores is not None else []  # 成绩列表

    @property
    def average(self):
        """计算平均分"""
        if not self.scores:
            return 0.0
        return round(sum(self.scores) / len(self.scores), 2)

    @property
    def total(self):
        """计算总分"""
        return sum(self.scores)

    def add_score(self, score):
        """添加一门成绩"""
        self.scores.append(score)

    def to_dict(self):
        """转为字典（方便JSON序列化）"""
        return {
            "student_id": self.student_id,
            "name": self.name,
            "age": self.age,
            "scores": self.scores,
        }

    @classmethod
    def from_dict(cls, data):
        """从字典创建学生对象"""
        return cls(
            student_id=data["student_id"],
            name=data["name"],
            age=data["age"],
            scores=data.get("scores", []),
        )

    def __str__(self):
        return (
            f"学号: {self.student_id}  "
            f"姓名: {self.name}  "
            f"年龄: {self.age}  "
            f"成绩: {self.scores}  "
            f"均分: {self.average}"
        )
```

### 步骤 2：数据持久化 `storage.py`

```python
"""
storage.py - JSON文件读写，负责数据持久化
"""
import json
import os
from models import Student


class Storage:
    """数据存储类"""

    def __init__(self, filepath="students.json"):
        self.filepath = filepath

    def save(self, students):
        """保存学生列表到JSON文件"""
        data = [s.to_dict() for s in students]
        with open(self.filepath, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
        print(f"✅ 数据已保存到 {self.filepath}（共 {len(students)} 条记录）")

    def load(self):
        """从JSON文件加载学生列表"""
        if not os.path.exists(self.filepath):
            print("⚠️ 数据文件不存在，将创建新文件")
            return []

        try:
            with open(self.filepath, "r", encoding="utf-8") as f:
                data = json.load(f)
            students = [Student.from_dict(item) for item in data]
            print(f"✅ 已从 {self.filepath} 加载 {len(students)} 条记录")
            return students
        except (json.JSONDecodeError, KeyError) as e:
            print(f"❌ 数据文件损坏: {e}")
            return []
```

### 步骤 3：工具函数 `utils.py`

```python
"""
utils.py - 输入验证等工具函数
"""

def input_int(prompt, min_val=None, max_val=None):
    """安全地读取一个整数"""
    while True:
        try:
            value = int(input(prompt))
            if min_val is not None and value < min_val:
                print(f"⚠️ 值不能小于 {min_val}，请重新输入")
                continue
            if max_val is not None and value > max_val:
                print(f"⚠️ 值不能大于 {max_val}，请重新输入")
                continue
            return value
        except ValueError:
            print("⚠️ 请输入一个有效的整数！")


def input_nonempty(prompt):
    """读取一个非空字符串"""
    while True:
        value = input(prompt).strip()
        if value:
            return value
        print("⚠️ 输入不能为空，请重新输入")


def input_scores():
    """读取成绩列表"""
    scores = []
    while True:
        score_str = input("请输入成绩（直接回车结束）: ").strip()
        if not score_str:
            break
        try:
            score = float(score_str)
            if 0 <= score <= 100:
                scores.append(score)
            else:
                print("⚠️ 成绩应在 0-100 之间")
        except ValueError:
            print("⚠️ 请输入有效的数字！")
    return scores


def confirm_action(prompt):
    """确认操作"""
    answer = input(f"{prompt} (y/n): ").strip().lower()
    return answer in ("y", "yes", "是")
```

### 步骤 4：业务逻辑 `service.py`

```python
"""
service.py - 学生管理核心业务逻辑
"""
from models import Student
from utils import input_int, input_nonempty, input_scores, confirm_action


class StudentService:
    """学生管理服务"""

    def __init__(self, storage):
        self.students = storage.load()      # 从文件加载
        self.storage = storage

    # ========== 增删改查 ==========

    def add_student(self):
        """添加学生"""
        print("\n--- 添加学生 ---")
        sid = input_nonempty("请输入学号: ")

        # 检查学号是否重复
        if self._find_by_id(sid):
            print(f"❌ 学号 {sid} 已存在！")
            return

        name = input_nonempty("请输入姓名: ")
        age = input_int("请输入年龄: ", min_val=1, max_val=120)
        scores = input_scores()

        student = Student(sid, name, age, scores)
        self.students.append(student)
        print(f"✅ 学生 {name} 添加成功！")

    def show_all(self):
        """显示所有学生"""
        print("\n--- 所有学生 ---")
        if not self.students:
            print("还没有学生记录，请先添加")
            return

        # 用 sorted 按学号排序展示
        sorted_students = sorted(self.students, key=lambda s: s.student_id)
        for i, stu in enumerate(sorted_students, 1):
            print(f"{i}. {stu}")

    def search_student(self):
        """查询学生（按学号或姓名）"""
        print("\n--- 查询学生 ---")
        keyword = input_nonempty("请输入学号或姓名关键字: ")

        # 用 filter 筛选
        results = list(filter(
            lambda s: keyword in s.student_id or keyword in s.name,
            self.students
        ))

        if not results:
            print(f"❌ 未找到与 '{keyword}' 相关的学生")
            return

        print(f"\n找到 {len(results)} 条记录:")
        for stu in results:
            print(f"  {stu}")

    def update_student(self):
        """修改学生信息"""
        print("\n--- 修改学生信息 ---")
        sid = input_nonempty("请输入要修改的学生学号: ")

        student = self._find_by_id(sid)
        if not student:
            print(f"❌ 学号 {sid} 不存在！")
            return

        print(f"当前信息: {student}")
        print("（直接回车保留原值）")

        new_name = input(f"新姓名 [{student.name}]: ").strip()
        if new_name:
            student.name = new_name

        new_age_str = input(f"新年龄 [{student.age}]: ").strip()
        if new_age_str:
            try:
                new_age = int(new_age_str)
                if 1 <= new_age <= 120:
                    student.age = new_age
                else:
                    print("⚠️ 年龄超出范围，保留原值")
            except ValueError:
                print("⚠️ 格式错误，保留原值")

        print("重新录入成绩？")
        if confirm_action("确认重新录入成绩"):
            student.scores = input_scores()

        print(f"✅ 学生 {student.name} 信息已更新！")

    def delete_student(self):
        """删除学生"""
        print("\n--- 删除学生 ---")
        sid = input_nonempty("请输入要删除的学生学号: ")

        student = self._find_by_id(sid)
        if not student:
            print(f"❌ 学号 {sid} 不存在！")
            return

        print(f"将删除: {student}")
        if confirm_action("确认删除？"):
            self.students.remove(student)
            print(f"✅ 学生 {student.name} 已删除！")

    # ========== 统计功能 ==========

    def show_statistics(self):
        """成绩统计分析"""
        print("\n--- 成绩统计 ---")

        if not self.students:
            print("还没有学生记录")
            return

        # 收集所有成绩
        all_scores = []
        for stu in self.students:
            all_scores.extend(stu.scores)

        if not all_scores:
            print("还没有录入任何成绩")
            return

        # 使用 Python 内置函数做统计
        avg_score = sum(all_scores) / len(all_scores)
        max_score = max(all_scores)
        min_score = min(all_scores)

        # 统计各分数段人数（列表推导式）
        excellent = len([s for s in all_scores if s >= 90])
        good = len([s for s in all_scores if 80 <= s < 90])
        medium = len([s for s in all_scores if 70 <= s < 80])
        passing = len([s for s in all_scores if 60 <= s < 70])
        failed = len([s for s in all_scores if s < 60])

        print(f"总人数: {len(self.students)}")
        print(f"总成绩数: {len(all_scores)}")
        print(f"平均分: {avg_score:.2f}")
        print(f"最高分: {max_score}")
        print(f"最低分: {min_score}")
        print(f"\n分数段分布:")
        print(f"  优秀 (90-100): {excellent} 人")
        print(f"  良好 (80-89):  {good} 人")
        print(f"  中等 (70-79):  {medium} 人")
        print(f"  及格 (60-69):  {passing} 人")
        print(f"  不及格 (<60):  {failed} 人")

        # 按平均分排名（sorted + lambda）
        print(f"\n🏆 学生排名（按均分降序）:")
        ranked = sorted(self.students, key=lambda s: s.average, reverse=True)
        for i, stu in enumerate(ranked, 1):
            medal = "🥇" if i == 1 else "🥈" if i == 2 else "🥉" if i == 3 else f"{i}."
            print(f"  {medal} {stu}")

    # ========== 辅助方法 ==========

    def _find_by_id(self, student_id):
        """按学号查找学生（返回对象或None）"""
        for student in self.students:
            if student.student_id == student_id:
                return student
        return None

    def save_and_exit(self):
        """保存并退出"""
        self.storage.save(self.students)
        print("👋 再见！")
```

### 步骤 5：主程序 `main.py`

```python
"""
main.py - 学生管理系统主入口
"""
from storage import Storage
from service import StudentService


def show_menu():
    """显示菜单"""
    print("\n" + "=" * 44)
    print("           🎓 学生管理系统 v1.0")
    print("=" * 44)
    print("  1. 添加学生           2. 查看所有学生")
    print("  3. 查询学生           4. 修改学生信息")
    print("  5. 删除学生           6. 成绩统计")
    print("  7. 保存数据           8. 从文件加载")
    print("  0. 退出系统")
    print("=" * 44)


def main():
    """主函数"""
    storage = Storage("students.json")
    service = StudentService(storage)

    menu_actions = {
        "1": service.add_student,
        "2": service.show_all,
        "3": service.search_student,
        "4": service.update_student,
        "5": service.delete_student,
        "6": service.show_statistics,
        "7": lambda: storage.save(service.students),
        "8": lambda: setattr(service, "students", storage.load()),
    }

    while True:
        show_menu()
        choice = input("请选择操作 (0-8): ").strip()

        if choice == "0":
            service.save_and_exit()
            break

        action = menu_actions.get(choice)
        if action:
            try:
                action()
            except Exception as e:
                print(f"❌ 操作出错: {e}")
        else:
            print("⚠️ 无效选项，请重新输入")


if __name__ == "__main__":
    main()
```

---

## 四、运行说明

```bash
# 1. 确保所有文件在同一目录下
#    main.py / models.py / service.py / storage.py / utils.py

# 2. 直接运行
python main.py

# 3. 首次运行会自动创建 students.json
#    之后每次启动自动加载
```

### 运行效果示例

```
============================================
           🎓 学生管理系统 v1.0
============================================
  1. 添加学生           2. 查看所有学生
  3. 查询学生           4. 修改学生信息
  5. 删除学生           6. 成绩统计
  7. 保存数据           8. 从文件加载
  0. 退出系统
============================================
请选择操作 (0-8): 1

--- 添加学生 ---
请输入学号: 2024001
请输入姓名: 张三
请输入年龄: 20
请输入成绩（直接回车结束）: 85
请输入成绩（直接回车结束）: 92
请输入成绩（直接回车结束）: 78
请输入成绩（直接回车结束）:
✅ 学生 张三 添加成功！
```

---

## 五、今日练习（3选1必做）

### 练习 1：扩展功能（推荐 ⭐）

给系统添加以下任一功能：
- **按均分排名输出 Top 3**
- **支持按姓名模糊搜索**
- **导出成绩报告为 CSV 文件**

### 练习 2：代码自查清单

对照检查你的代码是否用到以下知识点（每项打钩）：

- [ ] 列表的增删改查
- [ ] 字典（student.to_dict / from_dict）
- [ ] 集合/列表推导式
- [ ] 函数 def + 参数 + 返回值
- [ ] lambda 匿名函数
- [ ] sorted + key
- [ ] filter/map
- [ ] 类（class）的基本用法
- [ ] 文件读写（with open）
- [ ] JSON 序列化/反序列化
- [ ] try/except 异常处理
- [ ] if/else 条件判断
- [ ] for/while 循环
- [ ] f-string 格式化输出
- [ ] if __name__ == "__main__"
- [ ] 多模块 import

### 练习 3：我的代码有什么 bug？

下面这段代码有 3 个问题，找出并修正：

```python
# 找出问题
students = [
    {"id": "001", "name": "张三", "scores": [85, 92]},
    {"id": "002", "name": "李四", "scores": [78, 88]},
]

# 问题1：想打印每个学生的平均分，但出错了
for s in students:
    print(f"{s['name']}: {sum(s['avg']) / len(s['avg'])}")  # bug 在这行

# 问题2：想按均分排序，但没效果
students.sort(key=lambda s: sum(s["scores"]) / len(s["scores"]))

# 问题3：想安全读取用户输入的数字
age = input("请输入年龄: ")
if age >= 18:                    # bug 在这里
    print("成年")
```

<details>
<summary>点击查看答案</summary>

```python
# 问题1：字段名错误，应该是 'scores' 不是 'avg'
for s in students:
    print(f"{s['name']}: {sum(s['scores']) / len(s['scores'])}")

# 问题2：students 是列表（字典元素），sort 方法修改的是原列表本身。
# 排序确实执行了，但字典之间无法直接比较 —— 实际上这里代码是对的，
# key 返回的是数字，可以比较。真正的问题是：你期望 students 排序后
# 是 sorted 的新列表？那需要用 sorted() 而非 list.sort()
# （实际上 list.sort() 是就地排序，也生效了）

# 问题3：input() 返回的是字符串，不能直接和数字比较
age_str = input("请输入年龄: ")
try:
    age = int(age_str)
    if age >= 18:
        print("成年")
except ValueError:
    print("请输入数字！")
```
</details>

---

## 六、第一阶段学习自查表

| 检查项 | 会了吗？ |
|--------|---------|
| 能独立写一个带菜单的交互程序 | □ |
| 能用列表/字典组织数据 | □ |
| 能把代码拆成多个文件互相 import | □ |
| 能读写 JSON 文件持久化数据 | □ |
| 能用 try/except 处理用户输入错误 | □ |
| 能用 lambda + sorted 做排序 | □ |
| 能理解错误信息并自己改 bug | □ |
| 知道什么时候用 list / dict / tuple / set | □ |

> **如果 6 项以上打钩，恭喜你第一阶段通关！** 🎉

---

## 七、第二阶段预告（Day 15-28）

第一阶段结束，明天开始进入第二阶段 —— **Python 进阶与工程化**：

| 周次 | 主题 | 核心内容 |
|------|------|---------|
| 第3周 | 面向对象编程 | 类与对象、继承、多态、魔法方法、属性装饰器 |
| 第4周 | 进阶特性 | 迭代器/生成器、装饰器、上下文管理器、正则表达式 |

二阶段的实战项目：**个人博客数据爬虫 + 分析器**

---

## 八、今日避坑指南

1. **不要在循环中修改正在遍历的列表** — 用列表推导式或新建列表
2. **文件路径用 os.path.join() 拼接** — 不要手动拼 `/` 或 `\`，跨平台会炸
3. **JSON 保存中文记得 `ensure_ascii=False`** — 否则存的是 `\uXXXX`
4. **类的 `__str__` 只返回字符串** — 不要在里面 `print()`
5. **模块间的循环导入** — A import B，B import A 会死循环，需要重构

---

## 九、推荐资源（第一阶段配套）

| 资源 | 说明 |
|------|------|
| [Python官方教程](https://docs.python.org/zh-cn/3/tutorial/) | 最权威的中文教程 |
| [菜鸟教程 Python](https://www.runoob.com/python3/) | 快速查阅语法 |
| [廖雪峰 Python教程](https://www.liaoxuefeng.com/wiki/1016959663602400) | 经典中文教程 |
| [Python 100题](https://github.com/Runsen/100-Days-Of-Python) | 100天练习题库 |

---

> **今日金句**：代码是写给人看的，顺带能在机器上运行。
> 完成学生管理系统后，你就有了第一个完整的 Python 项目作品！
