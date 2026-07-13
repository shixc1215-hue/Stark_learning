# Python Day 35：Git 版本控制

> 写代码最怕什么？——改坏了回不去。Git 就是你的"时光机"，每一次保存都是一个存档点，随时可以回档重来。

---

## 一、版本控制是什么？

想象你在写一篇论文，改了十版，文件名变成这样：

```
报告_final.docx
报告_最终版.docx
报告_打死不改版.docx
报告_真的最后一版.docx
报告_终极版_v2.docx
```

这就是**没有版本控制**的后果。版本控制系统（VCS）能帮你：

| 功能 | 说明 |
|------|------|
| **记录历史** | 每次修改都有记录，谁在什么时候改了什么 |
| **随时回退** | 改坏了？一条命令回到之前的版本 |
| **分支开发** | 可以同时尝试多个方案，互不影响 |
| **多人协作** | 多人同时修改同一份代码，自动合并或提示冲突 |

### Git vs 其他工具

- **SVN / CVS**：集中式版本控制，所有记录存在一台服务器上，离线就无法操作
- **Git**：分布式版本控制，每个人的电脑上都有完整的仓库历史，离线也能提交

Git 是目前最流行的版本控制工具，由 Linux 之父 Linus Torvalds 于 2005 年创建。

---

## 二、安装与配置

### 2.1 安装 Git

- **Windows**：到 [https://git-scm.com/download/win](https://git-scm.com/download/win) 下载安装包，安装时一路"Next"即可
- **macOS**：`brew install git` 或安装 Xcode Command Line Tools
- **Linux（Ubuntu）**：`sudo apt install git`

安装完成后，打开终端（Windows 用户打开 Git Bash），输入：

```bash
git --version
# 输出示例：git version 2.45.0
```

能看到版本号就说明安装成功。

### 2.2 初始配置

Git 需要知道你是谁，这样每次提交都能记录作者：

```bash
# 设置用户名（替换成你自己的名字）
git config --global user.name "张三"

# 设置邮箱（替换成你自己的邮箱）
git config --global user.email "zhangsan@example.com"

# 查看当前配置
git config --list

# 设置默认分支名为 main（推荐）
git config --global init.defaultBranch main
```

> **为什么要设置 `user.name` 和 `user.email`？**
> 每次你提交代码时，Git 会自动记录这两个信息。在团队协作中，这能让大家知道这段代码是谁写的。

---

## 三、Git 核心概念

在开始操作之前，先理解 Git 的三个区域：

```
工作区（Working Directory）  →  暂存区（Staging Area）  →  本地仓库（Repository）
      你编辑代码的地方              你标记要提交的改动            永久保存的历史记录
```

还有一个**远程仓库（Remote）**，比如 GitHub、Gitee（码云）等服务器上的仓库。

```
┌──────────────┐   git add    ┌──────────────┐   git commit   ┌──────────────┐   git push    ┌──────────────┐
│   工作区      │ ──────────→  │   暂存区     │ ───────────→  │  本地仓库     │ ──────────→  │  远程仓库     │
│ (Working Dir)│              │  (Staging)   │               │  (Repository) │              │  (Remote)     │
└──────────────┘              └──────────────┘               └──────────────┘              └──────────────┘
```

---

## 四、基础操作流程

### 4.1 创建仓库

**方式一：把已有项目变成 Git 仓库**

```bash
# 进入你的项目目录
cd ~/projects/my_python_project

# 初始化 Git 仓库（会在当前目录创建一个隐藏的 .git 文件夹）
git init

# 查看仓库状态
git status
```

**方式二：从远程仓库克隆**

```bash
# 从 GitHub 克隆一个项目到本地
git clone https://github.com/user/project.git

# 克隆后自动创建项目文件夹，并进入
cd project
```

### 4.2 记录修改（日常最高频操作）

```bash
# 1. 查看当前状态（哪些文件被修改了）
git status

# 2. 查看具体改了什么（类似"diff"对比）
git diff

# 3. 把修改添加到暂存区
git add main.py              # 只添加 main.py
git add .                    # 添加所有修改（小心使用）
git add *.py                 # 添加所有 .py 文件

# 4. 提交到本地仓库（-m 后面写提交说明）
git commit -m "修复了登录功能的bug"

# 5. 查看提交历史
git log                      # 详细版本
git log --oneline            # 简洁版本（推荐）
```

> **💡 好习惯**：提交说明要写清楚做了什么。好的说明如"修复登录验证逻辑错误"、"新增用户注册接口"；不好的说明如"改了"、"update"。

### 4.3 查看与回退

```bash
# 查看提交历史（图形化）
git log --oneline --graph

# 回退到上一个版本（两种方式）
# 方式一：soft —— 只回退 commit，保留暂存区的改动
git reset --soft HEAD~1

# 方式二：hard —— 彻底回退，丢弃所有改动（⚠️ 危险！）
git reset --hard HEAD~1

# 方式三：checkout（只是查看，不会真正回退）
git checkout <commit-hash>   # 临时切换到某个历史版本查看
git checkout main            # 看完了，回到最新版本
```

> **⚠️ 警告**：`git reset --hard` 会永久丢弃未提交的修改！使用前请三思。如果不确定，先用 `git stash` 保存当前修改。

### 4.4 .gitignore 忽略文件

有些文件不需要被 Git 追踪（比如编译产物、临时文件、密码配置等）：

```bash
# 创建 .gitignore 文件
touch .gitignore
```

在 `.gitignore` 中写入：

```gitignore
# Python 编译产物
__pycache__/
*.pyc
*.pyo
*.egg-info/

# 虚拟环境
venv/
.venv/
env/

# IDE 配置
.vscode/
.idea/

# 系统文件
.DS_Store
Thumbs.db

# 敏感信息（密码、密钥等）
.env
*.pem
config/local.py

# 分发 / 打包
dist/
build/
*.egg
```

---

## 五、分支（Branch）

分支是 Git 最强大的功能之一。你可以创建一个分支来开发新功能，开发完成后再合并到主分支，全程不影响主分支的稳定。

### 5.1 分支基本操作

```bash
# 查看所有分支（* 标记当前分支）
git branch

# 创建新分支
git branch feature-login     # 创建一个叫 feature-login 的分支
git checkout feature-login   # 切换到这个分支
# 或者一步到位：
git checkout -b feature-login  # 创建并切换

# 在新分支上开发、提交...
git add login.py
git commit -m "实现登录功能"

# 切回主分支
git checkout main

# 合并新分支到主分支
git merge feature-login

# 合并完成后删除分支（保持整洁）
git branch -d feature-login
```

### 5.2 合并冲突

当两个分支修改了同一个文件的同一行时，Git 无法自动决定保留哪个，就会产生**冲突**：

```bash
git merge feature-login
# 如果有冲突，会提示：
# CONFLICT (content): Merge conflict in login.py
# Automatic merge failed; fix conflicts and then commit the result.
```

打开冲突文件，你会看到这样的标记：

```python
def login(username, password):
<<<<<<< feature-login
    # 使用 JWT 验证
    token = jwt_encode(username)
    return token
=======
    # 使用 Session 验证
    session_id = create_session(username)
    return session_id
>>>>>>> main
```

手动选择要保留的代码（或者合并两者的逻辑），删除 `<<<<<<`、`======`、`>>>>>>` 标记，然后：

```bash
git add login.py
git commit -m "解决合并冲突：采用JWT验证方案"
```

### 5.3 分支工作流推荐

对于 Python 项目，推荐使用以下分支策略：

```
main          （主分支，永远保持可发布状态）
  └── dev     （开发分支，日常开发在这里）
       ├── feature-login     （功能分支：登录功能）
       ├── feature-payment  （功能分支：支付功能）
       └── bugfix-42        （修复分支：修复42号bug）
```

```bash
# 1. 从 dev 创建功能分支
git checkout dev
git checkout -b feature-login

# 2. 开发完成后，切回 dev 合并
git checkout dev
git merge feature-login

# 3. 删除功能分支
git branch -d feature-login
```

---

## 六、远程仓库（GitHub / Gitee）

远程仓库让你的代码托管在云端，支持多人协作和代码备份。

### 6.1 关联远程仓库

```bash
# 在 GitHub 或 Gitee 上创建一个空仓库后，执行：

# 添加远程仓库（origin 是远程仓库的别名，可以自定义）
git remote add origin https://github.com/zhangsan/my_project.git

# 查看远程仓库
git remote -v

# 推送代码到远程仓库
git push -u origin main      # -u 参数会记住这个关联，以后只需 git push

# 拉取远程仓库的最新代码
git pull origin main

# 拉取但不自动合并（更安全）
git fetch origin
git merge origin/main
```

### 6.2 SSH vs HTTPS

| 方式 | 优点 | 缺点 |
|------|------|------|
| **HTTPS** | 简单，不用配置 | 每次推送可能要输入密码（可配置 token 免密） |
| **SSH** | 配置一次，永久免密 | 需要生成和添加 SSH 密钥 |

**配置 SSH（推荐）**：

```bash
# 1. 生成 SSH 密钥（一路回车即可）
ssh-keygen -t ed25519 -C "zhangsan@example.com"

# 2. 查看公钥
cat ~/.ssh/id_ed25519.pub

# 3. 复制公钥内容，到 GitHub/Gitee → Settings → SSH Keys → 添加

# 4. 测试连接
ssh -T git@github.com
# 输出：Hi zhangsan! You've successfully authenticated...
```

### 6.3 .gitignore 时机

如果忘记在第一次提交前创建 `.gitignore`，已经提交了不该提交的文件怎么办？

```bash
# 1. 先把文件从 Git 追踪中移除（但保留本地文件）
git rm --cached __pycache__/ -r

# 2. 提交这个改动
git commit -m "移除不需要追踪的文件"

# 3. 把规则加入 .gitignore
echo "__pycache__/" >> .gitignore
git add .gitignore
git commit -m "添加 .gitignore"
```

---

## 七、Python 项目中的 Git 最佳实践

### 7.1 提交粒度

```bash
# ❌ 不好：一次提交包含所有改动
git add .
git commit -m "大更新"

# ✅ 好的：一个逻辑改动一次提交
git add utils.py
git commit -m "提取重复代码为工具函数"

git add tests/test_utils.py
git commit -m "添加 utils 模块的单元测试"

git add main.py
git commit -m "main.py 使用新的工具函数重构"
```

### 7.2 提交信息规范（Conventional Commits）

推荐的提交信息格式：

```
<类型>(<范围>): <描述>

类型：
  feat     - 新功能
  fix      - 修复 bug
  docs     - 文档修改
  style    - 格式调整（不影响逻辑）
  refactor - 代码重构
  test     - 测试相关
  chore    - 构建/工具变动

示例：
  feat(auth): 新增用户注册接口
  fix(login): 修复密码验证时的空指针异常
  docs(readme): 更新安装说明
```

### 7.3 Git 与 Python 工具链集成

**requirements.txt 版本追踪**：

```bash
# 生成依赖列表
pip freeze > requirements.txt

# 每次添加新依赖后，更新并提交
git add requirements.txt
git commit -m "build: 添加 pandas 依赖"
```

**pre-commit 钩子（自动代码检查）**：

```bash
# 安装 pre-commit
pip install pre-commit

# 创建 .pre-commit-config.yaml
```

`.pre-commit-config.yaml` 示例：

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: '24.4.0'
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/PyCQA/flake8
    rev: '7.0.0'
    hooks:
      - id: flake8
```

```bash
# 安装钩子
pre-commit install

# 以后每次 commit 前自动运行代码格式化和检查
```

---

## 八、Git 常用命令速查表

### 仓库操作

| 命令 | 作用 |
|------|------|
| `git init` | 初始化新仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git status` | 查看当前状态 |

### 文件操作

| 命令 | 作用 |
|------|------|
| `git add <file>` | 添加到暂存区 |
| `git add .` | 添加所有改动 |
| `git commit -m "msg"` | 提交 |
| `git rm <file>` | 删除文件 |
| `git mv <old> <new>` | 重命名文件 |

| `git diff` | 查看工作区改动 |
| `git diff --staged` | 查看暂存区改动 |

### 分支操作

| 命令 | 作用 |
|------|------|
| `git branch` | 查看分支 |
| `git branch <name>` | 创建分支 |
| `git checkout <name>` | 切换分支 |
| `git checkout -b <name>` | 创建并切换 |
| `git merge <name>` | 合并分支 |
| `git branch -d <name>` | 删除分支 |

### 远程操作

| 命令 | 作用 |
|------|------|
| `git remote -v` | 查看远程仓库 |
| `git push` | 推送到远程 |
| `git pull` | 拉取远程更新 |
| `git fetch` | 获取远程更新（不合并） |

### 查看历史

| 命令 | 作用 |
|------|------|
| `git log` | 详细提交历史 |
| `git log --oneline` | 简洁历史 |
| `git log --graph` | 图形化分支历史 |
| `git show <hash>` | 查看某次提交的详情 |

### 撤销操作

| 命令 | 作用 |
|------|------|
| `git checkout -- <file>` | 撤销工作区修改 |
| `git reset HEAD <file>` | 取消暂存 |
| `git reset --soft HEAD~1` | 撤销上次提交（保留改动） |
| `git stash` | 暂存当前改动 |
| `git stash pop` | 恢复暂存的改动 |

---

## 九、综合实战：从零搭建一个 Python 项目的 Git 工作流

假设你要开发一个"记账本"程序，从零开始：

```bash
# ===== 第一步：创建项目 =====
mkdir expense_tracker
cd expense_tracker

# 初始化 Git
git init

# 创建 .gitignore
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
venv/
.env
*.egg-info/
dist/
EOF

# 第一次提交
git add .
git commit -m "chore: 初始化项目结构，添加 .gitignore"

# ===== 第二步：开发基础功能 =====
# 创建主程序
cat > main.py << 'EOF'
"""简易记账本程序"""
expenses = []

def add_expense(item: str, amount: float):
    """添加一笔支出"""
    expenses.append({"item": item, "amount": amount})
    print(f"已记录：{item} ¥{amount:.2f}")

def show_summary():
    """显示支出汇总"""
    total = sum(e["amount"] for e in expenses)
    print(f"\n共 {len(expenses)} 笔支出，合计 ¥{total:.2f}")
    for e in expenses:
        print(f"  - {e['item']}: ¥{e['amount']:.2f}")

if __name__ == "__main__":
    add_expense("午餐", 25.0)
    add_expense("地铁", 4.0)
    add_expense("咖啡", 35.0)
    show_summary()
EOF

git add main.py
git commit -m "feat(core): 实现记账本基础功能——添加支出和显示汇总"

# ===== 第三步：添加测试 =====
cat > test_main.py << 'EOF'
"""记账本单元测试"""
import unittest
from main import add_expense, show_summary, expenses

class TestExpenseTracker(unittest.TestCase):
    def setUp(self):
        """每次测试前清空列表"""
        expenses.clear()

    def test_add_expense(self):
        add_expense("午餐", 25.0)
        self.assertEqual(len(expenses), 1)
        self.assertEqual(expenses[0]["item"], "午餐")
        self.assertAlmostEqual(expenses[0]["amount"], 25.0)

    def test_show_summary_empty(self):
        """没有记录时也能正常显示"""
        show_summary()  # 不应报错

    def test_multiple_expenses(self):
        add_expense("午餐", 25.0)
        add_expense("地铁", 4.0)
        self.assertEqual(len(expenses), 2)

if __name__ == "__main__":
    unittest.main()
EOF

git add test_main.py
git commit -m "test: 添加记账本核心功能单元测试"

# ===== 第四步：关联远程仓库 =====
# 在 GitHub/Gitee 创建仓库后执行：
# git remote add origin https://github.com/zhangsan/expense_tracker.git
# git push -u origin main
```

### 使用分支开发新功能

```bash
# 创建功能分支：添加 CSV 导出功能
git checkout -b feature-csv-export

# 开发并提交
cat > export.py << 'EOF'
"""CSV 导出模块"""
import csv
from main import expenses

def export_to_csv(filename: str = "expenses.csv"):
    """将支出记录导出为 CSV 文件"""
    if not expenses:
        print("没有支出记录可导出")
        return

    with open(filename, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.DictWriter(f, fieldnames=["item", "amount"])
        writer.writeheader()
        writer.writerows(expenses)
    print(f"已导出到 {filename}")
EOF

git add export.py
git commit -m "feat(export): 实现 CSV 导出功能"

# 切回主分支并合并
git checkout main
git merge feature-csv-export
git branch -d feature-csv-export

# 推送到远程
# git push
```

---

## 十、练习题

### 练习一：基础操作（难度：⭐）

1. 在本地创建一个新目录 `git_practice`
2. 初始化 Git 仓库
3. 创建一个 `hello.py` 文件，内容为 `print("Hello Git!")`
4. 添加并提交，提交信息为 `feat: 添加 hello.py`
5. 修改文件为 `print("Hello Git! Version 2")`
6. 再次提交，提交信息为 `feat: 更新问候语`
7. 用 `git log --oneline` 查看提交历史

### 练习二：分支实战（难度：⭐⭐）

1. 创建 `calculator.py`，实现加减乘除四个函数
2. 提交到 main 分支
3. 创建 `feature-modulo` 分支
4. 在新分支上添加取模函数
5. 切回 main，合并分支
6. 删除 feature-modulo 分支

### 练习三：回退操作（难度：⭐⭐⭐）

1. 创建一个 `notes.txt`，写入 3 行文字，提交
2. 再添加 2 行文字，提交
3. 再修改第 1 行文字，提交
4. 用 `git log --oneline` 查看历史
5. 用 `git diff HEAD~1` 查看最近一次提交的改动
6. 尝试用 `git checkout` 查看第一次提交时的文件内容，然后回到最新版本

---

## 十一、常见问题（FAQ）

### Q1：`git push` 报错 "rejected" 怎么办？

```
! [rejected]        main -> main (fetch first)
error: failed to push some refs
```

**原因**：远程仓库有新的提交，你本地的历史落后了。

**解决**：

```bash
# 先拉取远程更新
git pull --rebase origin main
# 再推送
git push
```

### Q2：不小心 `git add` 了敏感文件（如密码），怎么办？

```bash
# 如果还没 commit：
git reset HEAD <敏感文件>
echo "敏感文件名" >> .gitignore

# 如果已经 commit 了：
git rm --cached <敏感文件>
git commit -m "security: 移除敏感文件"
echo "敏感文件名" >> .gitignore
git commit -am "添加 .gitignore 规则"

# 如果已经 push 了：需要用 git filter-branch 或 BFG 清理历史（高级操作）
```

### Q3：`git stash` 是什么？什么时候用？

`git stash` 用来临时保存当前未提交的修改：

```bash
# 场景：你正在开发功能 A，突然要切换到分支 B 修复一个紧急 bug

# 1. 暂存当前工作
git stash
# 或者带说明：
git stash save "开发到一半的登录功能"

# 2. 切换到修复分支
git checkout bugfix-42
# ... 修复并提交 ...

# 3. 切回来，恢复之前的工作
git checkout feature-login
git stash pop   # 恢复并删除暂存记录

# 查看所有暂存
git stash list
```

### Q4：GitHub 和 Gitee 有什么区别？

| 对比项 | GitHub | Gitee（码云） |
|--------|--------|---------------|
| 服务器位置 | 国外（访问可能较慢） | 国内（访问速度快） |
| 免费私有仓库 | ✅ 支持 | ✅ 支持 |
| 中文支持 | 一般 | ✅ 原生中文 |
| 适合场景 | 开源项目、国际协作 | 国内项目、学习使用 |

初学者建议先在 **Gitee** 上练习，速度快，全中文界面。

### Q5：如何撤销上一次 commit（但不删除改动）？

```bash
git reset --soft HEAD~1
# HEAD~1 表示"上一次提交"
# --soft 表示"软重置"：commit 撤销了，但改动还在暂存区
# 你可以修改后重新 commit
```

---

## 十二、免费学习资源

- **Pro Git 电子书（中文版）**：[https://git-scm.com/book/zh/v2](https://git-scm.com/book/zh/v2) —— Git 最权威的免费教材
- **廖雪峰 Git 教程**：[https://www.liaoxuefeng.com/wiki/896043488029600](https://www.liaoxuefeng.com/wiki/896043488029600) —— 简洁易懂
- **菜鸟教程 Git**：[https://www.runoob.com/git/git-tutorial.html](https://www.runoob.com/git/git-tutorial.html) —— 命令速查方便
- **GitHub 官方教程**：[https://docs.github.com/cn/get-started](https://docs.github.com/cn/get-started) —— 学会使用 GitHub
- **Learn Git Branching**：[https://learngitbranching.js.org/?locale=zh_CN](https://learngitbranching.js.org/?locale=zh_CN) —— 交互式学习 Git 分支（强烈推荐！）
- **Oh My Git!**：[https://ohmygit.org/](https://ohmygit.org/) —— Git 学习小游戏

---

> **Day 35 小结**：今天学习了 Git 版本控制的核心操作——`init`、`add`、`commit`、`branch`、`merge`、`push/pull`，以及 `.gitignore`、提交规范和分支工作流。Git 是每个程序员必备的技能，从今天开始养成"勤提交、写好说明"的习惯。
>
> **下一节预告**：Day 36 — requests 库，用 Python 发送 HTTP 请求，打开网络编程的大门。
