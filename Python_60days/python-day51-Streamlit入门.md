# Python Day 51 - Streamlit 入门

> 欢迎来到第 51 天！我们已经学完了 FastAPI 和 SQLAlchemy，掌握了构建后端 API 和操作数据库的能力。但从"有接口"到"用户能用"还差一块拼图——**前端界面**。今天我们要学习 **Streamlit**，一个让你用纯 Python 就能搭建交互式 Web 应用的神器。不需要写 HTML、CSS、JavaScript，只要你会 Python，就能做出漂亮的数据应用。

---

## 一、为什么学 Streamlit？

### 1.1 Streamlit 是什么？

Streamlit 是一个开源 Python 框架，专为**数据科学家和 Python 开发者**设计的 Web 应用工具。它的核心理念是：

> **用 Python 的方式写 Web 应用，而不是 Web 开发的方式。**

用传统方式做一个带图表、用户输入、数据展示的 Web 页面，你需要：
- 前端：HTML + CSS + JavaScript（或 React/Vue）
- 后端：Flask/Django/FastAPI
- 部署：Nginx + Gunicorn

用 Streamlit，**只需要一个 Python 文件**：

```python
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

st.title("📊 销售数据看板")
data = pd.read_csv("sales.csv")
st.dataframe(data)

fig, ax = plt.subplots()
ax.bar(data["月份"], data["销售额"])
st.pyplot(fig)
```

保存为 `app.py`，运行一行命令，浏览器就出现了完整的交互式页面。

### 1.2 Streamlit vs 传统方案对比

| 特性 | Streamlit | Flask/Django + 前端 |
|------|-----------|-------------------|
| 学习曲线 | 极低（会 Python 就行） | 高（需学前后端技术栈） |
| 开发速度 | 分钟级 | 小时/天级 |
| 前端代码 | 零 | 大量 HTML/CSS/JS |
| 交互性 | 内置（滑块、按钮等） | 需自行实现 |
| 适用场景 | 数据应用/原型/内部工具 | 复杂业务系统 |
| 定制性 | 中等（有主题和CSS注入） | 完全自由 |

### 1.3 Streamlit 的典型应用场景

- 📊 **数据看板**：销售报表、用户分析、KPI 展示
- 🔬 **机器学习 Demo**：模型训练界面、参数调优、结果可视化
- 🛠️ **内部工具**：数据库查询界面、配置管理面板
- 📝 **交互式教程**：教学演示、概念可视化
- 📈 **金融工具**：股票筛选器、投资组合分析

> 💡 **经验法则**：如果你需要在 **1 小时内** 做出一个带交互的数据展示页面，选 Streamlit；如果要做**复杂的业务系统**（用户登录、权限管理、复杂表单），选 FastAPI + 前端。

---

## 二、安装与第一个应用

### 2.1 安装

```bash
pip install streamlit
```

验证安装成功：

```bash
streamlit --version
```

### 2.2 启动方式

Streamlit 应用就是一个普通的 Python 文件（通常命名为 `app.py`），通过以下命令运行：

```bash
streamlit run app.py
```

运行后，Streamlit 会：
1. 在本地启动一个 Web 服务器（默认端口 8501）
2. 自动打开浏览器访问 `http://localhost:8501`
3. 开启**热重载**——修改代码保存后，浏览器页面自动刷新

常用启动参数：

```bash
# 指定端口
streamlit run app.py --server.port 8080

# 关闭自动打开浏览器
streamlit run app.py --server.headless true

# 指定网络地址（局域网访问）
streamlit run app.py --server.address 0.0.0.0
```

### 2.3 Hello World

创建文件 `app.py`：

```python
import streamlit as st

st.title("我的第一个 Streamlit 应用")
st.write("Hello, Streamlit! 🎉")
```

运行 `streamlit run app.py`，你就能在浏览器中看到一个标题和一段文字。

### 2.4 Streamlit 的执行模型（重要！）

Streamlit 的运行方式跟普通 Python 脚本很不一样，理解这一点至关重要：

```
每次用户与页面交互（点击按钮、拖动滑块、输入文字），
整个脚本从上到下重新执行一遍。
```

这意味着：
- **不要在全局作用域做耗时操作**（如加载大模型），否则每次交互都会重新执行
- **用 `@st.cache_data` 缓存耗时计算**（后面会讲）
- Streamlit 用"从头运行"代替了传统前端的事件驱动模型

```
用户点击按钮 → 整个脚本重新执行 → 页面重新渲染 → 展示新结果
```

这个模型叫 **从上到下的执行模型（Top-down execution model）**，是 Streamlit 的灵魂。

---

## 三、核心显示组件

### 3.1 文本展示

Streamlit 提供了多种文本展示方式，从简单到丰富：

```python
import streamlit as st

# 标题（h1）
st.title("一级标题")

# 子标题（h2）
st.header("二级标题")

# 三级标题（h3）
st.subheader("三级标题")

# 普通文本（支持 Markdown）
st.write("这是一段普通文本")
st.write("支持 **粗体**、*斜体*、~~删除线~~、`代码`")

# Markdown 渲染
st.markdown("""
### 这是一段 Markdown

- 列表项 1
- 列表项 2

> 这是一段引用文字

`代码片段`
""")

# 代码块（带语法高亮和复制按钮）
st.code("""
import pandas as pd
df = pd.DataFrame({"name": ["张三", "李四"]})
print(df)
""", language="python")

# 纯文本（不渲染 Markdown）
st.text("这是纯文本，**不会被加粗**")

# 分隔线
st.divider()

# LaTeX 公式
st.latex(r"e^{i\pi} + 1 = 0")
```

### 3.2 数据展示

这是 Streamlit 最强大的部分之一，专门为数据展示做了优化：

```python
import streamlit as st
import pandas as pd
import numpy as np

# 创建示例数据
df = pd.DataFrame({
    "姓名": ["张三", "李四", "王五", "赵六", "钱七"],
    "年龄": [25, 30, 28, 35, 22],
    "城市": ["北京", "上海", "广州", "深圳", "杭州"],
    "薪资": [15000, 20000, 18000, 25000, 12000]
})

# 1. 数据表格（最常用，可排序/搜索）
st.dataframe(df)

# 2. 静态表格（不可交互，适合小量数据）
st.table(df.head(3))

# 3. 列指标展示（关键数字大字显示）
col1, col2, col3 = st.columns(3)
col1.metric("总人数", len(df), "+2")
col2.metric("平均薪资", f"{df['薪资'].mean():.0f}元", "+5%")
col3.metric("最高薪资", f"{df['薪资'].max():.0f}元")

# 4. JSON 展示
st.json({
    "项目": "数据分析系统",
    "版本": "1.0",
    "功能": ["数据展示", "图表分析", "导出报告"]
})

# 5. 简单列表展示
st.write("城市列表：")
st.write(df["城市"].tolist())
```

### 3.3 图表展示

Streamlit 内置了对 Matplotlib、Seaborn、Altair、Plotly 等图表库的支持：

```python
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

# 准备数据
df = pd.DataFrame({
    "月份": ["1月", "2月", "3月", "4月", "5月", "6月"],
    "销售额": [120, 150, 180, 210, 195, 230],
    "利润": [30, 45, 55, 70, 60, 80]
})

# === 方法 1：使用 st.line_chart / st.bar_chart / st.area_chart ===
# 最简单的方式，基于 Altair 渲染
st.subheader("内置图表")
st.line_chart(df, x="月份", y=["销售额", "利润"])
st.bar_chart(df, x="月份", y="销售额")

# === 方法 2：Matplotlib 图表 ===
fig, ax = plt.subplots(figsize=(8, 4))
ax.bar(df["月份"], df["销售额"], color="steelblue", label="销售额")
ax.plot(df["月份"], df["利润"], "ro-", label="利润")
ax.set_xlabel("月份")
ax.set_ylabel("金额（万元）")
ax.set_title("2024年上半年销售与利润趋势")
ax.legend()
st.pyplot(fig)  # 用 st.pyplot() 展示 Matplotlib 图表

# === 方法 3：Seaborn 图表 ===
fig2, ax2 = plt.subplots(figsize=(8, 4))
sns.heatmap(
    df[["销售额", "利润"]].corr(),
    annot=True,
    cmap="YlOrRd",
    ax=ax2
)
ax2.set_title("相关性热力图")
st.pyplot(fig2)

# === 方法 4：Altair 图表（Streamlit 原生推荐） ===
import altair as alt
chart = alt.Chart(df).mark_bar().encode(
    x="月份:O",
    y="销售额:Q",
    tooltip=["月份", "销售额"]
)
st.altair_chart(chart, use_container_width=True)
```

### 3.4 多媒体展示

```python
import streamlit as st

# 图片展示
st.image(
    "https://streamlit.io/images/brand/streamlit-logo-secondary-colormark-darktext.png",
    caption="Streamlit Logo",
    width=300
)

# 音频播放
st.audio("https://example.com/sample.mp3")

# 视频播放
st.video("https://example.com/sample.mp4")
```

---

## 四、交互组件

Streamlit 的交互组件是它的核心亮点。每个交互组件都会返回一个值，脚本每次重新执行时会用新值重新渲染页面。

### 4.1 按钮与文本输入

```python
import streamlit as st

# === 按钮按钮 ===
if st.button("点我试试 🎉"):
    st.success("你点击了按钮！")
    st.balloons()  # 气球动画效果 🎈

# === 文本输入 ===
name = st.text_input("请输入你的名字：", placeholder="例如：张三")
if name:
    st.write(f"你好，{name}！")

# === 大文本输入（多行） ===
content = st.text_area("请输入你的想法：", height=150)
if content:
    st.write(f"你写了 {len(content)} 个字符")

# === 密码输入 ===
password = st.text_input("密码：", type="password")
```

### 4.2 选择组件

```python
import streamlit as st

# === 下拉选择框 ===
city = st.selectbox(
    "选择城市：",
    ["北京", "上海", "广州", "深圳", "杭州", "成都"],
    index=0,  # 默认选中第一项
    placeholder="请选择..."
)
st.write(f"你选择了：{city}")

# === 多选框 ===
colors = st.multiselect(
    "选择你喜欢的颜色：",
    ["红色", "蓝色", "绿色", "黄色", "紫色"],
    ["蓝色", "绿色"]  # 默认选中
)
st.write(f"你选了：{colors}")

# === 单选按钮 ===
gender = st.radio(
    "你的性别：",
    ["男", "女", "保密"],
    index=2,  # 默认选中"保密"
    horizontal=True  # 水平排列
)
st.write(f"性别：{gender}")

# === 复选框 ===
agree = st.checkbox("我同意用户协议")
if agree:
    st.write("感谢你的同意！")
```

### 4.3 数值与滑块

```python
import streamlit as st

# === 数字输入 ===
age = st.number_input("年龄：", min_value=0, max_value=150, value=25, step=1)
st.write(f"年龄：{age}")

# === 滑块 ===
temperature = st.slider(
    "设定温度（℃）：",
    min_value=-10,
    max_value=45,
    value=25,
    step=0.5
)
st.write(f"当前温度：{temperature}℃")

# === 范围滑块（选一个区间） ===
price_range = st.slider(
    "价格区间（元）：",
    min_value=0,
    max_value=1000,
    value=(200, 500)  # 默认区间
)
st.write(f"价格范围：{price_range[0]} - {price_range[1]} 元")
```

### 4.4 日期与时间

```python
import streamlit as st
from datetime import datetime

# === 日期选择 ===
birthday = st.date_input("出生日期：", datetime(2000, 1, 1))
st.write(f"出生日期：{birthday}")

# === 时间选择 ===
alarm = st.time_input("闹钟时间：", datetime(2024, 1, 1, 7, 30))
st.write(f"闹钟：{alarm}")

# === 日期范围 ===
from datetime import date
start, end = st.date_input(
    "选择日期范围：",
    value=(date(2024, 1, 1), date(2024, 12, 31))
)
st.write(f"从 {start} 到 {end}")
```

### 4.5 文件上传与下载

```python
import streamlit as st
import pandas as pd

# === 文件上传 ===
uploaded = st.file_uploader(
    "上传你的文件：",
    type=["csv", "xlsx", "json", "txt"],
    accept_multiple_files=False  # 设为 True 可上传多个
)

if uploaded:
    st.write(f"文件名：{uploaded.name}")
    st.write(f"文件大小：{uploaded.size / 1024:.1f} KB")

    # 根据 file_type 读取
    if uploaded.name.endswith(".csv"):
        df = pd.read_csv(uploaded)
        st.dataframe(df)

# === 文件下载 ===
st.download_button(
    label="下载示例数据",
    data=df.to_csv(index=False).encode("utf-8"),
    file_name="example.csv",
    mime="text/csv"
)
```

### 4.6 进度与状态

```python
import streamlit as st
import time

# === 进度条 ===
st.subheader("进度条示例")
progress = st.progress(0, text="处理中...")
for i in range(100):
    time.sleep(0.02)
    progress.progress(i + 1, text=f"已完成 {i+1}%")
progress.empty()  # 完成后清除

# === 加载动画（会阻塞执行） ===
with st.spinner("正在加载数据..."):
    time.sleep(2)
st.success("数据加载完成！")

# === 状态消息 ===
st.info("这是一条提示信息")
st.warning("这是一条警告信息")
st.error("这是一条错误信息")
st.success("操作成功！")
```

---

## 五、布局控制

### 5.1 侧边栏

侧边栏是 Streamlit 应用的标准布局元素，通常用来放控制组件：

```python
import streamlit as st

# 侧边栏内容
st.sidebar.title("控制面板")
st.sidebar.write("在这里调整参数")

# 侧边栏中的组件
city = st.sidebar.selectbox("城市", ["北京", "上海", "广州"])
salary = st.sidebar.slider("最低薪资", 0, 50000, 10000)

# 侧边栏选择器（特别常用）
page = st.sidebar.radio(
    "选择页面：",
    ["📊 数据总览", "📈 趋势分析", "👤 用户分析"]
)

if page == "📊 数据总览":
    st.title("数据总览")
    st.write("这里是总览页面")
elif page == "📈 趋势分析":
    st.title("趋势分析")
    st.write("这里是趋势页面")
elif page == "👤 用户分析":
    st.title("用户分析")
    st.write("这里是用户分析页面")
```

### 5.2 列布局

将页面分成多列并排展示：

```python
import streamlit as st

# 等分三列
col1, col2, col3 = st.columns(3)

with col1:
    st.metric("用户总数", "12,345", "+156")
    st.write("详细说明...")

with col2:
    st.metric("活跃用户", "8,901", "+89")
    st.write("详细说明...")

with col3:
    st.metric("转化率", "3.2%", "-0.1%")
    st.write("详细说明...")

# 不等分列（指定比例）
left, right = st.columns([2, 1])  # 左边占 2/3，右边占 1/3

with left:
    st.write("这是左侧主内容区域，占更大空间")
    st.dataframe(...)  # 展示数据表

with right:
    st.write("右侧")
    st.selectbox("筛选条件", ["A", "B", "C"])
```

### 5.3 标签页

在同一个页面内切换不同内容：

```python
import streamlit as st

tab1, tab2, tab3 = st.tabs(["📈 图表", "📋 数据表", "⚙️ 设置"])

with tab1:
    st.line_chart(...)
    st.bar_chart(...)

with tab2:
    st.dataframe(...)

with tab3:
    st.slider("参数调整", 0, 100)
```

### 5.4 折叠区域与容器

```python
import streamlit as st

# === 折叠区域 ===
with st.expander("点击查看详情"):
    st.write("这里是隐藏的详细内容")
    st.image("https://picsum.photos/400/200")
    st.code("print('Hello')", language="python")

# === 容器（方便统一管理） ===
with st.container():
    st.write("容器内的内容")
    st.button("容器内的按钮")
    # 还可以调用 container.empty() 等方法动态更新
```

---

## 六、缓存机制（性能优化关键）

### 6.1 为什么需要缓存？

回忆 Streamlit 的执行模型——**每次交互都从头运行脚本**。如果你的代码里有一个耗时 10 秒的数据加载操作：

```python
# ❌ 每次交互都会执行，页面卡 10 秒
df = pd.read_csv("huge_dataset.csv")  # 100MB 的文件
result = df.groupby("category").mean()  # 复杂计算
```

用户每点一下按钮，就要等 10 秒。这就是缓存要解决的问题。

### 6.2 st.cache_data —— 缓存数据

```python
import streamlit as st
import pandas as pd

@st.cache_data
def load_data():
    """加载数据，结果会被缓存"""
    df = pd.read_csv("huge_dataset.csv")
    return df

@st.cache_data
def expensive_computation(df):
    """耗时计算，结果会被缓存"""
    result = df.groupby("category").agg(["mean", "count"])
    return result

# 第一次调用：执行函数，缓存结果
data = load_data()

# 后续调用（即使脚本重新执行）：直接返回缓存结果，瞬间完成
data = load_data()  # ⚡ 几乎不花时间
```

`@st.cache_data` 会根据**函数名 + 参数值**生成一个唯一的缓存键。如果参数没变，直接返回缓存。

### 6.3 st.cache_resource —— 缓存资源

```python
import streamlit as st
from sqlalchemy import create_engine

@st.cache_resource
def get_db_engine():
    """创建数据库连接引擎（只创建一次）"""
    engine = create_engine("sqlite:///myapp.db")
    return engine

# 适合缓存的对象：
# - 数据库连接
# - ML 模型
# - 文件句柄
# - 网络会话（requests.Session）
```

### 6.4 缓存控制

```python
# 清除某个函数的缓存
load_data.clear()

# 清除所有缓存
st.cache_data.clear()

# 设置缓存过期时间
@st.cache_data(ttl=3600)  # 1 小时后过期
def fetch_realtime_data():
    """获取实时数据，1小时更新一次"""
    ...
```

### 6.5 何时用哪个缓存？

| 场景 | 用什么 | 原因 |
|------|--------|------|
| 数据计算结果（DataFrame、列表、数字） | `@st.cache_data` | 数据可序列化，安全深拷贝 |
| 数据库连接、ML 模型 | `@st.cache_resource` | 对象不可序列化，共享引用 |

> 💡 **最佳实践**：养成习惯，把所有耗时操作都用 `@st.cache_data` 或 `@st.cache_resource` 包起来。

---

## 七、综合实战：简易数据分析工具

把今天学到的内容组合起来，做一个实用的数据分析小工具：

```python
import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# ========== 页面配置 ==========
st.set_page_config(
    page_title="数据分析工具",
    page_icon="📊",
    layout="wide"  # 使用宽屏布局
)

# ========== 缓存数据加载 ==========
@st.cache_data
def generate_sample_data(n=500):
    """生成模拟销售数据"""
    np.random.seed(42)
    cities = ["北京", "上海", "广州", "深圳", "杭州", "成都"]
    categories = ["电子产品", "服装", "食品", "家居", "图书"]

    df = pd.DataFrame({
        "日期": pd.date_range("2024-01-01", periods=n, freq="D"),
        "城市": np.random.choice(cities, n),
        "类别": np.random.choice(categories, n),
        "销售额": np.random.randint(100, 10000, n),
        "数量": np.random.randint(1, 50, n),
        "评分": np.round(np.random.uniform(3.0, 5.0, n), 1)
    })
    return df

# ========== 加载数据 ==========
df = generate_sample_data()

# ========== 侧边栏 ==========
st.sidebar.title("控制面板")
st.sidebar.write("调整下方参数来筛选数据")

selected_city = st.sidebar.multiselect("选择城市", df["城市"].unique(), df["城市"].unique()[:2])
selected_category = st.sidebar.multiselect("选择类别", df["类别"].unique(), df["类别"].unique())

# 数据筛选
filtered = df[
    (df["城市"].isin(selected_city)) &
    (df["类别"].isin(selected_category))
]

# ========== 主页面 ==========
st.title("📊 销售数据分析工具")
st.write(f"共筛选出 **{len(filtered)}** 条记录")

# 关键指标
col1, col2, col3, col4 = st.columns(4)
with col1:
    st.metric("总销售额", f"¥{filtered['销售额'].sum():,}")
with col2:
    st.metric("平均销售额", f"¥{filtered['销售额'].mean():,.0f}")
with col3:
    st.metric("总销量", f"{filtered['数量'].sum():,}")
with col4:
    st.metric("平均评分", f"{filtered['评分'].mean():.1f}")

st.divider()

# 标签页展示不同分析
tab1, tab2, tab3 = st.tabs(["数据明细", "图表分析", "统计分析"])

with tab1:
    st.dataframe(filtered, use_container_width=True)
    csv = filtered.to_csv(index=False).encode("utf-8")
    st.download_button("📥 下载 CSV", csv, "sales_data.csv", "text/csv")

with tab2:
    fig, axes = plt.subplots(1, 2, figsize=(14, 5))

    # 左图：各城市销售额柱状图
    city_sales = filtered.groupby("城市")["销售额"].sum().sort_values(ascending=False)
    axes[0].barh(city_sales.index, city_sales.values, color="steelblue")
    axes[0].set_title("各城市销售额")
    axes[0].set_xlabel("销售额（元）")

    # 右图：各类别销售额饼图
    cat_sales = filtered.groupby("类别")["销售额"].sum()
    axes[1].pie(cat_sales, labels=cat_sales.index, autopct="%1.1f%%")
    axes[1].set_title("各类别销售占比")

    plt.tight_layout()
    st.pyplot(fig)

with tab3:
    st.write("### 各维度统计")
    st.dataframe(
        filtered.groupby("城市").agg({
            "销售额": ["sum", "mean", "count"],
            "评分": "mean"
        }).round(1),
        use_container_width=True
    )
```

把以上代码保存为 `app.py`，运行 `streamlit run app.py`，你就能得到一个带筛选功能、图表展示和数据导出的完整分析工具！

---

## 八、常见问题 FAQ

### Q1：Streamlit 运行很慢怎么办？
**A**：大部分性能问题都是因为缺少缓存。把耗时操作（数据加载、模型加载、API 调用）用 `@st.cache_data` 或 `@st.cache_resource` 包起来。另外避免在循环中频繁调用 `st.write()`，可以收集到列表里一次性输出。

### Q2：如何修改页面标题和图标？
**A**：在脚本最开头调用 `st.set_page_config()`：
```python
st.set_page_config(
    page_title="我的应用",
    page_icon="🚀",
    layout="wide",       # "wide" 或 "centered"
    initial_sidebar_state="expanded"  # 侧边栏默认展开
)
```
**注意**：`st.set_page_config()` 必须是第一个 Streamlit 命令，否则会报错。

### Q3：按钮点击后状态怎么消失了？
**A**：这是 Streamlit 执行模型导致的。`st.button()` 每次重新执行都会变回 False。如果需要保持状态，用 `st.session_state`（明天 Day 52 会详细讲）。

### Q4：Streamlit 能不能做用户登录？
**A**：Streamlit 没有内置的认证系统。可以用 `streamlit-authenticator` 第三方库，或者配合 FastAPI 后端做 JWT 认证。简单的方案是用密码输入 + `st.session_state` 模拟。

### Q5：如何部署 Streamlit 应用？
**A**：
- **Streamlit Community Cloud**（免费）：推送到 GitHub，在 [share.streamlit.io](https://share.streamlit.io) 连接仓库即可
- **自部署**：用 `Docker + Nginx`
- **云服务器**：`pip install streamlit` 后 `nohup streamlit run app.py &`

---

## 九、练习题

### 练习 1：个人 BMI 计算器（基础）
用 `st.number_input` 输入身高和体重，计算并显示 BMI 值，用颜色区分体重状态（偏瘦/正常/偏胖/肥胖）。

```python
# 提示：
# BMI = 体重(kg) / 身高(m)²
# < 18.5 偏瘦，18.5-24 正常，24-28 偏胖，>= 28 肥胖
```

### 练习 2：Markdown 实时预览器（进阶）
用 `st.text_area` 输入 Markdown 文本，旁边用 `st.markdown()` 实时渲染预览效果。

```python
# 提示：用 st.columns([1, 1]) 做左右两栏布局
```

### 练习 3：CSV 数据分析工具（挑战）
做一个能上传 CSV 文件、自动展示基本统计信息、生成柱状图和饼图的工具。支持用户选择要分析的列。

```python
# 提示：
# 1. st.file_uploader 接收 CSV
# 2. pd.read_csv 读取数据
# 3. st.selectbox 让用户选列
# 4. 根据列类型自动选择合适的图表
```

---

## 十、学习资源

- **Streamlit 官方文档**（最权威）：[https://docs.streamlit.io](https://docs.streamlit.io)
- **Streamlit 官方画廊**（大量示例）：[https://streamlit.io/gallery](https://streamlit.io/gallery)
- **Streamlit GitHub**（开源代码）：[https://github.com/streamlit/streamlit](https://github.com/streamlit/streamlit)
- **菜鸟教程 Streamlit**：[https://www.runoob.com](https://www.runoob.com)

---

> **Day 51 完成总结**：今天我们学习了 Streamlit 的核心功能——显示组件、交互组件、布局控制和缓存机制。Streamlit 的哲学是"用 Python 写 Web 应用"，非常适合数据应用和内部工具的快速开发。明天我们将深入学习 Streamlit 的组件系统，包括 `st.session_state` 状态管理和更高级的交互技巧。
