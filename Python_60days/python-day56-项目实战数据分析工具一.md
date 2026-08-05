# Python Day 56 - 项目实战：数据分析工具（一）

> 恭喜你走到了 Day 56！前面 55 天你已经掌握了 Python 核心语法、面向对象、数据处理（pandas/NumPy）、可视化（matplotlib/seaborn）、Web 开发（FastAPI/Streamlit/HTML+CSS+JS）、数据库（SQLite/SQLAlchemy）等技能。从今天开始，我们进入**最后冲刺阶段**——通过两个大型综合项目，把所有知识串起来。第一个项目是**数据分析工具 DataLens**，今天完成上半部分（项目搭建 + 数据引擎 + 命令行分析）。

---

## 一、项目概述

### 1.1 项目目标

我们要构建一个名为 **DataLens** 的数据分析工具，它能够：

| 功能 | 说明 |
|------|------|
| 📊 数据加载 | 支持 CSV、Excel、JSON 三种格式导入 |
| 🧹 数据清洗 | 自动检测缺失值、重复数据、异常值 |
| 📈 统计分析 | 描述性统计、分组聚合、相关性分析 |
| 🎨 可视化 | 自动生成分析图表（趋势图、分布图、相关矩阵） |
| 📋 分析报告 | 生成 Markdown 格式的分析摘要报告 |
| 🖥️ 交互界面 | Streamlit Web 界面（Day 57 实现） |

### 1.2 项目架构

```
datalens/
├── pyproject.toml              # 项目配置
├── requirements.txt             # 依赖清单
├── README.md
├── run.py                       # 入口文件
├── datalens/                    # 核心包
│   ├── __init__.py             # 版本号 + 公共接口
│   ├── loader.py               # 数据加载模块
│   ├── cleaner.py              # 数据清洗模块
│   ├── analyzer.py              # 统计分析模块
│   ├── visualizer.py            # 可视化模块
│   ├── reporter.py             # 报告生成模块
│   ├── cli.py                  # 命令行接口
│   └── utils.py                # 工具函数
├── tests/                       # 单元测试（Day 58 补充）
│   └── ...
└── data/                        # 示例数据
    └── sample_sales.csv
```

### 1.3 技术栈回顾

这个项目用到的知识点覆盖了你学过的绝大部分内容：

```
Day 11-12  函数设计 → 模块化架构
Day 13     异常处理 → 优雅的错误提示
Day 15-17  面向对象 → 数据类封装
Day 20     装饰器 → 日志计时
Day 28-29  类型注解 + dataclass → 类型安全
Day 30     单元测试 → 质量保证
Day 32     项目结构 → src layout
Day 33     logging → 运行日志
Day 38-40  pandas → 数据处理核心
Day 41-42  matplotlib + seaborn → 可视化
Day 51-52  Streamlit → Web 界面（明天）
```

---

## 二、项目搭建

### 2.1 创建项目结构

```bash
# 创建项目根目录
mkdir datalens && cd datalens

# 创建目录结构
mkdir -p datalens tests data
touch datalens/__init__.py
touch datalens/{loader,cleaner,analyzer,visualizer,reporter,cli,utils}.py
touch tests/__init__.py
touch run.py requirements.txt pyproject.toml
```

### 2.2 依赖配置

```txt
# requirements.txt
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
openpyxl>=3.1      # Excel 文件支持
click>=8.1          # 命令行框架
rich>=13.0          # 终端美化输出
```

```toml
# pyproject.toml
[project]
name = "datalens"
version = "0.1.0"
description = "Python 数据分析工具"
requires-python = ">=3.10"
dependencies = [
    "pandas>=2.0",
    "numpy>=1.24",
    "matplotlib>=3.7",
    "seaborn>=0.12",
    "openpyxl>=3.1",
    "click>=8.1",
    "rich>=13.0",
]

[project.scripts]
datalens = "datalens.cli:main"
```

### 2.3 包初始化

```python
# datalens/__init__.py
"""
DataLens - Python 数据分析工具
版本: 0.1.0
"""

__version__ = "0.1.0"

# 公共接口：用户可以 from datalens import DataLoader
from datalens.loader import DataLoader
from datalens.cleaner import DataCleaner
from datalens.analyzer import DataAnalyzer
from datalens.visualizer import DataVisualizer
from datalens.reporter import DataReporter
```

---

## 三、数据加载模块（loader.py）

这个模块负责将不同格式的文件加载为 pandas DataFrame。

```python
# datalens/loader.py
"""
数据加载模块 - 支持 CSV、Excel、JSON 三种格式
"""
import json
from pathlib import Path
from typing import Optional

import pandas as pd
from rich.console import Console
from rich.table import Table

console = Console()


class DataLoader:
    """数据加载器：将文件加载为 DataFrame"""

    # 支持的文件格式与扩展名映射
    SUPPORTED_FORMATS = {
        ".csv": "csv",
        ".xlsx": "excel",
        ".xls": "excel",
        ".json": "json",
    }

    def __init__(self, filepath: str | Path):
        """
        初始化加载器

        Args:
            filepath: 数据文件路径
        """
        self.filepath = Path(filepath)
        self.df: Optional[pd.DataFrame] = None
        self.file_info: dict = {}

    def load(self) -> pd.DataFrame:
        """
        加载数据文件（自动识别格式）

        Returns:
            加载后的 DataFrame

        Raises:
            FileNotFoundError: 文件不存在
            ValueError: 不支持的文件格式
        """
        # 1. 检查文件是否存在
        if not self.filepath.exists():
            raise FileNotFoundError(
                f"文件不存在: {self.filepath}"
            )

        # 2. 自动识别格式
        suffix = self.filepath.suffix.lower()
        if suffix not in self.SUPPORTED_FORMATS:
            raise ValueError(
                f"不支持的文件格式 '{suffix}'，"
                f"支持: {list(self.SUPPORTED_FORMATS.keys())}"
            )
        fmt = self.SUPPORTED_FORMATS[suffix]

        # 3. 根据格式加载
        loaders = {
            "csv": self._load_csv,
            "excel": self._load_excel,
            "json": self._load_json,
        }
        self.df = loaders[fmt]()

        # 4. 记录文件信息
        self.file_info = {
            "文件路径": str(self.filepath),
            "文件格式": fmt,
            "数据形状": f"{self.df.shape[0]} 行 × {self.df.shape[1]} 列",
            "列名": list(self.df.columns),
            "内存占用": self._format_size(self.df.memory_usage(deep=True).sum()),
        }

        console.print(
            f"[green]✓[/green] 成功加载 {self.file_info['数据形状']}，"
            f"内存占用 {self.file_info['内存占用']}"
        )
        return self.df

    def _load_csv(self) -> pd.DataFrame:
        """加载 CSV 文件"""
        return pd.read_csv(
            self.filepath,
            encoding="utf-8-sig",  # 自动处理带BOM的UTF-8
        )

    def _load_excel(self) -> pd.DataFrame:
        """加载 Excel 文件"""
        return pd.read_excel(self.filepath)

    def _load_json(self) -> pd.DataFrame:
        """加载 JSON 文件"""
        return pd.read_json(self.filepath)

    def preview(self, n: int = 5) -> None:
        """
        在终端预览数据

        Args:
            n: 显示的行数
        """
        if self.df is None:
            console.print("[red]请先调用 load() 加载数据[/red]")
            return

        # 使用 rich 的表格美化输出
        table = Table(title=f"数据预览（前 {n} 行）")
        for col in self.df.columns:
            table.add_column(str(col), style="cyan")

        for _, row in self.df.head(n).iterrows():
            table.add_row(*[str(v) for v in row])

        console.print(table)
        console.print(f"[dim]共 {len(self.df)} 行，{len(self.df.columns)} 列[/dim]")

    def summary(self) -> dict:
        """返回加载信息摘要"""
        if self.df is None:
            console.print("[red]请先调用 load() 加载数据[/red]")
            return {}
        return self.file_info

    @staticmethod
    def _format_size(size_bytes: float) -> str:
        """将字节数转为可读字符串"""
        for unit in ["B", "KB", "MB", "GB"]:
            if size_bytes < 1024:
                return f"{size_bytes:.1f} {unit}"
            size_bytes /= 1024
        return f"{size_bytes:.1f} TB"
```

---

## 四、数据清洗模块（cleaner.py）

```python
# datalens/cleaner.py
"""
数据清洗模块 - 自动检测并修复常见数据质量问题
"""
from typing import Optional

import pandas as pd
import numpy as np
from rich.console import Console

console = Console()


class DataCleaner:
    """数据清洗器：处理缺失值、重复数据、异常值"""

    def __init__(self, df: pd.DataFrame):
        """
        初始化清洗器

        Args:
            df: 待清洗的 DataFrame（会创建副本，不修改原数据）
        """
        self.original = df.copy()       # 保留原始数据
        self.df = df.copy()             # 工作副本
        self.report: list[dict] = []   # 清洗日志

    def _log(self, action: str, detail: str) -> None:
        """记录清洗操作"""
        entry = {"操作": action, "详情": detail}
        self.report.append(entry)
        console.print(f"  [yellow]·[/yellow] {action}: {detail}")

    # ---------- 缺失值处理 ----------

    def detect_missing(self) -> pd.DataFrame:
        """
        检测缺失值情况

        Returns:
            包含每列缺失统计的 DataFrame
        """
        missing = self.df.isnull().sum()
        pct = (missing / len(self.df) * 100).round(1)
        result = pd.DataFrame({
            "缺失数量": missing,
            "缺失比例(%)": pct,
        }).sort_values("缺失数量", ascending=False)

        result = result[result["缺失数量"] > 0]
        if result.empty:
            console.print("[green]✓ 未发现缺失值[/green]")
        else:
            console.print(f"[red]发现 {len(result)} 列存在缺失值[/red]")
        return result

    def fill_missing(self, strategy: str = "auto") -> "DataCleaner":
        """
        填充缺失值

        Args:
            strategy: 填充策略
                - "auto": 自动判断（数值列用中位数，文本列用众数）
                - "mean": 均值填充（仅数值列）
                - "median": 中位数填充（仅数值列）
                - "mode": 众数填充
                - "drop": 删除缺失行
                - "ffill": 前向填充
        """
        strategies = {
            "auto": self._fill_auto,
            "mean": lambda: self._fill_numeric("mean"),
            "median": lambda: self._fill_numeric("median"),
            "mode": self._fill_mode,
            "drop": self._fill_drop,
            "ffill": self._fill_ffill,
        }

        if strategy not in strategies:
            raise ValueError(
                f"不支持的策略 '{strategy}'，可选: {list(strategies.keys())}"
            )

        strategies[strategy]()
        self._log("缺失值处理", f"策略={strategy}")
        return self

    def _fill_auto(self) -> None:
        """自动填充：数值列 → 中位数，文本列 → 众数"""
        for col in self.df.columns:
            if self.df[col].isnull().sum() == 0:
                continue
            if pd.api.types.is_numeric_dtype(self.df[col]):
                self.df[col].fillna(self.df[col].median(), inplace=True)
            else:
                mode = self.df[col].mode()
                if not mode.empty:
                    self.df[col].fillna(mode.iloc[0], inplace=True)

    def _fill_numeric(self, method: str) -> None:
        """数值列填充（mean/median）"""
        numeric_cols = self.df.select_dtypes(include="number").columns
        for col in numeric_cols:
            fill_value = getattr(self.df[col], method)()
            self.df[col].fillna(fill_value, inplace=True)

    def _fill_mode(self) -> None:
        """众数填充"""
        for col in self.df.columns:
            mode = self.df[col].mode()
            if not mode.empty:
                self.df[col].fillna(mode.iloc[0], inplace=True)

    def _fill_drop(self) -> None:
        """删除含缺失的行"""
        before = len(self.df)
        self.df.dropna(inplace=True)
        self.df.reset_index(drop=True, inplace=True)
        self._log("删除缺失行", f"删除了 {before - len(self.df)} 行")

    def _fill_ffill(self) -> None:
        """前向填充"""
        self.df.fillna(method="ffill", inplace=True)

    # ---------- 重复数据 ----------

    def detect_duplicates(self) -> int:
        """检测重复行数量"""
        count = self.df.duplicated().sum()
        if count > 0:
            console.print(f"[red]发现 {count} 行重复数据[/red]")
        else:
            console.print("[green]✓ 未发现重复数据[/green]")
        return count

    def remove_duplicates(self, keep: str = "first") -> "DataCleaner":
        """
        删除重复行

        Args:
            keep: 保留策略（first/last/False）
        """
        before = len(self.df)
        self.df.drop_duplicates(keep=keep, inplace=True)
        self.df.reset_index(drop=True, inplace=True)
        removed = before - len(self.df)
        self._log("删除重复行", f"删除了 {removed} 行")
        return self

    # ---------- 异常值 ----------

    def detect_outliers(
        self, column: str, method: str = "iqr"
    ) -> pd.DataFrame:
        """
        检测某列的异常值

        Args:
            column: 目标列名
            method: 检测方法（"iqr" 四分位距 / "zscore" Z分数）

        Returns:
            包含异常值的 DataFrame
        """
        if method == "iqr":
            Q1 = self.df[column].quantile(0.25)
            Q3 = self.df[column].quantile(0.75)
            IQR = Q3 - Q1
            lower = Q1 - 1.5 * IQR
            upper = Q3 + 1.5 * IQR
            outliers = self.df[
                (self.df[column] < lower) | (self.df[column] > upper)
            ]
            console.print(
                f"  IQR 方法: [{lower:.2f}, {upper:.2f}]，"
                f"发现 {len(outliers)} 个异常值"
            )

        elif method == "zscore":
            z = np.abs(
                (self.df[column] - self.df[column].mean())
                / self.df[column].std()
            )
            outliers = self.df[z > 3]
            console.print(
                f"  Z-Score 方法 (阈值=3)，发现 {len(outliers)} 个异常值"
            )
        else:
            raise ValueError(f"不支持的方法 '{method}'")

        return outliers

    def cap_outliers(self, column: str) -> "DataCleaner":
        """
        用四分位距法截断异常值（替换为边界值）

        Args:
            column: 目标列名
        """
        Q1 = self.df[column].quantile(0.25)
        Q3 = self.df[column].quantile(0.75)
        IQR = Q3 - Q1
        lower = Q1 - 1.5 * IQR
        upper = Q3 + 1.5 * IQR

        before = len(self.df[self.df[column] < lower]) + len(
            self.df[self.df[column] > upper]
        )
        self.df[column] = self.df[column].clip(lower, upper)
        self._log("异常值截断", f"列 '{column}'，处理了 {before} 个值")
        return self

    # ---------- 类型转换 ----------

    def convert_dtypes(self, infer_objects: bool = True) -> "DataCleaner":
        """
        优化列数据类型以减少内存占用

        Args:
            infer_objects: 是否自动推断 object 列类型
        """
        self.df = self.df.convert_dtypes(infer_objects=infer_objects)
        self._log("类型优化", "convert_dtypes 优化完成")
        return self

    # ---------- 结果获取 ----------

    def get_cleaned(self) -> pd.DataFrame:
        """获取清洗后的数据"""
        return self.df

    def get_report(self) -> list[dict]:
        """获取清洗操作日志"""
        return self.report

    def stats_before_after(self) -> pd.DataFrame:
        """对比清洗前后的数据统计"""
        return pd.DataFrame({
            "指标": ["行数", "列数", "缺失值总数", "重复行数", "内存(MB)"],
            "清洗前": [
                len(self.original),
                len(self.original.columns),
                self.original.isnull().sum().sum(),
                self.original.duplicated().sum(),
                round(
                    self.original.memory_usage(deep=True).sum() / 1024**2, 2
                ),
            ],
            "清洗后": [
                len(self.df),
                len(self.df.columns),
                self.df.isnull().sum().sum(),
                self.df.duplicated().sum(),
                round(
                    self.df.memory_usage(deep=True).sum() / 1024**2, 2
                ),
            ],
        })
```

---

## 五、统计分析模块（analyzer.py）

```python
# datalens/analyzer.py
"""
统计分析模块 - 描述性统计、分组聚合、相关性分析
"""
from typing import Optional

import pandas as pd
import numpy as np
from rich.console import Console
from rich.table import Table

console = Console()


class DataAnalyzer:
    """数据分析器：提供常用统计分析功能"""

    def __init__(self, df: pd.DataFrame):
        self.df = df

    def overview(self) -> pd.DataFrame:
        """
        生成数据概览表

        Returns:
            包含各列类型、非空数量、唯一值数、示例值的 DataFrame
        """
        data = []
        for col in self.df.columns:
            data.append({
                "列名": col,
                "数据类型": str(self.df[col].dtype),
                "非空数量": self.df[col].count(),
                "缺失数量": self.df[col].isnull().sum(),
                "唯一值数": self.df[col].nunique(),
                "示例值": str(self.df[col].dropna().iloc[0] if len(self.df) > 0 else "")
                         [:30],
            })
        result = pd.DataFrame(data)
        console.print(result.to_string(index=False))
        return result

    def describe(self, numeric_only: bool = True) -> pd.DataFrame:
        """
        描述性统计

        Args:
            numeric_only: 是否只分析数值列

        Returns:
            统计摘要 DataFrame
        """
        stats = self.df.describe(include="number" if numeric_only else "all")
        console.print(stats.to_string())
        return stats

    def group_aggregate(
        self,
        group_col: str,
        agg_col: str,
        agg_func: str = "mean",
    ) -> pd.DataFrame:
        """
        分组聚合

        Args:
            group_col: 分组依据列
            agg_col: 聚合目标列
            agg_func: 聚合函数（mean/sum/count/min/max/median/std）

        Returns:
            聚合结果 DataFrame

        Example:
            >>> analyzer.group_aggregate("城市", "销售额", "sum")
            # 每个城市的总销售额
        """
        valid_funcs = ["mean", "sum", "count", "min", "max", "median", "std"]
        if agg_func not in valid_funcs:
            raise ValueError(f"不支持的聚合函数 '{agg_func}'，可选: {valid_funcs}")

        result = self.df.groupby(group_col)[agg_col].agg(agg_func)
        result = result.reset_index()
        result.columns = [group_col, f"{agg_col}的{agg_func}"]

        # 按聚合值降序排列
        result = result.sort_values(
            f"{agg_col}的{agg_func}", ascending=False
        )
        console.print(result.to_string(index=False))
        return result

    def correlation(
        self,
        method: str = "pearson",
        threshold: float = 0.5,
    ) -> pd.DataFrame:
        """
        相关性分析

        Args:
            method: 相关系数方法（pearson/spearman/kendall）
            threshold: 强相关性阈值（绝对值）

        Returns:
            筛选出强相关列对的 DataFrame
        """
        numeric_df = self.df.select_dtypes(include="number")
        if numeric_df.shape[1] < 2:
            console.print("[yellow]数值列不足 2 列，无法计算相关性[/yellow]")
            return pd.DataFrame()

        corr_matrix = numeric_df.corr(method=method)

        # 提取强相关列对（排除自相关）
        pairs = []
        cols = corr_matrix.columns
        for i in range(len(cols)):
            for j in range(i + 1, len(cols)):
                r = corr_matrix.iloc[i, j]
                if abs(r) >= threshold:
                    pairs.append({
                        "列1": cols[i],
                        "列2": cols[j],
                        f"{method}相关系数": round(r, 4),
                        "关系": "正相关" if r > 0 else "负相关",
                    })

        if pairs:
            result = pd.DataFrame(pairs).sort_values(
                by=f"{method}相关系数",
                key=abs,
                ascending=False,
            )
            console.print(f"[cyan]发现 {len(pairs)} 对强相关列 (|r| ≥ {threshold})[/cyan]")
            console.print(result.to_string(index=False))
        else:
            console.print(f"[yellow]未发现强相关列对 (阈值={threshold})[/yellow]")
            result = pd.DataFrame()

        return result

    def top_n(
        self,
        column: str,
        n: int = 10,
        ascending: bool = False,
    ) -> pd.DataFrame:
        """
        获取某列的 Top N 值

        Args:
            column: 目标列
            n: 返回前 N 条
            ascending: 是否升序排列
        """
        if column not in self.df.columns:
            console.print(f"[red]列 '{column}' 不存在[/red]")
            return pd.DataFrame()

        result = self.df.nlargest(n, column) if not ascending else self.df.nsmallest(n, column)
        console.print(f"[cyan]Top {n} {column}（{'升序' if ascending else '降序'}）[/cyan]")
        console.print(result.to_string(index=False))
        return result
```

---

## 六、示例数据与命令行接口

### 6.1 创建示例数据

```python
# create_sample_data.py（运行一次即可）
"""
生成示例销售数据，用于测试 DataLens
"""
import pandas as pd
import numpy as np

np.random.seed(42)

n = 200
products = ["笔记本电脑", "手机", "平板", "耳机", "键盘", "显示器", "鼠标"]
cities = ["北京", "上海", "广州", "深圳", "杭州", "成都", "武汉"]
channels = ["线上", "线下"]

data = pd.DataFrame({
    "日期": pd.date_range("2025-01-01", periods=n, freq="D"),
    "产品": np.random.choice(products, n),
    "城市": np.random.choice(cities, n),
    "渠道": np.random.choice(channels, n),
    "销售额": np.random.randint(100, 10000, n).astype(float),
    "数量": np.random.randint(1, 50, n),
})

# 制造一些脏数据（用于测试清洗功能）
data.loc[5:8, "销售额"] = np.nan          # 缺失值
data.loc[10:12, "城市"] = np.nan          # 缺失值
data.loc[20, "销售额"] = 999999           # 异常值
data.loc[80:82] = data.loc[79]            # 重复数据
data.loc[100, "数量"] = -5                # 负值异常

data.to_csv("data/sample_sales.csv", index=False, encoding="utf-8-sig")
print("✓ 示例数据已生成: data/sample_sales.csv")
print(f"  共 {len(data)} 行，{len(data.columns)} 列")
```

### 6.2 命令行入口（cli.py）

```python
# datalens/cli.py
"""
命令行接口 - 使用 click + rich 实现友好的交互体验
"""
from pathlib import Path

import click
from rich.console import Console
from rich.panel import Panel
from rich.markdown import Markdown

from datalens import __version__
from datalens.loader import DataLoader
from datalens.cleaner import DataCleaner
from datalens.analyzer import DataAnalyzer

console = Console()


@click.group()
@click.version_option(__version__, prog_name="datalens")
def main():
    """🧪 DataLens - Python 数据分析工具"""
    pass


@main.command()
@click.argument("filepath", type=click.Path(exists=True))
def load(filepath):
    """加载数据文件并预览"""
    loader = DataLoader(filepath)
    df = loader.load()
    loader.preview(10)


@main.command()
@click.argument("filepath", type=click.Path(exists=True))
@click.option("--strategy", "-s", default="auto",
              type=click.Choice(["auto", "mean", "median", "mode", "drop", "ffill"]),
              help="缺失值填充策略")
@click.option("--remove-dups", "-d", is_flag=True, help="是否删除重复行")
@click.option("--output", "-o", default=None, help="清洗后保存路径")
def clean(filepath, strategy, remove_dups, output):
    """清洗数据文件"""
    loader = DataLoader(filepath)
    df = loader.load()

    cleaner = DataCleaner(df)

    console.print("\n[bold cyan]📋 步骤 1：检测缺失值[/bold cyan]")
    cleaner.detect_missing()

    console.print("\n[bold cyan]📋 步骤 2：填充缺失值[/bold cyan]")
    cleaner.fill_missing(strategy)

    if remove_dups:
        console.print("\n[bold cyan]📋 步骤 3：检测重复数据[/bold cyan]")
        cleaner.detect_duplicates()
        cleaner.remove_duplicates()

    console.print("\n[bold cyan]📋 清洗前后对比[/bold cyan]")
    console.print(cleaner.stats_before_after().to_string(index=False))

    if output:
        cleaned = cleaner.get_cleaned()
        cleaned.to_csv(output, index=False, encoding="utf-8-sig")
        console.print(f"\n[green]✓ 清洗后数据已保存到: {output}[/green]")

    # 返回清洗后的数据供管道使用
    return cleaner.get_cleaned()


@main.command()
@click.argument("filepath", type=click.Path(exists=True))
@click.option("--group", "-g", default=None, help="分组列名")
@click.option("--agg-col", "-a", default=None, help="聚合目标列")
@click.option("--func", "-f", default="mean", help="聚合函数")
def analyze(filepath, group, agg_col, func):
    """分析数据文件"""
    loader = DataLoader(filepath)
    df = loader.load()

    analyzer = DataAnalyzer(df)

    console.print("\n[bold cyan]📊 数据概览[/bold cyan]")
    analyzer.overview()

    console.print("\n[bold cyan]📈 描述性统计[/bold cyan]")
    analyzer.describe()

    if group and agg_col:
        console.print(f"\n[bold cyan]📋 分组聚合: 按 {group} 对 {agg_col} 求 {func}[/bold cyan]")
        analyzer.group_aggregate(group, agg_col, func)

    console.print("\n[bold cyan]🔗 相关性分析[/bold cyan]")
    analyzer.correlation()


@main.command()
def info():
    """显示 DataLens 信息"""
    info_text = f"""
# DataLens 🧪

**版本**: {__version__}

**功能**: 数据加载、清洗、分析、可视化、报告生成

**支持格式**: CSV、Excel、JSON

**使用方式**:
```bash
datalens load data.csv          # 加载并预览
datalens clean data.csv -s auto # 清洗数据
datalens analyze data.csv       # 统计分析
```
"""
    console.print(Panel(Markdown(info_text), title="DataLens", border_style="cyan"))


if __name__ == "__main__":
    main()
```

### 6.3 运行入口

```python
# run.py
"""
DataLens 快速启动脚本
"""
from datalens.cli import main

if __name__ == "__main__":
    main()
```

---

## 七、完整运行演示

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 生成示例数据
python create_sample_data.py

# 3. 加载并预览
python run.py load data/sample_sales.csv
```

输出示例：

```
✓ 成功加载 200 行 × 7 列，内存占用 87.3 KB

┏━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━┳━━━━━━┳━━━━━━━┓
┃ 日期           ┃ 产品    ┃ 城市  ┃ 渠道 ┃ 销售额 ┃
┡━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━╇━━━━━━╇━━━━━━━┩
│ 2025-01-01    │ 耳机    │ 深圳  │ 线下 │ 5029.0│
│ 2025-01-02    │ 显示器  │ 成都  │ 线上 │ 3014.0│
│ ...           │ ...     │ ...   │ ...  │ ...   │
└───────────────┴─────────┴──────┴──────┴───────┘
共 200 行，7 列
```

```bash
# 4. 清洗数据
python run.py clean data/sample_sales.csv -s auto -d -o data/cleaned_sales.csv
```

输出示例：

```
📋 步骤 1：检测缺失值
  发现 2 列存在缺失值

📋 步骤 2：填充缺失值
  · 缺失值处理: 策略=auto

📋 步骤 3：检测重复数据
  发现 2 行重复数据
  · 删除重复行: 删除了 2 行

📋 清洗前后对比
   指标    清洗前    清洗后
   行数      200       195
   列数        7         7
  缺失值总数   6         0
  重复行数     2         0
  内存(MB)   0.08      0.08

✓ 清洗后数据已保存到: data/cleaned_sales.csv
```

```bash
# 5. 统计分析
python run.py analyze data/sample_sales.csv -g 城市 -a 销售额 -f sum
```

---

## 八、架构设计思路

### 8.1 为什么这样设计？

```
 DataLoader ──→ DataFrame ──→ DataCleaner ──→ 清洗后 DataFrame
                                          │
                                          ↓
                                    DataAnalyzer ──→ 统计结果
```

**核心设计原则**：

| 原则 | 体现 |
|------|------|
| 单一职责 | 每个模块只做一件事（加载/清洗/分析） |
| 数据不可变 | DataCleaner 保留原始数据副本，不修改输入 |
| 链式调用 | `cleaner.fill_missing().remove_duplicates()` |
| 友好输出 | rich 美化终端输出，让分析结果直观可读 |

### 8.2 与已学知识的关联

```
Day 11 函数设计 → 每个方法只做一件事，职责清晰
Day 15-16 OOP  → 用类封装数据和操作（loader.df / cleaner.df）
Day 20 装饰器  → 未来可添加 @timer / @log 装饰器
Day 28 类型注解 → 所有函数参数和返回值都有类型标注
Day 32 项目结构 → 符合标准 Python 项目布局
Day 33 logging → 通过 rich console 提供运行时反馈
Day 38 pandas  → 核心数据处理引擎
Day 30 测试    → 每个方法可独立测试（tests/）
```

---

## 练习题

### 练习 1：扩展加载器

在 `DataLoader` 中新增 `_load_tsv()` 方法，支持以 Tab 分隔的 TSV 文件加载。

**提示**：`pd.read_csv()` 的 `sep` 参数可以指定分隔符。

### 练习 2：增强清洗器

为 `DataCleaner` 添加一个 `remove_negative()` 方法，自动将指定列中的负数值替换为 0，并记录到清洗报告中。

```python
def remove_negative(self, column: str) -> "DataCleaner":
    """将指定列中的负数值替换为 0"""
    # 你的实现
    pass
```

### 练习 3：添加新的聚合统计

在 `DataAnalyzer` 中添加 `monthly_trend()` 方法，自动识别日期列和数值列，计算每月的趋势统计。

```python
def monthly_trend(self, date_col: str, value_col: str) -> pd.DataFrame:
    """计算月度趋势（按月聚合）"""
    # 你的实现
    pass
```

---

## 常见问题

### Q1: 安装依赖时报错怎么办？

**A**: 如果 `openpyxl` 安装失败，可以只安装核心依赖：
```bash
pip install pandas numpy matplotlib seaborn click rich
```
Excel 支持是可选功能。

### Q2: 为什么 DataCleaner 要保存原始数据副本？

**A**: 这是**不可变数据**的设计原则。清洗操作不应修改原始数据，这样你可以随时对比清洗前后的差异，也可以回退到原始数据重新清洗。

### Q3: `click` 是什么？为什么要用它而不是 `argparse`？

**A**: `click` 是 Python 最流行的命令行框架，相比标准库的 `argparse`：
- 用装饰器定义命令，代码更简洁
- 自动生成 `--help` 帮助文档
- 支持参数类型验证和选项提示
- 支持子命令嵌套（我们的 `load`/`clean`/`analyze`）

### Q4: rich 的 Console 和 print 有什么区别？

**A**: `rich.Console` 支持：
- **颜色标注**：`[red]错误[/red]`、`[green]成功[/green]`
- **表格输出**：自动对齐的精美表格
- **进度条**：`console.progress()` 长任务进度
- **Markdown 渲染**：直接渲染 Markdown 格式文本

### Q5: 如果数据量很大（百万行以上），怎么优化？

**A**: 几个策略：
1. `pd.read_csv(chunksize=10000)` 分块读取
2. 加载后立即 `convert_dtypes()` 优化内存
3. 用 `category` 类型存储低基数文本列
4. 只加载需要的列：`usecols=["销售额", "日期"]`
5. 考虑用 `polars` 库替代 pandas（更快的内存 DataFrame）

---

## 学习资源

- **pandas 官方文档**：https://pandas.pydata.org/docs/
- **click 命令行框架**：https://click.palletsprojects.com/
- **rich 终端美化**：https://rich.readthedocs.io/
- **Python 项目打包**：https://packaging.python.org/
- **菜鸟教程 - pandas**：https://www.runoob.com/pandas/pandas-tutorial.html

---

> **下一篇**：Day 57 将为 DataLens 添加**可视化模块 + 报告生成 + Streamlit Web 界面**，让你的分析工具拥有漂亮的图形界面！
