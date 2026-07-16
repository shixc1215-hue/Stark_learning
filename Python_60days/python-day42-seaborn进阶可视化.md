# Python Day 42：seaborn 进阶可视化 —— 统计图表的"美学升级"

> Day 41 我们掌握了 matplotlib 的核心绑图能力。今天学习 **seaborn** —— 一个基于 matplotlib 的高级统计可视化库。seaborn 的代码更简洁、图表更美观、内置大量统计图表类型，特别适合数据探索和汇报展示。一句话：**matplotlib 是画笔，seaborn 是调色板**。

---

## 一、回顾与过渡

Day 41 我们用 matplotlib 画了折线图、柱状图、饼图、散点图、直方图、箱线图。但 matplotlib 有几个痛点：

| 痛点 | 说明 |
|------|------|
| 默认样式不够美观 | 需要手动调颜色、字体、间距 |
| 统计图表要自己写 | 如热力图、小提琴图需要大量代码 |
| DataFrame 数据要手动提取 | 不能直接传 DataFrame 画图 |
| 分类变量处理繁琐 | 按类别分组、着色需要额外处理 |

**seaborn 正好解决这些问题**：它直接接受 pandas DataFrame，一行代码就能画出精美的统计图表。

今天的目标：**掌握 seaborn 的核心绑图函数，能画出 10+ 种统计图表**。

---

## 二、seaborn 是什么？

### 2.1 一句话介绍

**seaborn** 是基于 matplotlib 的高级统计可视化库，由 Stanford 大学研究员开发。它的设计哲学是：**让统计图表的创建尽可能简单**。

### 2.2 安装

```bash
pip install seaborn
```

### 2.3 seaborn vs matplotlib

| 特性 | matplotlib | seaborn |
|------|-----------|---------|
| 定位 | 基础绑图库 | 高级统计可视化 |
| 代码量 | 较多（需手动配置） | 较少（一行出图） |
| 默认美观度 | 一般，需调参 | 高，开箱即用 |
| 数据输入 | 数组、列表 | 直接传 DataFrame |
| 统计图表 | 需手写 | 内置 heatmap、violin 等 |
| 自定义灵活度 | 极高 | 高（底层仍是 matplotlib） |
| 学习曲线 | 较平缓 | 稍陡（概念较多） |

> **核心关系：** seaborn 底层就是 matplotlib。你用 seaborn 画图后，仍然可以用 matplotlib 的 `plt.title()`、`plt.xlabel()` 等方法来微调。

### 2.4 第一个 seaborn 图表

```python
import seaborn as sns
import matplotlib.pyplot as plt

# 设置中文显示（seaborn 底层仍用 matplotlib）
plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 加载内置数据集
tips = sns.load_dataset("tips")

# 一行代码画出带回归线的散点图
sns.lmplot(x="total_bill", y="tip", data=tips)

plt.title("账单金额与小费的关系")
plt.show()
```

> `sns.load_dataset()` 可以加载 seaborn 内置的经典数据集，非常适合练习。

---

## 三、seaborn 的四大核心图表函数

seaborn 的图表函数按**关系类型**分为四类：

| 函数 | 关系类型 | 用途 |
|------|---------|------|
| `sns.relplot()` | 关系型 | 变量之间的关系（散点、折线） |
| `sns.displot()` | 分布型 | 变量的分布（直方图、KDE） |
| `sns.catplot()` | 分类型 | 分类数据的比较（箱线、小提琴、柱状） |
| `sns.heatmap()` | 矩阵型 | 矩阵数据可视化（热力图、聚类图） |

下面逐一讲解。

---

## 四、关系型图表：sns.relplot()

展示两个数值变量之间的关系。

### 4.1 散点图（默认）

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 散点图，按"时间"（午餐/晚餐）用颜色区分
sns.relplot(x="total_bill", y="tip", hue="time", data=tips,
            height=6, aspect=1.5)  # height=高度(英寸), aspect=宽高比

plt.title("不同时段的账单金额与小费", fontsize=14)
plt.xlabel("账单金额（美元）")
plt.ylabel("小费金额（美元）")
plt.show()
```

**关键参数：**
- `hue` —— 用颜色区分某个分类变量
- `style` —— 用标记样式区分分类变量
- `size` —— 用散点大小区分某个数值变量
- `height` / `aspect` —— 图表尺寸

### 4.2 折线图

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 加载航班数据集
flights = sns.load_dataset("flights")

# 折线图：按年份展示乘客数趋势，用颜色区分月份
sns.relplot(x="year", y="passengers", hue="month",
            kind="line", data=flights,
            height=6, aspect=1.5,
            palette="tab10")

plt.title("1949-1960年各月航班乘客数", fontsize=14)
plt.xlabel("年份")
plt.ylabel("乘客数（万人）")
plt.show()
```

> `kind="line"` 指定画折线图。注意 seaborn 的折线图会自动计算并显示**置信区间**（浅色带状区域），非常直观。

### 4.3 多维度可视化

同时用颜色、样式、大小来编码信息：

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 颜色 = 用餐时间，样式 = 是否吸烟，大小 = 聚餐人数
sns.relplot(x="total_bill", y="tip", hue="time", style="smoker",
            size="size", data=tips,
            height=6, aspect=1.5,
            palette="Set2")

plt.title("多维信息散点图", fontsize=14)
plt.show()
```

---

## 五、分布型图表：sns.displot()

展示单个变量的分布情况。

### 5.1 直方图

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 直方图 + KDE曲线
sns.displot(tips["total_bill"], bins=25, kde=True,
            height=5, aspect=1.5,
            color="#3498db", edgecolor="white")

plt.title("账单金额分布", fontsize=14)
plt.xlabel("账单金额（美元）")
plt.ylabel("频次")
plt.show()
```

> `kde=True` 会同时显示核密度估计曲线（平滑的概率密度曲线）。

### 5.2 按类别分面（Facet）

seaborn 的杀手级功能——自动按类别拆分成多个子图：

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 按性别分行，按时段分列
sns.displot(tips, x="total_bill", col="time", row="sex",
            bins=20, kde=True, height=4, aspect=1.2)

plt.suptitle("不同性别和时段的账单金额分布", y=1.02, fontsize=14)
plt.show()
```

**分面参数：**
- `col` —— 按某列拆分成多列子图
- `row` —— 按某列拆分成多行子图
- `col_wrap` —— 限制每行显示几个子图

### 5.3 KDE 图（核密度估计）

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# KDE 图，按性别叠加显示
sns.displot(tips, x="total_bill", hue="sex",
            kind="kde", fill=True,       # fill=True 填充颜色
            height=5, aspect=1.5,
            palette=["#e74c3c", "#3498db"])

plt.title("不同性别的账单金额概率密度", fontsize=14)
plt.xlabel("账单金额（美元）")
plt.show()
```

> `fill=True` 让密度曲线下方填充颜色，视觉效果更好。

---

## 六、分类型图表：sns.catplot()

比较不同类别之间的数据分布，这是 seaborn 最强大的功能之一。

### 6.1 箱线图

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 不同时段的小费箱线图，按性别分颜色
sns.catplot(x="day", y="total_bill", hue="sex",
            kind="box", data=tips,
            height=6, aspect=1.2,
            palette="Set2")

plt.title("不同日期和性别的账单金额分布", fontsize=14)
plt.xlabel("星期")
plt.ylabel("账单金额（美元）")
plt.show()
```

### 6.2 小提琴图（Violin Plot）

小提琴图是箱线图的升级版，能同时展示数据的分布形状：

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

sns.catplot(x="day", y="total_bill", hue="sex",
            kind="violin", split=True,     # split=True 左右对比
            data=tips,
            height=6, aspect=1.2,
            palette="muted")

plt.title("不同日期的账单金额分布（小提琴图）", fontsize=14)
plt.xlabel("星期")
plt.ylabel("账单金额（美元）")
plt.show()
```

**箱线图 vs 小提琴图：**

| 特性 | 箱线图 | 小提琴图 |
|------|--------|---------|
| 信息量 | 五数概括 | 五数概括 + 密度形状 |
| 能否看出多峰 | 不能 | 能 |
| 视觉效果 | 简洁 | 更丰富 |
| 数据量少时 | 稳定 | 可能误导 |

### 6.3 条形图（带误差棒）

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 各日平均小费，用颜色区分性别
sns.catplot(x="day", y="tip", hue="sex",
            kind="bar", data=tips,
            height=5, aspect=1.3,
            palette="Set2",
            errorbar="sd")  # 误差棒显示标准差（默认是置信区间）

plt.title("各日平均小费金额", fontsize=14)
plt.xlabel("星期")
plt.ylabel("小费金额（美元）")
plt.show()
```

> seaborn 的条形图自动计算**均值**，误差棒默认显示 95% 置信区间。`errorbar="sd"` 改为显示标准差。

### 6.4 散点抖动图（Swarm Plot）

当数据点不多时，用散点抖动图可以看清每个数据点：

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

sns.catplot(x="day", y="total_bill", hue="sex",
            kind="swarm", data=tips,
            height=5, aspect=1.3,
            palette="Set2")

plt.title("各日账单金额散点分布", fontsize=14)
plt.xlabel("星期")
plt.ylabel("账单金额（美元）")
plt.show()
```

### 6.5 分类图表种类速查

`kind` 参数决定图表类型：

| kind | 图表类型 | 适用场景 |
|------|---------|---------|
| `"strip"` | 散点抖动图（默认） | 数据量小，看原始点 |
| `"swarm"` | 蜂群图（不重叠） | 数据量小，看清每个点 |
| `"box"` | 箱线图 | 比较分布，看异常值 |
| `"violin"` | 小提琴图 | 比较分布形状 |
| `"boxen"` | 增强箱线图 | 数据量大的分布比较 |
| `"bar"` | 条形图 | 比较均值 |
| `"count"` | 计数图 | 某类别的频次统计 |
| `"point"` | 点图 | 均值对比 + 误差棒 |

---

## 七、热力图：sns.heatmap()

展示矩阵数据，是数据分析中最常用的图表之一。

### 7.1 相关性热力图

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 计算数值列的相关系数矩阵
corr = tips[["total_bill", "tip", "size"]].corr()
print("相关系数矩阵：")
print(corr)

# 画热力图
sns.heatmap(corr, annot=True,         # 显示数值
            fmt=".2f",               # 数值保留2位小数
            cmap="RdBu_r",           # 红蓝色阶（正相关红，负相关蓝）
            center=0,                # 中心值为0
            square=True,             # 每个格子是正方形
            linewidths=0.5,          # 格子间白线
            vmin=-1, vmax=1)         # 颜色范围

plt.title("变量相关性热力图", fontsize=14)
plt.show()
```

**关键参数：**
- `annot` —— 是否在格子中显示数值
- `fmt` —— 数值格式（`.2f` = 两位小数，`.1%` = 百分比）
- `cmap` —— 颜色映射（`RdBu_r` 红蓝、`YlOrRd` 黄橙红、`Blues` 蓝色渐变）
- `center` —— 颜色中心值（0 表示正负对称）

### 7.2 枢纽表热力图

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

flights = sns.load_dataset("flights")

# 创建枢纽表：行=月份，列=年份，值=乘客数
flight_pivot = flights.pivot(index="month", column="year", values="passengers")

# 热力图
plt.figure(figsize=(12, 8))
sns.heatmap(flight_pivot, annot=True, fmt="d",    # 整数格式
            cmap="YlOrRd",                          # 黄→橙→红
            linewidths=0.5)

plt.title("1949-1960年航班乘客数（月×年）", fontsize=14)
plt.xlabel("年份")
plt.ylabel("月份")
plt.show()
```

### 7.3 常用色板（cmap）速查

| cmap | 颜色风格 | 适用场景 |
|------|---------|---------|
| `"RdBu_r"` | 红蓝反转 | 相关性（正=红，负=蓝） |
| `"YlOrRd"` | 黄→橙→红 | 数值大小（越大越热） |
| `"Blues"` | 白→深蓝 | 深度/权重 |
| `"Greens"` | 白→深绿 | 增长/正向指标 |
| `"coolwarm"` | 蓝→红冷热 | 发散型数据 |
| `"viridis"` | 紫绿黄 | 通用（色盲友好） |

---

## 八、成对关系图：sns.pairplot()

一张图看完 DataFrame 中所有数值变量的两两关系，是数据探索的利器。

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 数值列的成对关系图，按性别分颜色
sns.pairplot(tips, hue="sex",
             vars=["total_bill", "tip", "size"],  # 指定要展示的列
             palette="Set2",
             height=3,
             diag_kind="kde")  # 对角线显示KDE密度图

plt.suptitle("数值变量两两关系", y=1.02, fontsize=14)
plt.show()
```

**解读：**
- **对角线** —— 每个变量的分布（KDE 密度图）
- **非对角线** —— 两个变量之间的散点图
- **颜色** —— 按性别区分

> 数据探索阶段，`pairplot` 是第一个该画的图——它能快速帮你发现变量间的关系和异常。

---

## 九、seaborn 样式主题

seaborn 内置了多种美观的主题风格：

```python
import seaborn as sns
import matplotlib.pyplot as plt

# 设置主题（二选一）

# 方式1：用 sns.set_theme()（推荐）
sns.set_theme(style="whitegrid")   # 白底+网格线
# sns.set_theme(style="darkgrid")  # 深色背景+网格线
# sns.set_theme(style="white")      # 白底无网格线
# sns.set_theme(style="dark")       # 深色背景无网格线
# sns.set_theme(style="ticks")      # 带刻度线

# 方式2：更细粒度的控制
sns.set_theme(
    style="whitegrid",       # 样式
    palette="husl",          # 调色板
    font="SimHei",          # 字体
    font_scale=1.2           # 字体缩放比例
)

# 画图
tips = sns.load_dataset("tips")
sns.barplot(x="day", y="total_bill", data=tips)
plt.title("各日平均账单")
plt.show()
```

**主题对比：**

| 主题 | 背景 | 网格线 | 适用场景 |
|------|------|--------|---------|
| `whitegrid` | 白色 | 有 | 数据分析、汇报（最常用） |
| `darkgrid` | 深灰 | 有 | 深色主题演示 |
| `white` | 白色 | 无 | 干净简洁的图表 |
| `dark` | 深灰 | 无 | 深色背景简洁风 |
| `ticks` | 白色 | 有刻度线 | 学术论文 |

### 调色板（palette）

```python
# 预定义调色板
sns.set_theme(palette="deep")       # 深色调（默认）
sns.set_theme(palette="muted")      # 柔和色
sns.set_theme(palette="pastel")     # 粉彩色
sns.set_theme(palette="bright")     # 明亮色
sns.set_theme(palette="dark")       # 深色
sns.set_theme(palette="colorblind")  # 色盲友好
sns.set_theme(palette="husl")       # HUSL 均匀色

# 自定义颜色
sns.set_theme(palette=["#e74c3c", "#3498db", "#2ecc71", "#f39c12"])
```

---

## 十、综合实战：电商数据分析仪表盘

用 seaborn 对模拟的电商数据进行多维度可视化分析。

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

# ---- 0. 设置中文与主题 ----
plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False
sns.set_theme(style="whitegrid", font_scale=1.1)

# ---- 1. 生成模拟数据 ----
np.random.seed(42)
n = 3000

df = pd.DataFrame({
    "订单日期": [datetime(2025, 1, 1) + timedelta(days=np.random.randint(0, 365))
                 for _ in range(n)],
    "商品类别": np.random.choice(
        ["电子产品", "服装鞋帽", "家居用品", "食品饮料", "图书文具"], n,
        p=[0.25, 0.22, 0.20, 0.18, 0.15]
    ),
    "订单金额": np.clip(np.random.lognormal(6, 0.8, n), 20, 5000),
    "城市": np.random.choice(
        ["北京", "上海", "广州", "深圳", "杭州", "成都"], n
    ),
    "客户等级": np.random.choice(["普通", "银卡", "金卡", "钻石"], n,
                                p=[0.4, 0.3, 0.2, 0.1]),
    "支付方式": np.random.choice(["支付宝", "微信支付", "银行卡"], n)
})
df["订单日期"] = pd.to_datetime(df["订单日期"])
df["月份"] = df["订单日期"].dt.month
df["星期"] = df["订单日期"].dt.day_name()

print(f"数据概览：{len(df)} 条订单")
print(df.head())
print(df.describe())

# ---- 2. 创建 2x3 仪表盘 ----
fig, axes = plt.subplots(2, 3, figsize=(18, 11))
plt.suptitle("2025年电商销售数据可视化分析", fontsize=18, fontweight="bold", y=1.02)

# 图1：月度销售趋势（折线图）
monthly = df.groupby("月份")["订单金额"].sum().reset_index()
sns.lineplot(x="月份", y="订单金额", data=monthly,
             marker="o", linewidth=2.5, color="#e74c3c", ax=axes[0, 0])
axes[0, 0].set_title("月度销售趋势")
axes[0, 0].set_xlabel("月份")
axes[0, 0].set_ylabel("销售额（元）")
axes[0, 0].fill_between(monthly["月份"], monthly["订单金额"],
                        alpha=0.15, color="#e74c3c")

# 图2：各类别销售额（箱线图）
sns.boxplot(x="商品类别", y="订单金额", data=df,
            palette="Set2", ax=axes[0, 1])
axes[0, 1].set_title("各类别订单金额分布")
axes[0, 1].set_xlabel("商品类别")
axes[0, 1].set_ylabel("订单金额（元）")
axes[0, 1].tick_params(axis="x", rotation=30)

# 图3：客户等级订单金额分布（小提琴图）
sns.violinplot(x="客户等级", y="订单金额", data=df,
               palette="muted", order=["普通", "银卡", "金卡", "钻石"],
               ax=axes[0, 2])
axes[0, 2].set_title("不同客户等级的消费分布")
axes[0, 2].set_xlabel("客户等级")
axes[0, 2].set_ylabel("订单金额（元）")

# 图4：城市×类别 销售额热力图
pivot = df.pivot_table(values="订单金额", index="城市",
                        columns="商品类别", aggfunc="sum")
sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlOrRd",
            linewidths=0.5, ax=axes[1, 0])
axes[1, 0].set_title("城市×类别 销售额热力图")
axes[1, 0].set_xlabel("商品类别")
axes[1, 0].set_ylabel("城市")
axes[1, 0].tick_params(axis="x", rotation=30)

# 图5：各城市平均订单金额（带误差棒的条形图）
city_mean = df.groupby("城市")["订单金额"].mean().reset_index()
city_mean = city_mean.sort_values("订单金额", ascending=False)
sns.barplot(x="城市", y="订单金额", data=city_mean,
            palette="Blues_d", errorbar="sd", ax=axes[1, 1])
axes[1, 1].set_title("各城市平均订单金额")
axes[1, 1].set_xlabel("城市")
axes[1, 1].set_ylabel("平均金额（元）")
axes[1, 1].tick_params(axis="x", rotation=30)

# 图6：订单金额分布（直方图+KDE）
sns.histplot(df["订单金额"], bins=40, kde=True,
             color="#3498db", edgecolor="white", ax=axes[1, 2])
axes[1, 2].set_title("订单金额分布")
axes[1, 2].set_xlabel("订单金额（元）")
axes[1, 2].set_ylabel("频次")

plt.tight_layout()
plt.savefig("ecommerce_dashboard.png", dpi=150, bbox_inches="tight")
print("仪表盘已保存：ecommerce_dashboard.png")
plt.show()
```

---

## 十一、seaborn 与 matplotlib 混合使用

seaborn 画出的图表本质上是 matplotlib 对象，可以自由混搭：

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

tips = sns.load_dataset("tips")

# 用 seaborn 画底图
fig, ax = plt.subplots(figsize=(10, 6))

# seaborn 画散点图（传入 ax 参数）
sns.scatterplot(x="total_bill", y="tip", hue="time",
                data=tips, ax=ax, s=80, alpha=0.7)

# matplotlib 画趋势线
z = np.polyfit(tips["total_bill"], tips["tip"], 1)
p = np.poly1d(z)
x_line = np.linspace(tips["total_bill"].min(), tips["total_bill"].max(), 100)
ax.plot(x_line, p(x_line), "--", color="#e74c3c", linewidth=2)

# matplotlib 添加文字
ax.set_title("seaborn + matplotlib 混合使用", fontsize=14)
ax.set_xlabel("账单金额")
ax.set_ylabel("小费")

plt.show()
```

> **关键：** seaborn 的画图函数接受 `ax` 参数，可以指定画在哪个子图上。这是混合使用的核心。

---

## 十二、练习题

### 练习 1：基础（必做）

1. 用 seaborn 加载 `tips` 数据集
2. 画出一张 `total_bill` 的直方图（带 KDE 曲线）
3. 用 `hue="sex"` 区分性别
4. 保存为 `tips_histogram.png`

### 练习 2：进阶

1. 加载 `diamonds` 数据集（`sns.load_dataset("diamonds")`）
2. 创建一个 **2×2** 的子图布局：
   - 左上：用 `sns.boxplot` 展示不同 `cut`（切工）的 `price` 分布
   - 右上：用 `sns.violinplot` 展示不同 `cut` 的 `carat` 分布
   - 左下：用 `sns.barplot` 展示不同 `cut` 的平均 `price`
   - 右下：用 `sns.heatmap` 展示 `cut` 和 `clarity`（净度）的 `price` 均值热力图
3. 设置统一的主题和配色

### 练习 3：挑战

1. 自己创建一个 DataFrame，包含：学生姓名、科目（数学/英语/物理/化学）、成绩（60-100随机）、班级（A/B/C）
2. 用 seaborn 画出**至少 5 种不同类型的图表**，从多角度分析数据：
   - 成绩分布直方图
   - 各科平均成绩条形图
   - 各班级成绩箱线图
   - 科目×班级成绩热力图
   - 成绩的 pairplot 多维关系图
3. 每张图都有清晰的标题、标签和配色

---

## 十三、常见问题

### Q1：seaborn 中文显示还是方块怎么办？

seaborn 底层是 matplotlib，所以中文设置方式完全相同：

```python
plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False
```

如果 SimHei 不可用，试试微软雅黑：`["Microsoft YaHei"]`。

### Q2：sns.relplot() 和 sns.scatterplot() 有什么区别？

| 函数 | 区别 |
|------|------|
| `sns.scatterplot()` | 画在指定的 `ax` 上，适合嵌入子图 |
| `sns.relplot()` | 自动创建 `FacetGrid`，支持 `col/row` 分面 |

> 简单说：单图画 `scatterplot`，要分面用 `relplot`。`barplot`/`boxplot` 与 `catplot` 同理。

### Q3：数据量很大时 pairplot 太慢怎么办？

`pairplot` 会画 n×n 个子图，数据量大时非常慢。解决方案：

```python
# 方法1：只取部分列
sns.pairplot(df, vars=["col1", "col2", "col3"])

# 方法2：采样数据
sample = df.sample(n=500)  # 随机取500行
sns.pairplot(sample, hue="category")

# 方法3：用 corner=True 只画下三角
sns.pairplot(df, hue="category", corner=True)
```

### Q4：如何导出 seaborn 图表？

和 matplotlib 完全一样：

```python
plt.savefig("chart.png", dpi=300, bbox_inches="tight")
plt.savefig("chart.pdf", bbox_inches="tight")  # 矢量图
plt.savefig("chart.svg", bbox_inches="tight")  # 网页友好
```

### Q5：seaborn 的 `x` 和 `y` 参数传什么？

- **传列名字符串**：`x="total_bill"` —— 从 DataFrame 中取列
- **传数组**：`x=df["total_bill"]` —— 直接传数据

> 推荐**传列名字符串**，这样自动使用 DataFrame 的列名作为轴标签。

---

## 十四、免费学习资源

- **seaborn 官方文档**（最权威）：https://seaborn.pydata.org/
- **seaborn 图表画廊**（看示例学图表）：https://seaborn.pydata.org/examples/index.html
- **seaborn API 参考**（查阅函数参数）：https://seaborn.pydata.org/api.html
- **菜鸟教程 Seaborn**：https://www.runoob.com/seaborn/seaborn-tutorial.html
- **Kaggle 免费课程 Data Visualization**：https://www.kaggle.com/learn/data-visualization

---

## 十五、下节预告

Day 43 我们学习 **numpy 基础** —— Python 科学计算的基石。numpy 提供了高效的数组运算能力，是 pandas、seaborn、scikit-learn 等库的底层支撑。掌握 numpy 后，你的数据处理速度将提升一个量级。

---

> **今日小结：**
> 今天学习了 seaborn 的四大核心图表函数：`relplot`（关系型）、`displot`（分布型）、`catplot`（分类型）、`heatmap`（矩阵型），以及 `pairplot` 成对关系图和主题样式系统。seaborn 的优势在于：代码简洁、直接接受 DataFrame、内置统计功能、默认美观。**建议搭配 matplotlib 一起使用：seaborn 画主体，matplotlib 做微调**。下节进入 numpy 的世界，为后续的高效数据处理打基础。
