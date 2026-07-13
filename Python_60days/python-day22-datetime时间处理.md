# Python Day 22：datetime 时间处理

> **学习目标**：掌握 Python `datetime` 模块的核心用法，能够进行日期时间创建、格式化、计算时间差、时区处理，并结合 AMC 业务场景做日期逻辑判断。

---

## 一、为什么需要 datetime？

在数据开发工作中，日期时间无处不在：

| 场景 | 需求 |
|------|------|
| 监管报送 | 计算报送截止日期（如"次月15日前"） |
| 不良资产 | 计算资产处置时效（从收购日到处置日的天数） |
| ETL调度 | 判断数据是否已迟到（如"今日10点前产出"） |
| 报表标注 | 生成"截至YYYY年MM月DD日"的标题 |

如果你只会用字符串 `"2025-06-15"` 拼接日期，遇到计算、比较、格式转换就会很痛苦。`datetime` 模块就是你的"日期计算器"。

---

## 二、datetime 模块四大核心类

```
datetime 模块
├── date        → 只管日期（年月日）
├── time        → 只管时间（时分秒）
├── datetime    → 日期 + 时间（最常用）
└── timedelta   → 时间差（做加减运算）
```

### 2.1 date — 纯日期

```python
from datetime import date

# 创建日期
today = date.today()            # 今天
print(today)                     # 2026-06-21

specific = date(2026, 6, 15)    # 指定年月日
print(specific)                  # 2026-06-15

# 提取属性
print(specific.year)             # 2026
print(specific.month)            # 6
print(specific.day)              # 15

# 日期 → 字符串
print(specific.isoformat())      # '2026-06-15'（ISO标准格式）
```

### 2.2 time — 纯时间

```python
from datetime import time

# 创建时间
noon = time(12, 0, 0)           # 12点整
print(noon)                      # 12:00:00

morning = time(9, 30, 15)       # 9:30:15
print(morning.hour)              # 9
print(morning.minute)            # 30
```

> **实际使用频率**：`time` 类单独用得很少，大多数场景都用 `datetime`。

### 2.3 datetime — 日期 + 时间（⭐ 最常用）

```python
from datetime import datetime

# 获取当前时刻
now = datetime.now()
print(now)                        # 2026-06-21 10:40:40.123456

# 获取今天零点（只有日期）
today_zero = datetime.today()
print(today_zero)                 # 2026-06-21 10:40:40（注意：不是零点！）

# 指定具体时刻
deadline = datetime(2026, 7, 15, 18, 0, 0)  # 2026年7月15日18:00
print(deadline)

# 提取各部分
print(now.year, now.month, now.day)
print(now.hour, now.minute, now.second)
```

> ⚠️ `datetime.today()` 和 `datetime.now()` 效果类似，但 `now()` 可以传入时区参数。

### 2.4 timedelta — 时间差（⭐ 做计算的关键）

```python
from datetime import datetime, timedelta

# 创建时间差
delta_3days = timedelta(days=3)          # 3天
delta_1h30m = timedelta(hours=1, minutes=30)  # 1小时30分钟
delta_week = timedelta(weeks=1)          # 1周（=7天）

# 时间 + 时间差 = 新时间
now = datetime.now()
future = now + delta_3days
print(future)                            # 3天后

past = now - delta_week
print(past)                              # 7天前

# 两个时间相减 = 时间差
start = datetime(2026, 6, 1)
end = datetime(2026, 6, 21)
diff = end - start
print(diff.days)                         # 20（相差20天）
print(diff.total_seconds())              # 1728000.0（总秒数）
```

---

## 三、日期格式化与解析（⭐ 实战最频繁）

### 3.1 datetime → 字符串（strftime）

`strftime` = **string format**，把 datetime 变成你想要的字符串格式。

```python
from datetime import datetime

now = datetime.now()

# 常用格式符号
# %Y  → 四位年份    2026
# %m  → 两位月份    06
# %d  → 两位日期    21
# %H  → 24小时制    10
# %M  → 分钟        40
# %S  → 秒          40

# 举例
print(now.strftime("%Y-%m-%d"))          # '2026-06-21'
print(now.strftime("%Y年%m月%d日"))       # '2026年06月21日'
print(now.strftime("%Y/%m/%d %H:%M"))    # '2026/06/21 10:40'
print(now.strftime("%A"))                # 'Sunday'（星期名）
print(now.strftime("%B"))                # 'June'（月份名）
```

**AMC场景**：生成报表标题

```python
# "截至2026年6月21日的不良资产余额统计"
title = f"截至{now.strftime('%Y年%m月%d日')}的不良资产余额统计"
print(title)
```

### 3.2 字符串 → datetime（strptime）

`strptime` = **string parse**，把字符串解析回 datetime 对象。

```python
from datetime import datetime

# 解析标准格式
dt = datetime.strptime("2026-06-21", "%Y-%m-%d")
print(dt)                                # 2026-06-21 00:00:00

# 解析中文格式
dt2 = datetime.strptime("2026年06月21日", "%Y年%m月%d日")
print(dt2)

# 解析带时间的格式
dt3 = datetime.strptime("2026/06/21 10:40", "%Y/%m/%d %H:%M")
print(dt3)
```

> ⚠️ **strptime 最常见的错误**：格式符号和字符串不匹配！
> ```python
> # 错误示例
> datetime.strptime("2026-06-21", "%Y年%m月%d日")  # ❌ ValueError
> # 格式符必须和字符串的分隔符完全一致
> datetime.strptime("2026-06-21", "%Y-%m-%d")       # ✅
> ```

---

## 四、实战案例：AMC 业务中的日期计算

### 案例1：计算报送截止日期

```python
from datetime import datetime, timedelta

# 监管报送规则：数据月份的次月15日前报送
# 例如：6月的数据，7月15日前必须报送

data_month = datetime(2026, 6, 1)  # 数据所属月份

# 方法：加1个月 → 取第15天
# 注意：没有直接的 "加1个月" 操作，需要手动处理
if data_month.month == 12:
    deadline = datetime(data_month.year + 1, 1, 15)
else:
    deadline = datetime(data_month.year, data_month.month + 1, 15)

print(f"报送截止日期：{deadline.strftime('%Y-%m-%d')}")  # 2026-07-15

# 判断是否已超期
today = datetime.now()
if today > deadline:
    print("⚠️ 已超期！")
else:
    remaining = (deadline - today).days
    print(f"✅ 还剩 {remaining} 天")
```

### 案例2：计算资产处置时效

```python
from datetime import datetime

# 不良资产从收购日到处置日的天数
acquisition_date = datetime.strptime("2025-03-15", "%Y-%m-%d")
disposal_date = datetime.strptime("2026-06-10", "%Y-%m-%d")

holding_days = (disposal_date - acquisition_date).days
print(f"持有天数：{holding_days}天")  # 451天

# 判断是否超过1年处置时限
if holding_days > 365:
    print(f"⚠️ 超过1年时限，已持有 {holding_days}天")
else:
    print(f"✅ 在1年时限内，还剩 {365 - holding_days}天")
```

### 案例3：批量处理日期字符串

```python
from datetime import datetime

# 模拟从数据库读出的日期字符串列表
date_strings = ["2026-01-15", "2026-02-20", "2026-03-10", "2026-04-05"]

# 找出最近的日期
dates = [datetime.strptime(d, "%Y-%m-%d") for d in date_strings]
latest = max(dates)
print(f"最近日期：{latest.strftime('%Y-%m-%d')}")

# 找出距今超过90天的（过期数据）
today = datetime.now()
expired = [d for d in dates if (today - d).days > 90]
print(f"过期数据：{[d.strftime('%Y-%m-%d') for d in expired]}")
```

---

## 五、时区处理（进阶）

如果你的项目涉及跨时区数据（比如海外资产），需要了解时区：

```python
from datetime import datetime, timezone, timedelta

# UTC时间（国际标准时间）
utc_now = datetime.now(timezone.utc)
print(f"UTC时间：{utc_now}")

# 中国时区 UTC+8
china_tz = timezone(timedelta(hours=8))
china_now = datetime.now(china_tz)
print(f"中国时间：{china_now}")

# UTC转中国时间
china_from_utc = utc_now.astimezone(china_tz)
print(f"转换后：{china_from_utc}")
```

> 对于 AMC 国内业务，通常只用北京时间，不需要复杂时区处理。了解即可。

---

## 六、常用格式符号速查表

| 符号 | 含义 | 示例 |
|------|------|------|
| `%Y` | 四位年份 | 2026 |
| `%y` | 两位年份 | 26 |
| `%m` | 月份(01-12) | 06 |
| `%d` | 日期(01-31) | 21 |
| `%H` | 24小时(00-23) | 10 |
| `%I` | 12小时(01-12) | 10 |
| `%M` | 分钟(00-59) | 40 |
| `%S` | 秒(00-59) | 40 |
| `%A` | 星期全名 | Sunday |
| `%a` | 星期缩写 | Sun |
| `%B` | 月份全名 | June |
| `%b` | 月份缩写 | Jun |
| `%f` | 微秒 | 123456 |
| `%p` | AM/PM | AM |

---

## 七、练习题

### 练习1：日期格式转换

把字符串 `"2026/06/21"` 转成 `"2026年6月21日"` 格式。

```python
from datetime import datetime

# 你的代码
dt = datetime.strptime("2026/06/21", "%Y/%m/%d")
result = dt.strftime("%Y年%m月%d日")
print(result)  # 2026年06月21日
```

### 练习2：计算工作日

给定一个开始日期，计算 N 个工作日（跳过周末）后的日期。

```python
from datetime import datetime, timedelta

start = datetime(2026, 6, 21)  # 周日
n = 10  # 10个工作日

current = start
workdays_count = 0

while workdays_count < n:
    current += timedelta(days=1)
    # 周一到周五是工作日（weekday()返回0-6，0=周一，5=周六，6=周日）
    if current.weekday() < 5:
        workdays_count += 1

print(f"{n}个工作日后：{current.strftime('%Y-%m-%d %A')}")
```

### 练习3：报送超期检测

写一个函数，接收数据月份字符串 `"2026-06"` 和报送日期字符串 `"2026-07-14"`，判断是否超期（次月15日前）。

```python
from datetime import datetime

def check_report_deadline(data_month_str, report_date_str):
    """判断报送是否超期"""
    # 解析数据月份
    data_month = datetime.strptime(data_month_str, "%Y-%m")
    
    # 计算截止日期（次月15日）
    if data_month.month == 12:
        deadline = datetime(data_month.year + 1, 1, 15)
    else:
        deadline = datetime(data_month.year, data_month.month + 1, 15)
    
    # 解析报送日期
    report_date = datetime.strptime(report_date_str, "%Y-%m-%d")
    
    if report_date > deadline:
        return f"❌ 超期！截止{deadline.strftime('%Y-%m-%d')}, 报送于{report_date_str}"
    else:
        return f"✅ 合规，报送于{report_date_str}"

# 测试
print(check_report_deadline("2026-06", "2026-07-14"))  # ✅ 合规
print(check_report_deadline("2026-06", "2026-07-16"))  # ❌ 超期
```

---

## 八、常见问题

**Q1：怎么获取"今天零点"？**
```python
from datetime import datetime
today_zero = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
```

**Q2：怎么判断两个日期是不是同一天？**
```python
dt1.date() == dt2.date()  # 取date部分比较
```

**Q3：怎么给日期加减月份？**
datetime 没有"加1个月"的直接方法，需要手动：
```python
def add_months(dt, months):
    month = dt.month - 1 + months
    year = dt.year + month // 12
    month = month % 12 + 1
    day = min(dt.day, [31,28,31,30,31,30,31,31,30,31,30,31][month-1])
    return dt.replace(year=year, month=month, day=day)
```

**Q4：strftime 和 strptime 怎么区分？**
- **f** = format → datetime **变成** 字符串
- **p** = parse → 字符串 **解析成** datetime

---

## 推荐学习资源

- [廖雪峰 - datetime](https://www.liaoxuefeng.com/wiki/1016959663602400/1017786753950592)
- [菜鸟教程 - Python 日期和时间](https://www.runoob.com/python/python-date-time.html)
- [Python 官方文档 - datetime](https://docs.python.org/3/library/datetime.html)

---

> 📌 **今日总结**：`datetime` 是 Python 日期处理的瑞士军刀。记住四个类：`date`、`time`、`datetime`、`timedelta`，两个方法：`strftime`（格式化输出）和 `strptime`（解析输入）。结合 AMC 业务场景，日期计算和报送截止判断是最常见的应用。
