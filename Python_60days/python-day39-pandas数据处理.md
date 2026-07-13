# Python Day 39：pandas 数据处理 —— 数据清洗与转换的核心技能

> Day 38 我们学会了 pandas 的基础操作——创建 DataFrame、读取文件、筛选数据、分组统计、处理缺失值。今天进入 pandas 的**核心战场：数据清洗与转换**。在实际工作中，原始数据往往千疮百孔——重复行、格式混乱、列名不规范、数据类型不对……掌握今天的内容，你就能把"脏数据"变成"干净可用"的数据。

---

## 一、回顾与过渡

Day 38 我们学了这些：

| 操作 | 方法 | 示例 |
|------|------|------|
| 读取 CSV | `pd.read_csv()` | `df = pd.read_csv("data.csv")` |
| 查看数据 | `head() / info() / describe()` | `df.head(5)` |
| 选择列 | `df[列名]` 或 `df[["列1", "列2"]]` | `df[["name", "age"]]` |
| 条件筛选 | `df[条件]` | `df[df["age"] > 18]` |
| 分组统计 | `groupby() + agg()` | `df.groupby("city").mean()` |
| 缺失值处理 | `fillna() / dropna()` | `df.fillna(0)` |

今天要深入学习的主题：

1. **数据类型转换** —— `astype()`、`pd.to_numeric()`、`pd.to_datetime()`
2. **字符串处理** —— `.str` 访问器的强大方法
3. **合并与连接** —— `merge()`、`concat()`、`join()`
4. **数据透视** —— `pivot_table()`、`melt()`、`crosstab()`
5. **apply 高级应用** —— 对行/列自定义函数处理
6. **窗口函数** —— `rolling()`、`shift()`、`rank()`
7. **综合实战** —— 从"脏数据"到"分析报告"

---

## 二、数据类型转换

现实中的数据经常"类型不对"——数字被读成了字符串、日期成了 object 类型。pandas 提供了灵活的类型转换工具。

### 2.1 查看当前数据类型

```python
import pandas as pd

# 创建示例数据（模拟从 CSV 读入的"脏数据"）
df = pd.DataFrame({
    "商品编码": ["001", "002", "003", "004", "005"],
    "价格": ["29.9", "59.9", "128.0", "15.5", "299.9"],
    "销量": ["100", "200", "50", "300", "150"],
    "日期": ["2026-01-15", "2026-02-20", "2026-03-10", "2026-04-05", "2026-05-18"],
    "是否促销": ["是", "否", "是", "否", "是"]
})

print(df.dtypes)
# 商品编码     object（字符串）
# 价格        object（字符串，但实际是数字！）
# 销量        object（字符串，但实际是数字！）
# 日期        object（字符串，但实际是日期！）
# 是否促销     object（字符串，但实际是布尔值！）
```

> **问题**：所有列都是 `object`（字符串）类型，无法做数值计算或日期排序。

### 2.2 astype() —— 基础类型转换

```python
# 把价格和销量转为数值类型
df["价格"] = df["价格"].astype("float64")   # 转为浮点数
df["销量"] = df["销量"].astype("int64")      # 转为整数

print(df["价格"].mean())  # 现在可以计算平均价格了！
# 输出：106.64

print(df["销量"].sum())   # 可以统计总销量！
# 输出：800
```

### 2.3 pd.to_numeric() —— 容错转换（推荐）

`astype()` 遇到无法转换的值会直接报错。`pd.to_numeric()` 可以设置容错模式：

```python
# 模拟有脏数据的列
df_dirty = pd.DataFrame({
    "价格": ["29.9", "免费", "128.0", "N/A", "299.9"]
})

# 方式1：errors="coerce" —— 无法转换的变成 NaN
df_dirty["价格_数值"] = pd.to_numeric(df_dirty["价格"], errors="coerce")
print(df_dirty)
#    价格   价格_数值
# 0  29.9    29.9
# 1   免费    NaN
# 2  128.0  128.0
# 3   N/A    NaN
# 4  299.9  299.9

# 方式2：errors="ignore" —— 无法转换的保持原样（不推荐）
# 方式3：不设置 errors —— 遇到脏数据直接报 ValueError
```

### 2.4 pd.to_datetime() —— 日期转换

```python
# 把字符串日期转为 datetime 类型
df["日期"] = pd.to_datetime(df["日期"])

# 转换后可以做日期计算
print(df["日期"].dtype)         # datetime64[ns]
print(df["日期"].max())         # 2026-05-18 00:00:00

# 提取年、月、日
df["年份"] = df["日期"].dt.year
df["月份"] = df["日期"].dt.month
df["星期"] = df["日期"].dt.day_name()

print(df[["日期", "年份", "月份", "星期"]])
```

### 2.5 map() + 字典 —— 分类数据转换

```python
# 把"是/否"映射为 True/False
df["是否促销"] = df["是否促销"].map({"是": True, "否": False})

# 或者用 replace（不会对不匹配的值产生 NaN）
df["是否促销"] = df["是否促销"].replace({"是": True, "否": False})
```

> **map vs replace**：`map()` 不在字典中的键会变成 NaN；`replace()` 只替换匹配的值，其余保持不变。不确定数据是否干净时，优先用 `replace()`。

---

## 三、字符串处理 —— .str 访问器

pandas 的 `.str` 访问器让你像操作普通字符串一样，对整列数据进行批量处理。

### 3.1 常用字符串方法

```python
df = pd.DataFrame({
    "姓名": ["张三", "李四 ", " 王五", "赵六abc", "  钱七  "],
    "邮箱": ["zhangsan@qq.com", "LISI@163.COM", "wangwu@gmail.com", "invalid", "qianqi@qq.com"],
    "手机号": ["13812345678", "159-8765-4321", "187 6543 2100", "12345678901", "17698765432"]
})

# 去除两端空白
df["姓名"] = df["姓名"].str.strip()
print(df["姓名"])
# 0     张三
# 1     李四
# 2     王五
# 3    赵六abc
# 4    钱七

# 全部转小写
df["邮箱_小写"] = df["邮箱"].str.lower()
# 0    zhangsan@qq.com
# 1    lisi@163.com
# 2    wangwu@gmail.com
# ...

# 判断是否包含某个子串
df["邮箱有效"] = df["邮箱"].str.contains("@", na=False)
# 0     True
# 1     True
# 2     True
# 3    False
# 4     True

# 提取手机号中的纯数字（去掉横杠和空格）
df["手机号_干净"] = df["手机号"].str.replace(r"\D", "", regex=True)
# 0    13812345678
# 1    15987654321
# 2    18765432100
# 3    12345678901
# 4    17698765432
```

### 3.2 正则提取与分割

```python
# 用正则提取邮箱的用户名部分（@ 之前的内容）
df["邮箱用户"] = df["邮箱"].str.extract(r"(.+)@")

# 用正则验证手机号格式（1开头，11位数字）
df["手机号合法"] = df["手机号_干净"].str.match(r"^1\d{10}$")

# 按分隔符拆分
df2 = pd.DataFrame({
    "地址": ["北京市-朝阳区-建国路88号", "上海市-浦东新区-世纪大道1号"]
})
# expand=True 返回 DataFrame
df2[["省", "市", "详细"]] = df2["地址"].str.split("-", expand=True)
print(df2)
```

### 3.3 字符串方法速查表

| 方法 | 作用 | 示例 |
|------|------|------|
| `.str.strip()` | 去两端空白 | `" hello ".strip()` → `"hello"` |
| `.str.lower()` | 转小写 | `"ABC".lower()` → `"abc"` |
| `.str.upper()` | 转大写 | `"abc".upper()` → `"ABC"` |
| `.str.contains()` | 是否包含子串 | 包含则 True |
| `.str.startswith()` | 是否以某字符串开头 | |
| `.str.endswith()` | 是否以某字符串结尾 | |
| `.str.replace()` | 替换 | 支持正则 |
| `.str.extract()` | 正则提取 | 返回匹配组 |
| `.str.split()` | 分割字符串 | `expand=True` 返回多列 |
| `.str.len()` | 字符串长度 | |
| `.str.cat()` | 拼接字符串 | |
| `.str.slice()` | 切片 | 类似 `[start:stop]` |

---

## 四、数据合并与连接

数据分析经常需要把多张表"拼"在一起。pandas 提供了三种合并方式。

### 4.1 concat() —— 纵向/横向拼接

```python
# 纵向拼接（行合并，类似 SQL 的 UNION ALL）
df_q1 = pd.DataFrame({
    "月份": ["1月", "2月", "3月"],
    "销售额": [10000, 12000, 15000]
})
df_q2 = pd.DataFrame({
    "月份": ["4月", "5月", "6月"],
    "销售额": [13000, 16000, 18000]
})

result = pd.concat([df_q1, df_q2], ignore_index=True)
print(result)
#   月份    销售额
# 0  1月  10000
# 1  2月  12000
# 2  3月  15000
# 3  4月  13000
# 4  5月  16000
# 5  6月  18000

# 横向拼接（列合并）
df_info = pd.DataFrame({
    "姓名": ["张三", "李四"],
    "部门": ["技术部", "市场部"]
})
df_salary = pd.DataFrame({
    "姓名": ["张三", "李四"],
    "月薪": [15000, 12000]
})

result2 = pd.concat([df_info, df_salary], axis=1)  # axis=1 表示按列拼接
```

### 4.2 merge() —— 类似 SQL 的 JOIN

```python
# 创建两张表
df_orders = pd.DataFrame({
    "订单号": ["O001", "O002", "O003", "O004"],
    "客户ID": ["C01", "C02", "C01", "C03"],
    "金额": [500, 300, 800, 200]
})
df_customers = pd.DataFrame({
    "客户ID": ["C01", "C02", "C04"],
    "客户名": ["张三", "李四", "王五"],
    "城市": ["北京", "上海", "广州"]
})

# 内连接（inner join）—— 只保留两表都有的客户
inner = pd.merge(df_orders, df_customers, on="客户ID", how="inner")
print(inner)
#   订单号 客户ID  金额 客户名 城市
# 0  O001   C01  500   张三  北京
# 1  O002   C02  300   李四  上海
# 2  O003   C01  800   张三  北京

# 左连接（left join）—— 保留左表所有行，右表没有的填 NaN
left = pd.merge(df_orders, df_customers, on="客户ID", how="left")
# C03 的客户名和城市为 NaN

# 外连接（outer join）—— 保留所有行
outer = pd.merge(df_orders, df_customers, on="客户ID", how="outer")

# 使用不同的列名进行连接（left_on / right_on）
df_a = pd.DataFrame({"key": [1, 2], "value": ["a", "b"]})
df_b = pd.DataFrame({"id": [1, 2], "score": [90, 85]})
pd.merge(df_a, df_b, left_on="key", right_on="id")
```

### 4.3 join() —— 基于索引的连接

```python
# join 基于 DataFrame 的索引进行连接（适合索引即键的情况）
df1 = pd.DataFrame({"A": [1, 2]}, index=["K01", "K02"])
df2 = pd.DataFrame({"B": [3, 4]}, index=["K01", "K03"])

df1.join(df2, how="outer")
#       A    B
# K01  1.0  3.0
# K02  2.0  NaN
# K03  NaN  4.0
```

### 4.4 四种连接方式对比

| 方式 | SQL 等价 | 说明 |
|------|----------|------|
| `inner` | INNER JOIN | 只保留两表都匹配的行 |
| `left` | LEFT JOIN | 保留左表全部，右表无匹配填 NaN |
| `right` | RIGHT JOIN | 保留右表全部，左表无匹配填 NaN |
| `outer` | FULL OUTER JOIN | 保留所有行，无匹配填 NaN |

---

## 五、数据透视 —— reshape

### 5.1 pivot_table() —— 数据透视表

类似 Excel 的数据透视表，按行/列分组后做聚合计算。

```python
df_sales = pd.DataFrame({
    "城市": ["北京", "北京", "上海", "上海", "广州", "广州",
            "北京", "上海", "广州", "北京"],
    "品类": ["手机", "电脑", "手机", "电脑", "手机", "电脑",
            "手机", "手机", "电脑", "电脑"],
    "销售额": [5000, 8000, 4500, 7000, 3000, 5500, 6200, 5100, 4800, 9000],
    "数量": [10, 8, 9, 7, 6, 5, 12, 10, 8, 10]
})

# 按城市和品类，汇总销售额
pivot = df_sales.pivot_table(
    values="销售额",    # 要聚合的值
    index="城市",        # 行分组
    columns="品类",      # 列分组
    aggfunc="sum",       # 聚合方式（sum/mean/count/min/max）
    fill_value=0,        # 缺失值填充
    margins=True,        # 显示行/列总计
    margins_name="合计"
)

print(pivot)
# 品类    手机     电脑     合计
# 城市
# 北京   11200  17000  28200
# 广州    3000  10300  13300
# 上海    9600   7000  16600
# 合计   23800  34300  58100

# 同时查看多个指标
pivot2 = df_sales.pivot_table(
    values=["销售额", "数量"],
    index="城市",
    columns="品类",
    aggfunc="sum"
)
```

### 5.2 melt() —— 宽表转长表（逆透视）

把"宽"的表变成"长"的表，适合做后续分析或绘图。

```python
# 宽表格式
df_wide = pd.DataFrame({
    "学生": ["张三", "李四", "王五"],
    "语文": [90, 85, 78],
    "数学": [95, 88, 92],
    "英语": [80, 76, 85]
})

# 转为长表格式
df_long = df_wide.melt(
    id_vars=["学生"],       # 保持不变的列
    value_vars=["语文", "数学", "英语"],  # 要"融化"的列
    var_name="科目",         # 新列名：原来的列名
    value_name="成绩"        # 新列名：原来的值
)

print(df_long)
#    学生 科目  成绩
# 0  张三  语文   90
# 1  李四  语文   85
# 2  王五  语文   78
# 3  张三  数学   95
# 4  李四  数学   88
# 5  王五  数学   92
# 6  张三  英语   80
# 7  李四  英语   76
# 8  王五  英语   85
```

### 5.3 crosstab() —— 交叉表

快速统计两个分类变量的频次分布：

```python
# 统计每个城市每个品类的订单数量
ct = pd.crosstab(
    df_sales["城市"],
    df_sales["品类"],
    margins=True,
    margins_name="合计"
)
print(ct)
# 品类  电脑  手机  合计
# 城市
# 北京    3    2    5
# 广州    2    1    3
# 上海    1    3    4
# 合计    6    6   12

# 添加 normalize 参数可显示比例
ct_pct = pd.crosstab(df_sales["城市"], df_sales["品类"], normalize="all")
```

---

## 六、apply() —— 自定义函数处理

当你需要的操作没有内置方法时，`apply()` 让你对每行/每列施加自定义函数。

### 6.1 对列应用函数

```python
df = pd.DataFrame({
    "姓名": ["张三", "李四", "王五", "赵六"],
    "语文": [90, 85, 78, 92],
    "数学": [95, 88, 92, 76],
    "英语": [80, 76, 85, 88]
})

# 对每列应用函数
df[["语文", "数学", "英语"]].apply(lambda x: x.max())
# 语文    92
# 数学    95
# 英语    88

# 计算每科的标准差
df[["语文", "数学", "英语"]].apply(lambda x: x.std())
```

### 6.2 对行应用函数

```python
# 计算每个人的总分和平均分
df["总分"] = df[["语文", "数学", "英语"]].apply(lambda row: row.sum(), axis=1)
df["平均分"] = df[["语文", "数学", "英语"]].apply(lambda row: row.mean(), axis=1)

# 或者更简洁的写法（行求和直接用 sum(1)）
df["总分"] = df[["语文", "数学", "英语"]].sum(axis=1)

# 根据分数评等级
def get_grade(row):
    avg = row[["语文", "数学", "英语"]].mean()
    if avg >= 90:
        return "优秀"
    elif avg >= 80:
        return "良好"
    elif avg >= 70:
        return "中等"
    else:
        return "需努力"

df["等级"] = df.apply(get_grade, axis=1)
print(df[["姓名", "总分", "平均分", "等级"]])
```

### 6.3 apply vs 向量化操作

```python
# 慢的方式（apply，逐行处理）
df["总分_慢"] = df[["语文", "数学", "英语"]].apply(lambda row: row.sum(), axis=1)

# 快的方式（向量化操作，推荐！）
df["总分_快"] = df["语文"] + df["数学"] + df["英语"]

# 向量化操作通常比 apply 快 10-100 倍！
# 优先使用向量化操作，只有没有现成方法时才用 apply
```

> **性能提示**：能用内置方法就不要用 apply，能用向量化操作就不要用 apply。apply 的本质是 Python 循环，大数据量时会很慢。

---

## 七、窗口函数

窗口函数让你能"看到"相邻行的数据，做滑动计算。在时间序列分析中极为常用。

### 7.1 rolling() —— 滑动窗口

```python
import pandas as pd

# 模拟30天的销售数据
dates = pd.date_range("2026-01-01", periods=30)
sales = [100 + i * 5 + (i % 7) * 10 for i in range(30)]  # 有波动的数据
df = pd.DataFrame({"日期": dates, "销售额": sales})

# 7天移动平均（平滑数据，看清趋势）
df["7日均线"] = df["销售额"].rolling(window=7).mean()

# 7天滑动最大值
df["7日最高"] = df["销售额"].rolling(window=7).max()

# 7天滑动标准差（衡量波动性）
df["7日波动"] = df["销售额"].rolling(window=7).std()

print(df[["日期", "销售额", "7日均线"]].head(15))
```

### 7.2 shift() —— 数据移位

```python
# 获取前一天的数据（用于计算环比）
df["前一天销售额"] = df["销售额"].shift(1)

# 计算日环比增长率
df["环比增长率"] = (df["销售额"] / df["销售额"].shift(1) - 1) * 100
# 第一天会是 NaN（因为没有前一天的数据）

# 获取后一天的数据
df["后一天销售额"] = df["销售额"].shift(-1)
```

### 7.3 rank() —— 排名

```python
df_students = pd.DataFrame({
    "姓名": ["张三", "李四", "王五", "赵六", "钱七"],
    "成绩": [95, 88, 92, 88, 76]
})

# 按成绩排名（默认升序，分数越高排名越靠后）
df_students["排名"] = df_students["成绩"].rank(ascending=False)
print(df_students)
#    姓名  成绩  排名
# 0  张三   95  1.0
# 2  王五   92  2.0
# 1  李四   88  3.5   ← 并列第3（同分取平均）
# 3  赵六   88  3.5
# 4  钱七   76  5.0

# method="min" 并列取最小名次；"first" 按出现顺序
df_students["排名_最早"] = df_students["成绩"].rank(method="first", ascending=False)
```

### 7.4 cumsum() / cummax() —— 累计计算

```python
# 累计销售额
df["累计销售额"] = df["销售额"].cumsum()

# 累计最大值
df["历史最高"] = df["销售额"].cummax()

# 累计计数
df["累计天数"] = range(1, len(df) + 1)
```

---

## 八、综合实战：电商销售数据清洗与分析

让我们把今天学的所有内容串联起来，完成一个完整的数据分析流程。

### 8.1 创建模拟的"脏数据"

```python
import pandas as pd
import numpy as np

# 模拟从系统导出的原始数据（包含各种"脏数据"问题）
raw_data = """
订单号,客户姓名,商品,单价,数量,下单日期,支付金额,省份
ORD-001, 张三 ,iPhone 15,5999.00,2,2026/01/15,11998.00,北京
ORD-002,李四,MacBook Pro,12999,1,2026/01/16,12999,上海
ORD-003, 王五 ,AirPods,1299.00,3,2026-01-17,NaN,广东
ORD-004,赵六,iPad Air,4799,NaN,2026/01/18,9598,北京
ORD-005,  张三  ,iPhone 15,5999,1,2026/01/19,,上海
ORD-006,李四,,8999.00,1,2026/01/20,8999,浙江
ORD-007,王五,Apple Watch,2999.00,2,2026/01/21,5998,广东
ORD-008,赵六,MacBook Pro,12999.00,1,2026-01-22,12999.00,北京
ORD-009,钱七,iPad Air,4799,2,2026/01/23,9598,江苏
ORD-010,张三,AirPods,1299,1,2026/01/24,1299,上海
ORD-011,钱七,,5999,1,2026/01/25,5999,浙江
ORD-012,王五,iPhone 15,5999,1,2026/01/26,5999,广东
"""

# 读取数据
from io import StringIO
df = pd.read_csv(StringIO(raw_data.strip()))
print(df.head())
print("\n数据形状:", df.shape)
print("\n各列数据类型:\n", df.dtypes)
print("\n缺失值统计:\n", df.isna().sum())
```

### 8.2 数据清洗流程

```python
# ========== 第1步：清洗字符串列 ==========
# 去除客户姓名两端空白
df["客户姓名"] = df["客户姓名"].str.strip()

# 去除商品名两端空白
df["商品"] = df["商品"].str.strip()

# ========== 第2步：类型转换 ==========
# 单价转为数值（容错处理）
df["单价"] = pd.to_numeric(df["单价"], errors="coerce")

# 数量转为整数
df["数量"] = pd.to_numeric(df["数量"], errors="coerce")

# 支付金额转为数值
df["支付金额"] = pd.to_numeric(df["支付金额"], errors="coerce")

# 日期转换（支持多种格式）
df["下单日期"] = pd.to_datetime(df["下单日期"], format="mixed")

# ========== 第3步：补全缺失值 ==========
# 商品为空的行，用"未知商品"填充
df["商品"] = df["商品"].fillna("未知商品")

# 数量为空的行，根据支付金额和单价推算（如果都有的话）
mask = df["数量"].isna() & df["支付金额"].notna() & df["单价"].notna()
df.loc[mask, "数量"] = df.loc[mask, "支付金额"] / df.loc[mask, "单价"]

# 仍然缺失的数量，填 1
df["数量"] = df["数量"].fillna(1).astype(int)

# 支付金额为空的，用 单价 × 数量 计算
mask2 = df["支付金额"].isna() & df["单价"].notna()
df.loc[mask2, "支付金额"] = df.loc[mask2, "单价"] * df.loc[mask2, "数量"]

print("清洗后缺失值:\n", df.isna().sum())
```

### 8.3 数据分析与输出

```python
# ========== 第4步：新增分析列 ==========
df["下单月份"] = df["下单日期"].dt.month
df["下单星期"] = df["下单日期"].dt.day_name()

# ========== 第5步：核心指标 ==========
print("=" * 50)
print("📊 销售分析报告")
print("=" * 50)

print(f"\n总订单数: {len(df)}")
print(f"总销售额: ¥{df['支付金额'].sum():,.2f}")
print(f"平均客单价: ¥{df['支付金额'].mean():,.2f}")
print(f"最大单笔: ¥{df['支付金额'].max():,.2f}")

# ========== 第6步：各省份销售额排名 ==========
province_stats = df.groupby("省份").agg(
    订单数=("订单号", "count"),
    总销售额=("支付金额", "sum"),
    平均客单价=("支付金额", "mean")
).sort_values("总销售额", ascending=False)
print("\n各省份销售排名:\n", province_stats)

# ========== 第7步：商品销售透视表 ==========
product_pivot = df.pivot_table(
    values="支付金额",
    index="商品",
    columns="省份",
    aggfunc="sum",
    fill_value=0,
    margins=True,
    margins_name="合计"
)
print("\n商品-省份交叉销售额:\n", product_pivot)

# ========== 第8步：客户消费统计 ==========
customer_stats = df.groupby("客户姓名").agg(
    订单数=("订单号", "count"),
    总消费=("支付金额", "sum"),
    平均客单价=("支付金额", "mean")
).sort_values("总消费", ascending=False)
customer_stats["消费排名"] = customer_stats["总消费"].rank(ascending=False).astype(int)
print("\n客户消费排行:\n", customer_stats)

# ========== 第9步：保存清洗后的数据 ==========
df.to_csv("sales_cleaned.csv", index=False, encoding="utf-8-sig")
print("\n✅ 清洗后的数据已保存为 sales_cleaned.csv")
```

---

## 九、练习题

### 练习 1：数据清洗（基础）

给定以下 DataFrame，完成清洗任务：

```python
df = pd.DataFrame({
    "产品": [" A产品", "B产品 ", "C产品", " A产品 ", "D产品"],
    "价格": ["100", "200元", "300", "400", "abc"],
    "库存": ["50", "30", "缺货", "20", "10"],
    "上架日期": ["2026-01-01", "2026/02/15", "2026.03.20", "2026-04-10", "2026/05/05"]
})
```

要求：
1. 去除"产品"列两端空白
2. 把"价格"列转为数值（"200元"和"abc"要容错处理）
3. 把"库存"列转为数值（"缺货"变成 0）
4. 把"上架日期"转为 datetime 类型（注意多种分隔符）

### 练习 2：数据合并（中等）

有如下两张表：

```python
df_students = pd.DataFrame({
    "学号": ["S001", "S002", "S003", "S004", "S005"],
    "姓名": ["张三", "李四", "王五", "赵六", "钱七"],
    "班级": ["A班", "B班", "A班", "B班", "A班"]
})

df_scores = pd.DataFrame({
    "学号": ["S001", "S002", "S003", "S006"],
    "科目": ["数学", "数学", "数学", "数学"],
    "成绩": [95, 88, 76, 92]
})
```

要求：
1. 用左连接合并两张表（保留所有学生）
2. 用成绩填充合并后的 NaN 值
3. 按班级统计平均成绩

### 练习 3：综合分析（进阶）

给定一个月度销售数据（自己构造 50 行以上数据），要求：
1. 计算每个产品的 7 日移动平均销售额
2. 找出销售额环比增长超过 50% 的日期
3. 用 `pivot_table` 生成产品-星期的交叉汇总表
4. 找出每个产品销售额最高的前 3 天

---

## 十、常见问题（FAQ）

### Q1：astype() 和 pd.to_numeric() 有什么区别？

- `astype()` 简单直接，遇到无法转换的值直接报错
- `pd.to_numeric()` 支持 `errors="coerce"`（无法转换的变 NaN）和 `errors="ignore"`（保持原样）
- 处理可能有脏数据的列时，推荐用 `pd.to_numeric()`

### Q2：merge() 的 on 参数 vs left_on/right_on？

- `on`：两张表的连接键列名**相同**时使用
- `left_on` + `right_on`：连接键列名**不同**时使用
- 还有 `left_index` + `right_index`：基于索引连接

### Q3：apply() 很慢，有替代方案吗？

是的！优先使用以下方式：
- **向量化操作**：`df["A"] + df["B"]` 代替 `apply(lambda row: row["A"] + row["B"], axis=1)`
- **内置方法**：`df.sum()`、`df.mean()` 代替 `apply(lambda x: x.sum())`
- **`np.where()`**：条件赋值比 `apply` 快
- 只有找不到内置方法时才用 `apply`

### Q4：melt() 和 pivot_table() 是互逆的吗？

不完全是，但可以理解为一对操作：
- `pivot_table()`：长表 → 宽表（做聚合）
- `melt()`：宽表 → 长表（不做聚合，只是"拆开"）
- `unstack()`：类似 pivot_table 但不做聚合

### Q5：日期列显示为 NaT 怎么办？

`NaT`（Not a Time）是日期版的 NaN。常见原因：
- 原始数据格式不统一（混用 `2026/01/15` 和 `2026-01-15`）
- 有无法解析的值（如 "无"）

解决方法：
```python
# 使用 format="mixed" 让 pandas 自动推断格式
df["日期"] = pd.to_datetime(df["日期"], format="mixed")

# 或者指定多种格式（先尝试一种，失败的用另一种）
df["日期"] = pd.to_datetime(df["日期"], errors="coerce")
# 然后对 NaT 的行单独处理
```

---

## 十一、推荐学习资源

1. **pandas 官方文档**：https://pandas.pydata.org/docs/ —— 最权威的参考，但内容多
2. **廖雪峰 pandas 教程**：https://www.liaoxuefeng.com/wiki/1016959663602400 —— 中文友好，有实际案例
3. **菜鸟教程 pandas**：https://www.runoob.com/pandas/pandas-tutorial.html —— 简洁实用，适合快速查阅
4. **pandas Cheat Sheet**：https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf —— 官方速查表，打印出来贴在桌上
5. **Kaggle Learn pandas**：https://www.kaggle.com/learn/pandas —— 交互式学习，边学边练

---

> **Day 39 小结**：今天掌握了 pandas 数据处理的核心技能——类型转换、字符串处理、数据合并、透视表、apply 自定义函数、窗口函数。这些是数据分析日常工作中最高频使用的操作。明天我们将学习 **pandas 进阶**，包括多级索引、时间序列处理和性能优化。
