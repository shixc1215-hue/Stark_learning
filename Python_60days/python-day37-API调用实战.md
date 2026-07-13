# Python Day 37：API 调用实战 —— 让 Python 对话全世界

> Day 36 我们学会了 requests 库的基本用法。今天要更进一步——调用**真实公开 API**，获取天气、新闻、汇率、地图等真实数据，构建完整的网络数据应用。这是 Python 最实用的技能之一。

---

## 一、什么是 API？

**API（Application Programming Interface，应用程序编程接口）** 简单来说，就是**软件之间交流的桥梁**。

打个比方：

| 现实生活 | API 对应 |
|---------|---------|
| 餐厅的菜单 | API 文档——告诉你能"点"什么 |
| 服务员帮你下单 | API 调用——你发请求，服务器执行 |
| 菜端上来 | API 响应——服务器返回数据 |
| 你付钱才给菜单 | API 认证——需要密钥才能访问 |

### 1.1 REST API 核心概念

现在最主流的 API 风格叫 **REST API**。它的核心思想是：用 HTTP 方法（GET/POST/PUT/DELETE）操作"资源"（URL）。

```
HTTP 方法   作用              类比
GET         获取数据          读取
POST        创建数据          新增
PUT         更新数据（全量）   修改
DELETE      删除数据          删除
```

示例：

```
GET    https://api.example.com/users        → 获取所有用户
GET    https://api.example.com/users/1       → 获取 ID 为 1 的用户
POST   https://api.example.com/users        → 创建一个新用户
DELETE https://api.example.com/users/1       → 删除 ID 为 1 的用户
```

### 1.2 API 文档怎么看？

调用任何 API 之前，**一定要先读文档**。好的 API 文档会告诉你：

- **Base URL**：API 的根地址
- **端点（Endpoint）**：具体的请求路径
- **请求参数**：需要传什么数据
- **认证方式**：怎么证明你的身份
- **响应格式**：返回数据的结构
- **错误码**：各种错误代表什么

---

## 二、API 认证 —— "请出示您的证件"

很多 API 需要认证才能使用。常见三种方式：

### 2.1 API Key（API 密钥）

最简单的认证方式。服务提供商给你一个字符串，你每次请求都带上它：

```python
import requests

# 方式一：放在 URL 参数里
response = requests.get(
    "https://api.example.com/data",
    params={"api_key": "你的密钥"}
)

# 方式二：放在请求头里（更安全，推荐）
response = requests.get(
    "https://api.example.com/data",
    headers={"X-API-Key": "你的密钥"}
)
```

### 2.2 Bearer Token（令牌）

比 API Key 更安全，通常由登录流程获得：

```python
import requests

token = "eyJhbGciOiJIUzI1NiIs..."  # 登录后获得的令牌

response = requests.get(
    "https://api.example.com/me",
    headers={"Authorization": f"Bearer {token}"}
)
```

### 2.3 基本认证（Basic Auth）

用户名 + 密码直接放在请求头里（已编码）：

```python
import requests

response = requests.get(
    "https://api.example.com/data",
    auth=("用户名", "密码")
)
```

> **安全提示**：永远不要把密钥硬编码在代码里！应该用环境变量或配置文件管理。Day 34 学过的 `python-dotenv` 就是干这个的。

---

## 三、实战一：GitHub API —— 获取仓库信息

GitHub 提供了非常完善的免费公开 API，非常适合学习。不需要任何认证就能获取公开信息。

### 3.1 获取用户公开信息

```python
import requests


def get_github_profile(username: str) -> dict:
    """获取 GitHub 用户公开信息"""
    url = f"https://api.github.com/users/{username}"
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "User-Agent": "Python-Learner"
    }

    try:
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        return {"error": str(e)}


# 查看某用户的信息
data = get_github_profile("torvalds")  # Linux 之父
if "error" not in data:
    print(f"用户名: {data['name']}")
    print(f"简介: {data.get('bio', '无')}")
    print(f"位置: {data.get('location', '未知')}")
    print(f"公开仓库: {data['public_repos']} 个")
    print(f"粉丝: {data['followers']} 人")
    print(f"注册时间: {data['created_at']}")
```

### 3.2 获取仓库列表

```python
import requests


def get_user_repos(username: str, count: int = 5) -> list:
    """获取用户最近更新的仓库列表"""
    url = f"https://api.github.com/users/{username}/repos"
    params = {
        "sort": "updated",        # 按更新时间排序
        "direction": "desc",      # 降序（最新的在前）
        "per_page": count
    }

    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        repos = response.json()

        # 只提取我们关心的信息
        result = []
        for repo in repos:
            result.append({
                "name": repo["name"],
                "desc": repo.get("description", "无描述"),
                "stars": repo["stargazers_count"],
                "language": repo.get("language", "未知"),
                "url": repo["html_url"]
            })
        return result

    except requests.RequestException as e:
        print(f"请求失败: {e}")
        return []


repos = get_user_repos("python", 10)
for i, repo in enumerate(repos, 1):
    lang = repo["language"]
    print(f"{i:2d}. {repo['name']:<30} ⭐{repo['stars']:>6}  [{lang}]")
    print(f"    {repo['desc']}")
```

运行效果：

```
 1. cpython                        ⭐  68000  [C]
    The Python programming language
 2. devguide                       ⭐   2100  [Python]
    Python Developer's Guide
 3. bedrock                        ⭐    500  [Python]
    Reference implementation of Python
 ...
```

### 3.3 搜索仓库

```python
import requests


def search_github(keyword: str, language: str = None) -> list:
    """搜索 GitHub 仓库"""
    url = "https://api.github.com/search/repositories"
    query = keyword
    if language:
        query += f" language:{language}"

    params = {
        "q": query,
        "sort": "stars",
        "per_page": 10
    }

    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        data = response.json()
        return data.get("items", [])
    except requests.RequestException as e:
        print(f"搜索失败: {e}")
        return []


print("=== 搜索 Python 爬虫项目 ===")
repos = search_github("web scraper", "python")
for repo in repos:
    print(f"⭐{repo['stargazers_count']:>6}  {repo['full_name']}")
    print(f"    {repo.get('description', '无描述')}")
```

---

## 四、实战二：天气 API —— 多功能天气助手

使用 **Open-Meteo**（完全免费、无需注册的天气 API）构建一个功能丰富的天气助手。

### 4.1 Open-Meteo API 简介

Open-Meteo 是一个**完全免费、开源**的天气 API：
- 无需注册，无需 API Key
- 提供全球天气数据
- 支持当前天气、预报、历史数据

API 文档：https://open-meteo.com/en/docs

### 4.2 先获取城市坐标

Open-Meteo 需要经纬度坐标，我们可以用它的地理编码 API 先查坐标：

```python
import requests


def geocode(city: str) -> dict | None:
    """根据城市名获取经纬度"""
    url = "https://geocoding-api.open-meteo.com/v1/search"
    params = {"name": city, "count": 1, "language": "zh"}

    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        data = response.json()

        if "results" in data and len(data["results"]) > 0:
            result = data["results"][0]
            return {
                "name": result["name"],
                "country": result.get("country", ""),
                "latitude": result["latitude"],
                "longitude": result["longitude"],
            }
        else:
            return None

    except requests.RequestException:
        return None


# 测试
info = geocode("上海")
if info:
    print(f"{info['name']}, {info['country']}")
    print(f"坐标: ({info['latitude']}, {info['longitude']})")
```

### 4.3 获取天气预报

```python
import requests


def get_weather_forecast(city: str, days: int = 7) -> dict:
    """获取城市天气预报"""
    # 第一步：获取坐标
    geo_url = "https://geocoding-api.open-meteo.com/v1/search"
    geo_resp = requests.get(geo_url, params={"name": city, "count": 1}, timeout=10)
    geo_data = geo_resp.json()

    if "results" not in geo_data or len(geo_data["results"]) == 0:
        return {"error": f"找不到城市: {city}"}

    loc = geo_data["results"][0]
    lat, lon = loc["latitude"], loc["longitude"]
    city_name = loc["name"]

    # 第二步：获取天气预报
    weather_url = "https://api.open-meteo.com/v1/forecast"
    params = {
        "latitude": lat,
        "longitude": lon,
        "daily": "temperature_2m_max,temperature_2m_min,"
                  "weathercode,precipitation_sum,windspeed_10m_max",
        "timezone": "auto",
        "forecast_days": days,
    }

    try:
        resp = requests.get(weather_url, params=params, timeout=10)
        resp.raise_for_status()
        weather = resp.json()

        return {
            "city": city_name,
            "country": loc.get("country", ""),
            "forecast": weather["daily"],
        }

    except requests.RequestException as e:
        return {"error": str(e)}


# 天气代码对应中文描述
WEATHER_CODES = {
    0: "☀️ 晴天", 1: "🌤️ 大部晴朗", 2: "⛅ 多云", 3: "☁️ 阴天",
    45: "🌫️ 雾", 48: "🌫️ 雾凇",
    51: "🌦️ 小雨", 53: "🌦️ 中雨", 55: "🌧️ 大雨",
    61: "🌧️ 小雨", 63: "🌧️ 中雨", 65: "🌧️ 大雨",
    71: "🌨️ 小雪", 73: "🌨️ 中雪", 75: "❄️ 大雪",
    80: "🌦️ 阵雨", 81: "🌧️ 阵雨", 82: "🌧️ 暴雨",
    95: "⛈️ 雷暴", 96: "⛈️ 雷暴冰雹",
}


def display_forecast(data: dict) -> None:
    """格式化显示天气预报"""
    if "error" in data:
        print(f"错误: {data['error']}")
        return

    forecast = data["forecast"]
    print(f"\n📍 {data['city']}, {data['country']} — 未来 {len(forecast['time'])} 天预报")
    print("=" * 58)

    for i in range(len(forecast["time"])):
        date = forecast["time"][i]
        temp_max = forecast["temperature_2m_max"][i]
        temp_min = forecast["temperature_2m_min"][i]
        weather_code = forecast["weathercode"][i]
        rain = forecast["precipitation_sum"][i]
        wind = forecast["windspeed_10m_max"][i]

        weather_desc = WEATHER_CODES.get(weather_code, "❓ 未知")
        rain_str = f"{rain:.1f}mm" if rain > 0 else "无降水"

        print(f"  {date}  {weather_desc}")
        print(f"    🌡️ {temp_min}°C ~ {temp_max}°C  |  💧 {rain_str}  |  💨 {wind:.0f}km/h")
        print()


# 测试
data = get_weather_forecast("北京", 5)
display_forecast(data)
```

---

## 五、分页处理 —— 数据太多了怎么办？

很多 API 返回的数据会很多，不可能一次全给你，所以采用了**分页**机制。

### 5.1 分页原理

```
第1页: GET /api/users?page=1&per_page=10    → 返回第 1~10 条
第2页: GET /api/users?page=2&per_page=10    → 返回第 11~20 条
第3页: GET /api/users?page=3&per_page=10    → 返回第 21~30 条
...
```

### 5.2 通用分页获取函数

```python
import requests
from typing import Any


def fetch_all_pages(
    base_url: str,
    params: dict | None = None,
    page_param: str = "page",
    per_page: int = 100,
    max_pages: int = 10,
    data_key: str = "items",
    headers: dict | None = None,
) -> list[dict]:
    """
    通用分页数据获取函数

    :param base_url: API 地址
    :param params: 基础查询参数
    :param page_param: 页码参数名（有的是 page，有的是 offset）
    :param per_page: 每页数量
    :param max_pages: 最大页数（防止无限循环）
    :param data_key: 响应中数据列表的键名
    :param headers: 请求头
    :return: 所有数据的列表
    """
    if params is None:
        params = {}

    all_data = []

    for page in range(1, max_pages + 1):
        page_params = {**params, page_param: page, "per_page": per_page}

        try:
            response = requests.get(base_url, params=page_params, headers=headers, timeout=10)
            response.raise_for_status()
            result = response.json()

            items = result.get(data_key, [])
            if not items:
                break  # 没有数据了，停止

            all_data.extend(items)
            print(f"  第 {page} 页：获取 {len(items)} 条（累计 {len(all_data)} 条）")

        except requests.RequestException as e:
            print(f"  第 {page} 页获取失败: {e}")
            break

    return all_data


# 示例：获取 GitHub 上 Python 相关的热门仓库（所有页）
print("=== 搜索 Python 热门项目 ===")
repos = fetch_all_pages(
    base_url="https://api.github.com/search/repositories",
    params={"q": "language:python", "sort": "stars"},
    page_param="page",
    per_page=30,
    max_pages=3,       # 示例只取 3 页
    data_key="items",
    headers={"Accept": "application/vnd.github.v3+json"}
)

print(f"\n共获取 {len(repos)} 个仓库")
for repo in repos[:10]:  # 只显示前 10 个
    print(f"  ⭐{repo['stargazers_count']:>7}  {repo['full_name']}")
```

---

## 六、限速处理 —— 别催，排队！

很多 API 有**请求频率限制**（Rate Limit），比如"每分钟最多 60 次请求"。超出了会被暂时拒绝（返回 429 状态码）。

### 6.1 GitHub API 的限速规则

```python
import requests


def check_rate_limit():
    """查看 GitHub API 剩余请求数"""
    url = "https://api.github.com/rate_limit"
    headers = {"Accept": "application/vnd.github.v3+json"}

    response = requests.get(url, headers=headers, timeout=10)
    data = response.json()

    core = data["resources"]["core"]
    print(f"请求限额: {core['limit']} 次/小时")
    print(f"剩余次数: {core['remaining']}")
    print(f"重置时间: {core['reset']}")

    # 计算距离重置还有多久
    import time
    reset_time = core["reset"]
    wait_seconds = reset_time - int(time.time())
    if wait_seconds > 0:
        minutes = wait_seconds // 60
        print(f"约 {minutes} 分钟后重置")

check_rate_limit()
```

### 6.2 自动限速的请求函数

```python
import requests
import time
from typing import Any


def safe_request(
    url: str,
    max_retries: int = 3,
    initial_delay: float = 1.0,
    **kwargs
) -> requests.Response | None:
    """
    带自动重试和限速处理的请求函数

    :param url: 请求地址
    :param max_retries: 最大重试次数
    :param initial_delay: 初始等待时间（秒），每次重试会翻倍
    :param kwargs: 传给 requests.get/post 的其他参数
    :return: Response 对象或 None
    """
    delay = initial_delay

    for attempt in range(1, max_retries + 1):
        try:
            response = requests.get(url, timeout=10, **kwargs)

            if response.status_code == 200:
                return response

            elif response.status_code == 429:
                # 被限速了，等待后重试
                # 优先从响应头获取等待时间
                retry_after = response.headers.get("Retry-After")
                if retry_after:
                    wait = int(retry_after)
                else:
                    wait = delay

                print(f"  ⚠️ 限速 (429)，等待 {wait} 秒后重试（第 {attempt}/{max_retries} 次）")
                time.sleep(wait)
                delay *= 2  # 指数退避

            elif response.status_code == 403:
                print(f"  ❌ 访问被拒绝 (403)，可能需要认证")
                return None

            elif response.status_code >= 500:
                print(f"  ⚠️ 服务器错误 ({response.status_code})，等待后重试")
                time.sleep(delay)
                delay *= 2

            else:
                print(f"  ❌ 请求失败，状态码: {response.status_code}")
                return None

        except requests.Timeout:
            print(f"  ⚠️ 请求超时，等待后重试（第 {attempt}/{max_retries} 次）")
            time.sleep(delay)
            delay *= 2

        except requests.ConnectionError:
            print(f"  ⚠️ 网络连接失败，等待后重试（第 {attempt}/{max_retries} 次）")
            time.sleep(delay)
            delay *= 2

        except requests.RequestException as e:
            print(f"  ❌ 请求异常: {e}")
            return None

    print(f"  ❌ 超过最大重试次数 ({max_retries})，放弃请求")
    return None


# 测试：批量请求多个 GitHub 用户信息
users = ["torvalds", "gaearon", "tj", "sindresorhus", "ruanyf"]
for username in users:
    resp = safe_request(
        f"https://api.github.com/users/{username}",
        headers={"Accept": "application/vnd.github.v3+json"}
    )
    if resp:
        data = resp.json()
        print(f"  ✓ {data['name'] or username} — {data.get('bio', '无简介')[:40]}")
```

---

## 七、综合实战：汇率查询工具

用免费汇率 API 构建一个实用的汇率查询和换算工具。

```python
import requests
from datetime import datetime


class CurrencyConverter:
    """汇率查询与换算工具"""

    BASE_URL = "https://api.exchangerate-api.com/v4/latest"

    def __init__(self):
        self.cache = {}       # 缓存汇率数据
        self.cache_time = {}  # 缓存时间

    def get_rates(self, base: str = "USD") -> dict | None:
        """
        获取以 base 为基准的所有汇率

        :param base: 基准货币代码，如 "USD", "CNY", "EUR"
        :return: 汇率字典或 None
        """
        # 检查缓存（1 小时内有效）
        now = datetime.now().timestamp()
        if base in self.cache and (now - self.cache_time[base]) < 3600:
            print(f"  [缓存] 使用 {base} 汇率缓存")
            return self.cache[base]

        url = f"{self.BASE_URL}/{base}"

        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            data = response.json()

            rates = data["rates"]
            self.cache[base] = rates
            self.cache_time[base] = now

            print(f"  [网络] 获取 {base} 汇率成功（共 {len(rates)} 种货币）")
            return rates

        except requests.RequestException as e:
            print(f"  获取汇率失败: {e}")
            return None

    def convert(self, amount: float, from_currency: str, to_currency: str) -> float | None:
        """
        货币换算

        :param amount: 金额
        :param from_currency: 源货币代码
        :param to_currency: 目标货币代码
        :return: 换算结果或 None
        """
        rates = self.get_rates(from_currency)
        if rates is None:
            return None

        if to_currency not in rates:
            print(f"  不支持的货币: {to_currency}")
            return None

        return round(amount * rates[to_currency], 4)

    def show_popular(self, base: str = "CNY") -> None:
        """显示热门货币汇率"""
        rates = self.get_rates(base)
        if rates is None:
            return

        # 热门货币列表
        popular = ["USD", "EUR", "GBP", "JPY", "KRW", "HKD", "AUD", "CAD", "SGD", "THB"]

        print(f"\n💱 以 {base} 为基准的热门汇率：")
        print("-" * 40)
        for currency in popular:
            if currency in rates:
                rate = rates[currency]
                # 根据大小选择合适的精度
                if rate >= 100:
                    formatted = f"{rate:.2f}"
                elif rate >= 1:
                    formatted = f"{rate:.4f}"
                else:
                    formatted = f"{rate:.6f}"
                print(f"  {currency:>4} → {base}: {formatted}")

    def interactive(self) -> None:
        """交互式汇率查询"""
        print("=" * 50)
        print("💱 汇率查询与换算工具（输入 q 退出）")
        print("=" * 50)

        # 先展示热门汇率
        self.show_popular("CNY")

        while True:
            print("\n" + "-" * 50)
            print("  1. 货币换算")
            print("  2. 查看热门汇率（选基准货币）")
            print("  q. 退出")

            choice = input("\n请选择: ").strip().lower()

            if choice == "q":
                print("再见！")
                break

            elif choice == "1":
                try:
                    amount = float(input("金额: "))
                    from_cur = input("源货币代码 (如 CNY): ").strip().upper()
                    to_cur = input("目标货币代码 (如 USD): ").strip().upper()

                    result = self.convert(amount, from_cur, to_cur)
                    if result is not None:
                        print(f"\n  💰 {amount} {from_cur} = {result} {to_cur}")
                    else:
                        print("  换算失败，请检查货币代码")
                except ValueError:
                    print("  金额输入有误")

            elif choice == "2":
                base = input("基准货币代码 (如 USD): ").strip().upper()
                self.show_popular(base)
                self.show_popular(base)

            else:
                print("  无效选择")


if __name__ == "__main__":
    converter = CurrencyConverter()

    # 快速演示
    print("=== 快速演示 ===")
    print(f"100 CNY = {converter.convert(100, 'CNY', 'USD')} USD")
    print(f"100 CNY = {converter.convert(100, 'CNY', 'EUR')} EUR")
    print(f"100 CNY = {converter.convert(100, 'CNY', 'JPY')} JPY")
    print(f"  1 USD = {converter.convert(1, 'USD', 'CNY')} CNY")

    # 交互模式（取消下面注释即可启动）
    # converter.interactive()
```

---

## 八、API 调用最佳实践

### 8.1 DO（应该做的）

| 实践 | 说明 |
|-----|------|
| **始终设置 timeout** | 防止程序卡死 |
| **使用 Session** | 复用连接，提高性能 |
| **添加 User-Agent** | 避免被拒绝 |
| **做好异常处理** | 网络不稳定是常态 |
| **缓存结果** | 减少重复请求 |
| **尊重限速** | 避免被封禁 |
| **阅读文档** | 不要靠猜 |

### 8.2 DONT（不要做的）

| 反模式 | 原因 |
|-------|------|
| 硬编码 API Key | 安全隐患 |
| 不设 timeout | 程序可能永远卡住 |
| 忽略状态码 | 请求失败了还当成功处理 |
| 疯狂请求 | 被封禁 / 被起诉 |
| 不处理异常 | 一个网络波动就崩溃 |

---

## 九、练习题

### 练习 1：随机名言获取器

利用免费的 quotable API 获取随机名言：

```python
import requests


def get_random_quote():
    """获取一条随机名言"""
    url = "https://api.quotable.io/random"

    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        data = response.json()

        print(f"\n📖 "{data['content']}"")
        print(f"   —— {data['author']}\n")
        return data

    except requests.RequestException as e:
        print(f"获取失败: {e}")
        return None


# 获取 5 条随机名言
print("=== 今日名言推荐 ===")
for i in range(5):
    get_random_quote()
```

### 练习 2：国家信息查询

使用免费的 REST Countries API 查询国家信息：

```python
import requests


def get_country_info(name: str):
    """根据国家名查询详细信息"""
    url = "https://restcountries.com/v3.1/name/{name}"

    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        data = response.json()

        if not data:
            print(f"找不到国家: {name}")
            return

        country = data[0]
        print(f"\n📍 {country['name']['common']}")
        print(f"   首都: {country.get('capital', ['未知'])[0]}")
        print(f"   人口: {country['population']:,}")
        print(f"   面积: {country['area']:,.0f} km²")
        print(f"   语言: {', '.join(country['languages'].values())}")
        print(f"   货币: ", end="")
        for code, info in country.get("currencies", {}).items():
            print(f"{info['name']} ({code}) ", end="")
        print()

    except requests.RequestException as e:
        print(f"查询失败: {e}")


# 测试
get_country_info("China")
get_country_info("Japan")
get_country_info("France")
```

### 练习 3（挑战）：构建"每日一报"工具

综合使用多个免费 API，生成一份"每日信息简报"：

```python
import requests
from datetime import datetime


def daily_briefing():
    """生成每日信息简报"""
    print("=" * 50)
    print(f"  📋 每日信息简报 — {datetime.now().strftime('%Y-%m-%d')}")
    print("=" * 50)

    # 1. 随机名言
    print("\n📖 每日名言")
    try:
        resp = requests.get("https://api.quotable.io/random", timeout=10)
        if resp.ok:
            quote = resp.json()
            print(f'   "{quote["content"]}"')
            print(f'   —— {quote["author"]}')
    except Exception:
        print("   获取失败")

    # 2. 汇率快报
    print("\n💱 主要汇率（基于 USD）")
    try:
        resp = requests.get("https://api.exchangerate-api.com/v4/latest/USD", timeout=10)
        if resp.ok:
            rates = resp.json()["rates"]
            for pair in [("CNY", "人民币"), ("EUR", "欧元"), ("JPY", "日元")]:
                code, name = pair
                print(f"   1 USD = {rates[code]:.4f} {code}（{name}）")
    except Exception:
        print("   获取失败")

    # 3. IP 地理位置（你的公网 IP）
    print("\n🌐 你的公网 IP 信息")
    try:
        resp = requests.get("https://ipapi.co/json/", timeout=10)
        if resp.ok:
            info = resp.json()
            print(f"   IP: {info['ip']}")
            print(f"   位置: {info['city']}, {info['region']}, {info['country_name']}")
            print(f"   运营商: {info.get('org', '未知')}")
    except Exception:
        print("   获取失败")

    print("\n" + "=" * 50)


daily_briefing()
```

---

## 十、常见问题

### Q1：API 返回 401 Unauthorized 怎么办？

说明认证失败。检查：
1. API Key 是否正确
2. 是否放在了正确的请求头中
3. 认证方式是否符合文档要求（Key vs Token vs Basic Auth）

### Q2：API 返回 429 Too Many Requests？

说明你请求太频繁了。解决方案：
1. 降低请求频率
2. 使用上面教的 `safe_request` 函数自动重试
3. 如果有多个 API 可用，可以轮换使用

### Q3：怎么找到免费可用的 API？

推荐以下免费 API 聚合网站：
- [Public APIs (github.com)](https://github.com/public-apis/public-apis) —— 超全的免费 API 列表
- [RapidAPI](https://rapidapi.com/) —— 部分免费额度
- [GitHub API](https://docs.github.com/en/rest) —— 无需注册即可获取公开数据

### Q4：JSON 响应解析失败怎么办？

```python
# 先检查 Content-Type 是否为 JSON
if "application/json" in response.headers.get("Content-Type", ""):
    data = response.json()
else:
    # 可能返回了 HTML（比如错误页面）
    print("响应不是 JSON 格式")
    print("响应内容:", response.text[:200])
```

### Q5：API 返回的数据结构和文档不一致？

这种情况很常见，原因可能是：
1. 文档过时了
2. API 版本更新了
3. 参数不同导致返回结构不同

**建议**：先用 `response.json()` 打印完整响应，看清楚实际结构再解析。

---

## 十一、免费学习资源

- [RESTful API 设计指南](https://restfulapi.net/) —— 了解 RESTful 设计原则
- [GitHub REST API 文档](https://docs.github.com/en/rest) —— 最推荐的练手 API
- [Open-Meteo 天气 API](https://open-meteo.com/en/docs) —— 免费、无需注册
- [public-apis/public-apis](https://github.com/public-apis/public-apis) —— 免费大全
- [廖雪峰 - Python 网络编程](https://www.liaoxuefeng.com/wiki/1016959663602400/1183249758853888) —— requests 入门

---

## 今日小结

| 知识点 | 掌握程度 |
|-------|---------|
| REST API 概念与端点 | ⭐⭐⭐⭐ |
| API 认证方式（Key/Token/Basic） | ⭐⭐⭐⭐⭐ |
| GitHub API 实战调用 | ⭐⭐⭐⭐⭐ |
| Open-Meteo 天气 API | ⭐⭐⭐⭐ |
| 分页数据获取 | ⭐⭐⭐⭐ |
| 限速处理与自动重试 | ⭐⭐⭐⭐⭐ |
| 缓存策略 | ⭐⭐⭐ |
| 综合项目：汇率工具 | ⭐⭐⭐⭐ |

> **下一站预告**：Day 38 —— pandas 入门。告别手动处理数据，学会用 Python 最强大的数据分析库，让数据"自己说话"。
