# Python Day 55 - 项目实战：Web 应用（二）

> Day 54 我们搭建了 MoneyTracker 的后端 API、数据库、认证模块和前端页面骨架。今天我们在这个项目基础上**继续打磨**：添加数据可视化（分类饼图 + 月度趋势折线图）、记录编辑功能、CSV 导出，最后学习如何将项目打包部署，让它在互联网上可以被任何人访问！

---

## 一、回顾：项目现状

Day 54 我们完成了这些模块：

| 模块 | 状态 |
|------|------|
| FastAPI 后端 API | ✅ 已完成（注册/登录/记账CRUD/统计） |
| SQLAlchemy + SQLite 数据库 | ✅ 已完成（User + Record 模型） |
| JWT 认证 | ✅ 已完成（密码哈希 + Token 生成验证） |
| HTML 前端页面 | ✅ 已完成（登录/注册/仪表盘） |
| CSS 样式 | ✅ 已完成（响应式布局） |
| JavaScript 交互逻辑 | ✅ 已完成（API 调用 + 动态渲染） |
| **数据可视化图表** | ❌ 今天实现 |
| **记录编辑功能** | ❌ 今天实现 |
| **CSV 导出** | ❌ 今天实现 |
| **项目部署** | ❌ 今天学习 |

今天的目录结构将在 Day 54 基础上扩展：

```
moneytracker/
├── app/
│   ├── main.py              # 今天补全：图表API + CSV导出
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   ├── auth.py
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── records.py       # 今天补全：图表数据 + CSV导出
│   ├── static/
│   │   ├── css/style.css
│   │   └── js/app.js        # 今天补全：编辑 + 图表
│   └── utils.py             # 🆕 图表生成工具
├── templates/
│   ├── base.html
│   ├── login.html
│   ├── register.html
│   └── dashboard.html       # 今天补全：图表区域
├── requirements.txt
└── run.py
```

---

## 二、后端：数据可视化 API

### 2.1 新增依赖

```bash
# 追加到 requirements.txt
matplotlib==3.9.2
```

### 2.2 图表生成工具（app/utils.py）

我们用一个独立的工具模块来生成图表图片。这体现了 **Day 18 学过的模块化思想**——把图表生成逻辑抽离出来，让路由代码保持简洁。

```python
# app/utils.py
import io
import base64
from typing import Dict, Any

import matplotlib
matplotlib.use('Agg')  # 无GUI后端，服务器环境必须设置！
import matplotlib.pyplot as plt

# 设置中文字体（解决图表中文显示为方块的问题）
plt.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False  # 解决负号显示问题


def generate_pie_chart(category_data: Dict[str, float], title: str = "分类占比") -> str:
    """
    生成分类饼图，返回 base64 编码的图片字符串

    Args:
        category_data: {分类名: 金额} 字典
        title: 图表标题

    Returns:
        base64 编码的 PNG 图片字符串（可直接嵌入 <img> 标签）
    """
    if not category_data:
        return ""

    fig, ax = plt.subplots(figsize=(6, 4))

    # 准备数据
    labels = list(category_data.keys())
    values = list(category_data.values())

    # 颜色方案
    colors = plt.cm.Set3(range(len(labels)))

    # 绘制饼图
    wedges, texts, autotexts = ax.pie(
        values,
        labels=labels,
        autopct='%1.1f%%',
        colors=colors,
        startangle=90,
        pctdistance=0.75,
        textprops={'fontsize': 10}
    )

    # 设置百分比文字样式
    for autotext in autotexts:
        autotext.set_fontsize(9)

    ax.set_title(title, fontsize=14, fontweight='bold', pad=15)

    # 转为 base64
    buf = io.BytesIO()
    fig.savefig(buf, format='png', dpi=100, bbox_inches='tight')
    buf.seek(0)
    img_base64 = base64.b64encode(buf.read()).decode('utf-8')
    plt.close(fig)

    return img_base64


def generate_trend_chart(
    daily_data: Dict[str, Dict[str, float]],
    title: str = "月度趋势"
) -> str:
    """
    生成月度收支趋势折线图，返回 base64 编码的图片字符串

    Args:
        daily_data: {
            "2026-07-01": {"income": 200, "expense": 50},
            "2026-07-02": {"income": 0, "expense": 30},
            ...
        }
        title: 图表标题

    Returns:
        base64 编码的 PNG 图片字符串
    """
    if not daily_data:
        return ""

    fig, ax = plt.subplots(figsize=(10, 4))

    dates = sorted(daily_data.keys())
    # 提取日期的"日"部分作为x轴标签
    day_labels = [d.split('-')[2] for d in dates]
    incomes = [daily_data[d].get('income', 0) for d in dates]
    expenses = [daily_data[d].get('expense', 0) for d in dates]

    # 绘制折线图（Day 41 学过的）
    ax.plot(day_labels, incomes, marker='o', markersize=4,
            color='#27ae60', linewidth=2, label='收入')
    ax.plot(day_labels, expenses, marker='s', markersize=4,
            color='#e74c3c', linewidth=2, label='支出')

    # 填充区域
    ax.fill_between(day_labels, incomes, alpha=0.1, color='#27ae60')
    ax.fill_between(day_labels, expenses, alpha=0.1, color='#e74c3c')

    ax.set_title(title, fontsize=14, fontweight='bold', pad=15)
    ax.set_xlabel('日期', fontsize=11)
    ax.set_ylabel('金额（¥）', fontsize=11)
    ax.legend(loc='upper right')
    ax.grid(True, alpha=0.3)

    # x轴标签太多时只显示部分
    if len(day_labels) > 15:
        step = len(day_labels) // 10
        ax.set_xticks(day_labels[::step])

    plt.tight_layout()

    # 转为 base64
    buf = io.BytesIO()
    fig.savefig(buf, format='png', dpi=100, bbox_inches='tight')
    buf.seek(0)
    img_base64 = base64.b64encode(buf.read()).decode('utf-8')
    plt.close(fig)

    return img_base64
```

> **要点**：
> - `matplotlib.use('Agg')` 必须在 `import pyplot` **之前**调用。`Agg` 是非交互式后端，专门用于服务器环境（没有显示器）。如果漏掉这一行，服务器环境会报错 `_tkinter.TclError: no display name`。
> - 用 `io.BytesIO()` 把图片写入内存缓冲区，再转 base64——这样图表不需要存成磁盘文件，直接作为字符串传给前端。
> - `plt.close(fig)` **必须调用**！否则每次请求都会泄漏一个 matplotlib figure 对象，内存很快耗尽。

### 2.3 新增图表与导出 API（routers/records.py 补充）

在 `routers/records.py` 文件末尾追加以下接口：

```python
# app/routers/records.py（追加以下代码）

import io
from fastapi.responses import StreamingResponse
from app.utils import generate_pie_chart, generate_trend_chart


@router.get("/chart/pie")
def get_pie_chart(
    year: int = Query(..., description="年份"),
    month: int = Query(..., ge=1, le=12, description="月份"),
    chart_type: str = Query("expense", description="income或expense"),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """获取分类饼图（返回base64图片）"""
    records = db.query(Record).filter(
        and_(
            Record.user_id == current_user.id,
            func.strftime("%Y", Record.date) == str(year),
            func.strftime("%m", Record.date) == f"{month:02d}",
            Record.record_type == chart_type
        )
    ).all()

    # 按分类汇总
    category_data = {}
    for r in records:
        category_data[r.category] = category_data.get(r.category, 0) + r.amount

    title = f"{year}年{month}月{'收入' if chart_type == 'income' else '支出'}分类"
    img_b64 = generate_pie_chart(category_data, title)

    return {"image": img_b64, "categories": category_data}


@router.get("/chart/trend")
def get_trend_chart(
    year: int = Query(..., description="年份"),
    month: int = Query(..., ge=1, le=12, description="月份"),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """获取月度趋势折线图（返回base64图片）"""
    records = db.query(Record).filter(
        and_(
            Record.user_id == current_user.id,
            func.strftime("%Y", Record.date) == str(year),
            func.strftime("%m", Record.date) == f"{month:02d}"
        )
    ).all()

    # 按日期汇总
    daily_data = {}
    for r in records:
        day_str = r.date.strftime("%Y-%m-%d") if hasattr(r.date, 'strftime') else str(r.date)[:10]
        if day_str not in daily_data:
            daily_data[day_str] = {"income": 0.0, "expense": 0.0}
        daily_data[day_str][r.record_type] += r.amount

    title = f"{year}年{month}月收支趋势"
    img_b64 = generate_trend_chart(daily_data, title)

    return {"image": img_b64, "daily_data": daily_data}


@router.get("/export/csv")
def export_csv(
    start_date: Optional[str] = Query(None, description="开始日期"),
    end_date: Optional[str] = Query(None, description="结束日期"),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """导出记账记录为CSV文件（利用Day 23学的csv模块）"""
    import csv

    query = db.query(Record).filter(Record.user_id == current_user.id)
    if start_date:
        query = query.filter(Record.date >= start_date)
    if end_date:
        query = query.filter(Record.date <= end_date)

    records = query.order_by(Record.date.desc()).all()

    # 写入内存中的CSV
    output = io.StringIO()
    writer = csv.writer(output)

    # 写入表头（BOM头解决Excel中文乱码）
    output.write('\ufeff')  # UTF-8 BOM
    writer.writerow(['日期', '类型', '分类', '金额', '备注'])

    for r in records:
        date_str = r.date.strftime("%Y-%m-%d") if hasattr(r.date, 'strftime') else str(r.date)[:10]
        type_str = '收入' if r.record_type == 'income' else '支出'
        writer.writerow([date_str, type_str, r.category, r.amount, r.remark])

    # 转为字节流返回
    csv_bytes = output.getvalue().encode('utf-8-sig')
    output.close()

    return StreamingResponse(
        io.BytesIO(csv_bytes),
        media_type='text/csv',
        headers={'Content-Disposition': 'attachment; filename=records.csv'}
    )
```

> **要点**：
> - 图表 API 返回 JSON 格式 `{"image": "base64字符串"}`，前端收到后直接赋值给 `<img>` 标签的 `src` 属性。
> - CSV 导出使用 `StreamingResponse` 流式返回——不用先写文件到磁盘，直接从内存发送。`\ufeff` 是 UTF-8 BOM 头，能让 Excel 正确识别中文编码。
> - `export_csv` 同时支持日期范围筛选，和 `list_records` 共用一套筛选逻辑。

---

## 三、前端：补全仪表盘页面

### 3.1 更新 dashboard.html 模板

在 `templates/dashboard.html` 的记录列表 `<div class="record-list">` 之前，插入图表区域和编辑弹窗：

```html
<!-- 在 dashboard.html 的筛选栏和记录列表之间添加以下内容 -->

<!-- ========== 图表区域 ========== -->
<div class="charts-section">
    <div class="chart-tabs">
        <button class="chart-tab active" onclick="switchChart('expense')">支出饼图</button>
        <button class="chart-tab" onclick="switchChart('income')">收入饼图</button>
        <button class="chart-tab" onclick="switchChart('trend')">趋势图</button>
    </div>
    <div class="chart-container" id="chartContainer">
        <img id="chartImage" src="" alt="数据图表">
        <p id="chartEmpty" style="text-align:center;color:#999;padding:3rem;display:none">
            暂无数据，请先添加记录
        </p>
    </div>
</div>

<!-- ========== 编辑弹窗 ========== -->
<div id="editModal" class="modal" style="display:none">
    <div class="modal-content">
        <h3>编辑记录</h3>
        <form id="editForm" onsubmit="handleEditRecord(event)">
            <input type="hidden" id="editId">
            <div class="form-group">
                <label>类型</label>
                <select id="editType" required>
                    <option value="expense">支出</option>
                    <option value="income">收入</option>
                </select>
            </div>
            <div class="form-group">
                <label>金额</label>
                <input type="number" id="editAmount" step="0.01" min="0.01" required>
            </div>
            <div class="form-group">
                <label>分类</label>
                <select id="editCategory" required>
                    <option value="餐饮">餐饮</option>
                    <option value="交通">交通</option>
                    <option value="购物">购物</option>
                    <option value="娱乐">娱乐</option>
                    <option value="住房">住房</option>
                    <option value="医疗">医疗</option>
                    <option value="教育">教育</option>
                    <option value="工资">工资</option>
                    <option value="奖金">奖金</option>
                    <option value="其他">其他</option>
                </select>
            </div>
            <div class="form-group">
                <label>日期</label>
                <input type="date" id="editDate" required>
            </div>
            <div class="form-group">
                <label>备注</label>
                <input type="text" id="editRemark" maxlength="255">
            </div>
            <div class="modal-actions">
                <button type="button" class="btn btn-cancel" onclick="closeEditModal()">取消</button>
                <button type="submit" class="btn btn-primary">保存</button>
            </div>
        </form>
    </div>
</div>
```

### 3.2 追加 CSS 样式

在 `app/static/css/style.css` 末尾追加：

```css
/* ========== 图表区域 ========== */
.charts-section {
    background: white;
    padding: 1.5rem;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
    margin-bottom: 1.5rem;
}

.chart-tabs {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
}

.chart-tab {
    padding: 0.5rem 1rem;
    border: 1px solid #ddd;
    background: white;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    font-size: 0.9rem;
}

.chart-tab.active {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    border-color: transparent;
}

.chart-tab:hover:not(.active) {
    border-color: #667eea;
    color: #667eea;
}

.chart-container {
    text-align: center;
    min-height: 200px;
}

.chart-container img {
    max-width: 100%;
    border-radius: 8px;
}

/* ========== 弹窗 ========== */
.modal {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

.modal-content {
    background: white;
    padding: 2rem;
    border-radius: 12px;
    width: 90%;
    max-width: 450px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.modal-content h3 {
    margin-bottom: 1.5rem;
    text-align: center;
}

.modal-actions {
    display: flex;
    gap: 1rem;
    margin-top: 1.5rem;
    justify-content: flex-end;
}

.btn-cancel {
    background: #95a5a6;
    color: white;
}
```

### 3.3 追加 JavaScript 逻辑

在 `app/static/js/app.js` 末尾追加：

```javascript
// ========== 图表功能 ==========

// 当前图表类型
let currentChartType = 'expense';

// 切换图表类型
function switchChart(type) {
    currentChartType = type;

    // 更新按钮状态
    document.querySelectorAll('.chart-tab').forEach(btn => btn.classList.remove('active'));
    event.target.classList.add('active');

    loadChart();
}

// 加载图表
async function loadChart() {
    const monthInput = document.getElementById('filterMonth');
    if (!monthInput) return;

    const month = monthInput.value || new Date().toISOString().slice(0, 7);
    const [year, m] = month.split('-');

    const chartImg = document.getElementById('chartImage');
    const chartEmpty = document.getElementById('chartEmpty');

    try {
        let url;
        if (currentChartType === 'trend') {
            url = `/api/records/chart/trend?year=${year}&month=${m}`;
        } else {
            url = `/api/records/chart/pie?year=${year}&month=${m}&chart_type=${currentChartType}`;
        }

        const response = await apiRequest(url);
        const data = await response.json();

        if (data.image) {
            chartImg.src = 'data:image/png;base64,' + data.image;
            chartImg.style.display = 'block';
            chartEmpty.style.display = 'none';
        } else {
            chartImg.style.display = 'none';
            chartEmpty.style.display = 'block';
        }
    } catch (error) {
        console.error('加载图表失败:', error);
        chartImg.style.display = 'none';
        chartEmpty.style.display = 'block';
        chartEmpty.textContent = '加载失败: ' + error.message;
    }
}


// ========== 编辑功能 ==========

// 打开编辑弹窗
function openEditModal(id, recordType, amount, category, date, remark) {
    document.getElementById('editId').value = id;
    document.getElementById('editType').value = recordType;
    document.getElementById('editAmount').value = amount;
    document.getElementById('editCategory').value = category;
    // 日期格式转换
    document.getElementById('editDate').value = date.split('T')[0];
    document.getElementById('editRemark').value = remark || '';
    document.getElementById('editModal').style.display = 'flex';
}

// 关闭编辑弹窗
function closeEditModal() {
    document.getElementById('editModal').style.display = 'none';
}

// 提交编辑
async function handleEditRecord(event) {
    event.preventDefault();

    const id = document.getElementById('editId').value;
    const data = {
        amount: parseFloat(document.getElementById('editAmount').value),
        category: document.getElementById('editCategory').value,
        record_type: document.getElementById('editType').value,
        date: document.getElementById('editDate').value + 'T00:00:00',
        remark: document.getElementById('editRemark').value
    };

    try {
        const response = await apiRequest(`/api/records/${id}`, {
            method: 'PUT',
            body: JSON.stringify(data)
        });

        if (!response.ok) {
            const err = await response.json();
            throw new Error(err.detail || '更新失败');
        }

        closeEditModal();
        loadRecords();
        loadStats();
        loadChart();

    } catch (error) {
        alert('更新失败: ' + error.message);
    }
}


// ========== CSV 导出 ==========

async function exportCSV() {
    const monthInput = document.getElementById('filterMonth');
    let params = [];

    if (monthInput && monthInput.value) {
        const [year, month] = monthInput.value.split('-');
        params.push(`start_date=${year}-${month}-01`);
        const lastDay = new Date(year, month, 0).getDate();
        params.push(`end_date=${year}-${month}-${lastDay}`);
    }

    const url = '/api/records/export/csv' + (params.length ? '?' + params.join('&') : '');

    try {
        const response = await apiRequest(url);
        const blob = await response.blob();

        // 创建下载链接
        const downloadUrl = window.URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = downloadUrl;
        a.download = `记账记录_${monthInput?.value || 'all'}.csv`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        window.URL.revokeObjectURL(downloadUrl);
    } catch (error) {
        alert('导出失败: ' + error.message);
    }
}
```

### 3.4 更新 loadRecords 函数

修改 `loadRecords` 函数中的表格行渲染部分，为每条记录添加"编辑"按钮：

```javascript
// 替换 loadRecords 中的 tbody.innerHTML 部分
tbody.innerHTML = records.map(r => `
    <tr>
        <td>${formatDate(r.date)}</td>
        <td><span class="${r.record_type}-tag">${r.record_type === 'income' ? '收入' : '支出'}</span></td>
        <td>${r.category}</td>
        <td>¥${r.amount.toFixed(2)}</td>
        <td>${r.remark || '-'}</td>
        <td>
            <button class="btn btn-primary btn-sm" onclick="openEditModal(
                ${r.id}, '${r.record_type}', ${r.amount}, '${r.category}', '${r.date}', '${r.remark}'
            )">编辑</button>
            <button class="btn btn-danger btn-sm" onclick="deleteRecord(${r.id})">删除</button>
        </td>
    </tr>
`).join('');
```

同时在筛选栏旁边添加导出按钮（修改 dashboard.html 的筛选栏部分）：

```html
<!-- 修改后的筛选栏 -->
<div class="filter-bar">
    <input type="month" id="filterMonth" onchange="loadRecords(); loadChart()">
    <input type="text" id="filterCategory" placeholder="按分类筛选"
           onchange="loadRecords()">
    <button class="btn btn-primary btn-sm" onclick="exportCSV()">导出CSV</button>
</div>
```

> **要点**：`openEditModal` 把记录的当前值作为参数传入弹窗表单，用户修改后点击"保存"调用 `PUT` 接口更新。注意 `r.date` 的格式可能包含时间部分（如 `2026-07-01T00:00:00`），用 `.split('T')[0]` 只取日期部分。

---

## 四、后端补全：main.py 更新

在 `main.py` 中补充图表所需的页面路由，以及确保启动时正确加载模板：

```python
# app/main.py（完整版）
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from fastapi.requests import Request
from fastapi.responses import HTMLResponse, RedirectResponse

from app.database import engine, Base
from app.routers import users, records

# 创建所有数据库表
Base.metadata.create_all(bind=engine)

app = FastAPI(title="MoneyTracker", version="2.0")

# 注册API路由
app.include_router(users.router, prefix="/api/users", tags=["用户"])
app.include_router(records.router, prefix="/api/records", tags=["记账"])

# 挂载静态文件
app.mount("/static", StaticFiles(directory="app/static"), name="static")

# 模板引擎
templates = Jinja2Templates(directory="templates")


# ========== 页面路由 ==========

@app.get("/", response_class=HTMLResponse)
async def root():
    return RedirectResponse(url="/login")


@app.get("/login", response_class=HTMLResponse)
async def login_page(request: Request):
    return templates.TemplateResponse("login.html", {"request": request})


@app.get("/register", response_class=HTMLResponse)
async def register_page(request: Request):
    return templates.TemplateResponse("register.html", {"request": request})


@app.get("/dashboard", response_class=HTMLResponse)
async def dashboard_page(request: Request):
    return templates.TemplateResponse("dashboard.html", {"request": request})
```

---

## 五、完整项目运行测试

### 5.1 启动步骤

```bash
# 1. 进入项目目录
cd moneytracker

# 2. 安装所有依赖（含 matplotlib）
pip install -r requirements.txt

# 3. 启动服务
python run.py
```

### 5.2 完整测试流程

```
① 打开浏览器访问 http://localhost:8000
② 自动跳转登录页 → 注册新账号 → 登录
③ 进入仪表盘，添加 5-10 条收支记录（不同分类、不同日期）
④ 点击"支出饼图"标签，查看分类占比
⑤ 点击"收入饼图"标签，查看收入分布
⑥ 点击"趋势图"标签，查看每日收支折线
⑦ 切换月份筛选器，图表自动刷新
⑧ 点击某条记录的"编辑"按钮 → 修改金额 → 保存
⑨ 点击"导出CSV"按钮 → 下载文件 → 用 Excel 打开查看
⑩ 打开 http://localhost:8000/docs 查看更新后的API文档
```

---

## 六、项目部署：让全世界都能访问

项目在本地跑通了，接下来学习如何**部署到互联网**。

### 6.1 部署方案对比

| 方案 | 难度 | 费用 | 适合场景 |
|------|------|------|----------|
| PythonAnywhere | ⭐ | 免费额度 | 学习和小项目 |
| Railway | ⭐⭐ | 免费额度+按量 | 小型API服务 |
| 腾讯云/阿里云服务器 | ⭐⭐⭐ | 按月付费（学生机约10元/月） | 正式项目 |
| Docker 容器 | ⭐⭐⭐⭐ | 取决于服务器 | 专业部署 |

### 6.2 最简单的方式：PythonAnywhere

PythonAnywhere 是一个免费的 Python 云托管平台，非常适合学习阶段部署项目。

```bash
# 步骤1：注册 PythonAnywhere 账号
# 访问 https://www.pythonanywhere.com/ 注册（免费版即可）

# 步骤2：上传项目文件
# 通过网页控制台的 Files 标签页，上传整个 moneytracker 文件夹

# 步骤3：打开 Bash 控制台，安装依赖
pip install --user fastapi uvicorn sqlalchemy python-jose passlib python-multipart jinja2 matplotlib

# 步骤4：修改数据库路径（SQLite 在云平台需要绝对路径）
# 在 database.py 中改为：
DATABASE_URL = "sqlite:////home/你的用户名/moneytracker/moneytracker.db"

# 步骤5：通过 Web 标签页添加一个 Web App
# 选择 "Manual configuration" → Python 3.10+
# WSGI 配置文件填：/home/你的用户名/moneytracker/wsgi.py

# 步骤6：创建 wsgi.py 文件
```

```python
# wsgi.py（用于 PythonAnywhere 部署）
from app.main import app

# PythonAnywhere 使用 ASGI 模式运行 FastAPI
```

在 PythonAnywhere 的 Web 标签页设置：
- **Virtualenv**: 建议创建一个虚拟环境
- **WSGI file**: `/home/你的用户名/moneytracker/wsgi.py`
- **Command**: 直接运行即可，PythonAnywhere 会自动配置

### 6.3 更通用的方式：云服务器 + Docker

对于正式项目，推荐使用 Docker 容器化部署：

```dockerfile
# Dockerfile
# 使用官方Python镜像
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 先复制依赖文件（利用Docker缓存层加速构建）
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制项目代码
COPY . .

# 暴露端口
EXPOSE 8000

# 启动命令（生产环境用 gunicorn 替代 uvicorn --reload）
CMD ["gunicorn", "app.main:app", "-w", "2", "-k", "uvicorn.workers.UvicornWorker", "-b", "0.0.0.0:8000"]
```

```bash
# 构建Docker镜像
docker build -t moneytracker:latest .

# 运行容器
docker run -d -p 8000:8000 --name moneytracker moneytracker:latest
```

### 6.4 部署前的安全清单

在把项目放到互联网之前，**必须**检查以下几点：

| 项目 | 说明 | 状态 |
|------|------|------|
| SECRET_KEY | 改为环境变量，不要硬编码在代码里 | 🔴 必须 |
| SQLite 文件 | 不要提交到 Git（加入 .gitignore） | 🔴 必须 |
| CORS 配置 | 限制允许的域名，不要用 `*` 通配符 | 🟡 建议 |
| HTTPS | 生产环境必须使用 HTTPS 加密传输 | 🔴 必须 |
| 数据库备份 | 设置定时备份脚本 | 🟡 建议 |
| 错误信息 | 生产环境关闭 `echo=True` | 🟡 建议 |

将 SECRET_KEY 改为环境变量读取：

```python
# app/auth.py（修改 SECRET_KEY 部分）
import os

# 从环境变量读取，如果不存在则使用默认值（仅开发环境）
SECRET_KEY = os.getenv("SECRET_KEY", "dev-only-change-in-production")

# 生产环境启动时设置环境变量：
# export SECRET_KEY="一个随机的长字符串"
# 或者在 .env 文件中配置（Day 34 学过的 python-dotenv）
```

---

## 七、完整项目文件速查

以下是最终版 `requirements.txt`：

```
fastapi==0.111.0
uvicorn==0.30.1
sqlalchemy==2.0.31
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
jinja2==3.1.4
matplotlib==3.9.2
gunicorn==22.0.0
```

项目核心文件清单：

| 文件 | 功能 | 行数 |
|------|------|------|
| `app/database.py` | 数据库连接配置 | ~30行 |
| `app/models.py` | SQLAlchemy 数据模型 | ~40行 |
| `app/schemas.py` | Pydantic 请求/响应模型 | ~80行 |
| `app/auth.py` | 认证（密码哈希/JWT） | ~70行 |
| `app/utils.py` | 图表生成工具 | ~90行 |
| `app/routers/users.py` | 用户注册/登录API | ~40行 |
| `app/routers/records.py` | 记账CRUD/图表/导出API | ~180行 |
| `app/main.py` | FastAPI应用入口+页面路由 | ~50行 |
| `app/static/css/style.css` | 前端样式 | ~250行 |
| `app/static/js/app.js` | 前端交互逻辑 | ~300行 |
| `templates/dashboard.html` | 仪表盘页面模板 | ~100行 |
| `templates/login.html` | 登录页面 | ~30行 |
| `templates/register.html` | 注册页面 | ~30行 |
| **合计** | | **~1290行** |

一个 1300 行左右的完整 Web 应用——这就是你 55 天学习成果的浓缩！

---

## 八、常见问题（FAQ）

**Q1：matplotlib 在服务器上报 `no display name` 错误？**
A：这是因为服务器没有图形界面。在 `app/utils.py` 中，确保 `import matplotlib.pyplot as plt` **之前**执行 `matplotlib.use('Agg')`。`Agg` 是纯文件输出后端，不需要显示器。

**Q2：图表中文显示为方块怎么办？**
A：这是因为 matplotlib 找不到中文字体。Windows 上通常有 `SimHei` 和 `Microsoft YaHei`，代码中已配置。Linux 服务器可能需要安装中文字体包：`sudo apt install fonts-wqy-microhei`，然后在代码中改为 `'WenQuanYi Micro Hei'`。

**Q3：编辑弹窗中的日期输入框为空？**
A：检查 `openEditModal` 传入的 `date` 参数格式。API 返回的日期格式是 ISO 8601（如 `2026-07-01T00:00:00`），用 `split('T')[0]` 截取后赋值给 `<input type="date">`。

**Q4：导出的 CSV 在 Excel 中乱码？**
A：代码中已添加了 UTF-8 BOM 头（`\ufeff`），Excel 会正确识别编码。如果仍有问题，确保使用 `utf-8-sig` 编码导出（代码中已使用）。

**Q5：部署后 API 返回 404？**
A：检查 `main.py` 中的 `StaticFiles` 挂载路径和模板目录是否正确。部署环境的文件路径可能与本地不同（比如 `/home/user/moneytracker/app/static`），需要使用绝对路径。

---

## 九、练习题

### 练习1：添加预算功能（基础）
在统计卡片区域新增一个"本月预算"输入框，当支出超过预算时，结余卡片变为红色并显示警告文字 "已超出预算！"。需要新增一个 `Budget` 模型存储每月预算金额。

### 练习2：添加柱状图对比（进阶）
在 `app/utils.py` 中新增一个 `generate_bar_chart` 函数，生成最近 6 个月的收入 vs 支出对比柱状图。前端新增一个"月度对比"标签页显示此图。

### 练习3：使用 Docker 完整部署（挑战）
编写完整的 Dockerfile 和 docker-compose.yml，包含：
- FastAPI 应用容器
- Nginx 反向代理容器（处理 HTTPS）
- 数据卷持久化 SQLite 数据库文件
- 环境变量配置 SECRET_KEY

---

## 十、今日学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| matplotlib 中文教程 | https://www.runoob.com/matplotlib/matplotlib-tutorial.html | 菜鸟教程，适合快速查阅 |
| FastAPI 部署文档 | https://fastapi.tiangolo.com/zh/deployment/ | 官方部署指南（Docker/HTTPS等） |
| PythonAnywhere 教程 | https://help.pythonanywhere.com/ | 免费Python云平台使用指南 |
| Docker 入门教程 | https://www.runoob.com/docker/docker-tutorial.html | 菜鸟教程Docker篇 |
| Streamlit 部署 | https://docs.streamlit.io/deploy | 如果你更喜欢用Streamlit |

---

> **下一讲预告**：Day 56 开始进入数据分析工具实战——我们将用 Pandas + Matplotlib + Streamlit 构建一个**数据分析工具（一）**，从真实数据集出发，练习数据清洗、探索性分析和可视化报表的完整流程！

---

*学习进度：Day 55/60 | 距离完成还有 5 天*
