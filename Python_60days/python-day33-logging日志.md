# Python Day 33：logging 日志模块

> 学会使用 logging，让你的程序学会"说话"——记录运行状态、排查问题、监控性能。

---

## 一、为什么需要日志？

你已经学过 `print()` 来调试程序，但它有几个致命缺点：

| 对比项 | `print()` | `logging` 模块 |
|--------|-----------|----------------|
| 输出控制 | 无法关闭，全删或全留 | 按级别过滤，灵活控制 |
| 输出目标 | 只能打印到控制台 | 控制台、文件、网络、邮件等 |
| 时间戳 | 需要手动添加 | 自动附带时间、行号、模块名 |
| 线程安全 | 不安全 | 安全（多线程环境下可靠） |
| 格式化 | 手动拼接 | 丰富的格式模板 |

**一句话总结**：`print()` 适合临时调试，`logging` 适合正式项目的长期记录。

---

## 二、日志级别（LogLevel）

logging 定义了 5 个标准级别，从低到高：

```python
import logging

# 级别从低到高：DEBUG < INFO < WARNING < ERROR < CRITICAL
logging.debug("调试信息 - 变量值、函数入口等细节")   # 10
logging.info("一般信息 - 程序正常运行的状态")        # 20
logging.warning("警告信息 - 不影响运行但需注意")    # 30
logging.error("错误信息 - 功能无法正常执行")         # 40
logging.critical("严重错误 - 程序可能无法继续")      # 50
```

**默认行为**：logging 的默认级别是 `WARNING`，所以 `debug` 和 `info` 消息不会显示。

```python
import logging

logging.warning("这条会显示")   # ✅ 级别 >= WARNING
logging.info("这条不会显示")    # ❌ 级别 < WARNING
```

**各级别的使用场景**：

- **DEBUG**：开发阶段排查问题，记录变量值、分支走向
- **INFO**：确认程序按预期运行，如"启动成功""处理了100条数据"
- **WARNING**：程序仍可运行，但出现了意外情况，如"配置文件缺失，使用默认值"
- **ERROR**：某个功能失败了，如"数据库连接失败"
- **CRITICAL**：严重到程序可能要崩溃，如"磁盘空间不足，无法写入"

---

## 三、基本配置 — basicConfig

`logging.basicConfig()` 是最简单的配置方式，适合小脚本和快速调试。

### 3.1 常用参数

```python
import logging

# 配置日志的基本参数
logging.basicConfig(
    level=logging.DEBUG,           # 设置最低输出级别
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',  # 日志格式
    datefmt='%Y-%m-%d %H:%M:%S',  # 时间格式
    filename='app.log',            # 输出到文件（不指定则输出到控制台）
    filemode='a',                  # 'a'追加 / 'w'覆盖
    encoding='utf-8',             # 文件编码
)
```

> **注意**：`basicConfig` 只能调用一次。多次调用时，只有第一次会生效。

### 3.2 format 格式字段大全

```python
format_string = (
    '%(asctime)s'    ' - '    # 日志时间：2026-07-03 09:30:00,123
    '%(levelname)s'  ' - '    # 级别名称：INFO
    '%(name)s'       ' - '    # 日志器名称：root
    '%(filename)s'   ':'      # 文件名：app.py
    '%(lineno)d'     ' - '    # 行号：42
    '%(funcName)s'   ' - '    # 函数名：main
    '%(message)s'              # 日志消息本身
)
```

常用字段速查表：

| 字段 | 含义 | 示例 |
|------|------|------|
| `%(asctime)s` | 时间 | `2026-07-03 09:30:00,123` |
| `%(levelname)s` | 级别名 | `INFO` |
| `%(levelno)d` | 级别数字 | `20` |
| `%(name)s` | logger名称 | `root` |
| `%(message)s` | 日志消息 | `用户登录成功` |
| `%(filename)s` | 文件名 | `app.py` |
| `%(lineno)d` | 行号 | `42` |
| `%(funcName)s` | 函数名 | `main` |
| `%(pathname)s` | 完整路径 | `/home/user/app.py` |
| `%(thread)d` | 线程ID | `1402345` |
| `%(process)d` | 进程ID | `12345` |

### 3.3 同时输出到控制台和文件

`basicConfig` 只能选择一个输出目标。要同时输出到控制台和文件，需要用 Handler：

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# 格式
fmt = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')

# 控制台处理器
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)         # 控制台只显示 INFO 及以上
console_handler.setFormatter(fmt)

# 文件处理器
file_handler = logging.FileHandler('app.log', encoding='utf-8')
file_handler.setLevel(logging.DEBUG)            # 文件记录所有 DEBUG 及以上
file_handler.setFormatter(fmt)

# 添加处理器
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 测试
logger.debug("这条只在文件中")      # 仅文件
logger.info("这条在控制台和文件中")   # 两者都有
logger.error("出错了！")             # 两者都有
```

---

## 四、Logger、Handler、Formatter、Filter

这是 logging 的四大核心组件，理解它们的关系是掌握 logging 的关键：

```
Logger（日志记录器）
  ├── Handler（处理器）— 决定日志发到哪里
  │     ├── Formatter（格式化器）— 决定日志长什么样
  │     └── Filter（过滤器）— 额外的过滤条件
  │
  ├── Handler（可以有多个处理器）
  │     ├── Formatter
  │     └── Filter
  │
  └── ...
```

### 4.1 Logger — 日志记录器

```python
import logging

# 获取 logger（推荐用模块名作为 logger 名）
logger = logging.getLogger(__name__)   # __name__ 就是当前模块名
logger.setLevel(logging.DEBUG)

# 不要用 logging.getLogger() 不带参数（会获取 root logger）
# 最好每个模块用自己的 logger
```

**为什么用 `__name__`？**
因为 `__name__` 在不同模块中会自动变成模块的完整路径（如 `package.module`），方便你追踪日志来自哪个文件。

### 4.2 Handler — 处理器

常用 Handler 类型：

```python
import logging
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

# 1. StreamHandler — 输出到控制台
console = logging.StreamHandler()

# 2. FileHandler — 输出到文件
file = logging.FileHandler('app.log', encoding='utf-8')

# 3. RotatingFileHandler — 按大小轮转（推荐！）
#    当日志文件达到 maxBytes 时自动备份，最多保留 backupCount 个备份
rotating = RotatingFileHandler(
    'app.log',
    maxBytes=10 * 1024 * 1024,   # 10MB
    backupCount=5,                # 最多5个备份文件
    encoding='utf-8',
)

# 4. TimedRotatingFileHandler — 按时间轮转
#    每天午夜自动创建新文件，保留7天的日志
timed = TimedRotatingFileHandler(
    'app.log',
    when='midnight',             # 每天轮转
    interval=1,                   # 每隔1天
    backupCount=7,               # 保留7天
    encoding='utf-8',
)
```

**when 参数选项**：
- `'S'` — 秒
- `'M'` — 分钟
- `'H'` — 小时
- `'D'` — 天
- `'midnight'` — 每天午夜
- `'W0'`~`'W6'` — 每周几（W0=周一）

### 4.3 Filter — 过滤器

Filter 可以按自定义条件过滤日志：

```python
import logging

# 只允许 ERROR 级别的日志通过
error_filter = logging.Filter()
error_filter.filter = lambda record: record.levelno >= logging.ERROR

handler = logging.StreamHandler()
handler.addFilter(error_filter)

logger = logging.getLogger('filtered')
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("被过滤掉了")     # 不会显示
logger.error("这条通过了")    # 正常显示
```

### 4.4 完整配置示例

```python
import logging
from logging.handlers import RotatingFileHandler

def setup_logger(name='app', log_file='app.log', level=logging.DEBUG):
    """创建并配置一个 logger"""
    logger = logging.getLogger(name)
    logger.setLevel(level)

    # 防止重复添加 handler
    if logger.handlers:
        return logger

    formatter = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(name)s | %(filename)s:%(lineno)d | %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )

    # 控制台
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(formatter)
    logger.addHandler(console)

    # 文件（带轮转）
    file = RotatingFileHandler(
        log_file, maxBytes=5*1024*1024, backupCount=3, encoding='utf-8'
    )
    file.setLevel(logging.DEBUG)
    file.setFormatter(formatter)
    logger.addHandler(file)

    return logger


# 使用
logger = setup_logger('myapp')

logger.debug("开始处理数据...")
logger.info("共读取 1024 条记录")
logger.warning("发现 3 条空记录，已跳过")
logger.error("第 42 条记录格式异常")
```

---

## 五、日志分层与传播（Logger层级）

logging 使用层级结构，类似 Python 的包结构：

```
root
 └── myapp
      ├── myapp.web
      └── myapp.db
```

**传播规则**：子 logger 的日志默认会向上传播给父 logger（直到 root）。

```python
import logging

# 设置 root logger
logging.basicConfig(level=logging.WARNING, format='[%(name)s] %(message)s')

app_logger = logging.getLogger('myapp')
db_logger = logging.getLogger('myapp.db')

# 子 logger 的日志会传播到 root，root 级别是 WARNING
app_logger.info("app info")    # ❌ 被 root 过滤
app_logger.warning("app warn") # ✅ 显示 [myapp] app warn

db_logger.warning("db warn")   # ✅ 显示 [myapp.db] db warn（传播到 root）

# 关闭传播（子 logger 独立处理）
db_logger.propagate = False
db_logger.addHandler(logging.StreamHandler())
db_logger.setLevel(logging.INFO)
```

> **最佳实践**：在主程序入口配置 root logger，各子模块只管用 `logging.getLogger(__name__)` 记录日志。

---

## 六、异常日志记录

logging 提供了专门的异常记录方法，会自动附带完整的堆栈信息：

```python
import logging

logging.basicConfig(level=logging.ERROR, format='%(asctime)s - %(levelname)s - %(message)s')

def divide(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        # exc_info=True 会记录完整的异常堆栈
        logging.error("除零错误！", exc_info=True)
        # 或者更简洁地：
        # logging.exception("除零错误！")  # 等价于 error + exc_info=True

def main():
    divide(10, 0)

main()
```

输出示例：
```
2026-07-03 09:30:00,123 - ERROR - 除零错误！
Traceback (most recent call last):
  File "app.py", line 7, in divide
    result = a / b
ZeroDivisionError: division by zero
```

---

## 七、使用 JSON 格式记录日志

在生产环境中，JSON 格式的日志方便 ELK（Elasticsearch + Logstash + Kibana）等工具分析：

```python
import logging
import json

class JsonFormatter(logging.Formatter):
    """自定义 JSON 格式化器"""
    def format(self, record):
        log_data = {
            'time': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'file': record.filename,
            'line': record.lineno,
            'message': record.getMessage(),
        }
        # 如果有异常信息，也加进去
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        return json.dumps(log_data, ensure_ascii=False)

# 使用
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())

logger = logging.getLogger('json_example')
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("用户登录成功", extra={'user_id': 1001})
```

---

## 八、综合实战：带完整日志系统的学生管理系统

```python
"""
学生成绩管理系统 - 完整日志配置
演示：日志分层、轮转文件、异常记录
"""
import logging
from logging.handlers import RotatingFileHandler
import os

# ========== 日志配置 ==========
def setup_logging():
    """配置整个项目的日志系统"""

    # 1. 创建格式
    detail_fmt = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(name)-15s | '
        '%(filename)s:%(lineno)-3d | %(funcName)s | %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    simple_fmt = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(message)s',
        datefmt='%H:%M:%S'
    )

    # 2. 根 logger 配置
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.DEBUG)

    # 控制台（简洁格式）
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(simple_fmt)
    root_logger.addHandler(console)

    # 全局日志文件（详细格式，按天轮转）
    log_dir = 'logs'
    os.makedirs(log_dir, exist_ok=True)

    file_handler = RotatingFileHandler(
        os.path.join(log_dir, 'app.log'),
        maxBytes=10 * 1024 * 1024,
        backupCount=10,
        encoding='utf-8',
    )
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(detail_fmt)
    root_logger.addHandler(file_handler)

    # 错误日志单独一个文件
    error_handler = RotatingFileHandler(
        os.path.join(log_dir, 'error.log'),
        maxBytes=10 * 1024 * 1024,
        backupCount=5,
        encoding='utf-8',
    )
    error_handler.setLevel(logging.ERROR)
    error_handler.setFormatter(detail_fmt)
    root_logger.addHandler(error_handler)


# ========== 学生管理模块 ==========
class StudentManager:
    """学生成绩管理系统"""

    def __init__(self):
        self.logger = logging.getLogger(f'{self.__class__.__module__}.{self.__class__.__name__}')
        self.students = {}
        self.logger.info("学生管理系统初始化完成")

    def add_student(self, student_id, name, scores=None):
        """添加学生"""
        if student_id in self.students:
            self.logger.warning(f"学号 {student_id} 已存在，跳过添加")
            return False

        if not name or not name.strip():
            self.logger.error(f"添加失败：学号 {student_id} 姓名为空")
            return False

        self.students[student_id] = {
            'name': name.strip(),
            'scores': scores or {},
        }
        self.logger.info(f"添加学生：{student_id} - {name}，成绩：{scores or '无'}")
        return True

    def add_score(self, student_id, subject, score):
        """为学生添加单科成绩"""
        try:
            student = self.students[student_id]
        except KeyError:
            self.logger.error(f"学号 {student_id} 不存在", exc_info=True)
            return False

        if not (0 <= score <= 100):
            self.logger.warning(
                f"学号 {student_id} 的 {subject} 成绩 {score} 不在 0-100 范围内"
            )
            return False

        student['scores'][subject] = score
        self.logger.debug(f"更新成绩：{student_id} {subject}={score}")
        return True

    def get_average(self, student_id):
        """获取学生平均分"""
        try:
            scores = self.students[student_id]['scores'].values()
        except KeyError:
            self.logger.error(f"计算平均分失败：学号 {student_id} 不存在")
            return None

        if not scores:
            self.logger.warning(f"学号 {student_id} 暂无成绩")
            return 0.0

        avg = sum(scores) / len(scores)
        self.logger.info(f"学号 {student_id} 平均分：{avg:.1f}")
        return avg

    def get_ranking(self):
        """获取成绩排行榜"""
        self.logger.info("生成成绩排行榜")

        rankings = []
        for sid, info in self.students.items():
            avg = self.get_average(sid)
            if avg is not None:
                rankings.append((sid, info['name'], avg))

        rankings.sort(key=lambda x: x[2], reverse=True)

        self.logger.info(f"排行榜生成完成，共 {len(rankings)} 名学生")
        return rankings


# ========== 主程序 ==========
def main():
    # 初始化日志
    setup_logging()
    logger = logging.getLogger('main')

    logger.info("===== 系统启动 =====")

    # 创建管理器
    manager = StudentManager()

    # 添加学生
    manager.add_student("001", "张三", {"数学": 90, "语文": 85, "英语": 88})
    manager.add_student("002", "李四", {"数学": 78, "语文": 92, "英语": 95})
    manager.add_student("003", "王五", {"数学": 65, "语文": 70, "英语": 72})
    manager.add_student("001", "赵六")  # 重复学号（会被警告）

    # 添加成绩
    manager.add_score("001", "物理", 93)
    manager.add_score("999", "数学", 80)  # 不存在的学号（会记录错误）
    manager.add_score("002", "数学", 150)  # 超范围成绩（会警告）

    # 排行榜
    logger.info("===== 成绩排行榜 =====")
    for rank, (sid, name, avg) in enumerate(manager.get_ranking(), 1):
        print(f"  第{rank}名：{name}（{sid}）— 平均分 {avg:.1f}")

    logger.info("===== 系统结束 =====")


if __name__ == '__main__':
    main()
```

---

## 九、常见问题与排错

### Q1：日志重复打印了两遍？

**原因**：同一个 handler 被添加了多次，或者子 logger 的日志传播到了 root logger，而 root logger 也有 handler。

**解决**：
```python
# 方法1：关闭传播
logger.propagate = False

# 方法2：添加 handler 前检查
if not logger.handlers:
    logger.addHandler(handler)
```

### Q2：basicConfig 没有生效？

**原因**：`basicConfig` 只在 root logger 没有任何 handler 的情况下才会生效。如果你之前已经手动添加了 handler，basicConfig 就会"静默失败"。

**解决**：在调用 `basicConfig` 之前确保没有添加过任何 handler。

### Q3：日志文件出现乱码？

**原因**：Windows 下默认编码可能不是 UTF-8。

**解决**：始终显式指定 `encoding='utf-8'`：
```python
handler = logging.FileHandler('app.log', encoding='utf-8')
```

### Q4：如何让不同模块的日志输出到不同文件？

**解决**：为不同模块的 logger 添加不同的 FileHandler：
```python
# web 模块日志输出到 web.log
web_logger = logging.getLogger('myapp.web')
web_logger.addHandler(logging.FileHandler('web.log', encoding='utf-8'))

# db 模块日志输出到 db.log
db_logger = logging.getLogger('myapp.db')
db_logger.addHandler(logging.FileHandler('db.log', encoding='utf-8'))
```

### Q5：logging 和 print 能混用吗？

**建议不要混用**。在正式项目中，统一使用 logging，把 `print()` 全部替换掉。这样你可以通过调整日志级别来控制输出的详细程度，而不需要逐个去删 `print()` 语句。

---

## 十、练习题

### 练习1：基础练习 — 创建带日志的计算器

编写一个计算器程序，要求：
- 支持加、减、乘、除四种运算
- 每次运算都用 DEBUG 级别记录参数
- 除法除零时用 ERROR 级别记录异常
- 除法结果为小数时用 INFO 级别记录结果

```python
# 提示框架
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')

def calculate(a, b, op):
    """实现四种运算，并记录日志"""
    pass  # 请补充
```

### 练习2：进阶练习 — 日志轮转管理器

编写一个日志管理模块，要求：
- 创建按天轮转的日志文件，保留最近 7 天
- ERROR 级别以上的日志额外写入 `error.log`
- 控制台只显示 INFO 及以上级别
- 格式包含时间、级别、文件名、行号、消息

### 练习3：实战练习 — 为之前的 StudentManager 添加日志

回顾 Day 29 的 dataclass 版本待办事项管理系统，为其添加完整的 logging 日志系统：
- 使用分层 logger（main、TodoManager、TodoItem 各一个）
- 所有关键操作（添加/完成/删除）都记录日志
- 异常操作用 `logging.exception()` 记录堆栈

---

## 十一、推荐学习资源

- **[菜鸟教程 - Python logging](https://www.runoob.com/python/python-logging.html)** — 快速上手参考
- **[廖雪峰 - Python logging](https://www.liaoxuefeng.com/wiki/1016959663602400/1017787694025344)** — 讲解清晰，配有实例
- **[Python 官方文档 - logging cookbook](https://docs.python.org/zh-cn/3/howto/logging-cookbook.html)** — 各种场景的最佳实践示例
- **[Python 官方文档 - logging HOWTO](https://docs.python.org/zh-cn/3/howto/logging.html)** — 最权威的教程
- **[Real Python - Logging in Python](https://realpython.com/python-logging/)** — 英文深度教程

---

## 十二、下节预告

**Day 34：配置管理** — 学习如何管理程序的各种配置：环境变量、配置文件（INI/JSON/YAML/TOML）、`python-dotenv`、`pydantic-settings`，让项目的配置不再硬编码。

> 💡 **小贴士**：从今天开始，把之前练习中的 `print()` 逐步替换成 `logging`，养成良好的日志记录习惯。当你以后开发真实项目时，这个习惯会帮你省下大量调试时间！
