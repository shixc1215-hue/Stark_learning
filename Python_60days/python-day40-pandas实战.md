# Python Day 40：pandas 实战 —— 从真实数据到分析报告

> Day 38 学了 pandas 入门，Day 39 深入了数据清洗与转换。今天是 pandas 模块的**收官之战：实战演练**。我们将用 pandas 完成一个完整的数据分析项目——从原始数据出发，经过清洗、分析、可视化，最终产出一份分析报告。这是你在 60 天计划中第一次做真正的"数据分析"。

---

## 一、回顾与过渡

Day 39 我们学了这些核心技能：

| 操作 | 方法 | 示例 |
|------|------|------|
| 数据类型转换 | `astype()`、`pd.to_numeric()` | `df["age"] = df["age"].astype(int)` |
| 字符串处理 | `.str` 访问器 | `df["name"].str.upper()` |
| 合并连接 | `merge()`、`concat()` | `pd.merge(df1, df2, on="id")` |
| 数据透视 | `pivot_table()`、`melt()` | `df.pivot_table(index="city", values="salary")` |
| 窗口函数 | `rolling()`、`shift()` | `df["price"].rolling(7).mean()` |

今天的目标：**把前面学过的 pandas 知识串联起来，做一个完整项目**。

---

## 二、项目概述：电商销售数据分析

### 项目背景

假设你是一家电商公司的数据分析师，拿到了一份过去 12 个月的**销售订单数据**（CSV 文件）。你的任务是：

1. **清洗数据** —— 处理缺失值、重复值、异常值
2. **探索性分析** —— 了解数据全貌
3. **业务分析** —— 回答关键业务问题
4. **生成报告** —— 用 pandas 输出汇总表格

### 我们会回答的问题

| 编号 | 业务问题 | 涉及的 pandas 技能 |
|------|---------|-------------------|
| Q1 | 哪个月销售额最高？ | `groupby` + 聚合 |
| Q2 | 哪个商品类别贡献最大？ | `groupby` + 排序 |
| Q3 | 用户消费金额分布如何？ | `describe` + 分箱 |
| Q4 | 有多少回头客（下单超过 1 次）？ | `value_counts` + 筛选 |
| Q5 | 月度销售趋势如何？ | `resample` + 时间索引 |

---

## 三、第一步：创建模拟数据集

由于我们还没有真实数据文件，先用 pandas 自己生成一份模拟数据集：

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# 设置随机种子，保证结果可复现
np.random.seed(42)

# ---- 生成基础数据 ----
n_orders = 2000  # 共 2000 条订单

# 日期范围：2025-01-01 到 2025-12-31
start_date = datetime(2025, 1, 1)
end_date = datetime(2025, 12, 31)
dates = [start_date + timedelta(days=np.random.randint(0, 365)) for _ in range(n_orders)]

# 商品类别
categories = ["电子产品", "服装鞋帽", "家居用品", "食品饮料", "图书文具"]
# 城市
cities = ["北京", "上海", "广州", "深圳", "杭州", "成都", "武汉", "南京"]
# 支付方式
payment_methods = ["支付宝", "微信支付", "银行卡", "信用卡"]

# 生成 DataFrame
df = pd.DataFrame({
    "订单ID": [f"ORD{str(i+10001).zfill(5)}" for i in range(n_orders)],
    "下单时间": dates,
    "用户ID": [f"U{np.random.randint(1, 501)}" for _ in range(n_orders)],
    "商品类别": np.random.choice(categories, n_orders),
    "商品名称": np.random.choice([
        "蓝牙耳机", "手机壳", "运动鞋", "保温杯", "零食大礼包",
        "Python编程书", "笔记本电脑", "T恤", "台灯", "咖啡豆",
        "键盘", "背包", "充电宝", "雨伞", "笔记本"
    ], n_orders),
    "数量": np.random.randint(1, 6, n_orders),
    "单价": np.round(np.random.uniform(10, 2000, n_orders), 2),
    "城市": np.random.choice(cities, n_orders),
    "支付方式": np.random.choice(payment_methods, n_orders)
})

# 计算订单金额 = 数量 × 单价
df["订单金额"] = df["数量"] * df["单价"]

# ---- 有意制造一些"脏数据"，方便练习清洗 ----
# 1. 制造缺失值：随机将 5% 的"城市"设为 NaN
mask = np.random.random(n_orders) < 0.05
df.loc[mask, "城市"] = np.nan

# 2. 制造重复订单：复制前 20 条追加到末尾
df = pd.concat([df, df.head(20)], ignore_index=True)

# 3. 制造异常值：随机将 10 条订单金额设为负数
outlier_idx = np.random.choice(df.index, 10, replace=False)
df.loc[outlier_idx, "订单金额"] = df.loc[outlier_idx, "订单金额"] * -1

# 4. 制造格式不一致："用户ID"列有些是大写有些是小写
mask2 = np.random.random(len(df)) < 0.1
df.loc[mask2, "用户ID"] = df.loc[mask2, "用户ID"].str.lower()

# 保存到 CSV
df.to_csv("sales_data_raw.csv", index=False, encoding="utf-8-sig")
print(f"原始数据已生成：{len(df)} 条记录")
print(df.head())
```

运行后会生成一个 `sales_data_raw.csv` 文件，里面有大约 2020 条数据，包含缺失值、重复行、异常值等问题。

---

## 四、第二步：数据清洗

数据清洗是数据分析中**最耗时**的环节，通常占整个分析工作的 60%-80%。我们来一步步处理。

```python
import pandas as pd
import numpy as np

# ---- 读取原始数据 ----
df = pd.read_csv("sales_data_raw.csv")
print(f"原始数据：{len(df)} 行, {len(df.columns)} 列")

# ---- 4.1 查看数据概况 ----
print("\n=== 数据概况 ===")
print(df.info())          # 数据类型 + 非空数量
print("\n=== 前 5 行 ===")
print(df.head())
print("\n=== 统计摘要 ===")
print(df.describe())      # 数值列的统计信息
print("\n=== 缺失值统计 ===")
print(df.isnull().sum())  # 每列有多少缺失值

# ---- 4.2 处理重复数据 ----
dup_count = df.duplicated().sum()
print(f"\n重复行数量：{dup_count}")
df = df.drop_duplicates()
print(f"删除重复后：{len(df)} 行")

# ---- 4.3 处理缺失值 ----
# 城市列缺失值：用众数（出现最多的城市）填充
city_mode = df["城市"].mode()[0]
print(f"城市列众数：{city_mode}")
df["城市"] = df["城市"].fillna(city_mode)
print(f"填充后缺失值数量：{df['城市'].isnull().sum()}")

# ---- 4.4 处理异常值 ----
# 订单金额为负数的，取绝对值修正（假设是录入错误）
negative_count = (df["订单金额"] < 0).sum()
print(f"\n负数订单金额：{negative_count} 条")
df["订单金额"] = df["订单金额"].abs()

# ---- 4.5 数据格式统一 ----
# 用户ID统一为大写
df["用户ID"] = df["用户ID"].str.upper()

# ---- 4.6 数据类型转换 ----
# "下单时间"转为 datetime 类型
df["下单时间"] = pd.to_datetime(df["下单时间"])
# 提取月份，方便后续分析
df["月份"] = df["下单时间"].dt.month
df["日期"] = df["下单时间"].dt.date

# ---- 4.7 验证清洗结果 ----
print("\n=== 清洗后数据概况 ===")
print(f"总记录数：{len(df)}")
print(f"缺失值总数：{df.isnull().sum().sum()}")
print(f"重复行数：{df.duplicated().sum()}")
print(f"订单金额范围：{df['订单金额'].min():.2f} ~ {df['订单金额'].max():.2f}")

# 保存清洗后的数据
df.to_csv("sales_data_clean.csv", index=False, encoding="utf-8-sig")
print("清洗后数据已保存：sales_data_clean.csv")
```

### 清洗步骤总结

| 步骤 | 操作 | 方法 | 说明 |
|------|------|------|------|
| 1 | 查看概况 | `info()`、`describe()` | 先了解数据全貌 |
| 2 | 删除重复 | `drop_duplicates()` | 去除完全相同的行 |
| 3 | 填充缺失 | `fillna()` | 用众数填充城市缺失值 |
| 4 | 修正异常 | `.abs()` | 修正负数金额 |
| 5 | 格式统一 | `.str.upper()` | 用户ID统一大写 |
| 6 | 类型转换 | `pd.to_datetime()` | 时间列转为日期类型 |

---

## 五、第三步：探索性数据分析（EDA）

清洗完数据后，先做一个全面的**探索性分析**，了解数据的整体特征。

```python
import pandas as pd

df = pd.read_csv("sales_data_clean.csv")
df["下单时间"] = pd.to_datetime(df["下单时间"])

# ---- 5.1 基本统计 ----
total_revenue = df["订单金额"].sum()
total_orders = len(df)
avg_order = df["订单金额"].mean()
unique_users = df["用户ID"].nunique()

print("=" * 50)
print("         销售数据总览")
print("=" * 50)
print(f"总订单数：{total_orders:,} 单")
print(f"总销售额：¥{total_revenue:,.2f}")
print(f"平均客单价：¥{avg_order:,.2f}")
print(f"总用户数：{unique_users:,} 人")
print(f"人均消费：¥{total_revenue / unique_users:,.2f}")

# ---- 5.2 各类别销售情况 ----
category_stats = df.groupby("商品类别").agg(
    订单数=("订单ID", "count"),
    总销售额=("订单金额", "sum"),
    平均单价=("订单金额", "mean"),
    总数量=("数量", "sum")
).sort_values("总销售额", ascending=False)

category_stats["销售额占比"] = (
    category_stats["总销售额"] / category_stats["总销售额"].sum() * 100
).round(2)

print("\n=== 各商品类别销售情况 ===")
print(category_stats.to_string())

# ---- 5.3 城市销售排名 ----
city_stats = df.groupby("城市").agg(
    订单数=("订单ID", "count"),
    总销售额=("订单金额", "sum")
).sort_values("总销售额", ascending=False)

city_stats["销售额占比"] = (
    city_stats["总销售额"] / city_stats["总销售额"].sum() * 100
).round(2)

print("\n=== 各城市销售排名（TOP 8）===")
print(city_stats.to_string())

# ---- 5.4 用户消费分布 ----
user_stats = df.groupby("用户ID").agg(
    下单次数=("订单ID", "count"),
    总消费=("订单金额", "sum"),
    平均客单价=("订单金额", "mean")
).sort_values("总消费", ascending=False)

print("\n=== TOP 10 消费用户 ===")
print(user_stats.head(10).to_string())

# 回头客分析
repeat_users = (user_stats["下单次数"] > 1).sum()
print(f"\n回头客数量：{repeat_users} 人（占比 {repeat_users / len(user_stats) * 100:.1f}%）")
```

---

## 六、第四步：回答关键业务问题

### Q1：哪个月销售额最高？

```python
import pandas as pd

df = pd.read_csv("sales_data_clean.csv")
df["下单时间"] = pd.to_datetime(df["下单时间"])
df["月份"] = df["下单时间"].dt.month

# 按月份汇总
monthly_sales = df.groupby("月份").agg(
    订单数=("订单ID", "count"),
    总销售额=("订单金额", "sum"),
    平均客单价=("订单金额", "mean")
)

month_names = ["1月", "2月", "3月", "4月", "5月", "6月",
               "7月", "8月", "9月", "10月", "11月", "12月"]
monthly_sales.index = month_names

print("=== 月度销售数据 ===")
print(monthly_sales.to_string())

# 找出最佳月份
best_month = monthly_sales["总销售额"].idxmax()
best_revenue = monthly_sales["总销售额"].max()
print(f"\n销售冠军月份：{best_month}，销售额 ¥{best_revenue:,.2f}")

# 环比增长率
monthly_sales["环比增长率"] = monthly_sales["总销售额"].pct_change() * 100
monthly_sales["环比增长率"] = monthly_sales["环比增长率"].round(2)
print("\n=== 含环比增长率 ===")
print(monthly_sales.to_string())
```

**关键方法说明：**
- `pct_change()` —— 计算环比增长率，每个值与前一个值相比的变化百分比

### Q2：哪个商品类别贡献最大？

```python
# 按类别统计
category_sales = df.groupby("商品类别")["订单金额"].sum()
category_pct = (category_sales / category_sales.sum() * 100).round(2)

# 按贡献从大到小排列
result = category_pct.sort_values(ascending=False)
print("=== 各类别销售额占比 ===")
for cat, pct in result.items():
    bar = "█" * int(pct / 2)  # 简易柱状图
    print(f"{cat:8s}  {pct:5.1f}%  {bar}")
```

输出效果类似：

```
=== 各类别销售额占比 ===
电子产品  22.3%  ██████████
服装鞋帽  20.1%  █████████
家居用品  19.8%  █████████
食品饮料  18.9%  ████████
图书文具  18.9%  ████████
```

### Q3：用户消费金额分布如何？

```python
user_spending = df.groupby("用户ID")["订单金额"].sum()

# 基本统计
print("=== 用户消费金额分布 ===")
print(user_spending.describe())

# 消费分箱（将用户按消费金额分组）
bins = [0, 500, 1000, 3000, 5000, 10000, float("inf")]
labels = ["0-500", "500-1K", "1K-3K", "3K-5K", "5K-10K", "10K+"]

user_spending_bin = pd.cut(user_spending, bins=bins, labels=labels)
distribution = user_spending_bin.value_counts().sort_index()

print("\n=== 消费金额分段 ===")
for label, count in distribution.items():
    pct = count / len(user_spending) * 100
    bar = "█" * int(pct)
    print(f"{label:8s}  {count:4d} 人  ({pct:5.1f}%)  {bar}")
```

### Q4：有多少回头客？

```python
user_orders = df.groupby("用户ID")["订单ID"].count()

# 用户下单次数分布
print("=== 用户下单次数分布 ===")
print(user_orders.value_counts().sort_index())

# 计算复购率
total_users = len(user_orders)
repeat = (user_orders > 1).sum()
first_time = (user_orders == 1).sum()
loyal = (user_orders >= 5).sum()

print(f"\n总用户数：{total_users}")
print(f"首次购买用户：{first_time}（{first_time/total_users*100:.1f}%）")
print(f"回头客（>1次）：{repeat}（{repeat/total_users*100:.1f}%）")
print(f"忠实用户（>=5次）：{loyal}（{loyal/total_users*100:.1f}%）")
print(f"复购率：{repeat/total_users*100:.1f}%")
```

### Q5：月度销售趋势

```python
# 设置日期为索引
df_indexed = df.set_index("下单时间")

# 按月重采样（resample）
monthly_trend = df_indexed["订单金额"].resample("M").agg(["count", "sum", "mean"])
monthly_trend.columns = ["订单数", "总销售额", "平均客单价"]
monthly_trend.index = monthly_trend.index.strftime("%Y-%m")

print("=== 月度销售趋势 ===")
print(monthly_trend.to_string())

# 计算移动平均（3个月）
monthly_trend["3月移动平均"] = monthly_trend["总销售额"].rolling(3).mean().round(2)
print("\n=== 含 3 个月移动平均 ===")
print(monthly_trend.to_string())
```

**关键方法说明：**
- `set_index("下单时间")` —— 将日期列设为索引，是时间序列分析的前提
- `resample("M")` —— 按月重新采样（M = Month），类似 SQL 的 `GROUP BY month` 但更灵活
- `rolling(3).mean()` —— 计算滚动平均值，用于观察趋势、平滑波动

---

## 七、第五步：生成分析报告

最后，将分析结果整理成一份结构化的报告：

```python
import pandas as pd
from datetime import datetime

df = pd.read_csv("sales_data_clean.csv")
df["下单时间"] = pd.to_datetime(df["下单时间"])
df["月份"] = df["下单时间"].dt.month

report_lines = []
report_lines.append(f"# 电商销售数据分析报告")
report_lines.append(f"\n> 报告生成时间：{datetime.now().strftime('%Y-%m-%d %H:%M')}")
report_lines.append(f"\n> 数据范围：2025-01-01 至 2025-12-31")
report_lines.append("")

# ---- 概览 ----
report_lines.append("## 一、数据概览")
report_lines.append("")
total = df["订单金额"].sum()
orders = len(df)
users = df["用户ID"].nunique()
report_lines.append(f"| 指标 | 数值 |")
report_lines.append(f"|------|------|")
report_lines.append(f"| 总订单数 | {orders:,} |")
report_lines.append(f"| 总销售额 | ¥{total:,.2f} |")
report_lines.append(f"| 平均客单价 | ¥{df['订单金额'].mean():,.2f} |")
report_lines.append(f"| 总用户数 | {users:,} |")
report_lines.append(f"| 人均消费 | ¥{total/users:,.2f} |")

# ---- 类别分析 ----
report_lines.append("")
report_lines.append("## 二、商品类别分析")
report_lines.append("")
cat = df.groupby("商品类别")["订单金额"].sum().sort_values(ascending=False)
report_lines.append("| 类别 | 销售额 | 占比 |")
report_lines.append("|------|--------|------|")
for name, val in cat.items():
    pct = val / cat.sum() * 100
    report_lines.append(f"| {name} | ¥{val:,.2f} | {pct:.1f}% |")

# ---- 城市排名 ----
report_lines.append("")
report_lines.append("## 三、城市销售排名")
report_lines.append("")
city = df.groupby("城市")["订单金额"].sum().sort_values(ascending=False)
report_lines.append("| 排名 | 城市 | 销售额 |")
report_lines.append("|------|------|--------|")
for i, (name, val) in enumerate(city.items(), 1):
    report_lines.append(f"| {i} | {name} | ¥{val:,.2f} |")

# ---- 用户分析 ----
report_lines.append("")
report_lines.append("## 四、用户分析")
report_lines.append("")
user_orders = df.groupby("用户ID")["订单ID"].count()
repeat = (user_orders > 1).sum()
report_lines.append(f"- 总用户数：{users:,}")
report_lines.append(f"- 回头客数量：{repeat}（复购率 {repeat/users*100:.1f}%）")
report_lines.append(f"- TOP 消费用户：")
top3 = df.groupby("用户ID")["订单金额"].sum().nlargest(3)
for uid, amt in top3.items():
    report_lines.append(f"  - {uid}：¥{amt:,.2f}")

# ---- 保存报告 ----
report = "\n".join(report_lines)

with open("sales_analysis_report.md", "w", encoding="utf-8") as f:
    f.write(report)

print("分析报告已生成：sales_analysis_report.md")
print("\n" + "=" * 40)
print(report)
```

---

## 八、完整项目代码（一次性运行）

把上面所有步骤合到一起，就是一个完整的分析流程：

```python
"""
电商销售数据分析 - 完整项目
============================
流程：生成数据 → 清洗 → 分析 → 生成报告
"""

import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# ============ 第1步：生成模拟数据 ============
np.random.seed(42)
n = 2000

df = pd.DataFrame({
    "订单ID": [f"ORD{str(i+10001).zfill(5)}" for i in range(n)],
    "下单时间": [datetime(2025, 1, 1) + timedelta(days=np.random.randint(0, 365))
                 for _ in range(n)],
    "用户ID": [f"U{np.random.randint(1, 501)}" for _ in range(n)],
    "商品类别": np.random.choice(["电子产品", "服装鞋帽", "家居用品",
                                    "食品饮料", "图书文具"], n),
    "数量": np.random.randint(1, 6, n),
    "单价": np.round(np.random.uniform(10, 2000, n), 2),
    "城市": np.random.choice(["北京", "上海", "广州", "深圳",
                              "杭州", "成都", "武汉", "南京"], n),
    "支付方式": np.random.choice(["支付宝", "微信支付", "银行卡", "信用卡"], n)
})
df["订单金额"] = df["数量"] * df["单价"]

# 制造脏数据
df.loc[np.random.random(n) < 0.05, "城市"] = np.nan
df = pd.concat([df, df.head(20)], ignore_index=True)
outlier_idx = np.random.choice(df.index, 10, replace=False)
df.loc[outlier_idx, "订单金额"] *= -1

# ============ 第2步：数据清洗 ============
df.drop_duplicates(inplace=True)
df["城市"] = df["城市"].fillna(df["城市"].mode()[0])
df["订单金额"] = df["订单金额"].abs()
df["用户ID"] = df["用户ID"].str.upper()
df["下单时间"] = pd.to_datetime(df["下单时间"])
df["月份"] = df["下单时间"].dt.month

print(f"✓ 清洗完成：{len(df)} 条有效数据")

# ============ 第3步：分析 ============
total = df["订单金额"].sum()
users = df["用户ID"].nunique()

print(f"\n{'='*45}")
print(f"  总销售额：¥{total:,.2f}")
print(f"  总订单数：{len(df):,}")
print(f"  总用户数：{users:,}")
print(f"  复购率：{(df.groupby('用户ID').size().gt(1).sum() / users * 100):.1f}%")
print(f"{'='*45}")

# ============ 第4步：保存清洗后数据 ============
df.to_csv("sales_data_clean.csv", index=False, encoding="utf-8-sig")
print("\n✓ 数据文件：sales_data_clean.csv")
print("✓ 分析完成！")
```

---

## 九、pandas 分析的常见思维模式

这个项目用到了 pandas 分析的几种**核心思维模式**，值得记住：

### 1. 拆解-聚合-组合（Split-Apply-Combine）

```
原始数据 → 按某列分组(groupby) → 对每组做计算(agg) → 合并结果
```

```python
# 几乎所有分析问题的核心套路
df.groupby("分组列").agg(
    新列名=("计算列", "计算方法")
)
```

### 2. 链式操作

```python
# 一行代码完成多步操作
result = (
    df[df["订单金额"] > 100]          # 筛选
    .groupby("商品类别")              # 分组
    .agg(销售额=("订单金额", "sum"))  # 聚合
    .sort_values("销售额", ascending=False)  # 排序
    .head(5)                          # 取前5
)
```

### 3. 时间序列思维

```python
# 先设日期索引 → 再用 resample/rolling 等时间方法
df.set_index("下单时间")["订单金额"].resample("M").sum()
```

---

## 十、练习题

### 练习 1：基础（必做）

基于上面的项目代码，完成以下分析：

1. 统计每种**支付方式**的使用次数和总金额
2. 找出**订单金额最大**的 5 笔订单
3. 计算每个**城市**的平均客单价，找出最高和最低的

### 练习 2：进阶

1. 分析**各季度**的销售表现（提示：用 `df["下单时间"].dt.quarter` 提取季度）
2. 计算**用户平均下单间隔天数**（提示：对每个用户按日期排序后用 `diff()`）
3. 找出**连续 3 天都有下单**的用户

### 练习 3：挑战

尝试用 pandas 分析你自己的数据——比如：
- 导出你的微信/支付宝账单 CSV
- 用 pandas 分析每月消费趋势、消费类别分布
- 找出你的"消费习惯"

---

## 十一、常见问题

### Q1：pandas 处理大数据时会卡吗？

pandas 默认把所有数据加载到内存中。数据量超过几 GB 时确实会卡。解决方案：
- 只读取需要的列：`pd.read_csv("file.csv", usecols=["col1", "col2"])`
- 分块读取：`pd.read_csv("file.csv", chunksize=10000)`
- 或者使用 **Dask** 库（后续进阶内容会涉及）

### Q2：groupby 之后的数据怎么还原？

`groupby()` 返回的是 GroupBy 对象，需要跟聚合函数一起使用：

```python
# ❌ 错误：groupby 本身不返回 DataFrame
df.groupby("类别")  # 这是一个 GroupBy 对象

# ✅ 正确：必须跟聚合函数
df.groupby("类别").sum()        # 求和
df.groupby("类别").mean()       # 平均
df.groupby("类别").agg(...)     # 自定义聚合
```

### Q3：`merge` 和 `concat` 什么时候用哪个？

| 场景 | 使用方法 | 说明 |
|------|---------|------|
| 纵向拼接（行变多） | `pd.concat([df1, df2])` | 列名相同的表上下拼起来 |
| 横向拼接（列变多） | `pd.concat([df1, df2], axis=1)` | 按索引左右拼起来 |
| 按某个键关联 | `pd.merge(df1, df2, on="id")` | 类似 SQL 的 JOIN |
| 按索引关联 | `df1.join(df2)` | 类似 merge，但用索引作为键 |

---

## 十二、免费学习资源

- **pandas 官方文档**（最权威的参考）：https://pandas.pydata.org/docs/
- **pandas 10分钟入门**：https://pandas.pydata.org/docs/user_guide/10min.html
- **Kaggle 免费课程 Pandas**：https://www.kaggle.com/learn/pandas （英文，有练习）
- **廖雪峰 - Python 实例**：https://www.liaoxuefeng.com/wiki/1016959663602400
- **菜鸟教程 Pandas**：https://www.runoob.com/pandas/pandas-tutorial.html
- **和鲸社区数据分析实战**：https://www.heywhale.com/home （中文，有大量真实项目案例）

---

## 十三、下节预告

Day 41 我们进入可视化领域——学习 **matplotlib** 绘图库。有了 pandas 做数据处理，再配合 matplotlib 做图表展示，你的数据分析能力将大幅提升。今天生成的销售数据集，到时候正好用 matplotlib 画出来！

---

> **今日小结：**
> 今天完成了一个完整的 pandas 数据分析项目：生成数据 → 清洗 → 探索分析 → 回答业务问题 → 生成报告。你学会了数据分析的**标准流程**和 pandas 的**核心思维模式**（拆解-聚合-组合、链式操作、时间序列分析）。从零散的技能到完整的项目实战，这是从"会写代码"到"能做事情"的关键一步。
