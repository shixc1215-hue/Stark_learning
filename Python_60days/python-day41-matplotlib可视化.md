# Python Day 41：matplotlib 可视化 —— 用图表讲数据的故事

> Day 38-40 我们用 pandas 完成了数据处理的完整流程。今天进入一个全新领域——**数据可视化**。matplotlib 是 Python 最经典、最基础的绑图库，几乎所有你见过的 Python 图表底层都在用它。今天学完，你就能让数据"开口说话"。

---

## 一、回顾与过渡

Day 40 我们完成了一个电商销售数据分析项目，用 pandas 得到了各种统计数字。但纯文字报表不够直观——如果老板问你"各月销售趋势怎么样"，一张折线图比一堆数字有说服力得多。

| 阶段 | 工具 | 输出 |
|------|------|------|
| 数据获取 | requests / API | 原始数据 |
| 数据清洗 | pandas | 干净的 DataFrame |
| 数据分析 | pandas | 统计结果 |
| **数据展示** | **matplotlib** | **图表** |

今天的目标：**掌握 matplotlib 的核心绑图方法，能画出常用图表**。

---

## 二、matplotlib 是什么？

### 2.1 一句话介绍

**matplotlib** 是 Python 的 2D 绑图库，能生成出版级质量的图表。它名字来源于 MATLAB（一个工程计算软件）的 plot 模块，所以用法和 MATLAB 类似。

### 2.2 安装

```bash
pip install matplotlib
```

> 大多数 Python 环境已经预装了 matplotlib。如果没装，运行上面的命令即可。

### 2.3 第一个图表

```python
import matplotlib.pyplot as plt

# 准备数据
x = [1, 2, 3, 4, 5]
y = [2, 4, 6, 8, 10]

# 绑图
plt.plot(x, y)

# 显示
plt.show()
```

运行后会弹出一个窗口，显示一条从左下到右上的直线。恭喜，你的第一个图表诞生了！

> **注意：** 在 Jupyter Notebook 中，如果想让图表直接显示在页面上，需要在开头加一行魔法命令 `%matplotlib inline`。

---

## 三、matplotlib 的两种绘图风格

matplotlib 有两种常用的绘图方式，你需要都了解。

### 3.1 Pyplot 接口（函数式风格）

最简单直接的方式，适合快速画图：

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(8, 4))    # 创建画布，8英寸宽，4英寸高
plt.plot([1, 2, 3], [4, 5, 6])  # 画线
plt.title("我的第一个图表")       # 标题
plt.xlabel("X轴")               # X轴标签
plt.ylabel("Y轴")               # Y轴标签
plt.show()                      # 显示
```

### 3.2 面向对象风格（推荐进阶使用）

更灵活、更可控的方式，适合复杂图表：

```python
import matplotlib.pyplot as plt

# 创建画布和坐标系
fig, ax = plt.subplots(figsize=(8, 4))

# 在坐标系上画图
ax.plot([1, 2, 3], [4, 5, 6])
ax.set_title("我的第一个图表")
ax.set_xlabel("X轴")
ax.set_ylabel("Y轴")

plt.show()
```

**两种风格对比：**

| 特性 | Pyplot 风格 | 面向对象风格 |
|------|------------|-------------|
| 写法 | `plt.xxx()` | `ax.xxx()` |
| 控制粒度 | 全局操作 | 精确到子图 |
| 适用场景 | 快速绑图、单图 | 多子图、复杂图表 |
| 学习难度 | 简单 | 稍高 |

> **建议：** 初学阶段先用 Pyplot 风格（简单直观），熟练后转向面向对象风格（更灵活）。本教程主要用 Pyplot 风格演示。

---

## 四、中文显示问题（重要）

matplotlib 默认不支持中文，画图时中文会显示为方块。解决方法：

### 方法一：临时设置（推荐，简单）

```python
import matplotlib.pyplot as plt

# 设置中文字体
plt.rcParams["font.sans-serif"] = ["SimHei"]  # Windows 用黑体
# macOS 用户用下面这行：
# plt.rcParams["font.sans-serif"] = ["Arial Unicode MS"]

# 解决负号显示问题
plt.rcParams["axes.unicode_minus"] = False
```

### 方法二：永久设置

找到 matplotlib 的配置文件 `matplotlibrc`，添加：
```
font.sans-serif: SimHei
axes.unicode_minus: False
```

> **本教程后续所有代码都假设你已经设置了中文字体**。实际运行时记得加上这两行。

---

## 五、最常用的 6 种图表

### 5.1 折线图（Line Plot）

**用途：** 展示数据随时间/顺序的变化趋势。

```python
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 月度销售数据
months = ["1月", "2月", "3月", "4月", "5月", "6月",
          "7月", "8月", "9月", "10月", "11月", "12月"]
sales = [120, 132, 101, 134, 90, 230, 210, 182, 191, 250, 220, 280]

plt.figure(figsize=(10, 5))
plt.plot(months, sales, marker="o", linewidth=2, color="#e74c3c",
         markersize=8, markerfacecolor="white", markeredgewidth=2)

plt.title("2025年月度销售额趋势", fontsize=16)
plt.xlabel("月份", fontsize=12)
plt.ylabel("销售额（万元）", fontsize=12)
plt.grid(True, alpha=0.3)       # 添加网格线，透明度 0.3
plt.ylim(0, 300)               # Y轴范围 0-300

plt.show()
```

**常用参数说明：**

| 参数 | 含义 | 示例 |
|------|------|------|
| `marker` | 数据点标记样式 | `"o"` 圆点, `"s"` 方块, `"^"` 三角 |
| `linewidth` | 线条宽度 | `2`（默认 1） |
| `color` | 线条颜色 | `"red"`, `"#e74c3c"`, `"#333333"` |
| `markersize` | 数据点大小 | `8` |
| `linestyle` | 线条样式 | `"-"` 实线, `"--"` 虚线, `":"` 点线 |

### 5.2 柱状图（Bar Chart）

**用途：** 比较不同类别之间的数值大小。

```python
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 各商品类别销售额
categories = ["电子产品", "服装鞋帽", "家居用品", "食品饮料", "图书文具"]
revenue = [285, 256, 251, 240, 238]

plt.figure(figsize=(10, 6))

# 画柱状图
bars = plt.bar(categories, revenue, color=["#e74c3c", "#3498db", "#2ecc71",
                                             "#f39c12", "#9b59b6"],
               width=0.6, edgecolor="white", linewidth=1.5)

# 在每个柱子顶部显示数值
for bar, val in zip(bars, revenue):
    plt.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 5,
             f"{val}万", ha="center", va="bottom", fontsize=11)

plt.title("各商品类别销售额对比", fontsize=16)
plt.ylabel("销售额（万元）", fontsize=12)
plt.ylim(0, 320)
plt.grid(axis="y", alpha=0.3)    # 只显示水平网格线

plt.show()
```

**小技巧：** `plt.text()` 可以在图表任意位置添加文字，`ha="center"` 表示水平居中，`va="bottom"` 表示垂直底部对齐。

### 5.3 饼图（Pie Chart）

**用途：** 展示各部分占整体的百分比。

```python
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 支付方式占比
payment = ["支付宝", "微信支付", "银行卡", "信用卡"]
counts = [42, 35, 15, 8]  # 百分比
colors = ["#1677ff", "#07c160", "#faad14", "#eb2f96"]
explode = (0.05, 0, 0, 0)  # 让第一块稍微突出

plt.figure(figsize=(8, 8))
plt.pie(counts, labels=payment, colors=colors, explode=explode,
       autopct="%1.1f%%",     # 显示百分比，保留1位小数
       startangle=90,          # 从12点方向开始
       shadow=True,            # 添加阴影
       textprops={"fontsize": 13})

plt.title("支付方式占比", fontsize=16)
plt.legend(payment, loc="lower right")  # 图例位置：右下角

plt.show()
```

### 5.4 散点图（Scatter Plot）

**用途：** 观察两个变量之间的关系（相关性）。

```python
import matplotlib.pyplot as plt
import numpy as np

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 模拟数据：广告投入 vs 销售额
np.random.seed(42)
ad_spend = np.random.uniform(10, 100, 50)  # 广告投入（万元）
sales = ad_spend * 2.5 + np.random.normal(0, 20, 50)  # 销售额，带随机噪声
sales = np.clip(sales, 0, 300)  # 限制在合理范围

plt.figure(figsize=(9, 6))
plt.scatter(ad_spend, sales, s=80,        # s=散点大小
            c=ad_spend,                     # 颜色按广告投入映射
            cmap="RdYlBu",                  # 红黄蓝渐变色
            alpha=0.7,                     # 透明度
            edgecolors="white",             # 散点边框颜色
            linewidths=0.5)

# 添加趋势线
z = np.polyfit(ad_spend, sales, 1)   # 一次多项式拟合（直线）
p = np.poly1d(z)
x_line = np.linspace(10, 100, 100)
plt.plot(x_line, p(x_line), "--", color="#e74c3c", linewidth=2,
         label=f"趋势线: y = {z[0]:.1f}x + {z[1]:.1f}")

plt.colorbar(label="广告投入（万元）")  # 颜色条图例
plt.title("广告投入与销售额的关系", fontsize=16)
plt.xlabel("广告投入（万元）", fontsize=12)
plt.ylabel("销售额（万元）", fontsize=12)
plt.legend(fontsize=11)
plt.grid(True, alpha=0.3)

plt.show()
```

> `np.polyfit()` 和 `np.poly1d()` 用于拟合趋势线，是 numpy 的功能。Day 43 会系统学习 numpy。

### 5.5 直方图（Histogram）

**用途：** 展示数据的分布情况（频次/密度）。

```python
import matplotlib.pyplot as plt
import numpy as np

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 模拟用户消费金额
np.random.seed(42)
spending = np.random.lognormal(mean=6.5, sigma=0.8, size=1000)

plt.figure(figsize=(10, 6))
plt.hist(spending, bins=30,              # 分成 30 个区间
         color="#3498db",
         edgecolor="white",
         alpha=0.8,
         density=False)                 # False=频次, True=概率密度

plt.title("用户消费金额分布", fontsize=16)
plt.xlabel("消费金额（元）", fontsize=12)
plt.ylabel("用户数", fontsize=12)
plt.grid(axis="y", alpha=0.3)

# 添加统计信息文字
mean_val = np.mean(spending)
median_val = np.median(spending)
plt.axvline(mean_val, color="#e74c3c", linestyle="--", linewidth=2,
            label=f"均值: {mean_val:.0f}元")
plt.axvline(median_val, color="#2ecc71", linestyle="--", linewidth=2,
            label=f"中位数: {median_val:.0f}元")
plt.legend(fontsize=11)

plt.show()
```

**关键概念：**
- `bins` —— 把数据分成多少个区间，越多越精细，越少越概括
- `axvline()` —— 画一条垂直参考线，适合标注均值、中位数等

### 5.6 箱线图（Box Plot）

**用途：** 展示数据分布的五数概括（最小值、Q1、中位数、Q3、最大值）和异常值。

```python
import matplotlib.pyplot as plt
import numpy as np

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# 各类别用户消费金额
np.random.seed(42)
electronics = np.random.lognormal(7, 0.6, 200)
clothing = np.random.lognormal(6.5, 0.5, 200)
food = np.random.lognormal(6, 0.7, 200)

plt.figure(figsize=(9, 6))
data = [electronics, clothing, food]
labels = ["电子产品", "服装鞋帽", "食品饮料"]

plt.boxplot(data, labels=labels, patch_artist=True,
            boxprops=dict(facecolor="#3498db", alpha=0.5),
            medianprops=dict(color="#e74c3c", linewidth=2),
            flierprops=dict(marker="o", markerfacecolor="#e74c3c",
                            markersize=5))

plt.title("各品类用户消费金额分布", fontsize=16)
plt.ylabel("消费金额（元）", fontsize=12)
plt.grid(axis="y", alpha=0.3)
plt.show()
```

**箱线图解读：**

```
        ┌─────┐          ← 最大值（须的顶端）
        │     │
    ┌───┤     ├───┐
    │   │  ── │   │      ← 中位数（红色粗线）
    └───┤     ├───┘
        │     │          ← Q1（下四分位数）和 Q3（上四分位数）之间的箱子
        └─────┘          ← 最小值（须的底端）
         ○   ○            ← 异常值（超出1.5倍IQR的数据点）
```

---

## 六、图表美化技巧

### 6.1 修改图表样式

matplotlib 内置了多种风格，一键切换：

```python
import matplotlib.pyplot as plt

# 查看所有可用样式
print(plt.style.available)

# 使用某一种样式
plt.style.use("seaborn-v0_8")  # 类似 seaborn 的风格，最推荐
# 其他选择：
# "ggplot"         - 类似 R 语言的 ggplot2
# "fivethirtyeight" -  FiveThirtyEight 网站风格
# "bmh"            - 贝叶斯黑客风格
# "dark_background" - 深色背景

plt.plot([1, 2, 3, 4], [10, 20, 25, 30])
plt.title("使用 seaborn 风格")
plt.show()
```

### 6.2 多条线 + 图例

```python
import matplotlib.pyplot as plt

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

months = list(range(1, 13))

# 三个产品线的数据
product_a = [120, 132, 101, 134, 90, 230, 210, 182, 191, 250, 220, 280]
product_b = [80, 95, 88, 102, 75, 160, 145, 130, 140, 180, 165, 200]
product_c = [50, 60, 55, 70, 48, 110, 95, 85, 90, 120, 108, 130]

plt.figure(figsize=(12, 6))
plt.plot(months, product_a, marker="o", linewidth=2, label="产品A")
plt.plot(months, product_b, marker="s", linewidth=2, label="产品B")
plt.plot(months, product_c, marker="^", linewidth=2, label="产品C")

plt.title("三条产品线的月度销售趋势", fontsize=16)
plt.xlabel("月份", fontsize=12)
plt.ylabel("销售额（万元）", fontsize=12)
plt.xticks(months)                    # X轴刻度设为1-12
plt.legend(fontsize=12, loc="upper left")  # 图例
plt.grid(True, alpha=0.3)
plt.show()
```

### 6.3 多子图（Subplots）

当需要在同一张画布上展示多个图表时：

```python
import matplotlib.pyplot as plt
import numpy as np

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

np.random.seed(42)

fig, axes = plt.subplots(2, 2, figsize=(12, 10))  # 2行2列

# 第1个子图：折线图
x = np.linspace(0, 10, 100)
axes[0, 0].plot(x, np.sin(x), color="#e74c3c")
axes[0, 0].set_title("正弦函数")
axes[0, 0].grid(True, alpha=0.3)

# 第2个子图：柱状图
categories = ["A", "B", "C", "D"]
values = [23, 45, 56, 78]
axes[0, 1].bar(categories, values, color="#3498db")
axes[0, 1].set_title("柱状图")

# 第3个子图：散点图
x = np.random.normal(0, 1, 200)
y = x + np.random.normal(0, 0.5, 200)
axes[1, 0].scatter(x, y, alpha=0.5, s=20, color="#2ecc71")
axes[1, 0].set_title("散点图")

# 第4个子图：饼图
sizes = [30, 25, 20, 15, 10]
axes[1, 1].pie(sizes, labels=["A", "B", "C", "D", "E"],
               autopct="%1.0f%%", startangle=90)
axes[1, 1].set_title("饼图")

# 调整子图之间的间距
plt.tight_layout()
plt.suptitle("四种常用图表类型", fontsize=18, y=1.02)
plt.show()
```

**关键方法：**
- `plt.subplots(行, 列)` —— 创建多子图布局，返回 `fig`（画布）和 `axes`（坐标系数组）
- `plt.tight_layout()` —— 自动调整子图间距，防止标签重叠
- `plt.suptitle()` —— 为整个画布添加总标题

### 6.4 保存图表

```python
plt.plot([1, 2, 3], [4, 5, 6])
plt.title("保存示例")

# 保存为 PNG 图片（300 DPI，适合打印）
plt.savefig("my_chart.png", dpi=300, bbox_inches="tight")

# 保存为 SVG 矢量图（适合网页嵌入，可无损缩放）
plt.savefig("my_chart.svg", bbox_inches="tight")

# 保存为 PDF
plt.savefig("my_chart.pdf", bbox_inches="tight")

plt.show()  # 保存后再 show()
```

> **注意：** `bbox_inches="tight"` 会自动裁剪空白区域。`savefig()` 必须在 `show()` 之前调用，因为 `show()` 会清空画布。

---

## 七、实战：为 Day 40 的数据画图

把 Day 40 的分析结果用图表展示出来：

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False

# ---- 生成数据（复用 Day 40 的代码） ----
np.random.seed(42)
n = 2000
df = pd.DataFrame({
    "下单时间": [datetime(2025, 1, 1) + timedelta(days=np.random.randint(0, 365))
                 for _ in range(n)],
    "商品类别": np.random.choice(["电子产品", "服装鞋帽", "家居用品",
                                    "食品饮料", "图书文具"], n),
    "订单金额": np.random.uniform(50, 3000, n),
    "城市": np.random.choice(["北京", "上海", "广州", "深圳",
                              "杭州", "成都", "武汉", "南京"], n),
    "支付方式": np.random.choice(["支付宝", "微信支付", "银行卡", "信用卡"], n)
})
df["下单时间"] = pd.to_datetime(df["下单时间"])
df["月份"] = df["下单时间"].dt.month

# ---- 创建 2x2 子图 ----
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
plt.style.use("seaborn-v0_8")

# ---- 图1：月度销售趋势（折线图） ----
monthly = df.groupby("月份")["订单金额"].sum()
months_zh = [f"{m}月" for m in range(1, 13)]
axes[0, 0].plot(months_zh, monthly, marker="o", linewidth=2.5,
                color="#e74c3c", markersize=8)
axes[0, 0].fill_between(months_zh, monthly, alpha=0.1, color="#e74c3c")
axes[0, 0].set_title("月度销售趋势", fontsize=14, fontweight="bold")
axes[0, 0].set_ylabel("销售额（元）")
axes[0, 0].tick_params(axis="x", rotation=45)
axes[0, 0].grid(True, alpha=0.3)

# ---- 图2：各类别销售额（柱状图） ----
cat_sales = df.groupby("商品类别")["订单金额"].sum().sort_values(ascending=True)
cat_sales.plot(kind="barh", ax=axes[0, 1], color="#3498db", edgecolor="white")
axes[0, 1].set_title("各类别销售额排名", fontsize=14, fontweight="bold")
axes[0, 1].set_xlabel("销售额（元）")

# ---- 图3：城市销售额（横向柱状图） ----
city_sales = df.groupby("城市")["订单金额"].sum().sort_values(ascending=True)
city_sales.plot(kind="barh", ax=axes[1, 0], color="#2ecc71", edgecolor="white")
axes[1, 0].set_title("各城市销售额排名", fontsize=14, fontweight="bold")
axes[1, 0].set_xlabel("销售额（元）")

# ---- 图4：支付方式占比（饼图） ----
payment_counts = df["支付方式"].value_counts()
colors = ["#1677ff", "#07c160", "#faad14", "#eb2f96"]
axes[1, 1].pie(payment_counts, labels=payment_counts.index,
               autopct="%1.1f%%", colors=colors, startangle=90,
               textprops={"fontsize": 11})
axes[1, 1].set_title("支付方式占比", fontsize=14, fontweight="bold")

# ---- 全局设置 ----
plt.tight_layout()
plt.savefig("sales_dashboard.png", dpi=150, bbox_inches="tight")
print("图表已保存：sales_dashboard.png")
plt.show()
```

运行后会生成一张包含四个子图的综合仪表盘图片。

---

## 八、颜色选择指南

好看的图表离不开合适的颜色。这里推荐几种配色方案：

### 8.1 常用单色

| 颜色名 | 色值 | 适用场景 |
|--------|------|---------|
| 警告红 | `#e74c3c` | 需要强调、警示 |
| 天空蓝 | `#3498db` | 主数据、中性展示 |
| 翡翠绿 | `#2ecc71` | 正向指标、增长 |
| 向日黄 | `#f39c12` | 辅助数据 |
| 皇室紫 | `#9b59b6` | 对比数据 |

### 8.2 matplotlib 内置色板

```python
# 使用 tab10 色板（适合分类数据，最多 10 种颜色）
plt.cm.tab10(0)  # 取第 1 种颜色
plt.cm.tab10(1)  # 取第 2 种颜色
# ... 以此类推

# 使用 Set2 色板（柔和配色）
plt.cm.Set2(np.linspace(0, 1, 5))  # 取 Set2 中的 5 种颜色
```

---

## 九、练习题

### 练习 1：基础（必做）

1. 用 matplotlib 画一张**柱状图**，展示你最近一周每天的学习时长（小时）
2. 在柱状图上添加**数值标签**（每个柱子顶部显示具体数字）
3. 设置合适的标题、坐标轴标签和颜色

### 练习 2：进阶

1. 用 `plt.subplots(1, 3)` 创建一行三列的子图布局
2. 子图1：折线图（用 `np.sin()` 和 `np.cos()` 画正弦和余弦曲线）
3. 子图2：柱状图（展示 5 个城市的温度对比）
4. 子图3：饼图（展示你一周时间分配：学习/工作/娱乐/运动/睡眠）

### 练习 3：挑战

1. 读取 Day 40 生成的 `sales_data_clean.csv`
2. 创建一个 **3x2** 的子图仪表盘，包含：
   - 月度销售趋势折线图
   - 各类别销售额柱状图
   - 用户消费金额分布直方图
   - 支付方式饼图
   - 城市销售额箱线图
   - 周几销售分布柱状图（提示：`df["下单时间"].dt.dayofweek` 提取星期几）
3. 保存为 `sales_full_dashboard.png`

---

## 十、常见问题

### Q1：为什么我的图表中文显示为方块？

这是因为 matplotlib 默认字体不支持中文。参考第四节的解决方案，添加这两行：

```python
plt.rcParams["font.sans-serif"] = ["SimHei"]
plt.rcParams["axes.unicode_minus"] = False
```

如果 SimHei 不行，试试 `["Microsoft YaHei"]`（微软雅黑）或 `["STSong"]`（华文宋体）。

### Q2：`plt.show()` 和 `plt.savefig()` 的顺序？

**`savefig()` 要在 `show()` 之前！** 因为 `show()` 会清空当前画布：

```python
plt.plot(x, y)
plt.savefig("chart.png")  # 先保存
plt.show()                 # 后显示
```

### Q3：图表太大/太小怎么办？

- 画布大小：`plt.figure(figsize=(宽, 高))`，单位是英寸
- 字体大小：在 `title()`、`xlabel()` 等中设置 `fontsize=14`
- 保存时清晰度：`plt.savefig("chart.png", dpi=300)`，DPI 越高越清晰

### Q4：matplotlib 和 seaborn 有什么区别？

| 特性 | matplotlib | seaborn |
|------|-----------|---------|
| 定位 | 基础绑图库 | 高级统计绑图库 |
| 美观度 | 默认一般 | 默认就很美观 |
| 统计图表 | 需要手动实现 | 内置 heatmap、violin 等 |
| 自定义 | 非常灵活 | 基于 matplotlib，也灵活 |
| 关系 | 独立运行 | 底层依赖 matplotlib |

> Day 42 会专门学习 seaborn。

---

## 十一、免费学习资源

- **matplotlib 官方文档**（最权威的参考）：https://matplotlib.org/stable/contents.html
- **matplotlib 图表画廊**（看别人怎么画）：https://matplotlib.org/stable/gallery/index.html
- **菜鸟教程 Matplotlib**：https://www.runoob.com/matplotlib/matplotlib-tutorial.html
- **廖雪峰 - Python 绑图**：https://www.liaoxuefeng.com/wiki/1016959663602400
- **Kaggle 免费课程 Data Visualization**：https://www.kaggle.com/learn/data-visualization

---

## 十二、下节预告

Day 42 我们学习 **seaborn** —— 一个基于 matplotlib 的高级可视化库。seaborn 代码更简洁，图表更美观，特别适合做统计分析和数据探索。matplotlib 打基础，seaborn 做升华，两者配合才是完整的可视化方案。

---

> **今日小结：**
> 今天学习了 matplotlib 的核心功能：6 种常用图表（折线图、柱状图、饼图、散点图、直方图、箱线图）、中文显示配置、图表美化技巧、多子图布局、图表保存。matplotlib 是 Python 可视化的基石，掌握它之后学习 seaborn 和其他可视化库会事半功倍。记住：**图表的目的是让数据更容易被理解，不是为了好看而好看**。
