# Python Day 24：os 与 pathlib 文件路径操作

> **学习目标**：掌握 Python 中 `os` 模块和 `pathlib` 模块的核心用法，能够进行文件/目录的创建、删除、遍历、路径拼接等操作，理解传统 `os.path` 方式与现代 `pathlib` 面向对象方式的区别与联系。

---

## 一、为什么需要文件路径操作？

在日常编程中，文件操作几乎无处不在：

| 场景 | 说明 |
|------|------|
| 批量处理文件 | 遍历一个文件夹下所有 `.csv` 文件并逐个读取 |
| 自动创建目录 | 程序运行时自动创建 `output/` 文件夹存放结果 |
| 配置文件路径 | 根据操作系统不同，找到配置文件所在位置 |
| 日志管理 | 自动清理 7 天前的旧日志文件 |
| 项目组织 | 按照日期/类别自动生成目录结构 |

Python 提供了两种操作文件路径的方式：

- **`os` 模块**：传统方式，Python 2 时代就存在，函数式调用
- **`pathlib` 模块**：Python 3.4+ 引入，面向对象方式，代码更优雅

> **建议**：新项目优先使用 `pathlib`，旧项目或需要兼容老代码时使用 `os`。实际工作中两者都要会。

---

## 二、os 模块核心功能

### 2.1 获取当前工作目录

```python
import os

# 获取当前工作目录（你运行 Python 的位置）
print(os.getcwd())
# 输出示例：'C:\\Users\\admin\\Desktop\\project'

# 切换工作目录
os.chdir('C:\\Users\\admin\\Documents')
print(os.getcwd())
# 输出示例：'C:\\Users\\admin\\Documents'
```

### 2.2 获取文件和目录信息

```python
import os

# 列出当前目录下的所有内容（文件和文件夹）
print(os.listdir('.'))           # 当前目录
print(os.listdir('C:\\Users'))    # 指定目录

# 判断路径是否存在
print(os.path.exists('test.txt'))           # 文件是否存在 → True/False
print(os.path.exists('some_folder'))         # 文件夹是否存在 → True/False

# 判断是文件还是目录
print(os.path.isfile('test.txt'))            # 是否是文件 → True/False
print(os.path.isdir('some_folder'))          # 是否是目录 → True/False

# 获取文件大小（字节）
print(os.path.getsize('test.txt'))
# 输出示例：1024

# 获取文件/目录的修改时间（时间戳）
import time
mtime = os.path.getmtime('test.txt')
print(time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(mtime)))
# 输出示例：'2026-06-23 09:30:00'
```

### 2.3 路径拼接与拆分

```python
import os

# 路径拼接（推荐用 os.path.join，自动处理斜杠问题）
path1 = os.path.join('folder', 'subfolder', 'file.txt')
print(path1)
# Windows 输出：'folder\\subfolder\\file.txt'
# Linux 输出：'folder/subfolder/file.txt'

# 获取路径的各部分
filepath = 'C:\\Users\\admin\\Desktop\\test.txt'

print(os.path.basename(filepath))   # 文件名 → 'test.txt'
print(os.path.dirname(filepath))    # 目录部分 → 'C:\\Users\\admin\\Desktop'
print(os.path.splitext(filepath))   # 分离扩展名 → ('C:\\Users\\admin\\Desktop\\test', '.txt')
print(os.path.splitext('report.tar.gz'))  # 注意：只分离最后一个点 → ('report.tar', '.gz')

# 获取绝对路径
print(os.path.abspath('test.txt'))
# 输出示例：'C:\\Users\\admin\\Desktop\\project\\test.txt'

# 获取路径的规范形式（消除 . 和 ..）
messy_path = 'C:\\Users\\admin\\Desktop\\..\\Documents\\.\\file.txt'
print(os.path.normpath(messy_path))
# 输出：'C:\\Users\\admin\\Documents\\file.txt'
```

### 2.4 创建和删除目录

```python
import os

# 创建单层目录
os.mkdir('new_folder')
# 如果目录已存在会报错：FileExistsError

# 创建多层目录（自动创建父目录）
os.makedirs('parent/child/grandchild', exist_ok=True)
# exist_ok=True 表示如果目录已存在不报错

# 删除空目录
os.rmdir('new_folder')   # 只能删除空目录
os.removedirs('parent/child/grandchild')  # 递归删除空的父目录

# 删除文件
os.remove('unwanted.txt')     # 删除指定文件
os.unlink('unwanted.txt')     # 和 remove 功能一样
```

### 2.5 重命名与移动

```python
import os

# 重命名文件或目录
os.rename('old_name.txt', 'new_name.txt')

# 移动文件（本质是 rename）
os.rename('file.txt', 'other_folder/file.txt')
```

### 2.6 遍历目录树

```python
import os

# 方式一：os.walk —— 递归遍历所有子目录
for dirpath, dirnames, filenames in os.walk('.'):
    print(f'当前目录: {dirpath}')
    print(f'  子目录: {dirnames}')
    print(f'  文件:   {filenames}')
    print('-' * 40)

# 方式二：os.scandir —— 非递归，但比 listdir 更高效
with os.scandir('.') as entries:
    for entry in entries:
        if entry.is_file():
            print(f'文件: {entry.name}，大小: {entry.stat().st_size} 字节')
        elif entry.is_dir():
            print(f'目录: {entry.name}')
```

> **`os.walk` vs `os.scandir`**：`os.walk` 会递归遍历所有子目录，适合"搜索全部文件"的场景。`os.scandir` 只遍历当前层，但性能更好（它返回的 `DirEntry` 对象自带 `stat` 信息缓存）。

### 2.7 环境变量操作

```python
import os

# 获取环境变量
print(os.environ.get('HOME'))          # Linux/Mac 的家目录
print(os.environ.get('USERPROFILE'))    # Windows 的家目录

# 安全获取（如果不存在返回默认值）
home = os.environ.get('MY_VAR', 'default_value')

# 设置环境变量（只在当前进程有效，不影响系统）
os.environ['MY_VAR'] = 'hello'

# 获取用户家目录的跨平台写法
print(os.path.expanduser('~'))
# Windows: 'C:\\Users\\admin'
# Linux:   '/home/admin'
```

---

## 三、pathlib 模块 —— 现代替代方案

`pathlib` 是 Python 3.4+ 引入的模块，用面向对象的方式处理路径。路径是一个 **对象**，而不是字符串。

### 3.1 创建路径对象

```python
from pathlib import Path

# 纯路径（不访问文件系统）
p = Path('folder/subfolder/file.txt')
print(p)
# 输出：folder\subfolder\file.txt（Windows 下用反斜杠）

# 获取当前工作目录
print(Path.cwd())
# 输出：WindowsPath('C:/Users/admin/Desktop/project')

# 获取用户家目录
print(Path.home())
# 输出：WindowsPath('C:/Users/admin')
```

### 3.2 路径拼接 —— `/` 运算符

`pathlib` 最优雅的特性：用 `/` 符号拼接路径（重载了除法运算符）。

```python
from pathlib import Path

# 用 / 拼接路径（自动处理分隔符）
base = Path('project')
data_dir = base / 'data'
csv_file = data_dir / '2026' / 'report.csv'

print(csv_file)
# 输出：project\data\2026\report.csv（Windows）

# 也可以用 joinpath 方法
csv_file = base.joinpath('data', '2026', 'report.csv')
```

### 3.3 路径的属性和方法

```python
from pathlib import Path

p = Path('C:/Users/admin/Desktop/project/data/report.csv')

# 路径各部分
print(p.name)           # 文件名（含扩展名）→ 'report.csv'
print(p.stem)           # 文件名（不含扩展名）→ 'report'
print(p.suffix)         # 扩展名 → '.csv'
print(p.suffixes)       # 所有扩展名（列表）→ ['.csv']
print(p.parent)         # 父目录 → WindowsPath('C:/Users/admin/Desktop/project/data')
print(p.parents[0])     # 上一级 → 同 parent
print(p.parents[1])     # 上上级
print(p.parts)          # 所有组成部分的元组

# 判断与检查
print(p.exists())       # 路径是否存在 → True/False
print(p.is_file())      # 是否是文件
print(p.is_dir())       # 是否是目录
print(p.is_absolute())  # 是否是绝对路径
```

### 3.4 创建、删除文件和目录

```python
from pathlib import Path

# 创建目录
Path('new_folder').mkdir()                         # 单层
Path('parent/child').mkdir(parents=True)           # 多层（自动创建父目录）
Path('logs').mkdir(parents=True, exist_ok=True)    # 存在也不报错

# 删除目录
Path('new_folder').rmdir()                         # 只能删空目录

# 创建空文件（如果文件已存在不报错）
Path('notes.txt').touch()

# 删除文件
Path('unwanted.txt').unlink(missing_ok=True)        # missing_ok=True 文件不存在也不报错
```

### 3.5 读写文件（超简洁）

```python
from pathlib import Path

p = Path('hello.txt')

# 写入文件
p.write_text('你好，Python！\n欢迎学习 pathlib。', encoding='utf-8')

# 读取文件
content = p.read_text(encoding='utf-8')
print(content)

# 写入/读取二进制文件
p.write_bytes(b'binary data here')
data = p.read_bytes()
```

> **注意**：`read_text()` 和 `write_text()` 适合小文件。大文件还是用 `open()` + 逐行读取更安全，避免内存溢出。

### 3.6 遍历目录

```python
from pathlib import Path

# 遍历当前目录（非递归）
for item in Path('.').iterdir():
    print(f'{"📁 " if item.is_dir() else "📄 "}{item.name}')

# 递归遍历所有文件 —— rglob 通配符
for py_file in Path('.').rglob('*.py'):
    print(py_file)

# 只查找当前层 —— glob 通配符
for md_file in Path('.').glob('*.md'):
    print(md_file)

# 常用通配符
# *.py        所有 .py 文件
# data/*.csv  data 目录下所有 .csv 文件
# **/*.log    所有子目录中的 .log 文件（等价于 rglob）
```

### 3.7 路径转换

```python
from pathlib import Path

p = Path('folder/file.txt')

# 转为绝对路径
print(p.resolve())

# 转为字符串（需要传给不支持 pathlib 的函数时）
print(str(p))
# 'folder\\file.txt'

# 转为 POSIX 格式（正斜杠，用于 Web URL 等）
print(p.as_posix())
# 'folder/file.txt'

# 打开文件（pathlib 对象可以直接传给 open）
with open(p, 'r') as f:
    content = f.read()
```

---

## 四、os vs pathlib 对照表

| 功能 | os 方式 | pathlib 方式 |
|------|---------|-------------|
| 当前目录 | `os.getcwd()` | `Path.cwd()` |
| 家目录 | `os.path.expanduser('~')` | `Path.home()` |
| 路径拼接 | `os.path.join('a', 'b')` | `Path('a') / 'b'` |
| 文件名 | `os.path.basename(p)` | `Path(p).name` |
| 目录部分 | `os.path.dirname(p)` | `Path(p).parent` |
| 扩展名 | `os.path.splitext(p)[1]` | `Path(p).suffix` |
| 是否存在 | `os.path.exists(p)` | `Path(p).exists()` |
| 是否是文件 | `os.path.isfile(p)` | `Path(p).is_file()` |
| 是否是目录 | `os.path.isdir(p)` | `Path(p).is_dir()` |
| 创建目录 | `os.makedirs('a/b')` | `Path('a/b').mkdir(parents=True)` |
| 列出内容 | `os.listdir('.')` | `list(Path('.').iterdir())` |
| 递归查找 | `os.walk('.')` | `Path('.').rglob('*')` |
| 读文件 | `open(p).read()` | `Path(p).read_text()` |
| 写文件 | `open(p, 'w').write(s)` | `Path(p).write_text(s)` |

---

## 五、实战：批量文件整理工具

将散落在一个文件夹中的文件按扩展名分类到不同子文件夹：

```python
"""
文件整理工具 —— 将当前目录下的文件按扩展名分类到子文件夹
"""
from pathlib import Path
import shutil

def organize_files(source_dir='.', target_dir='organized'):
    """按扩展名整理文件"""
    source = Path(source_dir)
    target = Path(target_dir)

    # 定义分类规则
    categories = {
        '图片': ['.jpg', '.jpeg', '.png', '.gif', '.bmp', '.svg'],
        '文档': ['.txt', '.md', '.pdf', '.doc', '.docx', '.xlsx'],
        '代码': ['.py', '.js', '.html', '.css', '.java', '.cpp'],
        '数据': ['.csv', '.json', '.xml', '.sql'],
        '压缩包': ['.zip', '.rar', '.7z', '.tar', '.gz'],
    }

    # 其他文件归入 "其他" 分类
    all_extensions = set()
    for exts in categories.values():
        all_extensions.update(exts)

    # 遍历源目录中的所有文件
    file_count = 0
    for filepath in source.iterdir():
        if not filepath.is_file():
            continue
        if filepath.name.startswith('.'):  # 跳过隐藏文件
            continue

        suffix = filepath.suffix.lower()

        # 找到文件所属分类
        target_category = '其他'
        for category, exts in categories.items():
            if suffix in exts:
                target_category = category
                break

        # 创建目标子目录并移动文件
        dest_dir = target / target_category
        dest_dir.mkdir(parents=True, exist_ok=True)

        dest_file = dest_dir / filepath.name
        shutil.move(str(filepath), str(dest_file))
        file_count += 1
        print(f'  [{target_category}] {filepath.name}')

    print(f'\n整理完成！共移动 {file_count} 个文件')

if __name__ == '__main__':
    organize_files()
```

---

## 六、实战：日志自动清理

删除 7 天前的旧日志文件：

```python
"""
日志清理工具 —— 自动删除超过指定天数的日志文件
"""
from pathlib import Path
import time

def clean_old_logs(log_dir='logs', max_age_days=7):
    """删除超过 max_age_days 天的日志文件"""
    log_path = Path(log_dir)

    if not log_path.exists():
        print(f'日志目录 {log_dir} 不存在，无需清理')
        return

    # 计算截止时间（秒）
    cutoff = time.time() - (max_age_days * 24 * 60 * 60)

    deleted_count = 0
    deleted_size = 0

    for log_file in log_path.rglob('*.log'):
        # 获取文件最后修改时间
        file_mtime = log_file.stat().st_mtime

        if file_mtime < cutoff:
            file_size = log_file.stat().st_size
            log_file.unlink()
            deleted_count += 1
            deleted_size += file_size

            # 格式化删除时间
            del_time = time.strftime('%Y-%m-%d', time.localtime(file_mtime))
            print(f'  已删除: {log_file.name}（修改于 {del_time}）')

    size_mb = deleted_size / (1024 * 1024)
    print(f'\n清理完成！删除 {deleted_count} 个文件，释放 {size_mb:.2f} MB 空间')

if __name__ == '__main__':
    clean_old_logs('logs', max_age_days=7)
```

---

## 七、常见问题与排错

### Q1：Windows 和 Linux 的路径分隔符不一样怎么办？

```python
# 错误写法 —— 硬编码分隔符（跨平台会出问题）
path = 'folder\\file.txt'      # 只能在 Windows 用
path = 'folder/file.txt'       # 只能在 Linux 用

# 正确写法 —— 用 os.path.join 或 pathlib
import os
path = os.path.join('folder', 'file.txt')    # 自动适配

from pathlib import Path
path = Path('folder') / 'file.txt'            # 自动适配
```

### Q2：`os.path.exists()` 返回 False，但文件确实存在？

```python
# 原因 1：路径写错了（注意大小写、空格、中文）
path = 'data/repor.csv'  # 少了个 t → 找不到

# 原因 2：工作目录不是你以为的目录
import os
print(os.getcwd())  # 先确认当前目录

# 原因 3：相对路径 vs 绝对路径
# 解决：用绝对路径
path = Path('report.csv').resolve()
print(path)  # 查看完整的绝对路径
```

### Q3：`mkdir` 报错 `FileExistsError`？

```python
# 原因：目录已存在
from pathlib import Path

# 解决：加 exist_ok=True
Path('output').mkdir(parents=True, exist_ok=True)
```

### Q4：处理中文路径时出错？

```python
# Windows 下中文路径可能出现编码问题
# 解决方案 1：使用 pathlib（自动处理编码）
from pathlib import Path
p = Path('数据/报告/第一季度.csv')

# 解决方案 2：open 时指定编码
with open(str(p), 'r', encoding='utf-8') as f:
    content = f.read()

# 解决方案 3：路径中避免特殊字符
# 如果文件名中有特殊符号，用原始字符串
p = Path(r'C:\Users\admin\Desktop\2026年数据\报告.pdf')
```

### Q5：pathlib 和 open() 函数不兼容？

```python
from pathlib import Path

p = Path('test.txt')

# Python 3.6+ 的 open() 直接支持 Path 对象
with open(p, 'r') as f:
    content = f.read()

# 如果遇到不支持 Path 的老函数，用 str() 转换
content = some_old_function(str(p))
```

---

## 八、练习题

### 练习 1：目录结构生成器（基础）

编写一个函数，自动创建如下目录结构：

```
project/
├── src/
│   ├── utils/
│   └── models/
├── tests/
├── docs/
└── data/
    ├── raw/
    └── processed/
```

<details>
<summary>参考答案</summary>

```python
from pathlib import Path

def create_project_structure(base='project'):
    """创建项目目录结构"""
    folders = [
        'src/utils',
        'src/models',
        'tests',
        'docs',
        'data/raw',
        'data/processed',
    ]

    for folder in folders:
        path = Path(base) / folder
        path.mkdir(parents=True, exist_ok=True)
        print(f'已创建: {path}')

create_project_structure()
```

</details>

### 练习 2：文件搜索器（中级）

编写一个函数，接收一个目录路径和文件扩展名，递归搜索该目录下所有匹配的文件，并按文件大小降序排列。

<details>
<summary>参考答案</summary>

```python
from pathlib import Path

def search_files(directory, extension, top_n=10):
    """搜索指定扩展名的文件，按大小降序排列"""
    dir_path = Path(directory)
    if not dir_path.exists():
        print(f'目录不存在: {directory}')
        return []

    # 递归搜索
    pattern = f'*.{extension.lstrip(".")}'
    files = []
    for filepath in dir_path.rglob(pattern):
        size = filepath.stat().st_size
        files.append((filepath, size))

    # 按大小降序排列
    files.sort(key=lambda x: x[1], reverse=True)

    # 输出结果
    print(f'找到 {len(files)} 个 .{extension} 文件：')
    print(f'{"排名":<4} {"文件路径":<50} {"大小":>10}')
    print('-' * 70)

    for i, (fp, size) in enumerate(files[:top_n], 1):
        size_kb = size / 1024
        print(f'{i:<4} {str(fp):<50} {size_kb:>9.1f} KB')

    return files

# 使用示例
search_files('.', 'py', top_n=5)
```

</details>

### 练习 3：磁盘空间分析器（进阶）

编写一个函数，分析指定目录下每个子目录占用的大小，并生成报告。

<details>
<summary>参考答案</summary>

```python
from pathlib import Path

def analyze_disk_usage(directory='.'):
    """分析目录下每个子目录的大小"""
    dir_path = Path(directory)

    results = []
    for subdir in dir_path.iterdir():
        if subdir.is_dir():
            total_size = sum(
                f.stat().st_size for f in subdir.rglob('*') if f.is_file()
            )
            results.append((subdir.name, total_size))

    # 按大小降序排列
    results.sort(key=lambda x: x[1], reverse=True)

    # 输出报告
    print(f'目录分析: {dir_path.resolve()}')
    print(f'{"子目录":<30} {"大小":>12} {"占比":>8}')
    print('-' * 55)

    total = sum(size for _, size in results)
    for name, size in results:
        size_mb = size / (1024 * 1024)
        percent = (size / total * 100) if total > 0 else 0
        bar = '█' * int(percent / 2)
        print(f'{name:<30} {size_mb:>9.2f} MB {percent:>5.1f}% {bar}')

    print(f'\n总计: {total / (1024*1024):.2f} MB')

analyze_disk_usage('.')
```

</details>

---

## 九、学习资源推荐

1. **pathlib 官方文档**：https://docs.python.org/zh-cn/3/library/pathlib.html
2. **os 模块官方文档**：https://docs.python.org/zh-cn/3/library/os.html
3. **廖雪峰 Python 教程 —— 文件 IO**：https://www.liaoxuefeng.com/wiki/1016959663602400/1017608811286784
4. **菜鸟教程 —— Python OS 文件/目录方法**：https://www.runoob.com/python/os-file-methods.html
5. **Real Python —— pathlib 教程**：https://realpython.com/python-pathlib/

---

## 十、本节要点总结

```
os 模块（传统函数式）
├── os.getcwd() / os.chdir()          → 工作目录
├── os.listdir() / os.walk()          → 遍历目录
├── os.path.join()                    → 路径拼接
├── os.path.exists() / isfile() / isdir()  → 路径检查
├── os.mkdir() / os.makedirs()        → 创建目录
├── os.remove() / os.rmdir()          → 删除文件/目录
└── os.environ / os.path.expanduser() → 环境变量

pathlib 模块（现代面向对象）
├── Path.cwd() / Path.home()          → 工作目录/家目录
├── Path('a') / 'b' / 'c'            → 路径拼接（/运算符）
├── p.name / p.stem / p.suffix       → 路径属性
├── p.exists() / p.is_file()         → 路径检查
├── p.mkdir() / p.touch() / p.unlink() → 创建/删除
├── p.read_text() / p.write_text()   → 读写文件
└── p.glob() / p.rglob()             → 通配符搜索
```

> **下节预告**：Day 25 将学习 **多线程基础**，了解线程的概念、`threading` 模块的使用、线程同步与锁，以及多线程编程的常见问题。
