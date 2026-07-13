# Python Day 38：pandas 入门 —— 数据分析的瑞士军刀

> Day 37 我们学会了用 requests 调用 API 获取数据。但拿到一堆原始数据后，怎么**高效地筛选、计算、统计**？手动写循环处理列表和字典？太累了！今天正式进入 **pandas** 的世界——Python 数据分析的核心工具，一个库搞定几乎所有表格数据操作。

---

## 一、什么是 pandas？

**pandas** 是 Python 最流行的数据分析库，名字来自 "panel data"（面板数据）。它提供了两个核心数据结构，让你能像用 Excel 一样操作数据，但快得多、灵活得多。

### 1.1 为什么学 pandas？

| 场景 | 不用 pandas | 用 pandas |
|------|------------|-----------|
| 读取 CSV | 手动 split、类型转换 | `pd.read_csv("data.csv")` 一行搞定 |
| 筛选数据 | 嵌套 for + if | `df[df["age"] > 18]` |
| 分组统计 | 字典手动累加 | `df.groupby("city").mean()` |
| 处理缺失值 | 一堆 if 判断 | `df.fillna(0)` |
| 数据透视 | 写几百行代码 | `df.pivot_table()` |

### 1.2 安装

```bash
# 使用 pip 安装
pip install pandas

# 验证安装
python -c "import pandas; print(pandas.__version__)"
```

> **约定俗成**：导入时使用别名 `pd`
> ```python
> import pandas as pd
> ```

---

## 二、pandas 的两个核心数据结构

pandas 有两大法宝：**Series**（一维）和 **DataFrame**（二维）。

### 2.1 Series —— 一维数据（类似带索引的列表）

Series 是一行或一列数据，每个元素都有一个**标签（index）**。

```python
import pandas as pd

# 从列表创建 Series
s = pd.Series([90, 85, 78, 92, 88])
print(s)

# 输出：
# 0    90
# 1    85
# 2    78
# 3    92
# 4    88
# dtype: int64

# 指定自定义索引
s = pd.Series(
    [90, 85, 78, 92, 88],
    index=["语文", "数学", "英语", "物理", "化学"]
)
print(s)

# 输出：
# 语文    90
# 数学    85
# 英语    78
# 物理    92
# 化学    88
# dtype: int64

# 从字典创建 Series（字典的 key 自动成为索引）
s = pd.Series({"北京": 2100, "上海": 2400, "广州": 1800})
print(s["北京"])  # 2100

# Series 的常用属性
print(s.index)     # Index(['北京', '上海', '广州'], dtype='object')
print(s.values)    # [2100 2400 1800]
print(s.dtype)     # int64
print(s.shape)     # (3,)
print(s.size)      # 3
```

### 2.2 DataFrame —— 二维数据（类似 Excel 表格）

DataFrame 是 pandas 最核心的结构，可以理解为一个**带行标签和列标签的二维表格**。

```python
import pandas as pd

# 方式一：从字典创建（最常用）
data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [25, 30, 28, 35],
    "城市": ["北京", "上海", "广州", "深圳"],
    "薪资": [8000, 12000, 9500, 15000]
}
df = pd.DataFrame(data)
print(df)

# 输出：
#    姓名  年龄  城市    薪资
# 0  张三   25  北京   8000
# 1  李四   30  上海  12000
# 2  王五   28  广州   9500
# 3  赵六   35  深圳  15000

# 方式二：从列表的列表创建
data = [
    ["张三", 25, "北京", 8000],
    ["李四", 30, "上海", 12000],
]
df = pd.DataFrame(data, columns=["姓名", "年龄", "城市", "薪资"])

# 方式三：从嵌套字典创建
df = pd.DataFrame({
    "语文": {"张三": 90, "李四": 85},
    "数学": {"张三": 95, "李四": 88},
})
```

### 2.3 DataFrame 的核心属性

```python
# 查看基本信息
print(df.shape)       # (4, 4)  —— 行数、列数
print(df.columns)     # Index(['姓名', '年龄', '城市', '薪资'], dtype='object')
print(df.index)       # RangeIndex(start=0, stop=4, step=1)
print(df.dtypes)      # 每列的数据类型
print(df.values)      # 底层的 NumPy 数组（二维）
print(df.ndim)        # 维度数，DataFrame 固定为 2
print(df.size)        # 总元素数 16

# 快速预览
print(df.head(2))     # 前两行
print(df.tail(2))     # 后两行
print(df.info())      # 完整信息摘要（类型、非空数量、内存占用）
print(df.describe())  # 数值列的统计摘要（均值、标准差、最值等）
```

---

## 三、读取和保存数据

pandas 支持非常多的数据格式，最常用的是 CSV 和 Excel。

### 3.1 读取 CSV

```python
# 基本读取
df = pd.read_csv("employees.csv")

# 指定编码（中文 CSV 常用 utf-8-sig 或 gbk）
df = pd.read_csv("employees.csv", encoding="utf-8-sig")

# 指定分隔符（TSV 文件用 \t）
df = pd.read_csv("data.tsv", sep="\t")

# 只读取前 N 行（大文件用）
df = pd.read_csv("big_data.csv", nrows=1000)

# 指定某些列作为索引
df = pd.read_csv("data.csv", index_col="id")

# 跳过前几行（有些 CSV 前面有说明文字）
df = pd.read_csv("data.csv", skiprows=2)
```

### 3.2 保存 CSV

```python
# 保存到 CSV（index=False 不保存行号）
df.to_csv("output.csv", index=False, encoding="utf-8-sig")

# 只保存某些列
df[["姓名", "薪资"]].to_csv("salary.csv", index=False)
```

### 3.3 读取和保存 Excel

```python
# 读取 Excel（需要安装 openpyxl：pip install openpyxl）
df = pd.read_excel("report.xlsx", sheet_name="Sheet1")

# 读取指定列
df = pd.read_excel("data.xlsx", usecols=["姓名", "年龄"])

# 保存 Excel
df.to_excel("output.xlsx", sheet_name="数据", index=False)
```

### 3.4 其他格式

```python
# 从 JSON 读取
df = pd.read_json("data.json")

# 从剪贴板读取（方便从网页/Excel 复制数据）
# 先复制表格数据，然后：
df = pd.read_clipboard()

# 从 SQL 数据库读取
import sqlite3
conn = sqlite3.connect("mydb.db")
df = pd.read_sql("SELECT * FROM users", conn)
```

---

## 四、选择数据（索引与切片）

DataFrame 有多种方式选取数据，这是 pandas 最基础也最重要的操作。

### 4.1 选择列

```python
import pandas as pd

data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [25, 30, 28, 35],
    "城市": ["北京", "上海", "广州", "深圳"],
    "薪资": [8000, 12000, 9500, 15000]
}
df = pd.DataFrame(data)

# 选择单列 —— 返回 Series
print(df["姓名"])
print(df.姓名)        # 也可以用点语法（但列名含空格或中文时不可用，不推荐）

# 选择多列 —— 返回 DataFrame
print(df[["姓名", "薪资"]])
```

### 4.2 选择行 —— loc 和 iloc

这是 pandas 最重要的概念之一：

| 方法 | 含义 | 用法 |
|------|------|------|
| `loc` | **基于标签**（索引名/列名） | `df.loc[行标签, 列标签]` |
| `iloc` | **基于位置**（第几行第几列，从 0 开始） | `df.iloc[行位置, 列位置]` |

```python
# === loc：用标签选择 ===

# 选择单行
print(df.loc[0])           # 第一行（索引为 0）
print(df.loc[2])           # 第三行（索引为 2）

# 选择多行
print(df.loc[0:2])         # 索引 0 到 2（注意：loc 包含两端！）
print(df.loc[[0, 3]])      # 索引 0 和 3

# 选择某行某列
print(df.loc[0, "姓名"])          # 第 0 行的"姓名"列
print(df.loc[1, ["姓名", "薪资"]]) # 第 1 行的"姓名"和"薪资"

# 选择行和列的范围
print(df.loc[0:2, "姓名":"城市"])  # 0~2 行，姓名到城市列

# === iloc：用位置选择 ===

# 选择第一行（位置 0）
print(df.iloc[0])

# 选择前两行
print(df.iloc[0:2])       # 位置 0 到 1（注意：iloc 不包含右端！和 Python 切片一样）

# 选择第 1 行第 2 列
print(df.iloc[1, 2])      # "上海"

# 选择多行多列
print(df.iloc[0:3, 0:2])  # 前 3 行、前 2 列
```

### 4.3 布尔索引（条件筛选）

这是最实用的数据筛选方式——像 SQL 的 WHERE 子句一样：

```python
# 单条件筛选
print(df[df["年龄"] > 28])

# 多条件 —— 用 &（且）、|（或）、~（非），每个条件用括号包裹！
print(df[(df["年龄"] > 25) & (df["城市"] == "北京")])
print(df[(df["薪资"] > 9000) | (df["城市"] == "广州")])
print(df[~(df["城市"] == "上海")])   # 不等于上海

# 用 isin() 筛选多个值（类似 SQL 的 IN）
print(df[df["城市"].isin(["北京", "深圳"])])

# 用 between() 筛选范围
print(df[df["年龄"].between(25, 30)])

# 用字符串方法筛选
print(df[df["姓名"].str.contains("三")])
print(df[df["姓名"].str.startswith("张")])
```

---

## 五、数据操作（增删改）

### 5.1 新增列

```python
# 新增一列（直接赋值）
df["部门"] = ["技术", "市场", "技术", "管理"]
df["税后薪资"] = df["薪资"] * 0.9  # 基于现有列计算

# 用 assign（返回新 DataFrame，不修改原表）
df_new = df.assign(年薪=df["薪资"] * 12, 级别="P6")

# 用 cut 把连续值分箱
df["年龄段"] = pd.cut(
    df["年龄"],
    bins=[0, 25, 30, 40],
    labels=["年轻", "中年", "资深"]
)
```

### 5.2 新增行

```python
# 用 loc 追加一行（建议用 ignore_index=True）
df.loc[len(df)] = ["钱七", 27, "杭州", 10000, "技术", 9000.0, "年轻"]

# 推荐方式：创建新 DataFrame 然后 concat 合并
new_row = pd.DataFrame({"姓名": ["孙八"], "年龄": [32], "城市": ["成都"],
                         "薪资": [11000]})
df = pd.concat([df, new_row], ignore_index=True)
```

### 5.3 删除行和列

```python
# 删除列
df.drop("部门", axis=1, inplace=True)      # 删除单列
df.drop(["税后薪资", "年龄段"], axis=1, inplace=True)  # 删除多列

# 删除行
df.drop(0, axis=0, inplace=True)     # 删除索引为 0 的行
df.drop([1, 3], inplace=True)         # 删除多行

# 按条件删除（保留不满足条件的行）
df = df[df["城市"] != "广州"]
```

> **提示**：`inplace=True` 表示直接修改原 DataFrame（不返回新对象）。pandas 新版本推荐使用赋值方式：`df = df.drop(...)`，而不是 `inplace=True`。

### 5.4 修改数据

```python
# 修改某个值
df.loc[0, "薪资"] = 8500

# 按条件批量修改
df.loc[df["城市"] == "北京", "薪资"] *= 1.1   # 北京的薪资涨 10%

# 替换值
df["城市"].replace({"北京": "北京市", "上海": "上海市"}, inplace=True)

# 重命名列
df.rename(columns={"薪资": "月薪", "姓名": "名字"}, inplace=True)

# 重命名索引
df.rename(index={0: "first", 1: "second"}, inplace=True)
```

---

## 六、数据排序

```python
import pandas as pd

data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [25, 30, 28, 35],
    "城市": ["北京", "上海", "广州", "深圳"],
    "薪资": [8000, 12000, 9500, 15000]
}
df = pd.DataFrame(data)

# 按单列排序
print(df.sort_values("薪资"))                    # 默认升序
print(df.sort_values("薪资", ascending=False))   # 降序

# 按多列排序（先按城市升序，城市相同再按薪资降序）
print(df.sort_values(["城市", "薪资"], ascending=[True, False]))

# 按索引排序
print(df.sort_index())

# 取排序后的前 N 名（top N）
print(df.nlargest(3, "薪资"))   # 薪资最高的 3 人
print(df.nsmallest(2, "年龄")) # 年龄最小的 2 人
```

---

## 七、统计与聚合

```python
# 单列统计
print(df["薪资"].mean())    # 均值：11125.0
print(df["薪资"].sum())     # 总和：44500
print(df["薪资"].max())     # 最大值：15000
print(df["薪资"].min())     # 最小值：8000
print(df["薪资"].count())   # 非空值数量：4
print(df["薪资"].std())     # 标准差

# 唯一值与频数
print(df["城市"].unique())       # 唯一值列表
print(df["城市"].nunique())      # 唯一值个数
print(df["城市"].value_counts())  # 每个城市出现次数（降序）

# 对所有数值列做统计
print(df.describe())

# 分组统计（类似 SQL 的 GROUP BY）
print(df.groupby("城市")["薪资"].mean())          # 按城市分组求平均薪资
print(df.groupby("城市")["薪资"].agg(["mean", "max", "count"]))  # 多种统计

# 分组后对多列应用不同函数
result = df.groupby("城市").agg({
    "薪资": ["mean", "sum"],
    "年龄": ["mean", "count"]
})
print(result)
```

---

## 八、缺失值处理

真实数据几乎总有缺失值（NaN），pandas 提供了完善的处理机制。

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "姓名": ["张三", "李四", "王五", "赵六", None],
    "年龄": [25, None, 28, 35, 30],
    "薪资": [8000, 12000, None, 15000, 9500]
})

# 检测缺失值
print(df.isna())           # 返回布尔表（True 表示缺失）
print(df.isna().sum())     # 每列的缺失值数量
print(df.notna())          # 返回布尔表（True 表示非缺失）

# 删除缺失值
print(df.dropna())              # 删除任何包含 NaN 的行
print(df.dropna(subset=["年龄"]))  # 只检查"年龄"列
print(df.dropna(how="all"))     # 只删除全部为 NaN 的行
print(df.dropna(thresh=2))      # 至少有 2 个非空值的行才保留

# 填充缺失值
print(df.fillna(0))                         # 用 0 填充
print(df.fillna({"年龄": df["年龄"].mean(), "薪资": 0}))  # 不同列用不同值填充
print(df.fillna(method="ffill"))            # 用前一个有效值填充（forward fill）
print(df.fillna(method="bfill"))            # 用后一个有效值填充（backward fill）

# 用插值填充（适合数值型时间序列）
print(df["薪资"].interpolate())
```

> **经验法则**：
> - 数据量足够多 → 直接 `dropna()` 删掉缺失行
> - 关键字段缺失 → `dropna(subset=["关键字段"])` 只删关键字段为空的
> - 数值型数据 → 用均值/中位数填充
> - 分类数据 → 用众数填充

---

## 九、综合实战：员工信息管理系统

把今天学到的知识串起来，做一个完整的员工数据分析：

```python
import pandas as pd
import numpy as np

# ========== 1. 创建示例数据 ==========
np.random.seed(42)

departments = ["技术部", "市场部", "财务部", "人事部"]
cities = ["北京", "上海", "广州", "深圳", "杭州", "成都"]
names = ["张三", "李四", "王五", "赵六", "钱七", "孙八", "周九", "吴十",
         "郑十一", "冯十二", "陈十三", "褚十四", "卫十五", "蒋十六"]

employees = pd.DataFrame({
    "姓名": names,
    "年龄": np.random.randint(22, 45, size=14),
    "城市": np.random.choice(cities, size=14),
    "部门": np.random.choice(departments, size=14),
    "薪资": np.random.randint(6000, 25000, size=14),
    "绩效评分": np.round(np.random.uniform(60, 100, size=14), 1)
})

# 模拟一些缺失值
employees.loc[3, "薪资"] = np.nan
employees.loc[7, "绩效评分"] = np.nan

print("========== 原始数据 ==========")
print(employees)
print()

# ========== 2. 数据清洗 ==========
print("缺失值统计：")
print(employees.isna().sum())
print()

# 用部门平均薪资填充缺失值
employees["薪资"] = employees["薪资"].fillna(
    employees.groupby("部门")["薪资"].transform("mean")
)
# 用全体平均绩效填充缺失值
employees["绩效评分"] = employees["绩效评分"].fillna(
    employees["绩效评分"].mean()
)
print("========== 清洗后数据 ==========")
print(employees)
print()

# ========== 3. 新增计算列 ==========
employees["年薪"] = employees["薪资"] * 12
employees["薪资等级"] = pd.cut(
    employees["薪资"],
    bins=[0, 8000, 15000, 30000],
    labels=["初级", "中级", "高级"]
)
print("========== 新增列后 ==========")
print(employees[["姓名", "薪资", "年薪", "薪资等级"]])
print()

# ========== 4. 条件筛选 ==========
print("========== 薪资超过 12000 的员工 ==========")
high_salary = employees[employees["薪资"] > 12000]
print(high_salary[["姓名", "城市", "薪资"]])
print()

print("========== 北京或上海的高级员工 ==========")
senior_bj_sh = employees[
    (employees["城市"].isin(["北京", "上海"])) &
    (employees["薪资等级"] == "高级")
]
print(senior_bj_sh[["姓名", "城市", "薪资等级"]])
print()

# ========== 5. 分组统计 ==========
print("========== 各部门平均薪资与人数 ==========")
dept_stats = employees.groupby("部门").agg(
    人数=("姓名", "count"),
    平均薪资=("薪资", "mean"),
    最高薪资=("薪资", "max"),
    平均绩效=("绩效评分", "mean")
).round(1)
print(dept_stats)
print()

print("========== 各城市薪资排名 Top 3 ==========")
for city in employees["城市"].unique():
    city_df = employees[employees["城市"] == city]
    top3 = city_df.nlargest(3, "薪资")[["姓名", "薪资", "绩效评分"]]
    print(f"\n{city}：")
    print(top3.to_string(index=False))

# ========== 6. 保存结果 ==========
dept_stats.to_csv("部门统计报告.csv", encoding="utf-8-sig")
print("\n统计报告已保存到 部门统计报告.csv")
```

---

## 十、常见问题

### Q1：SettingWithCopyWarning 是什么？怎么解决？

```python
# 这个警告很常见，原因是你对"视图"进行了修改
df[df["薪资"] > 10000]["薪资"] = 0   # 触发警告！

# 正确做法：使用 loc 一步完成
df.loc[df["薪资"] > 10000, "薪资"] = 0
```

### Q2：pandas 和 Excel 哪个更好？

**各有优势**：
- **Excel**：适合小数据量（< 10万行）、需要可视化图表展示、非程序员使用
- **pandas**：适合大数据量、自动化处理流程、复杂统计分析、编程集成
- 建议：日常查看用 Excel，批量处理用 pandas

### Q3：DataFrame 显示不全怎么办？

```python
# 修改显示设置
pd.set_option("display.max_rows", 100)     # 最多显示 100 行
pd.set_option("display.max_columns", 20)   # 最多显示 20 列
pd.set_option("display.width", 200)        # 显示宽度
pd.set_option("display.max_colwidth", 50)  # 每列最大宽度
```

### Q4：读 CSV 中文乱码？

```python
# 尝试不同编码
df = pd.read_csv("data.csv", encoding="utf-8")      # 最常用
df = pd.read_csv("data.csv", encoding="utf-8-sig")   # 带 BOM 的 UTF-8
df = pd.read_csv("data.csv", encoding="gbk")         # Windows 中文
df = pd.read_csv("data.csv", encoding="gb18030")      # 更广的中文编码
```

### Q5：pandas 处理大数据太慢怎么办？

```python
# 1. 只读需要的列
df = pd.read_csv("big.csv", usecols=["姓名", "薪资"])

# 2. 指定数据类型（减少内存）
df = pd.read_csv("big.csv", dtype={"年龄": "int32", "薪资": "float32"})

# 3. 分块读取
chunks = pd.read_csv("huge.csv", chunksize=10000)
for chunk in chunks:
    process(chunk)  # 逐块处理

# 4. 使用 category 类型（适合重复值多的列，如城市、部门）
df["城市"] = df["城市"].astype("category")
```

---

## 十一、练习题

### 练习 1：基础操作（必做）

创建一个包含 5 名学生信息的 DataFrame（姓名、3 门课成绩），然后：
- 计算每人的总分和平均分
- 找出平均分最高的人
- 按总分降序排列

### 练习 2：数据清洗（进阶）

以下数据有缺失值，请清洗并分析：
```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "商品": ["手机", "电脑", "平板", "耳机", "手表", None],
    "价格": [3999, 6999, None, 299, None, 1999],
    "销量": [500, None, 200, 800, 350, None],
    "评分": [4.5, 4.8, 4.2, None, 4.6, None]
})
```
要求：填充缺失值、计算"销售额=价格×销量"、找出销售额最高的商品。

### 练习 3：综合分析（挑战）

从以下学生数据中：
- 统计每个班级的人数和平均成绩
- 找出每科成绩最高分的学生
- 筛选出有两门以上不及格（< 60 分）的学生
```python
students = pd.DataFrame({
    "姓名": [f"学生{i}" for i in range(1, 21)],
    "班级": np.random.choice(["A班", "B班", "C班"], 20),
    "数学": np.random.randint(40, 100, 20),
    "英语": np.random.randint(40, 100, 20),
    "物理": np.random.randint(40, 100, 20)
})
```

---

## 十二、免费学习资源

- **pandas 官方文档**（权威参考）：https://pandas.pydata.org/docs/
- **pandas 官方入门教程**：https://pandas.pydata.org/docs/getting_started/index.html
- **廖雪峰 pandas 教程**：https://www.liaoxuefeng.com/wiki/1016959663602400/1187197579476064
- **菜鸟教程 pandas**：https://www.runoob.com/pandas/pandas-tutorial.html
- **Kaggle pandas 微课程**（免费、交互式）：https://www.kaggle.com/learn/pandas
- **pandas-cookbook**（实战示例集）：https://pandas.pydata.org/docs/user_guide/cookbook.html

---

## 今日总结

| 知识点 | 要点 |
|--------|------|
| Series | 一维数据，带标签的数组 |
| DataFrame | 二维表格，pandas 核心 |
| 读取/保存数据 | `read_csv` / `to_csv` / `read_excel` |
| 选择数据 | `df["列"]` / `loc[标签]` / `iloc[位置]` / 布尔索引 |
| 增删改 | 直接赋值、`concat`、`drop`、`replace` |
| 排序 | `sort_values` / `nlargest` / `nsmallest` |
| 统计聚合 | `groupby` / `value_counts` / `describe` |
| 缺失值处理 | `isna` / `dropna` / `fillna` / `interpolate` |

> **明天预告**：Day 39 —— pandas 数据处理进阶，学习合并（merge/join）、重塑（melt/pivot）、字符串处理、窗口函数等高级操作，让数据分析能力再上一个台阶！
