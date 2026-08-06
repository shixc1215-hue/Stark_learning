# Python Day 57 - 项目实战：数据分析工具（二）

> 昨天我们完成了 DataLens 的核心引擎（数据加载、清洗、分析、命令行接口），今天继续完成下半部分：**可视化模块 + 报告生成 + Streamlit Web 界面**。完成后你将拥有一个可以直接在浏览器中操作的完整数据分析工具！

---

## 一、可视化模块（visualizer.py）

可视化是数据分析的"最后一公里"——把枯燥的数字变成直观的图表。这个模块封装了 matplotlib 和 seaborn，让图表生成变得像调用函数一样简单。

```python
# datalens/visualizer.py
"""
可视化模块 - 自动生成分析图表
"""
from pathlib import Path
from typing import Optional

import pandas as pd
import numpy as np
import matplotlib
matplotlib.use("Agg")  # 无头模式（不弹出窗口，适合服务器/Web）
import matplotlib.pyplot as plt
import seaborn as sns

# 中文字体配置（解决中文显示为方块的问题）
plt.rcParams["font.sans-serif"] = ["SimHei", "Microsoft YaHei", "Arial"]
plt.rcParams["axes.unicode_minus"] = False  # 负号正常显示

# 全局 seaborn 主题
sns.set_theme(style="whitegrid", font_scale=1.1)


class DataVisualizer:
    """数据可视化器：一键生成常用分析图表"""

    # 预设配色方案
    PALETTES = {
        "default": None,
        "blue": "Blues_d",
        "warm": "OrRd",
        "cool": "coolwarm",
        "viridis": "viridis",
        "dark": "dark",
    }

    def __init__(
        self,
        df: pd.DataFrame,
        output_dir: str | Path = "output",
        dpi: int = 150,
    ):
        """
        初始化可视化器

        Args:
            df: 待可视化的 DataFrame
            output_dir: 图表保存目录（默认 output/）
            dpi: 图片分辨率（默认 150）
        """
        self.df = df
        self.output_dir = Path(output_dir)
        self.dpi = dpi
        self.generated: list[str] = []  # 记录已生成的图表路径

        # 自动创建输出目录
        self.output_dir.mkdir(parents=True, exist_ok=True)

    def _save_and_close(
        self,
        fig,
        filename: str,
    ) -> str:
        """
        保存图表并关闭画布

        Args:
            fig: matplotlib Figure 对象
            filename: 文件名（不含目录）

        Returns:
            保存后的完整路径
        """
        filepath = self.output_dir / filename
        fig.savefig(filepath, dpi=self.dpi, bbox_inches="tight")
        plt.close(fig)
        self.generated.append(str(filepath))
        return str(filepath)

    # ---------- 趋势图 ----------

    def trend_line(
        self,
        date_col: str,
        value_col: str,
        title: Optional[str] = None,
        filename: str = "trend_line.png",
    ) -> str:
        """
        绘制时间趋势折线图

        Args:
            date_col: 日期列名
            value_col: 数值列名
            title: 图表标题（默认自动生成）
            filename: 保存文件名

        Returns:
            图表文件路径
        """
        # 确保日期列是 datetime 类型
        data = self.df.copy()
        data[date_col] = pd.to_datetime(data[date_col])

        # 按日期排序
        data = data.sort_values(date_col)

        fig, ax = plt.subplots(figsize=(12, 5))
        ax.plot(data[date_col], data[value_col],
                color="#4C72B0", linewidth=2, alpha=0.8)

        # 添加移动平均线
        if len(data) >= 7:
            ma = data[value_col].rolling(window=7, min_periods=1).mean()
            ax.plot(data[date_col], ma,
                    color="#DD8452", linewidth=1.5,
                    linestyle="--", label="7日移动平均")
            ax.legend()

        ax.set_title(title or f"{value_col} 随时间变化趋势")
        ax.set_xlabel(date_col)
        ax.set_ylabel(value_col)
        ax.tick_params(axis="x", rotation=45)

        return self._save_and_close(fig, filename)

    # ---------- 分布图 ----------

    def distribution(
        self,
        column: str,
        kind: str = "hist",
        filename: str = "distribution.png",
    ) -> str:
        """
        绘制数值分布图

        Args:
            column: 目标数值列
            kind: 图表类型
                - "hist": 直方图（默认）
                - "box": 箱线图
                - "violin": 小提琴图
                - "kde": 核密度估计图
            filename: 保存文件名

        Returns:
            图表文件路径
        """
        fig, ax = plt.subplots(figsize=(10, 5))

        plot_funcs = {
            "hist": lambda: sns.histplot(
                self.df[column], kde=True, ax=ax, color="#4C72B0"
            ),
            "box": lambda: sns.boxplot(
                y=self.df[column], ax=ax, color="#4C72B0"
            ),
            "violin": lambda: sns.violinplot(
                y=self.df[column], ax=ax, color="#4C72B0"
            ),
            "kde": lambda: sns.kdeplot(
                self.df[column], ax=ax, fill=True, color="#4C72B0"
            ),
        }

        if kind not in plot_funcs:
            raise ValueError(
                f"不支持的图表类型 '{kind}'，可选: {list(plot_funcs.keys())}"
            )

        plot_funcs[kind]()

        kind_names = {
            "hist": "直方图", "box": "箱线图",
            "violin": "小提琴图", "kde": "核密度图",
        }
        ax.set_title(f"{column} 的{kind_names[kind]}")

        return self._save_and_close(fig, filename)

    # ---------- 柱状图 ----------

    def bar_chart(
        self,
        category_col: str,
        value_col: str,
        agg: str = "sum",
        top_n: int = 15,
        horizontal: bool = False,
        filename: str = "bar_chart.png",
    ) -> str:
        """
        绘制分类柱状图（自动聚合后排序）

        Args:
            category_col: 分类列名
            value_col: 数值列名
            agg: 聚合方式（sum/mean/count）
            top_n: 显示前 N 个类别
            horizontal: 是否水平柱状图
            filename: 保存文件名

        Returns:
            图表文件路径
        """
        # 分组聚合
        grouped = (
            self.df.groupby(category_col)[value_col]
            .agg(agg)
            .sort_values(ascending=False)
            .head(top_n)
        )

        fig, ax = plt.subplots(figsize=(10, max(5, top_n * 0.4)))
        colors = sns.color_palette("Blues_d", n_colors=len(grouped))

        if horizontal:
            # 水平柱状图（类别多时更适合）
            grouped = grouped.sort_values()  # 升序排列，最长的在上面
            bars = ax.barh(grouped.index, grouped.values, color=colors)
            ax.set_xlabel(f"{value_col}（{agg}）")
            ax.set_ylabel(category_col)
        else:
            bars = ax.bar(grouped.index, grouped.values, color=colors)
            ax.set_ylabel(f"{value_col}（{agg}）")
            ax.set_xlabel(category_col)
            ax.tick_params(axis="x", rotation=45)

        # 在柱子上标注数值
        for bar in bars:
            width = bar.get_width() if horizontal else bar.get_height()
            if horizontal:
                ax.text(width + width * 0.01, bar.get_y() + bar.get_height() / 2,
                        f"{width:,.0f}", va="center", fontsize=9)
            else:
                ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height(),
                        f"{bar.get_height():,.0f}", ha="center", va="bottom", fontsize=8)

        agg_names = {"sum": "总计", "mean": "平均值", "count": "计数"}
        ax.set_title(f"各{category_col}的{value_col}{agg_names.get(agg, agg)}（Top {top_n}）")

        return self._save_and_close(fig, filename)

    # ---------- 相关性热力图 ----------

    def correlation_heatmap(
        self,
        method: str = "pearson",
        filename: str = "correlation_heatmap.png",
    ) -> str:
        """
        绘制相关性矩阵热力图

        Args:
            method: 相关系数方法（pearson/spearman/kendall）
            filename: 保存文件名

        Returns:
            图表文件路径
        """
        numeric_df = self.df.select_dtypes(include="number")
        if numeric_df.shape[1] < 2:
            raise ValueError("数值列不足 2 列，无法绘制相关性图")

        corr = numeric_df.corr(method=method)

        fig, ax = plt.subplots(figsize=(max(8, len(numeric_df.columns)), max(6, len(numeric_df.columns) * 0.6)))
        mask = np.triu(np.ones_like(corr, dtype=bool))  # 只显示下三角
        sns.heatmap(
            corr, mask=mask, annot=True, fmt=".2f",
            cmap="coolwarm", center=0,
            square=True, linewidths=0.5,
            ax=ax, vmin=-1, vmax=1,
        )
        ax.set_title(f"相关性矩阵（{method}）")

        return self._save_and_close(fig, filename)

    # ---------- 散点图 ----------

    def scatter_plot(
        self,
        x_col: str,
        y_col: str,
        hue: Optional[str] = None,
        filename: str = "scatter.png",
    ) -> str:
        """
        绘制散点图（支持按类别着色）

        Args:
            x_col: X 轴列名
            y_col: Y 轴列名
            hue: 分类着色列（可选）
            filename: 保存文件名

        Returns:
            图表文件路径
        """
        fig, ax = plt.subplots(figsize=(10, 6))
        sns.scatterplot(
            data=self.df, x=x_col, y=y_col,
            hue=hue, ax=ax, alpha=0.7, s=60,
        )
        ax.set_title(f"{x_col} vs {y_col}")

        return self._save_and_close(fig, filename)

    # ---------- 饼图 ----------

    def pie_chart(
        self,
        column: str,
        top_n: int = 8,
        filename: str = "pie_chart.png",
    ) -> str:
        """
        绘制分类占比饼图

        Args:
            column: 分类列名
            top_n: 显示前 N 个类别（其余合并为"其他"）
            filename: 保存文件名
        """
        counts = self.df[column].value_counts()

        # 超过 top_n 则合并尾部为"其他"
        if len(counts) > top_n:
            top = counts.head(top_n)
            other = pd.Series({"其他": counts.iloc[top_n:].sum()})
            counts = pd.concat([top, other])

        fig, ax = plt.subplots(figsize=(8, 8))
        colors = sns.color_palette("Set2", n_colors=len(counts))

        wedges, texts, autotexts = ax.pie(
            counts.values,
            labels=counts.index,
            autopct="%1.1f%%",
            colors=colors,
            startangle=90,
            pctdistance=0.75,
        )
        # 设置百分比文字样式
        for autotext in autotexts:
            autotext.set_fontsize(10)
            autotext.set_color("white")

        ax.set_title(f"{column} 的占比分布")

        return self._save_and_close(fig, filename)

    # ---------- 仪表盘（一键多图） ----------

    def generate_dashboard(
        self,
        date_col: Optional[str] = None,
        value_col: Optional[str] = None,
        category_col: Optional[str] = None,
    ) -> dict[str, str]:
        """
        一键生成分析仪表盘（多张图表）

        Args:
            date_col: 日期列名（用于趋势图）
            value_col: 数值列名（用于趋势图/分布图）
            category_col: 分类列名（用于柱状图/饼图）

        Returns:
            {"图表名": "文件路径"} 字典
        """
        results = {}
        numeric_cols = self.df.select_dtypes(include="number").columns.tolist()
        non_numeric_cols = self.df.select_dtypes(exclude="number").columns.tolist()

        # 自动推断列（如果用户未指定）
        if not value_col and numeric_cols:
            value_col = numeric_cols[0]
        if not category_col and non_numeric_cols:
            category_col = non_numeric_cols[0]
        if not date_col and non_numeric_cols:
            # 尝试识别日期列
            for col in non_numeric_cols:
                try:
                    pd.to_datetime(self.df[col].head(10))
                    date_col = col
                    break
                except (ValueError, TypeError):
                    continue

        # 1. 数值分布图
        if value_col:
            results[f"{value_col}_分布"] = self.distribution(
                value_col, "hist", f"{value_col}_distribution.png"
            )

        # 2. 趋势图
        if date_col and value_col:
            results["趋势图"] = self.trend_line(
                date_col, value_col, filename="trend.png"
            )

        # 3. 分类柱状图
        if category_col and value_col:
            results[f"按{category_col}分组"] = self.bar_chart(
                category_col, value_col, filename="group_bar.png"
            )

        # 4. 饼图
        if category_col:
            results[f"{category_col}_占比"] = self.pie_chart(
                category_col, filename="pie.png"
            )

        # 5. 相关性热力图（数值列 >= 2 时）
        if len(numeric_cols) >= 2:
            results["相关性矩阵"] = self.correlation_heatmap(
                filename="heatmap.png"
            )

        # 6. 散点图（前两个数值列）
        if len(numeric_cols) >= 2:
            results["散点图"] = self.scatter_plot(
                numeric_cols[0], numeric_cols[1],
                filename="scatter.png"
            )

        return results
```

### 1.1 可视化模块设计要点

| 设计决策 | 原因 |
|---------|------|
| `matplotlib.use("Agg")` | 无头模式——不弹窗，适合 Web 服务和服务器环境 |
| 每张图独立保存后关闭 | 避免 matplotlib 内存泄漏（不关闭 Figure 会累积在内存中） |
| 中文字体配置 | 默认 matplotlib 不支持中文，需手动指定 SimHei/Microsoft YaHei |
| `savefig + plt.close` | "生成即丢弃"模式——图存到文件，不留在内存 |
| 仪表盘自动推断列 | 用户只需传入 DataFrame，工具自动识别日期列/数值列/分类列 |

---

## 二、报告生成模块（reporter.py）

分析做完后，需要一份结构化的报告来总结发现。我们用 Markdown 格式生成报告，方便阅读和分享。

```python
# datalens/reporter.py
"""
报告生成模块 - 将分析结果整理为 Markdown 报告
"""
from datetime import datetime
from pathlib import Path
from typing import Optional

import pandas as pd


class DataReporter:
    """数据分析报告生成器"""

    def __init__(self, title: str = "数据分析报告"):
        """
        初始化报告生成器

        Args:
            title: 报告标题
        """
        self.title = title
        self.sections: list[str] = []

    def add_title(self, title: str, level: int = 1) -> "DataReporter":
        """
        添加标题

        Args:
            title: 标题文本
            level: 标题级别（1-4）
        """
        prefix = "#" * level
        self.sections.append(f"\n{prefix} {title}\n")
        return self

    def add_text(self, text: str) -> "DataReporter":
        """添加正文段落"""
        self.sections.append(f"\n{text}\n")
        return self

    def add_table(self, df: pd.DataFrame, max_rows: int = 20) -> "DataReporter":
        """
        添加 DataFrame 表格

        Args:
            df: 数据
            max_rows: 最大显示行数
        """
        display_df = df.head(max_rows).copy()
        # 转为字符串避免 Markdown 格式问题
        for col in display_df.columns:
            display_df[col] = display_df[col].apply(
                lambda x: str(round(x, 2)) if isinstance(x, float) else str(x)
            )
        markdown_table = display_df.to_markdown(index=False)
        self.sections.append(f"\n{markdown_table}\n")
        return self

    def add_code(self, code: str, language: str = "python") -> "DataReporter":
        """添加代码块"""
        self.sections.append(f"\n```{language}\n{code}\n```\n")
        return self

    def add_bullet(self, items: list[str]) -> "DataReporter":
        """添加无序列表"""
        bullets = "\n".join(f"- {item}" for item in items)
        self.sections.append(f"\n{bullets}\n")
        return self

    def add_divider(self) -> "DataReporter":
        """添加分割线"""
        self.sections.append("\n---\n")
        return self

    def add_image(self, image_path: str, alt_text: str = "") -> "DataReporter":
        """
        添加图片引用（相对路径）

        Args:
            image_path: 图片路径
            alt_text: 替代文本（鼠标悬停提示）
        """
        self.sections.append(
            f"\n![{alt_text or image_path}]({image_path})\n"
        )
        return self

    def generate_report(
        self,
        output_path: str | Path = "report.md",
    ) -> str:
        """
        生成完整 Markdown 报告

        Args:
            output_path: 输出文件路径

        Returns:
            报告文件路径
        """
        output_path = Path(output_path)

        # 组装报告内容
        now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        content = f"# {self.title}\n\n"
        content += f"> 生成时间：{now}\n\n"
        content += "".join(self.sections)

        # 保存文件
        output_path.write_text(content, encoding="utf-8")

        return str(output_path)

    @staticmethod
    def quick_report(
        df: pd.DataFrame,
        title: str = "数据分析报告",
        output_path: str = "report.md",
    ) -> str:
        """
        一键生成基础报告（自动包含概览 + 统计 + 数据预览）

        Args:
            df: 数据
            title: 报告标题
            output_path: 输出路径

        Returns:
            报告文件路径
        """
        reporter = DataReporter(title)

        # 数据概览
        reporter.add_title("数据概览", 2)
        reporter.add_text(
            f"- 数据形状：{df.shape[0]} 行 × {df.shape[1]} 列\n"
            f"- 内存占用：{df.memory_usage(deep=True).sum() / 1024:.1f} KB\n"
            f"- 列信息：{', '.join(df.columns)}"
        )

        # 列类型表
        reporter.add_title("列信息", 2)
        col_info = pd.DataFrame({
            "列名": df.columns,
            "类型": [str(df[c].dtype) for c in df.columns],
            "非空": [df[c].count() for c in df.columns],
            "缺失": [df[c].isnull().sum() for c in df.columns],
            "唯一值": [df[c].nunique() for c in df.columns],
        })
        reporter.add_table(col_info)

        # 描述性统计
        reporter.add_title("描述性统计", 2)
        numeric_df = df.select_dtypes(include="number")
        if not numeric_df.empty:
            reporter.add_table(numeric_df.describe().reset_index())
        else:
            reporter.add_text("（无数值列）")

        # 数据预览
        reporter.add_title("数据预览（前 10 行）", 2)
        reporter.add_table(df.head(10))

        return reporter.generate_report(output_path)
```

### 2.1 报告模块设计要点

| 设计决策 | 原因 |
|---------|------|
| **链式调用** | `reporter.add_title().add_text().add_table()` 代码流畅 |
| **Markdown 格式** | 通用性强——GitHub/GitLab/VS Code/飞书都能渲染 |
| **quick_report 快捷方法** | 无需手动拼装，一行代码生成基础报告 |
| **表格截断** | `max_rows=20` 防止大数据集生成超长报告 |

---

## 三、Streamlit Web 界面

有了命令行工具后，我们再做一个浏览器界面，让非技术用户也能使用。这是整个项目的"点睛之笔"。

```python
# datalens/app.py
"""
Streamlit Web 界面 - 浏览器中的数据分析工具
"""
from pathlib import Path
from io import StringIO

import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 页面配置
st.set_page_config(
    page_title="DataLens 数据分析工具",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded",
)

# 中文字体
plt.rcParams["font.sans-serif"] = ["SimHei", "Microsoft YaHei"]
plt.rcParams["axes.unicode_minus"] = False

# 自定义 CSS 美化
st.markdown("""
<style>
    .block-container { padding-top: 2rem; }
    .stMetric { background: #f0f4ff; border-radius: 10px; padding: 1rem; }
    h1 { color: #1a73e8; }
    h2 { color: #34a853; border-bottom: 2px solid #e8f0fe; padding-bottom: 0.5rem; }
</style>
""", unsafe_allow_html=True)


# ---------- 会话状态初始化 ----------
def init_session_state():
    """初始化 Streamlit session_state"""
    if "df" not in st.session_state:
        st.session_state.df = None
        st.session_state.original_df = None
        st.session_state.clean_report = []
        st.session_state.uploaded_file = None


init_session_state()


# ---------- 侧边栏：数据加载 ----------
with st.sidebar:
    st.header("📁 数据加载")

    # 文件上传
    uploaded = st.file_uploader(
        "上传数据文件",
        type=["csv", "xlsx", "json"],
        help="支持 CSV、Excel、JSON 格式",
    )

    if uploaded:
        st.session_state.uploaded_file = uploaded
        try:
            if uploaded.name.endswith(".csv"):
                df = pd.read_csv StringIO(uploaded.read().decode("utf-8-sig")))
            elif uploaded.name.endswith(".json"):
                df = pd.read_json StringIO(uploaded.read().decode("utf-8")))
            else:
                df = pd.read_excel(uploaded)

            st.session_state.df = df
            st.session_state.original_df = df.copy()
            st.success(f"✓ 加载成功：{df.shape[0]} 行 × {df.shape[1]} 列")
        except Exception as e:
            st.error(f"加载失败：{e}")

    st.divider()

    # 或者输入示例 URL / 文件路径
    st.subheader("🔄 其他选项")
    if st.button("加载示例数据"):
        # 内置生成示例数据
        import numpy as np
        np.random.seed(42)
        n = 200
        df = pd.DataFrame({
            "日期": pd.date_range("2025-01-01", periods=n, freq="D"),
            "产品": np.random.choice(
                ["笔记本电脑", "手机", "平板", "耳机", "键盘"], n
            ),
            "城市": np.random.choice(
                ["北京", "上海", "广州", "深圳", "杭州", "成都"], n
            ),
            "渠道": np.random.choice(["线上", "线下"], n),
            "销售额": np.random.randint(100, 10000, n).astype(float),
            "数量": np.random.randint(1, 50, n),
        })
        st.session_state.df = df
        st.session_state.original_df = df.copy()
        st.rerun()


# ---------- 主区域 ----------
st.title("📊 DataLens 数据分析工具")

if st.session_state.df is None:
    # 未加载数据时的欢迎页
    st.markdown("""
    ### 欢迎使用 DataLens！

    通过左侧面板上传数据文件，即可开始分析。

    **支持功能**：
    - 📋 数据预览与统计
    - 🧹 数据清洗（缺失值/重复值/异常值）
    - 📈 统计分析（描述统计/分组聚合/相关性）
    - 🎨 可视化图表
    - 📥 下载清洗后的数据
    """)
    st.stop()

df = st.session_state.df

# ---------- Tab 1: 数据预览 ----------
tab1, tab2, tab3, tab4, tab5 = st.tabs([
    "📋 预览", "🧹 清洗", "📈 分析", "🎨 可视化", "📥 导出"
])

with tab1:
    st.header("数据预览")
    st.write(f"当前数据：**{df.shape[0]}** 行 × **{df.shape[1]}** 列")

    # 指标卡片
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric("总行数", f"{df.shape[0]:,}")
    with col2:
        st.metric("总列数", df.shape[1])
    with col3:
        st.metric("缺失值", int(df.isnull().sum().sum()))
    with col4:
        st.metric("重复行", int(df.duplicated().sum()))

    # 数据表格
    st.subheader("数据内容")
    st.dataframe(df, use_container_width=True, height=400)

    # 列信息
    with st.expander("📋 列详情"):
        col_info = pd.DataFrame({
            "列名": df.columns,
            "类型": [str(df[c].dtype) for c in df.columns],
            "非空": [df[c].count() for c in df.columns],
            "缺失": [df[c].isnull().sum() for c in df.columns],
            "唯一值": [df[c].nunique() for c in df.columns],
        })
        st.dataframe(col_info, use_container_width=True)


# ---------- Tab 2: 数据清洗 ----------
with tab2:
    st.header("数据清洗")

    # 缺失值处理
    st.subheader("1️⃣ 缺失值处理")
    missing = df.isnull().sum()
    missing_cols = missing[missing > 0]
    if missing_cols.empty:
        st.success("✓ 无缺失值")
    else:
        st.warning(f"发现 {len(missing_cols)} 列存在缺失值")
        st.dataframe(
            pd.DataFrame({
                "缺失数量": missing_cols,
                "缺失比例": (missing_cols / len(df) * 100).round(1).astype(str) + "%"
            }),
            use_container_width=True,
        )
        fill_strategy = st.selectbox(
            "填充策略", ["auto", "mean", "median", "mode", "drop", "ffill"],
            format_func=lambda x: {
                "auto": "自动判断", "mean": "均值", "median": "中位数",
                "mode": "众数", "drop": "删除行", "ffill": "前向填充",
            }.get(x, x),
        )
        if st.button("执行缺失值填充"):
            df_clean = st.session_state.df.copy()
            for col in df_clean.columns:
                if df_clean[col].isnull().sum() == 0:
                    continue
                if fill_strategy == "auto":
                    if pd.api.types.is_numeric_dtype(df_clean[col]):
                        df_clean[col].fillna(df_clean[col].median(), inplace=True)
                    else:
                        mode = df_clean[col].mode()
                        if not mode.empty:
                            df_clean[col].fillna(mode.iloc[0], inplace=True)
                elif fill_strategy == "drop":
                    df_clean.dropna(inplace=True)
                elif fill_strategy == "ffill":
                    df_clean.fillna(method="ffill", inplace=True)
                else:
                    numeric_cols = df_clean.select_dtypes(include="number").columns
                    if col in numeric_cols:
                        fill_val = getattr(df_clean[col], fill_strategy)()
                        df_clean[col].fillna(fill_val, inplace=True)
            df_clean.reset_index(drop=True, inplace=True)
            st.session_state.df = df_clean
            st.success(f"✓ 清洗完成，剩余 {len(df_clean)} 行")
            st.rerun()

    st.divider()

    # 重复值处理
    st.subheader("2️⃣ 重复值处理")
    dup_count = df.duplicated().sum()
    if dup_count == 0:
        st.success("✓ 无重复行")
    else:
        st.warning(f"发现 {dup_count} 行重复数据")
        if st.button("删除重复行"):
            df_clean = st.session_state.df.drop_duplicates()
            df_clean.reset_index(drop=True, inplace=True)
            st.session_state.df = df_clean
            st.success(f"✓ 删除了 {dup_count} 行重复数据")
            st.rerun()

    st.divider()

    # 异常值处理
    st.subheader("3️⃣ 异常值检测")
    numeric_cols = df.select_dtypes(include="number").columns.tolist()
    if numeric_cols:
        outlier_col = st.selectbox("选择检测列", numeric_cols)
        method = st.radio("检测方法", ["IQR（四分位距）", "Z-Score"])
        if st.button("检测异常值"):
            col_data = df[outlier_col]
            if method.startswith("IQR"):
                Q1, Q3 = col_data.quantile(0.25), col_data.quantile(0.75)
                IQR = Q3 - Q1
                lower, upper = Q1 - 1.5 * IQR, Q3 + 1.5 * IQR
                outliers = df[(col_data < lower) | (col_data > upper)]
                st.info(f"IQR 范围：[{lower:.2f}, {upper:.2f}]")
            else:
                z = abs((col_data - col_data.mean()) / col_data.std())
                outliers = df[z > 3]
                st.info("Z-Score 阈值：3")
            st.write(f"发现 **{len(outliers)}** 个异常值")
            if not outliers.empty:
                st.dataframe(outliers, use_container_width=True)

    st.divider()

    # 重置按钮
    if st.button("🔄 恢复原始数据"):
        st.session_state.df = st.session_state.original_df.copy()
        st.rerun()


# ---------- Tab 3: 统计分析 ----------
with tab3:
    st.header("统计分析")

    # 描述性统计
    with st.expander("📈 描述性统计", expanded=True):
        numeric_df = df.select_dtypes(include="number")
        if not numeric_df.empty:
            st.dataframe(numeric_df.describe(), use_container_width=True)
        else:
            st.info("无数值列")

    # 分组聚合
    with st.expander("📊 分组聚合"):
        col1, col2 = st.columns(2)
        with col1:
            group_col = st.selectbox("分组列", df.columns, key="group")
        with col2:
            agg_col = st.selectbox(
                "聚合列",
                df.select_dtypes(include="number").columns,
                key="agg",
            )
        agg_func = st.selectbox(
            "聚合方式",
            ["sum", "mean", "count", "min", "max", "median", "std"],
            key="func",
        )
        if st.button("执行分组聚合"):
            result = (
                df.groupby(group_col)[agg_col]
                .agg(agg_func)
                .sort_values(ascending=False)
                .reset_index()
            )
            result.columns = [group_col, f"{agg_col}（{agg_func}）"]
            st.dataframe(result, use_container_width=True)

            # 同时显示柱状图
            fig, ax = plt.subplots(figsize=(10, 4))
            sns.barplot(
                data=result, x=group_col,
                y=f"{agg_col}（{agg_func}）",
                ax=ax, palette="Blues_d",
            )
            ax.set_title(f"按{group_col}分组的{agg_col}{agg_func}")
            ax.tick_params(axis="x", rotation=45)
            st.pyplot(fig)
            plt.close(fig)

    # 相关性分析
    with st.expander("🔗 相关性分析"):
        if len(numeric_df.columns) >= 2:
            corr = numeric_df.corr()
            fig, ax = plt.subplots(figsize=(8, 6))
            sns.heatmap(
                corr, annot=True, fmt=".2f",
                cmap="coolwarm", center=0, ax=ax,
            )
            ax.set_title("相关性矩阵")
            st.pyplot(fig)
            plt.close(fig)
        else:
            st.info("数值列不足 2 列")


# ---------- Tab 4: 可视化 ----------
with tab4:
    st.header("数据可视化")

    chart_type = st.selectbox(
        "选择图表类型",
        ["折线图", "柱状图", "饼图", "散点图", "分布图", "箱线图"],
    )

    if chart_type == "折线图":
        col1, col2 = st.columns(2)
        with col1:
            x_col = st.selectbox("X 轴（日期/序号）", df.columns, key="line_x")
        with col2:
            y_col = st.selectbox(
                "Y 轴（数值）",
                df.select_dtypes(include="number").columns,
                key="line_y",
            )
        fig, ax = plt.subplots(figsize=(12, 5))
        ax.plot(df[x_col], df[y_col], color="#4C72B0", linewidth=2)
        ax.set_title(f"{y_col} 随 {x_col} 的变化趋势")
        ax.tick_params(axis="x", rotation=45)
        st.pyplot(fig)
        plt.close(fig)

    elif chart_type == "柱状图":
        col1, col2 = st.columns(2)
        with col1:
            cat_col = st.selectbox("分类列", df.columns, key="bar_cat")
        with col2:
            val_col = st.selectbox(
                "数值列",
                df.select_dtypes(include="number").columns,
                key="bar_val",
            )
        agg = st.radio("聚合方式", ["sum", "mean", "count"], horizontal=True, key="bar_agg")
        result = df.groupby(cat_col)[val_col].agg(agg).sort_values(ascending=False)
        fig, ax = plt.subplots(figsize=(10, 5))
        sns.barplot(x=result.index, y=result.values, ax=ax, palette="Blues_d")
        ax.set_title(f"各{cat_col}的{val_col}（{agg}）")
        ax.tick_params(axis="x", rotation=45)
        st.pyplot(fig)
        plt.close(fig)

    elif chart_type == "饼图":
        cat_col = st.selectbox("分类列", df.columns, key="pie_col")
        counts = df[cat_col].value_counts().head(8)
        fig, ax = plt.subplots(figsize=(8, 8))
        ax.pie(counts.values, labels=counts.index, autopct="%1.1f%%",
               colors=sns.color_palette("Set2", len(counts)))
        ax.set_title(f"{cat_col} 的占比分布")
        st.pyplot(fig)
        plt.close(fig)

    elif chart_type == "散点图":
        num_cols = df.select_dtypes(include="number").columns.tolist()
        if len(num_cols) >= 2:
            col1, col2 = st.columns(2)
            with col1:
                x_col = st.selectbox("X 轴", num_cols, key="scat_x")
            with col2:
                y_col = st.selectbox("Y 轴", num_cols, index=1, key="scat_y")
            fig, ax = plt.subplots(figsize=(10, 6))
            sns.scatterplot(data=df, x=x_col, y=y_col, ax=ax, alpha=0.6, s=60)
            ax.set_title(f"{x_col} vs {y_col}")
            st.pyplot(fig)
            plt.close(fig)
        else:
            st.warning("数值列不足 2 列")

    elif chart_type == "分布图":
        val_col = st.selectbox(
            "选择数值列",
            df.select_dtypes(include="number").columns,
            key="dist_col",
        )
        fig, ax = plt.subplots(figsize=(10, 5))
        sns.histplot(df[val_col], kde=True, ax=ax, color="#4C72B0")
        ax.set_title(f"{val_col} 的分布")
        st.pyplot(fig)
        plt.close(fig)

    elif chart_type == "箱线图":
        val_col = st.selectbox(
            "选择数值列",
            df.select_dtypes(include="number").columns,
            key="box_col",
        )
        cat_col = st.selectbox(
            "分组列（可选）", ["不分组"] + df.columns.tolist(), key="box_group"
        )
        fig, ax = plt.subplots(figsize=(10, 5))
        if cat_col != "不分组":
            sns.boxplot(data=df, x=cat_col, y=val_col, ax=ax)
        else:
            sns.boxplot(y=df[val_col], ax=ax)
        ax.set_title(f"{val_col} 的箱线图")
        st.pyplot(fig)
        plt.close(fig)


# ---------- Tab 5: 导出 ----------
with tab5:
    st.header("数据导出")

    col1, col2 = st.columns(2)

    with col1:
        st.subheader("📥 下载清洗后的数据")
        csv_data = df.to_csv(index=False).encode("utf-8-sig")
        st.download_button(
            label="下载 CSV",
            data=csv_data,
            file_name="cleaned_data.csv",
            mime="text/csv",
        )

    with col2:
        st.subheader("📋 生成分析报告")
        if st.button("生成 Markdown 报告"):
            from datalens.reporter import DataReporter
            reporter = DataReporter("DataLens 分析报告")
            reporter.add_title("数据概览", 2)
            reporter.add_text(
                f"数据形状：{df.shape[0]} 行 × {df.shape[1]} 列"
            )
            reporter.add_title("描述性统计", 2)
            numeric_df = df.select_dtypes(include="number")
            if not numeric_df.empty:
                reporter.add_table(numeric_df.describe().reset_index())
            reporter.add_title("数据预览", 2)
            reporter.add_table(df.head(10))
            report_path = reporter.generate_report("report.md")
            st.success("✓ 报告已生成")
            with open(report_path, "r", encoding="utf-8") as f:
                report_content = f.read()
            st.download_button(
                label="下载报告",
                data=report_content.encode("utf-8"),
                file_name="report.md",
                mime="text/markdown",
            )
```

### 3.1 启动 Web 应用

```python
# run_app.py
"""
启动 Streamlit Web 界面
"""
# 注意：需要在 datalens 项目目录下运行
# streamlit run datalens/app.py --server.port 8501
import subprocess
import sys

subprocess.run([
    sys.executable, "-m", "streamlit", "run",
    "datalens/app.py", "--server.port", "8501"
])
```

运行后浏览器打开 `http://localhost:8501` 即可看到界面：

```
┌──────────────────────────────────────────────────────┐
│  📊 DataLens 数据分析工具                              │
├──────────┬───────────────────────────────────────────┤
│ 侧边栏    │  📋 预览 | 🧹 清洗 | 📈 分析 | 🎨 可视化 | 📥 导出 │
│          │                                           │
│ 📁 数据加载│  当前数据：200 行 × 6 列                   │
│ [上传文件] │                                           │
│          │  总行数: 200    总列数: 6                     │
│ [示例数据]│  缺失值: 6      重复行: 2                    │
│          │                                           │
│          │  ┌─────────────────────────────────────┐  │
│          │  │        数据表格（可交互）              │  │
│          │  │  日期 | 产品 | 城市 | 渠道 | 销售额  │  │
│          │  │  ...  | ...  | ...  | ...  | ...    │  │
│          │  └─────────────────────────────────────┘  │
└──────────┴───────────────────────────────────────────┘
```

### 3.2 Web 界面设计要点

| 设计决策 | 原因 |
|---------|------|
| **五个 Tab 页** | 预览/清洗/分析/可视化/导出，每步独立，不混乱 |
| **session_state 管理数据** | 上传一次数据，跨 Tab 共享状态 |
| **示例数据按钮** | 没有数据文件也能体验全部功能 |
| **侧边栏 + 主区域** | 左右分栏，操作和数据展示分离 |
| **一键下载** | CSV 导出 + Markdown 报告，结果可带走 |
| **matplotlib Agg 模式** | Streamlit 已管理图表渲染，不需要弹窗 |

---

## 四、完整项目文件速查

```
datalens/
├── pyproject.toml
├── requirements.txt
├── run.py                    # CLI 入口
├── run_app.py                # Streamlit 入口
├── create_sample_data.py     # 生成测试数据
├── datalens/
│   ├── __init__.py           # 包初始化 + 版本号
│   ├── loader.py             # Day 56 ✅ 数据加载
│   ├── cleaner.py            # Day 56 ✅ 数据清洗
│   ├── analyzer.py           # Day 56 ✅ 统计分析
│   ├── visualizer.py          # Day 57 ✅ 可视化
│   ├── reporter.py            # Day 57 ✅ 报告生成
│   ├── app.py                 # Day 57 ✅ Web 界面
│   ├── cli.py                # Day 56 ✅ 命令行
│   └── utils.py              # 工具函数
├── tests/
└── data/
```

---

## 五、项目运行完整流程

```bash
# 1. 安装依赖
pip install pandas numpy matplotlib seaborn openpyxl click rich streamlit

# 2. 创建项目结构
mkdir datalens && cd datalens
mkdir -p datalens tests data
touch datalens/__init__.py

# 3. 复制各模块代码到对应文件
# （loader.py, cleaner.py, analyzer.py → Day 56）
# （visualizer.py, reporter.py, app.py → Day 57）

# 4. 生成测试数据
python create_sample_data.py

# 5. 命令行分析
python run.py load data/sample_sales.csv
python run.py clean data/sample_sales.csv -s auto -d
python run.py analyze data/sample_sales.csv -g 城市 -a 销售额 -f sum

# 6. 启动 Web 界面
streamlit run datalens/app.py --server.port 8501
# 浏览器打开 http://localhost:8501
```

---

## 六、整体架构回顾

```
                    ┌─────────────────────┐
                    │    用户交互层         │
                    │  CLI (click)        │
                    │  Web (Streamlit)    │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │    业务逻辑层         │
                    │  DataLoader         │
                    │  DataCleaner        │
                    │  DataAnalyzer       │
                    │  DataVisualizer     │
                    │  DataReporter       │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │    数据层             │
                    │  pandas DataFrame    │
                    │  CSV / Excel / JSON  │
                    │  SQLite (扩展)       │
                    └─────────────────────┘
```

这个三层架构对应了真实的软件工程实践：

| 层级 | 对应知识 | 学过的 Day |
|------|---------|-----------|
| 用户交互层 | click CLI + Streamlit Web | Day 20/51/52 |
| 业务逻辑层 | OOP 封装 + 类型注解 | Day 15-17/28-29 |
| 数据层 | pandas + 文件读写 | Day 23/38-40 |

---

## 练习题

### 练习 1：增强可视化模块

在 `DataVisualizer` 中添加 `grouped_boxplot()` 方法，绘制按分类列分组的箱线图。

```python
def grouped_boxplot(
    self,
    category_col: str,
    value_col: str,
    filename: str = "grouped_box.png",
) -> str:
    """按分类列分组绘制箱线图"""
    # 你的实现
    pass
```

### 练习 2：HTML 报告

在 `DataReporter` 中添加 `generate_html_report()` 方法，将报告生成为 HTML 格式（可嵌入图表图片），便于在浏览器中直接打开查看。

```python
def generate_html_report(self, output_path: str = "report.html") -> str:
    """生成 HTML 格式报告（含嵌入图片）"""
    # 你的实现
    pass
```

### 练习 3：Web 界面添加新功能

在 Streamlit 界面的"可视化"Tab 中添加一个**一键仪表盘**功能：点击按钮后，自动在页面上展示 4 张图表（趋势图、柱状图、饼图、分布图），排成 2×2 网格。

```python
if st.button("一键仪表盘"):
    col1, col2 = st.columns(2)
    with col1:
        # 趋势图 + 饼图
        pass
    with col2:
        # 柱状图 + 分布图
        pass
```

---

## 常见问题

### Q1: Streamlit 启动时报 "ModuleNotFoundError"？

**A**: 确保在项目根目录（`datalens/` 的上级）运行，而不是在 `datalens/` 目录内部运行。Python 需要能找到 `datalens` 这个包名：

```bash
# 正确 ✅（项目根目录）
cd myproject
streamlit run datalens/app.py

# 错误 ❌（包内部）
cd myproject/datalens
streamlit run app.py
```

### Q2: 图表中中文显示为方块？

**A**: 这是 matplotlib 字体问题。代码中已配置了 `SimHei` 和 `Microsoft YaHei`，但如果你的系统没有这些字体：

```python
# 查看系统可用中文字体
import matplotlib.font_manager as fm
for f in fm.font_manager.ttflist:
    if "chinese" in f.name.lower() or "sim" in f.name.lower() or "yahei" in f.name.lower():
        print(f.name, f.fname)
```

macOS 用户可以用 `"PingFang SC"`，Linux 用户需安装 `fonts-wqy-zenhei`。

### Q3: 为什么用 `plt.close(fig)` 而不是 `plt.show()`？

**A**: Streamlit 通过 `st.pyplot(fig)` 已经接管了图表的显示。`plt.close()` 释放内存——不调用的话，每次交互都会在内存中累积 Figure 对象，最终导致内存溢出。

### Q4: 报告中的表格很长怎么办？

**A**: `add_table()` 默认只显示前 20 行。你可以调大 `max_rows`，或者在报告中用 `add_text()` 添加说明：

```python
reporter.add_text("（仅显示前 20 行，完整数据共 10,000 行）")
reporter.add_table(df.head(20))
```

### Q5: 如何把 DataLens 分享给别人使用？

**A**: 几种方式：
1. **打包安装**：`pip install -e .` 安装为可编辑包，其他人 clone 后就能用
2. **Docker 部署**：写个 Dockerfile，打包成容器镜像
3. **Streamlit Cloud**：推送到 GitHub，在 streamlit.io 一键部署 Web 版

---

## 学习资源

- **Streamlit 官方文档**：https://docs.streamlit.io/
- **matplotlib 中文指南**：https://matplotlib.org.cn/
- **seaborn 教程**：https://seaborn.pydata.org/tutorial.html
- **Python Markdown 生成**：https://github.com/tersesystems/blade
- **菜鸟教程 - matplotlib**：https://www.runoob.com/matplotlib/matplotlib-tutorial.html

---

> **下一篇**：Day 58 将为 DataLens 添加**代码优化与重构**——运用设计模式、性能优化和测试，把项目打磨到生产级质量！
