# Python Day 45 — 综合数据处理项目

> **学习目标**：综合运用 NumPy、Pandas、Matplotlib、Seaborn 四大工具，从零完成一个真实感的数据分析项目——"城市空气质量分析与可视化"，体验完整的数据科学工作流。

---

## 一、项目背景与目标

### 1.1 项目简介

在前面的学习中，我们分别掌握了：
- **NumPy**（Day 43-44）：高性能数值计算
- **Pandas**（Day 38-40）：数据清洗与分析
- **Matplotlib**（Day 41）：基础绑图
- **Seaborn**（Day 42）：高级统计图表

今天，我们将这四件"兵器"合在一起，完成一个完整的数据分析项目。

### 1.2 项目任务

模拟一份 **2024 年某城市全年空气质量监测数据**，完成以下分析：

| 编号 | 任务 | 使用工具 |
|------|------|----------|
| 1 | 生成模拟数据 | NumPy + Pandas |
| 2 | 数据清洗与预处理 | Pandas |
| 3 | 统计分析（月度/季度/季节） | Pandas + NumPy |
| 4 | 可视化（折线图、热力图、分布图等） | Matplotlib + Seaborn |
| 5 | 输出分析报告 | Python 字符串拼接 |

---

## 二、Step 1：生成模拟数据

真实项目中数据来自数据库或文件，但为了练习，我们先用代码生成一份结构完整的模拟数据集。

```python
import numpy as np
import pandas as pd

# 设置随机种子，保证每次运行结果一致（方便调试）
np.random.seed(42)

# ---------- 时间范围：2024年全年 ----------
dates = pd.date_range(start="2024-01-01", end="2024-12-31", freq="D")
n = len(dates)  # 366天（2024是闰年）
print(f"总天数: {n}")

# ---------- 模拟六项空气质量指标 ----------
# PM2.5: 主要污染物，冬季偏高、夏季偏低
base_pm25 = 35 + 25 * np.cos(2 * np.pi * (np.arange(n) - 30) / 365)
pm25 = base_pm25 + np.random.normal(0, 12, n)
pm25 = np.clip(pm25, 5, 300)  # 限制在合理范围内

# PM10: 与 PM2.5 正相关
pm10 = pm25 * 1.6 + np.random.normal(0, 10, n)
pm10 = np.clip(pm10, 10, 400)

# SO2: 工业排放，波动较小
so2 = 15 + np.random.normal(0, 5, n)
so2 = np.clip(so2, 2, 80)

# NO2: 交通相关，工作日偏高
weekday_factor = np.array([1 if d.weekday() < 5 else 0.7 for d in dates])
no2 = 30 * weekday_factor + np.random.normal(0, 8, n)
no2 = np.clip(no2, 5, 120)

# CO: 冬季取暖导致偏高
base_co = 0.8 + 0.4 * np.cos(2 * np.pi * (np.arange(n) - 30) / 365)
co = base_co + np.random.normal(0, 0.2, n)
co = np.clip(co, 0.2, 5.0)

# O3: 与其他污染物相反，夏季偏高（光化学反应）
base_o3 = 80 - 40 * np.cos(2 * np.pi * (np.arange(n) - 30) / 365)
o3 = base_o3 + np.random.normal(0, 15, n)
o3 = np.clip(o3, 10, 250)

# ---------- 组装 DataFrame ----------
df = pd.DataFrame({
    "日期": dates,
    "PM2.5": np.round(pm25, 1),
    "PM10": np.round(pm10, 1),
    "SO2": np.round(so2, 1),
    "NO2": np.round(no2, 1),
    "CO": np.round(co, 2),
    "O3": np.round(o3, 1),
})

# 添加辅助列
df["月份"] = df["日期"].dt.month
df["星期"] = df["日期"].dt.day_name()
df["季节"] = df["月份"].map({
    3: "春季", 4: "春季", 5: "春季",
    6: "夏季", 7: "夏季", 8: "夏季",
    9: "秋季", 10: "秋季", 11: "秋季",
    12: "冬季", 1: "冬季", 2: "冬季",
})

print(df.head(10))
print(f"\n数据形状: {df.shape}")
print(f"列名: {list(df.columns)}")
```

**输出示例**：
```
         日期  PM2.5  PM10   SO2   NO2    CO     O3  月份    星期  季节
0 2024-01-01   33.5  55.2  17.3  28.1  1.17  42.3    1  Monday  冬季
1 2024-01-02   27.8  46.1  12.5  25.3  0.98  38.7    1 Tuesday  冬季
...
数据形状: (366, 10)
列名: ['日期', 'PM2.5', 'PM10', 'SO2', 'NO2', 'CO', 'O3', '月份', '星期', '季节']
```

> **要点**：`np.clip()` 是 NumPy 的裁剪函数，将数值限制在 `[min, max]` 范围内，非常适合处理模拟数据中的异常值。`pd.date_range` 快速生成日期序列。

---

## 三、Step 2：数据清洗与预处理

即使是模拟数据，我们也按真实项目的标准走一遍清洗流程。

```python
# ---------- 2.1 基本检查 ----------
print("=== 数据概览 ===")
print(df.info())
print("\n=== 描述性统计 ===")
print(df.describe())

# ---------- 2.2 缺失值检查 ----------
print("\n=== 缺失值统计 ===")
print(df.isnull().sum())

# 模拟数据一般没有缺失值，但真实项目中这是第一步
# 如果有缺失值，可以这样处理：
# df["PM2.5"] = df["PM2.5"].fillna(df["PM2.5"].mean())  # 均值填充
# df = df.dropna()  # 直接删除缺失行

# ---------- 2.3 异常值检测（IQR 方法）----------
def detect_outliers_iqr(series: pd.Series, name: str = "") -> pd.Series:
    """用 IQR（四分位距）方法检测异常值"""
    Q1 = series.quantile(0.25)
    Q3 = series.quantile(0.75)
    IQR = Q3 - Q1
    lower = Q1 - 1.5 * IQR
    upper = Q3 + 1.5 * IQR
    outliers = series[(series < lower) | (series > upper)]
    print(f"  {name}: 异常值 {len(outliers)} 个 (范围: {lower:.1f} ~ {upper:.1f})")
    return outliers

print("\n=== 异常值检测（IQR 法）===")
for col in ["PM2.5", "PM10", "SO2", "NO2", "CO", "O3"]:
    detect_outliers_iqr(df[col], col)

# ---------- 2.4 AQI 等级计算 ----------
# 根据中国空气质量标准，PM2.5 浓度划分等级
def get_aqi_level(pm25: float) -> str:
    """根据 PM2.5 值返回空气质量等级"""
    if pm25 <= 35:
        return "优"
    elif pm25 <= 75:
        return "良"
    elif pm25 <= 115:
        return "轻度污染"
    elif pm25 <= 150:
        return "中度污染"
    elif pm25 <= 250:
        return "重度污染"
    else:
        return "严重污染"

df["等级"] = df["PM2.5"].apply(get_aqi_level)

# 查看各等级天数
print("\n=== 空气质量等级分布 ===")
print(df["等级"].value_counts())
```

> **要点**：IQR 是异常值检测的经典方法。Q1（第25百分位）到 Q3（第75百分位）为"正常区间"，超出 `[Q1-1.5*IQR, Q3+1.5*IQR]` 的值视为异常值。这是一种无监督方法，不需要标注数据。

---

## 四、Step 3：统计分析

### 3.1 月度趋势分析

```python
# ---------- 3.1 月度均值 ----------
monthly = df.groupby("月份")[["PM2.5", "PM10", "SO2", "NO2", "CO", "O3"]].mean()
print("=== 月度均值 ===")
print(monthly.round(1))

# ---------- 3.2 季度对比 ----------
seasonal = df.groupby("季节")[["PM2.5", "PM10", "SO2", "NO2", "CO", "O3"]].mean()
# 按季节自然顺序排列
season_order = ["春季", "夏季", "秋季", "冬季"]
seasonal = seasonal.reindex(season_order)
print("\n=== 季度均值 ===")
print(seasonal.round(1))

# ---------- 3.3 工作日 vs 周末 ----------
df["是否周末"] = df["星期"].isin(["Saturday", "Sunday"])
weekday_compare = df.groupby("是否周末")[["PM2.5", "PM10", "NO2", "O3"]].mean()
print("\n=== 工作日 vs 周末 ===")
print(weekday_compare.round(1))

# ---------- 3.4 相关性分析 ----------
pollutants = ["PM2.5", "PM10", "SO2", "NO2", "CO", "O3"]
corr = df[pollutants].corr()
print("\n=== 污染物相关性矩阵 ===")
print(corr.round(2))
```

### 3.2 用 NumPy 加速计算

```python
# ---------- 用 NumPy 计算年度统计 ----------
pm25_values = df["PM2.5"].values  # 转为 NumPy 数组

print(f"PM2.5 年度均值: {np.mean(pm25_values):.1f}")
print(f"PM2.5 年度标准差: {np.std(pm25_values):.1f}")
print(f"PM2.5 最大值: {np.max(pm25_values):.1f}（出现在第 {np.argmax(pm25_values) + 1} 天）")
print(f"PM2.5 最小值: {np.min(pm25_values):.1f}")
print(f"PM2.5 中位数: {np.median(pm25_values):.1f}")

# 优（<=35）的天数占比
good_days = np.sum(pm25_values <= 35)
total_days = len(pm25_values)
print(f"\n全年优良天数: {good_days} 天，占比: {good_days / total_days * 100:.1f}%")
```

> **要点**：将 Pandas 的 `Series` 转为 NumPy 数组（`.values` 属性）后，可以用 `np.mean()` 等函数计算。对于简单统计，Pandas 和 NumPy 性能差异不大；但当数据量达到百万级时，NumPy 的批量运算优势明显。

---

## 五、Step 4：数据可视化

### 4.1 设置全局样式

```python
import matplotlib.pyplot as plt
import seaborn as sns

# 设置中文字体（Windows 系统）
plt.rcParams["font.sans-serif"] = ["SimHei", "Microsoft YaHei"]
plt.rcParams["axes.unicode_minus"] = False  # 解决负号显示问题

# Seaborn 全局主题
sns.set_theme(style="whitegrid", palette="muted", font_scale=1.1)
```

### 4.2 图 1：全年 PM2.5 趋势图

```python
fig, ax = plt.subplots(figsize=(14, 5))

ax.plot(df["日期"], df["PM2.5"], color="steelblue", alpha=0.5, linewidth=0.8, label="每日值")

# 添加 7 天移动平均线，让趋势更清晰
df["PM2.5_MA7"] = df["PM2.5"].rolling(window=7, center=True).mean()
ax.plot(df["日期"], df["PM2.5_MA7"], color="red", linewidth=2, label="7日移动平均")

# 添加空气质量等级分界线
ax.axhline(y=35, color="green", linestyle="--", alpha=0.6, label="优 (≤35)")
ax.axhline(y=75, color="orange", linestyle="--", alpha=0.6, label="良 (≤75)")
ax.axhline(y=115, color="red", linestyle="--", alpha=0.6, label="轻度污染 (≤115)")

ax.set_title("2024年 PM2.5 全年趋势", fontsize=16, fontweight="bold")
ax.set_xlabel("日期")
ax.set_ylabel("PM2.5 (μg/m³)")
ax.legend(loc="upper right")
plt.tight_layout()
plt.savefig("chart1_pm25_trend.png", dpi=150)
plt.show()
```

### 4.3 图 2：月度均值对比柱状图

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# 左图：PM2.5 和 PM10 的月度均值
monthly[["PM2.5", "PM10"]].plot(
    kind="bar", ax=axes[0], color=["steelblue", "coral"], edgecolor="white"
)
axes[0].set_title("PM2.5 与 PM10 月度均值")
axes[0].set_xlabel("月份")
axes[0].set_ylabel("浓度 (μg/m³)")
axes[0].legend(title="指标")
axes[0].set_xticklabels(axes[0].get_xticklabels(), rotation=0)

# 右图：O3 的月度均值
monthly["O3"].plot(kind="bar", ax=axes[1], color="limegreen", edgecolor="white")
axes[1].set_title("O3 月度均值")
axes[1].set_xlabel("月份")
axes[1].set_ylabel("O3 (μg/m³)")
axes[1].set_xticklabels(axes[1].get_xticklabels(), rotation=0)

plt.suptitle("2024年月度污染物均值对比", fontsize=16, fontweight="bold", y=1.02)
plt.tight_layout()
plt.savefig("chart2_monthly_bar.png", dpi=150)
plt.show()
```

### 4.4 图 3：污染物相关性热力图

```python
fig, ax = plt.subplots(figsize=(8, 6))

sns.heatmap(
    corr,
    annot=True,        # 显示数值
    fmt=".2f",         # 保留两位小数
    cmap="RdBu_r",     # 红蓝配色（正相关红色，负相关蓝色）
    center=0,          # 以 0 为中心
    square=True,       # 正方形单元格
    linewidths=0.5,
    ax=ax
)
ax.set_title("污染物相关性矩阵", fontsize=14, fontweight="bold")
plt.tight_layout()
plt.savefig("chart3_heatmap.png", dpi=150)
plt.show()
```

### 4.5 图 4：季节箱线图

```python
fig, ax = plt.subplots(figsize=(10, 6))

# 准备数据：将宽表转为长表格式
df_melted = df.melt(
    id_vars=["季节"],
    value_vars=["PM2.5", "PM10", "NO2", "O3"],
    var_name="污染物",
    value_name="浓度"
)

sns.boxplot(
    data=df_melted,
    x="污染物",
    y="浓度",
    hue="季节",
    hue_order=season_order,
    palette=["#4CAF50", "#FF9800", "#FF5722", "#2196F3"],  # 春夏秋冬颜色
    ax=ax
)
ax.set_title("各污染物季节分布对比", fontsize=14, fontweight="bold")
ax.set_ylabel("浓度")
plt.tight_layout()
plt.savefig("chart4_season_box.png", dpi=150)
plt.show()
```

### 4.6 图 5：空气质量等级饼图

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 左图：全年等级分布
level_counts = df["等级"].value_counts()
level_order = ["优", "良", "轻度污染", "中度污染", "重度污染", "严重污染"]
level_counts = level_counts.reindex(level_order).dropna()

colors = ["#4CAF50", "#8BC34A", "#FFEB3B", "#FF9800", "#F44336", "#9C27B0"]
axes[0].pie(
    level_counts,
    labels=level_counts.index,
    autopct="%1.1f%%",
    colors=colors[:len(level_counts)],
    startangle=90,
    textprops={"fontsize": 10}
)
axes[0].set_title("全年空气质量等级分布")

# 右图：各季节优良率
season_good_rate = df.groupby("季节").apply(
    lambda x: (x["PM2.5"] <= 35).sum() / len(x) * 100
).reindex(season_order)
season_good_rate.plot(
    kind="bar", ax=axes[1],
    color=["#4CAF50", "#8BC34A", "#FF9800", "#2196F3"],
    edgecolor="white"
)
axes[1].set_title("各季节优良天数占比 (%)")
axes[1].set_ylabel("优良率 (%)")
axes[1].set_xticklabels(axes[1].get_xticklabels(), rotation=0)
for i, v in enumerate(season_good_rate):
    axes[1].text(i, v + 1, f"{v:.1f}%", ha="center", fontsize=10)

plt.suptitle("2024年空气质量总览", fontsize=16, fontweight="bold", y=1.02)
plt.tight_layout()
plt.savefig("chart5_overview.png", dpi=150)
plt.show()
```

---

## 六、Step 5：生成分析报告

把分析结论整理成结构化的 Markdown 报告。

```python
# ---------- 汇总关键指标 ----------
total_days = len(df)
good_days = (df["PM2.5"] <= 35).sum()
excellent_days = (df["PM2.5"] <= 75).sum()
worst_month = monthly["PM2.5"].idxmax()
best_month = monthly["PM2.5"].idxmin()
worst_season = seasonal["PM2.5"].idxmax()

# 生成报告
report = f"""# 2024年城市空气质量分析报告

## 一、数据概况

- 数据时间范围：2024-01-01 至 2024-12-31（共 {total_days} 天）
- 监测指标：PM2.5、PM10、SO2、NO2、CO、O3 共 6 项
- 数据来源：模拟数据（用于学习练习）

## 二、关键发现

### 2.1 全年总体表现

| 指标 | 数值 |
|------|------|
| PM2.5 年均值 | {df['PM2.5'].mean():.1f} μg/m³ |
| PM2.5 最高日均值 | {df['PM2.5'].max():.1f} μg/m³ |
| PM2.5 最低日均值 | {df['PM2.5'].min():.1f} μg/m³ |
| 优良天数（PM2.5≤75） | {excellent_days} 天（{excellent_days/total_days*100:.1f}%） |
| 优天数（PM2.5≤35） | {good_days} 天（{good_days/total_days*100:.1f}%） |

### 2.2 月度与季节趋势

- PM2.5 最高的月份：{worst_month}月（均值 {monthly.loc[worst_month, 'PM2.5']:.1f} μg/m³）
- PM2.5 最低的月份：{best_month}月（均值 {monthly.loc[best_month, 'PM2.5']:.1f} μg/m³）
- 空气质量最差的季节：{worst_season}（PM2.5 均值 {seasonal.loc[worst_season, 'PM2.5']:.1f} μg/m³）

### 2.3 污染物相关性

- PM2.5 与 PM10 高度正相关（r = {corr.loc['PM2.5', 'PM10']:.2f}），说明颗粒物来源一致
- O3 与其他污染物呈负相关，夏季光化学反应活跃导致 O3 升高
- 工作日 NO2 明显高于周末，验证了交通排放的影响

### 2.4 工作日 vs 周末

- NO2 工作日均值：{weekday_compare.loc[False, 'NO2']:.1f} μg/m³
- NO2 周末均值：{weekday_compare.loc[True, 'NO2']:.1f} μg/m³
- 差异约 {(weekday_compare.loc[False, 'NO2'] - weekday_compare.loc[True, 'NO2']):.1f} μg/m³，交通排放特征明显

## 三、可视化图表

1. **PM2.5 全年趋势图**：展示了日波动和7日均线趋势
2. **月度均值柱状图**：PM2.5/PM10 与 O3 的月度对比
3. **相关性热力图**：6种污染物之间的相关系数矩阵
4. **季节箱线图**：各污染物在不同季节的分布差异
5. **空气质量总览**：等级饼图 + 季节优良率

## 四、结论与建议

1. **冬季是空气污染防控重点**：{worst_season}PM2.5 均值最高，需加强监测
2. **夏季注意臭氧污染**：O3 在夏季显著升高，与 PM2.5 呈反向趋势
3. **交通管控有明确效果**：工作日 NO2 高于周末，建议重点区域限行
4. **优良天数占比 {excellent_days/total_days*100:.1f}%**：距离全年优良率 80% 的目标 {'已达成 ✓' if excellent_days/total_days >= 0.8 else '尚有差距，需持续治理'}

---
*报告生成时间：2024年度分析*
*工具：Python + NumPy + Pandas + Matplotlib + Seaborn*
"""

# 保存报告
with open("air_quality_report_2024.md", "w", encoding="utf-8") as f:
    f.write(report)

print("报告已生成: air_quality_report_2024.md")
print(report[:500])  # 预览前500字
```

---

## 七、完整代码整合

以上步骤可以整合到一个脚本中，方便一次性运行：

```python
"""
城市空气质量分析项目 — 完整脚本
整合 NumPy + Pandas + Matplotlib + Seaborn
"""

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# ============ 全局设置 ============
plt.rcParams["font.sans-serif"] = ["SimHei", "Microsoft YaHei"]
plt.rcParams["axes.unicode_minus"] = False
sns.set_theme(style="whitegrid", palette="muted", font_scale=1.1)
np.random.seed(42)

# ============ Step 1: 生成数据 ============
dates = pd.date_range(start="2024-01-01", end="2024-12-31", freq="D")
n = len(dates)
weekday_factor = np.array([1 if d.weekday() < 5 else 0.7 for d in dates])
cos_vals = np.cos(2 * np.pi * (np.arange(n) - 30) / 365)

df = pd.DataFrame({
    "日期": dates,
    "PM2.5": np.clip(35 + 25 * cos_vals + np.random.normal(0, 12, n), 5, 300).round(1),
    "PM10":  np.clip((35 + 25 * cos_vals) * 1.6 + np.random.normal(0, 10, n), 10, 400).round(1),
    "SO2":  np.clip(15 + np.random.normal(0, 5, n), 2, 80).round(1),
    "NO2":  np.clip(30 * weekday_factor + np.random.normal(0, 8, n), 5, 120).round(1),
    "CO":   np.clip(0.8 + 0.4 * cos_vals + np.random.normal(0, 0.2, n), 0.2, 5.0).round(2),
    "O3":   np.clip(80 - 40 * cos_vals + np.random.normal(0, 15, n), 10, 250).round(1),
})

df["月份"] = df["日期"].dt.month
df["季节"] = df["月份"].map({3:"春季",4:"春季",5:"春季",6:"夏季",7:"夏季",8:"夏季",
                              9:"秋季",10:"秋季",11:"秋季",12:"冬季",1:"冬季",2:"冬季"})

# ============ Step 2: 数据清洗 ============
df["等级"] = df["PM2.5"].apply(
    lambda x: "优" if x<=35 else "良" if x<=75 else "轻度污染" if x<=115
              else "中度污染" if x<=150 else "重度污染" if x<=250 else "严重污染"
)
df["PM2.5_MA7"] = df["PM2.5"].rolling(7, center=True).mean()

# ============ Step 3: 统计分析 ============
monthly = df.groupby("月份")[["PM2.5","PM10","SO2","NO2","CO","O3"]].mean()
season_order = ["春季", "夏季", "秋季", "冬季"]
seasonal = df.groupby("季节")[["PM2.5","PM10","SO2","NO2","CO","O3"]].mean().reindex(season_order)
corr = df[["PM2.5","PM10","SO2","NO2","CO","O3"]].corr()

print("=== 季度均值 ===")
print(seasonal.round(1))
print("\n=== 全年优良率 ===")
good_rate = (df["PM2.5"] <= 75).sum() / len(df) * 100
print(f"优良天数占比: {good_rate:.1f}%")

# ============ Step 4: 可视化 ============
fig, axes = plt.subplots(2, 3, figsize=(18, 10))

# (0,0) PM2.5 趋势
ax = axes[0, 0]
ax.plot(df["日期"], df["PM2.5"], alpha=0.4, linewidth=0.6, color="steelblue")
ax.plot(df["日期"], df["PM2.5_MA7"], color="red", linewidth=1.5)
ax.axhline(35, color="green", linestyle="--", alpha=0.5)
ax.axhline(75, color="orange", linestyle="--", alpha=0.5)
ax.set_title("PM2.5 全年趋势")

# (0,1) 月度均值柱状图
ax = axes[0, 1]
monthly[["PM2.5","O3"]].plot(kind="bar", ax=ax, color=["steelblue","limegreen"])
ax.set_title("月度均值 (PM2.5 vs O3)")
ax.set_xticklabels(ax.get_xticklabels(), rotation=0)

# (0,2) 相关性热力图
ax = axes[0, 2]
sns.heatmap(corr, annot=True, fmt=".2f", cmap="RdBu_r", ax=ax, square=True, linewidths=0.3)
ax.set_title("污染物相关性")

# (1,0) 季节箱线图
ax = axes[1, 0]
sns.boxplot(data=df, x="季节", y="PM2.5", order=season_order, ax=ax,
            palette=["#4CAF50","#FF9800","#FF5722","#2196F3"])
ax.set_title("PM2.5 季节分布")

# (1,1) 等级饼图
ax = axes[1, 1]
level_counts = df["等级"].value_counts().reindex(
    ["优","良","轻度污染","中度污染","重度污染","严重污染"]
).dropna()
ax.pie(level_counts, labels=level_counts.index, autopct="%1.1f%%",
       colors=["#4CAF50","#8BC34A","#FFEB3B","#FF9800","#F44336","#9C27B0"],
       textprops={"fontsize":9}, startangle=90)
ax.set_title("空气质量等级分布")

# (1,2) 散点图：PM2.5 vs O3
ax = axes[1, 2]
for season, color in zip(season_order, ["#4CAF50","#FF9800","#FF5722","#2196F3"]):
    subset = df[df["季节"] == season]
    ax.scatter(subset["PM2.5"], subset["O3"], alpha=0.3, s=15, label=season, color=color)
ax.set_xlabel("PM2.5")
ax.set_ylabel("O3")
ax.set_title("PM2.5 vs O3 散点图")
ax.legend(fontsize=9)

plt.suptitle("2024年城市空气质量综合分析仪表盘", fontsize=18, fontweight="bold", y=1.01)
plt.tight_layout()
plt.savefig("air_quality_dashboard.png", dpi=150, bbox_inches="tight")
plt.show()
print("\n仪表盘已保存: air_quality_dashboard.png")
```

运行以上完整脚本，你会得到一张包含 6 个子图的**综合分析仪表盘**。

---

## 八、练习题

### 练习 1（基础）

在上面的模拟数据中，找出 **PM2.5 最高的 10 天**，看看它们集中在哪个季节、哪几个月。

```python
# 提示：用 nlargest() 方法
top10 = df.nlargest(10, "PM2.5")[["日期", "PM2.5", "季节", "月份"]]
print(top10)
print(f"\nPM2.5 最高的 10 天中，{top10['季节'].mode().iloc[0]}有 {top10['季节'].value_counts().iloc[0]} 天")
```

### 练习 2（进阶）

为每种污染物分别生成一张**月度趋势图**（共 6 张子图），用 `plt.subplots(3, 2)` 排列，观察每种污染物的季节规律。

```python
# 提示：循环遍历污染物列表，在对应的 axes 上绘图
fig, axes = plt.subplots(3, 2, figsize=(14, 12))
pollutants = ["PM2.5", "PM10", "SO2", "NO2", "CO", "O3"]
colors = ["steelblue", "coral", "purple", "teal", "brown", "limegreen"]

for idx, (col, color) in enumerate(zip(pollutants, colors)):
    ax = axes[idx // 2, idx % 2]
    monthly[col].plot(kind="bar", ax=ax, color=color, edgecolor="white")
    ax.set_title(f"{col} 月度均值")
    ax.set_xticklabels(ax.get_xticklabels(), rotation=0)

plt.tight_layout()
plt.show()
```

### 练习 3（挑战）

扩展项目：**加入第二个城市的数据**，假设城市 B 的整体空气质量比城市 A 差 20%，然后生成两张城市的 PM2.5 对比图和统计对比表。

```python
# 提示：复制一份 df，乘以 1.2 的系数（加一些随机噪声避免完全线性）
df_b = df.copy()
df_b["PM2.5"] = df["PM2.5"] * 1.2 + np.random.normal(0, 3, len(df))
# 然后用月度均值画对比柱状图
```

---

## 九、常见问题

### Q1：图表中文字体显示为方块怎么办？

这是中文字体未正确配置的问题。Windows 系统在脚本开头加入：

```python
plt.rcParams["font.sans-serif"] = ["SimHei", "Microsoft YaHei"]
plt.rcParams["axes.unicode_minus"] = False  # 解决负号显示问题
```

如果 SimHei 不可用，可以用 `matplotlib.font_manager` 查找可用字体：

```python
from matplotlib.font_manager import fontManager
for f in fontManager.ttflist:
    if "SimHei" in f.name or "YaHei" in f.name:
        print(f.name, f.fname)
```

### Q2：`plt.show()` 弹出窗口后代码会暂停吗？

是的，`plt.show()` 会阻塞程序执行，直到关闭图表窗口。如果不想阻塞，可以在脚本开头设置：

```python
import matplotlib
matplotlib.use("Agg")  # 非交互式后端，不弹窗
```

这样图表只会保存到文件，不会弹出窗口。在 Jupyter Notebook 中则不受影响。

### Q3：数据量很大时绘图很慢怎么办？

- 用 `df.head(10000)` 或 `df.sample(10000)` 采样后再绘图
- 折线图数据点超过 1 万个时，可以先用 `resample("W").mean()` 按周聚合
- 避免 `plt.show()` 循环弹出多个窗口，用 `plt.savefig()` 批量保存

### Q4：为什么要用 `rolling().mean()` 计算移动平均？

原始每日数据波动很大（噪声多），移动平均能平滑这些波动，让整体趋势更清晰。这是时间序列分析中最常用的基础技巧。窗口越大越平滑，但滞后也越大。

### Q5：Seaborn 和 Matplotlib 能混用吗？

完全可以。Seaborn 本质上是对 Matplotlib 的高级封装，它的返回值往往是 Matplotlib 的 `Axes` 对象。你可以用 Seaborn 画图，再用 Matplotlib 的 `ax.set_title()` 等方法做微调，两种工具互补。

---

## 十、今日小结

| 知识点 | 对应工具 |
|--------|----------|
| 数值模拟与计算 | NumPy（clip, cos, random, mean, std） |
| 数据结构与管理 | Pandas（DataFrame, groupby, rolling, melt） |
| 基础绑图 | Matplotlib（plot, bar, pie, subplots） |
| 统计图表 | Seaborn（heatmap, boxplot, scatter） |
| 项目整合 | 脚本组织、报告生成 |

到这里，**数据科学基础五天（Day 41-45）全部完成**！你已经具备了用 Python 进行完整数据分析的能力。

---

## 十一、免费学习资源

- **NumPy 官方文档**：https://numpy.org/doc/stable/user/quickstart.html
- **Pandas 官方教程**：https://pandas.pydata.org/docs/getting_started/intro_tutorials/
- **Matplotlib 官方画廊**：https://matplotlib.org/stable/gallery/index.html（海量图表模板）
- **Seaborn 官方教程**：https://seaborn.pydata.org/tutorial.html
- **菜鸟教程 - NumPy**：https://www.runoob.com/numpy/numpy-tutorial.html
- **菜鸟教程 - Pandas**：https://www.runoob.com/pandas/pandas-tutorial.html

---

> **Day 46 预告**：进入第五阶段——Web 开发！将学习 **FastAPI 入门**，用 Python 构建高性能 Web API。

> *坚持到第 45 天，你已经掌握了 Python 数据分析的核心工具链。接下来的 Web 开发阶段会让你把数据处理能力转化为可交互的 Web 应用。加油！*
