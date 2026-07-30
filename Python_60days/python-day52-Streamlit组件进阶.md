# Python Day 52 - Streamlit 组件进阶

> 第 51 天我们入门了 Streamlit，学会了基础的数据展示和简单交互。今天我们要深入探索 Streamlit 的**丰富组件生态**——从表单控件到多媒体展示，从布局美化到状态管理。掌握这些组件，你就能构建出功能完整、体验优秀的 Web 应用。

---

## 一、文本输入组件全家桶

Streamlit 提供了多种文本输入方式，适用于不同场景。

### 1.1 单行文本输入 `st.text_input`

最常用的输入组件，用户输入一行文本：

```python
import streamlit as st

# 基础用法
name = st.text_input("你的名字", placeholder="请输入...")
st.write(f"你好，{name}！")

# 带默认值和标签
email = st.text_input(
    label="邮箱地址",
    value="user@example.com",
    max_chars=50,           # 限制最大字符数
    disabled=False,          # 是否禁用
    label_visibility="visible"  # visible / hidden / collapsed
)
st.write(f"邮箱：{email}")

# 输入类型控制（会显示对应键盘）
password = st.text_input("密码", type="password")  # 密码模式（隐藏输入）
age = st.text_input("年龄", type="default")        # 默认模式
```

### 1.2 多行文本输入 `st.text_area`

适用于输入大段文字（如反馈、备注、代码）：

```python
# 多行文本域
feedback = st.text_area(
    label="请留下你的反馈",
    height=150,          # 文本框高度（像素）
    max_chars=1000,      # 最大字符数
    placeholder="在这里写下你的建议..."
)

if feedback:
    st.success("感谢你的反馈！")
    st.write(f"你输入了 **{len(feedback)}** 个字符")
```

### 1.3 数字输入 `st.number_input`

```python
# 基础数字输入
age = st.number_input("年龄", min_value=0, max_value=150, value=25)

# 浮点数
price = st.number_input("价格", min_value=0.0, max_value=10000.0,
                        value=99.9, step=0.1, format="%.2f")

# 整数步进
quantity = st.number_input("数量", min_value=1, value=1, step=1)
st.write(f"总价：{price * quantity:.2f} 元")
```

### 文本组件速查表

| 组件 | 用途 | 关键参数 |
|------|------|----------|
| `st.text_input` | 单行文本 | `type`, `max_chars`, `placeholder` |
| `st.text_area` | 多行文本 | `height`, `max_chars` |
| `st.number_input` | 数字 | `min_value`, `max_value`, `step`, `format` |
| `st.text` | 静态展示文本 | — |
| `st.caption` | 小字说明 | — |
| `st.subheader` | 二级标题 | — |

---

## 二、选择类组件

### 2.1 下拉选择框 `st.selectbox`

从预设选项中选择一个：

```python
# 基础选择框
city = st.selectbox(
    "选择城市",
    options=["北京", "上海", "广州", "深圳", "杭州"]
)
st.write(f"你选择了：{city}")

# 动态选项
fruits = ["苹果", "香蕉", "橙子", "葡萄", "西瓜"]
favorite = st.selectbox("选择你最喜欢的水果", fruits, index=0)

# 配合格式化选项
import pandas as pd
df = pd.DataFrame({
    "name": ["Python", "Java", "Go", "Rust"],
    "year": [1991, 1995, 2009, 2010]
})

# 显示格式名，实际使用值
choice = st.selectbox(
    "选择编程语言",
    options=df["name"].tolist(),
    format_func=lambda x: f"🐍 {x}（{df[df['name']==x]['year'].values[0]}年）"
)
st.write(f"选择了：{choice}")
```

### 2.2 多选框 `st.multiselect`

选择多个选项：

```python
# 多选
selected = st.multiselect(
    "选择你掌握的技能",
    options=["Python", "JavaScript", "SQL", "Docker", "Git", "Linux"],
    default=["Python", "Git"]  # 默认选中
)

if selected:
    st.write(f"你掌握了 **{len(selected)}** 项技能：")
    for skill in selected:
        st.checkbox(skill, value=True, disabled=True)

# 配合 DataFrame 做数据筛选
df = pd.DataFrame({
    "产品": ["手机", "电脑", "平板", "耳机", "键盘"],
    "分类": ["电子产品", "电子产品", "电子产品", "配件", "配件"],
    "价格": [3999, 5999, 2999, 399, 299]
})

categories = st.multiselect(
    "筛选分类",
    options=df["分类"].unique().tolist(),
    default=df["分类"].unique().tolist()
)
filtered = df[df["分类"].isin(categories)]
st.dataframe(filtered)
```

### 2.3 单选按钮 `st.radio`

```python
# 单选按钮
gender = st.radio(
    "性别",
    options=["男", "女", "保密"],
    index=2,          # 默认选中索引
    horizontal=True  # 水平排列
)
st.write(f"性别：{gender}")

# 配合条件展示
mode = st.radio(
    "选择模式",
    ["📊 数据分析", "📈 可视化", "⚙️ 设置"]
)

if mode == "📊 数据分析":
    st.info("进入数据分析模式...")
elif mode == "📈 可视化":
    st.info("进入可视化模式...")
else:
    st.info("打开设置面板...")
```

### 2.4 复选框 `st.checkbox`

```python
# 单个复选框
agree = st.checkbox("我同意用户协议")
if agree:
    st.success("已同意协议")

# 复选框作为开关
show_chart = st.checkbox("显示图表", value=True)
show_table = st.checkbox("显示数据表", value=True)

if show_chart:
    st.line_chart([1, 3, 2, 5, 4])
if show_table:
    st.table({"数据": [10, 20, 30, 40, 50]})
```

### 选择类组件对比

| 组件 | 选择方式 | 适用场景 |
|------|---------|----------|
| `st.selectbox` | 单选下拉 | 选项较多（5+） |
| `st.radio` | 单选按钮 | 选项较少（2-5个），需全部可见 |
| `st.multiselect` | 多选 | 需要筛选多个标签/分类 |
| `st.checkbox` | 开关 | 是/否、显示/隐藏切换 |
| `st.toggle` | 开关（新版） | 功能启用/禁用 |

---

## 三、范围与滑块组件

### 3.1 滑块 `st.slider`

```python
# 基础滑块
temperature = st.slider("温度", min_value=-10, max_value=40, value=25)
st.write(f"当前温度：{temperature}°C")

# 带范围选择的滑块（选一个区间）
date_range = st.slider(
    "选择日期范围",
    min_value=1, max_value=31,
    value=[1, 15]  # 默认区间
)
st.write(f"从第 {date_range[0]} 天到第 {date_range[1]} 天")

# 步进和格式
salary = st.slider(
    "期望月薪",
    min_value=3000, max_value=100000,
    value=15000, step=1000,
    format="¥ %d"
)

# 带时间戳的滑块
from datetime import time, timedelta
t = st.slider(
    "选择时间",
    min_value=time(8, 0),
    max_value=time(22, 0),
    value=time(9, 30),
    step=timedelta(minutes=30),
    format="HH:mm"
)
st.write(f"选择的时间：{t}")
```

### 3.2 时间/日期选择器

```python
from datetime import datetime, date, time

# 日期选择器
birthday = st.date_input("出生日期", min_value=date(1950, 1, 1))
if birthday:
    age = (date.today() - birthday).days // 365
    st.write(f"你大约 {age} 岁")

# 时间选择器
meeting_time = st.time_input("会议时间", value=time(14, 0))
st.write(f"会议安排在 {meeting_time}")

# 日期+时间组合
appointment = st.date_input("预约日期")
appt_time = st.time_input("预约时间")
st.write(f"预约：{appointment} {appt_time}")
```

---

## 四、文件与媒体组件

### 4.1 文件上传 `st.file_uploader`

```python
# 基础文件上传
uploaded = st.file_uploader(
    "上传文件",
    type=["csv", "xlsx", "json"],  # 限制文件类型
    accept_multiple_files=False,   # 是否允许多文件
    label_visibility="visible"
)

if uploaded is not None:
    st.success(f"文件 **{uploaded.name}** 上传成功！")
    st.write(f"文件大小：{uploaded.size / 1024:.1f} KB")
    st.write(f"文件类型：{uploaded.type}")

    # 读取 CSV 文件
    if uploaded.name.endswith(".csv"):
        import pandas as pd
        df = pd.read_csv(uploaded)
        st.dataframe(df.head())

# 多文件上传
files = st.file_uploader("上传多张图片", type=["png", "jpg", "jpeg"],
                         accept_multiple_files=True)
if files:
    cols = st.columns(len(files))
    for i, f in enumerate(files):
        cols[i].image(f, caption=f.name, use_container_width=True)
```

### 4.2 摄像头 `st.camera_input`

```python
# 调用摄像头拍照
photo = st.camera_input("拍照")
if photo:
    st.image(photo, caption="拍摄的照片")
    # 可以进一步处理图片...
```

### 4.3 音频录制 `st.audio_input`（新版）

```python
# 录制音频
audio = st.audio_input("录音")
if audio:
    st.audio(audio)
```

### 4.4 图片与音频展示

```python
# 展示图片（URL 或本地路径）
st.image("https://picsum.photos/400/200", caption="示例图片")
st.image(["https://picsum.photos/200/200",
          "https://picsum.photos/200/200"],
         width=200)  # 多张图并排

# 展示音频
st.audio("https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3")

# 展示视频
st.video("https://www.w3schools.com/html/mov_bbb.mp4")
```

---

## 五、状态管理（Session State）

### 5.1 什么是 Session State？

Streamlit 每次用户交互都会**从头到尾重新执行整个脚本**。这意味着局部变量会丢失。`st.session_state` 就是用来**在多次运行之间保存数据**的：

```python
import streamlit as st

# 初始化 session state
if "count" not in st.session_state:
    st.session_state.count = 0
if "history" not in st.session_state:
    st.session_state.history = []

st.write(f"计数器：{st.session_state.count}")

if st.button("点击 +1"):
    st.session_state.count += 1
    st.session_state.history.append(f"第{st.session_state.count}次点击")
    st.rerun()  # 手动触发重跑，立即更新界面
```

### 5.2 Session State 与组件绑定

可以将组件直接绑定到 session_state 的键：

```python
import streamlit as st

# 方式一：用 key 参数（自动绑定）
st.text_input("你的名字", key="name")
st.write(f"名字：{st.session_state.name}")

st.slider("年龄", 0, 100, key="age")
st.write(f"年龄：{st.session_state.age}")

# 方式二：用 callback 回调函数
def on_name_change():
    st.session_state.greeting = f"你好，{st.session_state.new_name}！"

st.text_input("输入名字", key="new_name", on_change=on_name_change)
if "greeting" in st.session_state:
    st.success(st.session_state.greeting)
```

### 5.3 实战：简易待办清单（带持久化）

```python
import streamlit as st

# 初始化待办列表
if "todos" not in st.session_state:
    st.session_state.todos = []
if "next_id" not in st.session_state:
    st.session_state.next_id = 1

st.title("📋 待办清单")

# 添加新待办
with st.form("add_todo"):
    col1, col2 = st.columns([3, 1])
    with col1:
        task = st.text_input("任务内容", placeholder="输入新任务...")
    with col2:
        priority = st.selectbox("优先级", ["低", "中", "高"])
    submitted = st.form_submit_button("添加")
    if submitted and task:
        st.session_state.todos.append({
            "id": st.session_state.next_id,
            "task": task,
            "priority": priority,
            "done": False
        })
        st.session_state.next_id += 1
        st.rerun()

# 展示待办列表
for i, todo in enumerate(st.session_state.todos):
    col1, col2, col3, col4 = st.columns([0.5, 3, 1, 1])
    with col1:
        st.checkbox("", value=todo["done"], key=f"done_{todo['id']}")
    with col2:
        if todo["done"]:
            st.markdown(f"~~{todo['task']}~~")
        else:
            st.write(todo["task"])
    with col3:
        color = {"低": "🟢", "中": "🟡", "高": "🔴"}
        st.write(color[todo["priority"]])
    with col4:
        if st.button("删除", key=f"del_{todo['id']}"):
            st.session_state.todos.pop(i)
            st.rerun()

# 统计信息
total = len(st.session_state.todos)
done = sum(1 for t in st.session_state.todos if t["done"])
st.caption(f"总计 {total} 项，已完成 {done} 项")
```

---

## 六、布局美化进阶

### 6.1 分栏 `st.columns`

```python
# 多列布局
col1, col2, col3 = st.columns(3)

with col1:
    st.metric("总用户", "12,345", delta="↑ 234")
with col2:
    st.metric("活跃用户", "8,901", delta="↑ 156")
with col3:
    st.metric("转化率", "3.2%", delta="↓ 0.1%")

# 不等宽分栏
left, right = st.columns([2, 1])  # 左边占2/3，右边占1/3
with left:
    st.line_chart([1, 3, 2, 5, 4])
with right:
    st.bar_chart([3, 1, 4, 2, 5])
```

### 6.2 标签页 `st.tabs`

```python
tab1, tab2, tab3 = st.tabs(["📊 概览", "📋 详情", "⚙️ 设置"])

with tab1:
    st.header("数据概览")
    st.write("这里是概览页面的内容...")

with tab2:
    st.header("详细数据")
    import pandas as pd
    st.dataframe(pd.DataFrame({
        "名称": ["A", "B", "C"],
        "数值": [100, 200, 300]
    }))

with tab3:
    st.header("系统设置")
    st.slider("刷新频率", 1, 60, value=5)
```

### 6.3 折叠面板 `st.expander`

```python
with st.expander("点击展开更多信息", expanded=False):
    st.write("这是一段被折叠的内容。")
    st.write("可以放任何 Streamlit 组件。")
    st.code("print('Hello Streamlit!')")

# 用折叠面板展示 FAQ
with st.expander("❓ 常见问题", expanded=False):
    st.markdown("""
    **Q: Streamlit 支持哪些浏览器？**
    A: 支持所有现代浏览器（Chrome、Firefox、Safari、Edge）。

    **Q: Streamlit 应用能部署到公网吗？**
    A: 可以！官方提供 Streamlit Community Cloud 免费部署。

    **Q: 如何自定义主题？**
    A: 在 `.streamlit/config.toml` 文件中配置颜色和字体。
    """)
```

### 6.4 弹窗/模态框 `st.popover`

```python
# 点击按钮弹出浮层（新版功能）
with st.popover("📥 导出选项"):
    fmt = st.radio("导出格式", ["CSV", "Excel", "JSON"])
    st.checkbox("包含表头")
    st.button("确认导出")
```

### 6.5 状态容器 `st.status`

用于展示带步骤的进度状态：

```python
import time

with st.status("正在处理数据...", expanded=True) as status:
    st.write("📥 下载数据中...")
    time.sleep(1)
    st.write("🧹 清洗数据中...")
    time.sleep(1)
    st.write("📊 生成图表中...")
    time.sleep(1)
    status.update(label="处理完成！", state="complete", expanded=False)
```

### 6.6 完整布局模板

```python
import streamlit as st
import pandas as pd
import numpy as np

st.set_page_config(page_title="数据分析平台", layout="wide")

# 侧边栏导航
with st.sidebar:
    st.title("🧭 导航")
    page = st.radio("页面", ["📊 仪表盘", "📈 分析", "📋 数据"])

st.title("数据分析平台")

if page == "📊 仪表盘":
    # 顶部指标卡片
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric("总收入", "¥1,234,567", "↑ 12.5%")
    with col2:
        st.metric("订单数", "8,432", "↑ 8.3%")
    with col3:
        st.metric("客单价", "¥146.4", "↓ 2.1%")
    with col4:
        st.metric("退货率", "3.2%", "↓ 0.5%")

    st.divider()

    # 图表区域
    left, right = st.columns(2)
    with left:
        st.subheader("📈 月度趋势")
        data = pd.DataFrame({
            "月份": pd.date_range("2026-01", periods=6, freq="M"),
            "收入": np.random.randint(100, 500, 6) * 1000
        })
        st.line_chart(data, x="月份", y="收入")

    with right:
        st.subheader("🏷️ 分类占比")
        st.bar_chart({"电子产品": 45, "服装": 30, "食品": 25})

elif page == "📈 分析":
    tab1, tab2 = st.tabs(["趋势分析", "对比分析"])
    with tab1:
        st.write("趋势分析内容...")
    with tab2:
        st.write("对比分析内容...")
else:
    uploaded = st.file_uploader("上传数据文件", type=["csv"])
    if uploaded:
        df = pd.read_csv(uploaded)
        st.dataframe(df)
```

---

## 七、通知与反馈组件

| 组件 | 用途 | 示例 |
|------|------|------|
| `st.success` | 成功提示（绿色） | `st.success("保存成功！")` |
| `st.error` | 错误提示（红色） | `st.error("输入有误")` |
| `st.warning` | 警告提示（黄色） | `st.warning("注意风险")` |
| `st.info` | 信息提示（蓝色） | `st.info("提示信息")` |
| `st.exception` | 异常详情 | `st.exception(e)` |
| `st.toast` | 轻提示（右下角弹出） | `st.toast("操作完成")` |
| `st.balloons` | 庆祝动画 | `st.balloons()` |
| `st.snow` | 飘雪动画 | `st.snow()` |

```python
# Toast 轻提示（不占用布局空间）
st.toast("数据加载中...", icon="⏳")

# 各种通知
st.success("✅ 操作成功完成！")
st.info("💡 提示：你可以使用 Ctrl+S 保存")
st.warning("⚠️ 磁盘空间不足，仅剩 2GB")
st.error("❌ 连接超时，请检查网络")
```

---

## 八、综合实战：学生成绩管理应用

整合今天学到的所有组件，构建一个完整的学生成绩管理应用：

```python
import streamlit as st
import pandas as pd
import numpy as np

# ========== 页面配置 ==========
st.set_page_config(page_title="学生成绩管理系统", layout="wide")

# ========== 初始化数据 ==========
if "students" not in st.session_state:
    st.session_state.students = pd.DataFrame(
        np.random.randint(60, 100, size=(10, 5)),
        columns=["语文", "数学", "英语", "物理", "化学"]
    )
    st.session_state.students.insert(0, "姓名",
        [f"学生{i+1:02d}" for i in range(10)])

# ========== 侧边栏操作 ==========
with st.sidebar:
    st.header("🔧 操作面板")

    # 添加学生
    with st.expander("➕ 添加学生", expanded=False):
        with st.form("add_student"):
            name = st.text_input("姓名", placeholder="输入姓名")
            scores = {}
            for subject in ["语文", "数学", "英语", "物理", "化学"]:
                scores[subject] = st.number_input(
                    subject, min_value=0, max_value=100,
                    value=75, key=f"add_{subject}")
            submitted = st.form_submit_button("确认添加")
            if submitted and name:
                new_row = {"姓名": name, **scores}
                st.session_state.students = pd.concat(
                    [st.session_state.students, pd.DataFrame([new_row])],
                    ignore_index=True)
                st.toast(f"已添加学生：{name}")
                st.rerun()

    # 数据筛选
    st.divider()
    st.subheader("📊 数据筛选")
    min_avg = st.slider("最低平均分", 0, 100, value=0)
    subjects = st.multiselect(
        "选择显示科目",
        options=["语文", "数学", "英语", "物理", "化学"],
        default=["语文", "数学", "英语"])

# ========== 主内容区 ==========
st.title("📚 学生成绩管理系统")

# 指标统计
df = st.session_state.students
df["平均分"] = df[["语文", "数学", "英语", "物理", "化学"]].mean(axis=1)
filtered = df[df["平均分"] >= min_avg]

col1, col2, col3, col4 = st.columns(4)
with col1:
    st.metric("学生总数", len(df))
with col2:
    st.metric("全班均分", f"{df['平均分'].mean():.1f}")
with col3:
    st.metric("最高均分", f"{df['平均分'].max():.1f}")
with col4:
    st.metric("优秀率(≥90)", f"{(df['平均分']>=90).sum() / len(df) * 100:.1f}%")

st.divider()

# 标签页展示
tab1, tab2, tab3 = st.tabs(["📋 成绩表", "📊 图表分析", "🏆 排行榜"])

with tab1:
    display_cols = ["姓名"] + subjects + ["平均分"]
    st.dataframe(filtered[display_cols].sort_values("平均分", ascending=False),
                 use_container_width=True)

with tab2:
    left, right = st.columns(2)
    with left:
        st.subheader("各科平均分")
        avg_scores = df[["语文", "数学", "英语", "物理", "化学"]].mean()
        st.bar_chart(avg_scores)
    with right:
        st.subheader("成绩分布")
        st.line_chart(df["平均分"].value_counts().sort_index())

with tab3:
    top5 = df.nlargest(5, "平均分")[["姓名", "平均分"]]
    for i, (idx, row) in enumerate(top5.iterrows(), 1):
        medal = {1: "🥇", 2: "🥈", 3: "🥉"}.get(i, f"  {i}")
        st.write(f"{medal}  **{row['姓名']}**  —  平均分：{row['平均分']:.1f}")

# 删除操作
st.divider()
with st.popover("🗑️ 删除学生"):
    del_name = st.selectbox("选择要删除的学生", df["姓名"].tolist())
    if st.button("确认删除", type="secondary"):
        st.session_state.students = df[df["姓名"] != del_name].copy()
        st.toast(f"已删除学生：{del_name}")
        st.rerun()
```

---

## 九、练习题

### 练习 1（基础）：个人信息表单

创建一个包含以下字段的表单页面：姓名、年龄、性别（单选）、城市（下拉）、兴趣爱好（多选）、自我介绍（多行文本）。点击提交后，在侧边栏展示所有输入信息。

### 练习 2（进阶）：产品比价工具

读取或创建一个包含产品名称、价格、评分的 DataFrame。使用 `st.slider` 设置价格区间筛选，`st.multiselect` 筛选分类，用标签页展示"数据表"和"价格分布图"，侧边栏展示汇总统计。

### 练习 3（挑战）：完整日记本应用

使用 `st.session_state` 实现一个日记本：
- 用 `st.date_input` 选择日期
- 用 `st.text_area` 输入日记内容
- 用 `st.selectbox` 选择心情（开心/普通/难过/愤怒）
- 所有日记保存在 session_state 中
- 侧边栏可以搜索日记内容
- 主页面用标签页展示"写日记"和"查看日记"

---

## 十、常见问题

**Q1：为什么我的变量每次交互都重置了？**
A：Streamlit 每次交互都从头执行脚本。需要跨交互保存的数据必须存入 `st.session_state`。

**Q2：`st.rerun()` 和 `st.experimental_rerun()` 有什么区别？**
A：`st.experimental_rerun()` 是旧名，已在新版中统一为 `st.rerun()`。功能完全一样，用 `st.rerun()` 即可。

**Q3：如何实现"确认后执行"的按钮（防止误触）？**
A：用 `st.popover` 包裹按钮，或使用 `st.form` + `st.form_submit_button`（只在表单提交时触发）。

**Q4：session_state 数据在刷新浏览器后会丢失吗？**
A：默认会丢失。如果需要持久化，可以将数据保存到文件或数据库中，每次启动时加载。

**Q5：如何自定义 Streamlit 的主题和颜色？**
A：在项目根目录创建 `.streamlit/config.toml` 文件，配置 `[theme]` 部分的 `primaryColor`、`backgroundColor`、`font` 等参数。

---

## 十一、推荐学习资源

- 📖 [Streamlit 官方文档](https://docs.streamlit.io/) — 最权威的参考
- 📖 [Streamlit Gallery](https://streamlit.io/gallery) — 大量真实应用案例
- 📖 [Streamlit API 参考](https://docs.streamlit.io/library/api-reference) — 所有组件速查
- 📖 [菜鸟教程 - Streamlit](https://www.runoob.com/) — 中文入门教程
- 🎥 [Streamlit YouTube 频道](https://www.youtube.com/@Streamlit) — 官方视频教程

---

> 📌 **小结**：今天我们深入学习了 Streamlit 的各类组件——文本输入、选择控件、滑块、文件上传、Session State 状态管理、布局美化、通知反馈。这些组件是构建任何 Streamlit 应用时的"积木"。明天我们将学习**前端基础 HTML/CSS/JS**，了解 Web 开发的底层原理，为后续的综合项目实战打好基础。
