# Python Day 31：pip 与虚拟环境

> **学习目标**：掌握 Python 包管理工具 pip 的使用，学会创建和管理虚拟环境，理解依赖隔离的重要性，为后续独立开发项目打下基础。

---

## 一、为什么要学 pip 和虚拟环境？

在前面 30 天的学习中，我们一直在使用 Python 的内置功能。但在真实开发中，你几乎一定会用到**第三方库**——别人写好的代码包，拿来就能用。比如：

- `requests`：发送 HTTP 请求
- `pandas`：数据分析
- `flask` / `fastapi`：Web 开发
- `pygame`：游戏开发

这些库怎么安装？答案就是 **pip**。

而**虚拟环境**解决的是另一个问题：不同项目可能需要同一个库的不同版本。比如项目 A 用 `requests 2.28`，项目 B 用 `requests 2.31`。如果没有隔离，两个项目就会打架。虚拟环境就是给每个项目建一个"独立小房间"，互不干扰。

**一句话总结**：
- **pip** = 包管理器（安装/卸载/更新第三方库）
- **虚拟环境** = 项目隔离机制（每个项目有自己的依赖副本）

---

## 二、pip 包管理器

### 2.1 pip 是什么

pip 是 Python 官方的包管理工具，全称 "Pip Installs Packages"。从 Python 3.4 开始，pip 已经默认随 Python 一起安装了。

### 2.2 检查 pip 是否可用

```bash
# 在命令行（终端/PowerShell）中运行，不是在 Python 交互模式中
# 查看 pip 版本
pip --version
# 输出示例：pip 24.0 from ... (python 3.13)
```

> **常见问题**：如果命令行提示 `pip 不是内部或外部命令`，说明 pip 没有加入系统环境变量。可以尝试用 `python -m pip` 代替 `pip`。

### 2.3 常用 pip 命令

```bash
# 1. 安装包
pip install requests          # 安装最新版
pip install requests==2.28.0  # 安装指定版本
pip install requests>=2.28.0 # 安装大于等于该版本的最新版

# 2. 查看已安装的包
pip list                     # 列出所有已安装的包
pip show requests            # 查看某个包的详细信息

# 3. 升级包
pip install --upgrade requests   # 升级到最新版

# 4. 卸载包
pip uninstall requests        # 卸载包

# 5. 导出依赖列表
pip freeze > requirements.txt  # 把当前环境所有包及版本写入文件

# 6. 从文件批量安装
pip install -r requirements.txt  # 根据 requirements.txt 批量安装

# 7. 搜索包（在线搜索）
pip search requests           # 在 PyPI 上搜索相关包（可能需要额外配置）
```

### 2.4 requirements.txt 依赖文件

这是 Python 项目中最重要的文件之一。它记录了项目依赖的所有第三方库及其版本号。

```text
# requirements.txt 示例
requests==2.31.0
pandas==2.1.4
numpy==1.26.2
flask==3.0.0
```

```bash
# 生成：在虚拟环境中运行
pip freeze > requirements.txt

# 使用：新同事拿到代码后运行
pip install -r requirements.txt
```

> **小贴士**：`pip freeze > requirements.txt` 会把环境中**所有**包都列出来（包括间接依赖）。如果只想记录直接依赖，可以手动编写，或者使用 `pipreqs` 等工具自动生成。

### 2.5 使用国内镜像源加速下载

在国内使用 pip 默认从 PyPI（python.org）下载，速度可能很慢甚至超时。使用镜像源可以大幅加速：

```bash
# 临时使用镜像（以清华镜像为例）
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple

# 永久配置（推荐）
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

常用国内镜像：

| 镜像 | 地址 |
|------|------|
| 清华 | https://pypi.tuna.tsinghua.edu.cn/simple |
| 阿里云 | https://mirrors.aliyun.com/pypi/simple |
| 豆瓣 | https://pypi.douban.com/simple |
| 中科大 | https://pypi.mirrors.ustc.edu.cn/simple |

### 2.6 pip 安装原理简图

```
pip install requests
        │
        ▼
┌─────────────────┐
│  PyPI 仓库       │  ← 全球的 Python 包托管平台（python.org）
│  (在线仓库)      │
└────────┬────────┘
         │ 下载包文件
         ▼
┌─────────────────┐
│  本地环境        │  ← 安装到当前 Python 环境中
│  site-packages/  │
└─────────────────┘
```

---

## 三、虚拟环境

### 3.1 为什么需要虚拟环境

假设你有两个项目：

```
项目A（数据分析）：需要 pandas 1.5
项目B（Web开发）：需要 pandas 2.0
```

如果只用一个全局 Python 环境，你无法同时满足两个项目。虚拟环境让每个项目拥有独立的包安装目录，互不影响。

### 3.2 使用 venv 创建虚拟环境（推荐）

`venv` 是 Python 3.3+ 内置的虚拟环境模块，不需要额外安装。

```bash
# 步骤1：创建虚拟环境
python -m venv myproject_env
# 这会在当前目录下创建一个 myproject_env 文件夹

# 步骤2：激活虚拟环境

# Windows (PowerShell)：
myproject_env\Scripts\Activate.ps1

# Windows (CMD)：
myproject_env\Scripts\activate.bat

# macOS / Linux：
source myproject_env/bin/activate
```

激活后，命令行提示符前面会出现 `(myproject_env)`：

```bash
(myproject_env) D:\projects\myproject>
```

```bash
# 步骤3：在虚拟环境中安装包
pip install requests pandas

# 步骤4：确认包安装在虚拟环境中
pip list
# 只会看到虚拟环境中安装的包，不会看到全局的包

# 步骤5：退出虚拟环境
deactivate
```

### 3.3 虚拟环境目录结构

创建虚拟环境后，会生成如下目录结构：

```
myproject_env/
├── Scripts/          # Windows 下的脚本目录
│   ├── activate      # 激活脚本
│   ├── deactivate    # 退出脚本
│   ├── pip.exe       # pip 程序
│   └── python.exe    # Python 解释器（副本）
├── Lib/
│   └── site-packages/   # 第三方包安装位置
├── pyvenv.cfg        # 虚拟环境配置文件
└── ...
```

> **关键点**：虚拟环境不是复制整个 Python，而是创建一个"快捷方式"指向原 Python，并拥有独立的 `site-packages` 目录来存放包。

### 3.4 完整工作流示例

```bash
# 1. 创建项目文件夹
mkdir my_web_app
cd my_web_app

# 2. 创建虚拟环境
python -m venv venv

# 3. 激活虚拟环境
venv\Scripts\Activate.ps1    # Windows PowerShell

# 4. 安装项目依赖
pip install flask requests

# 5. 导出依赖
pip freeze > requirements.txt

# 6. 开始写代码、运行代码...
python app.py

# 7. 工作完毕，退出虚拟环境
deactivate
```

### 3.5 不要做的事（常见错误）

```
❌ 错误做法：直接在全局环境安装所有包
   pip install flask django requests pandas ...
   → 不同项目依赖冲突，环境越来越乱

❌ 错误做法：把虚拟环境文件夹传给同事
   → 体积大、路径不同无法直接使用
   → 正确做法是传 requirements.txt，让对方自己创建虚拟环境

❌ 错误做法：用系统 Python 的 pip 安装包后又在虚拟环境中找不到
   → 确保先激活虚拟环境，再执行 pip install
```

---

## 四、pip 与虚拟环境进阶

### 4.1 pip show 查看包详情

```bash
pip show flask
# 输出示例：
# Name: Flask
# Version: 3.0.0
# Summary: A simple framework for building web applications.
# Home-page: https://palletsprojects.com/p/flask/
# Author: Armin Ronacher
# Author-email: armin.ronacher@active-4.com
# License: BSD-3-Clause
# Location: /path/to/venv/lib/python3.13/site-packages
# Requires: Werkzeug, Jinja2, itsdangerous, click, blinker
# Required-by:
```

> 注意 `Requires` 字段——这就是为什么你只安装了一个包，`pip list` 却多了好几个。这些叫**间接依赖**（也叫子依赖）。

### 4.2 pip check 检查依赖冲突

```bash
pip check
# 检查已安装的包之间是否有版本冲突
# 如果没有冲突：No broken requirements found.
# 如果有冲突：会列出哪些包的依赖不兼容
```

### 4.3 pip cache 管理缓存

pip 下载的包会缓存在本地，下次安装同样版本时不需要重新下载。

```bash
# 查看缓存位置
pip cache dir

# 查看缓存信息
pip cache info

# 清理缓存（释放磁盘空间）
pip cache purge
```

### 4.4 conda 简介（了解即可）

除了 venv，**Anaconda / Miniconda** 也是流行的 Python 环境管理工具，尤其在数据科学领域：

```bash
# conda 创建环境
conda create -n myenv python=3.11

# conda 激活环境
conda activate myenv

# conda 安装包（从 conda 仓库安装）
conda install numpy

# conda 也能用 pip
pip install requests
```

> **venv vs conda 怎么选？**
> - 纯 Python Web 开发 → venv（轻量、内置、够用）
> - 数据科学/机器学习 → conda（方便管理科学计算库和 Python 本身版本）

---

## 五、综合实战：搭建一个规范的 Python 项目

让我们把今天学的知识整合起来，搭建一个标准的 Python 项目结构：

```
my_project/
├── venv/                    # 虚拟环境（不要提交到 Git）
├── src/                     # 源代码目录
│   └── main.py              # 主程序
├── tests/                   # 测试目录
│   └── test_main.py         # 测试文件
├── requirements.txt         # 依赖文件
├── README.md                # 项目说明
└── .gitignore               # Git 忽略文件
```

### .gitignore 文件内容

```text
# 虚拟环境
venv/
env/
.venv/

# Python 缓存
__pycache__/
*.pyc
*.pyo

# IDE 配置
.vscode/
.idea/

# 环境变量
.env
```

### src/main.py 示例代码

```python
"""我的第一个规范项目 - 使用虚拟环境中的第三方库"""

import requests  # 第三方库，通过 pip install 安装


def get_ip_info():
    """获取当前设备的公网IP信息"""
    try:
        response = requests.get("https://httpbin.org/ip", timeout=5)
        response.raise_for_status()  # 如果状态码不是200，抛出异常
        data = response.json()
        return data
    except requests.RequestException as e:
        print(f"请求失败: {e}")
        return None


def main():
    print("=" * 40)
    print("  欢迎来到我的 Python 项目")
    print("  当前运行在虚拟环境中")
    print("=" * 40)

    # 测试第三方库是否正常工作
    ip_info = get_ip_info()
    if ip_info:
        print(f"\n当前公网 IP: {ip_info.get('origin', '未知')}")
    print(f"\n项目启动成功！")


if __name__ == "__main__":
    main()
```

### tests/test_main.py 示例代码

```python
"""测试主模块"""
import unittest


class TestMain(unittest.TestCase):
    """测试 main 模块的基本功能"""

    def test_import_requests(self):
        """验证 requests 库可以正常导入"""
        import requests
        self.assertTrue(hasattr(requests, "get"))

    def test_import_main(self):
        """验证 main 模块可以正常导入"""
        from src import main
        self.assertTrue(hasattr(main, "get_ip_info"))


if __name__ == "__main__":
    unittest.main()
```

### 完整搭建流程（命令行操作）

```bash
# 1. 创建项目目录
mkdir my_project && cd my_project
mkdir src tests

# 2. 创建并激活虚拟环境
python -m venv venv
venv\Scripts\Activate.ps1          # Windows PowerShell

# 3. 创建 requirements.txt
echo "requests==2.31.0" > requirements.txt

# 4. 安装依赖
pip install -r requirements.txt

# 5. 创建 .gitignore
# （手动创建或用代码编辑器创建，内容见上方）

# 6. 运行项目
python src\main.py

# 7. 运行测试
python -m pytest tests/            # 需要先 pip install pytest
# 或
python -m unittest discover tests/
```

---

## 六、练习题

### 练习1：pip 基础操作

1. 创建一个虚拟环境
2. 在其中安装 `requests` 和 `beautifulsoup4` 两个包
3. 用 `pip list` 和 `pip show` 查看安装信息
4. 导出 `requirements.txt`
5. 卸载 `beautifulsoup4`，再通过 `requirements.txt` 重新安装

### 练习2：环境隔离验证

1. 创建两个虚拟环境 `env_a` 和 `env_b`
2. 在 `env_a` 中安装 `requests==2.28.0`
3. 在 `env_b` 中安装 `requests==2.31.0`
4. 分别用 `pip show requests` 确认版本不同
5. 理解两个环境的 `site-packages` 是完全独立的

### 练习3：搭建项目

按照第五节的规范，搭建一个 Python 项目，包含：
- 虚拟环境
- `src/` 目录和 `main.py`
- `tests/` 目录和测试文件
- `requirements.txt`
- `.gitignore`

---

## 七、常见问题（FAQ）

### Q1：`pip` 和 `pip3` 有什么区别？

`pip3` 特指 Python 3 的 pip。在大多数现代系统上，如果只安装了 Python 3，`pip` 和 `pip3` 指向同一个程序。如果系统同时有 Python 2 和 Python 3，建议用 `python -m pip` 来明确指定用哪个 Python 版本的 pip。

### Q2：虚拟环境创建失败怎么办？

常见原因：
- Python 版本太旧（需要 3.3+，但 3.3~3.5 需要先 `pip install venv`）
- 缺少 `venv` 模块（某些 Linux 发行版需要安装 `python3-venv` 包）

```bash
# Ubuntu/Debian 如果缺少 venv：
sudo apt-get install python3.12-venv
```

### Q3：激活虚拟环境时报错"禁止运行脚本"？

Windows PowerShell 默认禁止执行脚本。以管理员身份运行 PowerShell，执行：

```powershell
Set-ExecutionPolicy RemoteSigned
```

或者改用 CMD（命令提示符），使用 `venv\Scripts\activate.bat` 激活。

### Q4：`pip install` 时报 SSL 错误？

通常是网络环境（公司代理/防火墙）导致。尝试：
```bash
# 方法1：信任 PyPI 的证书
pip install --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host files.pythonhosted.org <包名>

# 方法2：使用国内镜像
pip install <包名> -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

### Q5：如何一次性升级所有已安装的包？

```bash
# Linux/macOS
pip list --outdated --format=freeze | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip install -U

# Windows PowerShell
pip list --outdated --format=freeze | ForEach-Object { $_.Split('=')[0] } | ForEach-Object { pip install --upgrade $_ }
```

> 不过不建议盲目升级所有包，可能导致兼容性问题。按需升级更安全。

---

## 八、总结

| 概念 | 作用 | 核心命令 |
|------|------|----------|
| pip | 包管理器 | `pip install/uninstall/list/show` |
| requirements.txt | 依赖清单 | `pip freeze > requirements.txt` |
| venv | 虚拟环境 | `python -m venv venv` |
| 激活/退出 | 切换环境 | `activate` / `deactivate` |
| PyPI | 包仓库 | https://pypi.org |

**今天的关键收获**：
1. pip 是 Python 的包管理器，能从 PyPI 安装第三方库
2. `requirements.txt` 是项目的"依赖清单"，方便团队协作
3. 虚拟环境让每个项目拥有独立的依赖，互不冲突
4. 每个项目都应该创建自己的虚拟环境
5. 虚拟环境文件夹不要提交到 Git

---

## 九、免费学习资源

- [廖雪峰 - pip 和虚拟环境](https://www.liaoxuefeng.com/wiki/1016959663602400/1019312825819520)
- [菜鸟教程 - pip](https://www.runoob.com/w3cnote/python-pip-install-usage.html)
- [Python 官方文档 - venv](https://docs.python.org/zh-cn/3/library/venv.html)
- [Python 官方文档 - pip 用户指南](https://pip.pypa.io/zh-cn/stable/user_guide/)
- [PyPI 官网](https://pypi.org/) — 搜索和浏览 Python 包

---

> **下一篇预告**：Day 32 — 项目结构规范，学习如何组织一个大型 Python 项目的目录结构、`__init__.py` 的作用、包的相对导入与绝对导入等内容。
