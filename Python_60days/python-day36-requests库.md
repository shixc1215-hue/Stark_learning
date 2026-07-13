# Python Day 36：requests 库 —— Python 网络请求利器

> 互联网时代，数据无处不在。天气信息、股票行情、新闻头条……它们都藏在某个网址后面。requests 库就是帮你"敲开这些大门"的万能钥匙。

---

## 一、什么是网络请求？

你在浏览器里输入一个网址、按回车，浏览器就帮你做了一次**网络请求**——向服务器要数据，服务器把网页内容返回来。Python 也可以做同样的事情。

打个比方：

| 现实生活 | Python 网络请求 |
|---------|----------------|
| 你打电话给餐厅问菜单 | Python 发请求给服务器要数据 |
| 餐厅告诉你今天的菜品 | 服务器把数据返回给 Python |
| 你听电话得到信息 | Python 解析返回的数据 |

Python 内置了 `urllib` 模块可以做网络请求，但它的接口比较麻烦。`requests` 是第三方库，**更简单、更人性**，是 Python 界做网络请求的事实标准。

---

## 二、安装 requests

```bash
pip install requests
```

安装完成后，在 Python 中导入：

```python
import requests
```

如果导入成功没有报错，说明安装 OK。

---

## 三、GET 请求 —— "我要看看"

GET 是最常见的请求方式，就像你**打开一个网页**，只看不改。

### 3.1 最简单的 GET 请求

```python
import requests

# 向一个测试网站发送 GET 请求
response = requests.get("https://httpbin.org/get")

# 查看响应状态码（200 = 成功）
print("状态码:", response.status_code)       # 状态码: 200

# 查看响应内容（文本格式）
print("响应文本:", response.text)

# 查看响应内容（字节格式）
print("响应字节:", response.content[:100])    # 只打印前100字节
```

### 3.2 状态码速查

| 状态码 | 含义 | 说明 |
|-------|------|------|
| **200** | OK | 请求成功 |
| **301/302** | 重定向 | 网址已经搬家了 |
| **404** | Not Found | 你要的东西不存在 |
| **500** | 服务器错误 | 服务器自己崩了 |

`requests` 还提供了更直观的判断方式：

```python
response = requests.get("https://httpbin.org/get")

# 这些都是布尔值，可以直接用 if 判断
print(response.ok)              # True（状态码 < 400）
print(response.status_code == 200)  # True

if response.ok:
    print("请求成功！")
else:
    print(f"请求失败，状态码：{response.status_code}")
```

### 3.3 带参数的 GET 请求

很多时候我们需要传参数，比如搜索关键词。`params` 参数帮你自动拼接到 URL 后面：

```python
import requests

# 方式一：手动拼接 URL（不推荐，容易出错）
url = "https://httpbin.org/get?name=张三&age=25"
response = requests.get(url)

# 方式二：用 params 字典（推荐，自动处理编码）
params = {
    "name": "张三",
    "age": 25,
    "city": "北京"
}
response = requests.get("https://httpbin.org/get", params=params)

# requests 会自动把中文编码，生成正确 URL
print("实际请求的 URL:", response.url)
# 实际请求的 URL: https://httpbin.org/get?name=%E5%BC%A0%E4%B8%89&age=25&city=%E5%8C%97%E4%BA%AC
```

### 3.4 添加请求头（Headers）

有些网站会检查"你是什么浏览器"。默认情况下，requests 发出的请求头标识自己是 Python 程序，某些网站可能会拒绝。我们可以伪装成浏览器：

```python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
}

response = requests.get("https://httpbin.org/headers", headers=headers)
print(response.text)
```

常用的请求头：

| 请求头 | 作用 |
|-------|------|
| `User-Agent` | 标识客户端（浏览器类型/版本） |
| `Accept` | 告诉服务器你想要什么格式的数据 |
| `Content-Type` | 发送数据的格式（POST 时用） |
| `Authorization` | 认证令牌（API Key 等） |
| `Referer` | 告诉服务器你从哪个页面来的 |

### 3.5 获取 JSON 数据

现代 API 大多返回 JSON 格式的数据，requests 提供了 `response.json()` 方法，直接把 JSON 转成 Python 字典：

```python
import requests

response = requests.get("https://httpbin.org/json")

# response.json() 直接把 JSON 响应转成 Python 字典
data = response.json()

print(type(data))              # <class 'dict'>
print("幻灯片标题:", data["slideshow"]["title"])
```

再举个例子，获取一个免费公开 API 的数据：

```python
import requests

# 免费的天气 API（无需注册）
response = requests.get("https://api.github.com")

if response.ok:
    data = response.json()
    print(f"GitHub 当前状态: {data.get('message', '未知')}")
else:
    print(f"请求失败: {response.status_code}")
```

---

## 四、POST 请求 —— "我要提交"

POST 请求用于**向服务器提交数据**，比如登录、发表评论、上传文件。

### 4.1 发送表单数据

```python
import requests

# 模拟登录请求
login_data = {
    "username": "myuser",
    "password": "mypassword"
}

response = requests.post("https://httpbin.org/post", data=login_data)

print("状态码:", response.status_code)
print(response.text)
```

### 4.2 发送 JSON 数据

现代 API 通常接收 JSON 格式：

```python
import requests

import json

# 方式一：手动转 JSON 字符串
payload = {"name": "张三", "age": 25}
response = requests.post(
    "https://httpbin.org/post",
    data=json.dumps(payload),
    headers={"Content-Type": "application/json"}
)

# 方式二：用 json 参数（推荐，requests 自动帮你转换）
response = requests.post(
    "https://httpbin.org/post",
    json={"name": "张三", "age": 25}
)

# 查看服务器收到的是什么
data = response.json()
print("服务器收到的数据:", data.get("json"))
# 服务器收到的数据: {'name': '张三', 'age': 25}
```

`data` vs `json` 参数的区别：

| 参数 | Content-Type | 数据格式 | 适用场景 |
|-----|-------------|---------|---------|
| `data=字典` | `application/x-www-form-urlencoded` | 表单 | 传统网页表单 |
| `json=字典` | `application/json` | JSON | REST API |

---

## 五、响应对象详解

`requests.get()` / `requests.post()` 返回的都是 `Response` 对象，它包含了服务器返回的所有信息：

```python
import requests

response = requests.get("https://httpbin.org/get")

# === 状态相关 ===
print(response.status_code)     # 200，状态码
print(response.ok)              # True，是否成功（< 400）
print(response.reason)          # "OK"，状态描述

# === 内容相关 ===
print(response.text)            # str，响应体的文本形式（自动解码）
print(response.content)         # bytes，响应体的原始字节
print(response.json())          # dict，把 JSON 文本转成字典

# === 编码相关 ===
print(response.encoding)        # 'utf-8'，requests 自动检测的编码
# 如果自动检测不对，可以手动设置
response.encoding = "gbk"
print(response.text)            # 用新编码解码

# === 头部信息 ===
print(response.headers)         # dict，响应头
print(response.headers["Content-Type"])  # 'application/json'
print(response.headers.get("Server"))    # 'gunicorn/19.9.0'

# === URL 相关 ===
print(response.url)              # 实际请求的 URL（可能被重定向过）
print(response.cookies)          # 响应中的 cookies
```

---

## 六、超时设置 —— 不要傻等

网络请求可能会"卡住"——服务器半天不回复。设置 `timeout` 可以避免程序无限等待：

```python
import requests
from requests.exceptions import Timeout

try:
    # timeout=5 表示：连接超时 5 秒
    response = requests.get("https://httpbin.org/delay/3", timeout=5)
    print("请求成功")
except Timeout:
    print("请求超时了！服务器没有及时响应")
except Exception as e:
    print(f"其他错误: {e}")
```

`timeout` 还可以设置元组，分别控制**连接超时**和**读取超时**：

```python
# 连接超时 5 秒 + 读取超时 30 秒
response = requests.get("https://httpbin.org/get", timeout=(5, 30))
```

---

## 七、异常处理

网络请求可能遇到各种问题：断网、DNS 解析失败、连接被拒绝……`requests` 定义了一系列异常：

```python
import requests
from requests.exceptions import (
    RequestException,    # 所有 requests 异常的基类
    Timeout,            # 请求超时
    ConnectionError,    # 网络连接问题（DNS失败/拒绝连接等）
    HTTPError,           # HTTP 状态码不是 2xx
    TooManyRedirects,   # 重定向次数过多
)

url = "https://httpbin.org/get"

try:
    response = requests.get(url, timeout=10)
    # 如果状态码不是 2xx，主动抛出 HTTPError
    response.raise_for_status()
    print("请求成功:", response.status_code)

except Timeout:
    print("请求超时")
except ConnectionError:
    print("网络连接失败，请检查网络")
except HTTPError as e:
    print(f"HTTP 错误: {e}")
except RequestException as e:
    print(f"请求异常: {e}")
```

异常继承关系：

```
RequestException                    ← 所有异常的基类
├── Timeout                         ← 超时
├── ConnectionError                 ← 连接错误
│   ├── ProxyError                  ← 代理错误
│   └── SSLError                    ← SSL 错误
├── HTTPError                       ← HTTP 状态码错误
├── TooManyRedirects               ← 太多重定向
├── URLRequired                    ← 缺少 URL
└── MissingSchema                  ← URL 格式错误
```

---

## 八、Session 会话 —— 保持登录状态

如果你需要连续访问同一个网站的多个页面（比如先登录再查看个人信息），用 `Session` 可以自动保持 cookies 和一些设置：

```python
import requests

# 创建一个 Session 对象
session = requests.Session()

# 第一次请求：登录（服务器会返回 cookie）
login_resp = session.post("https://httpbin.org/post", json={"user": "张三"})

# 后续请求：Session 自动带上之前的 cookie
info_resp = session.get("https://httpbin.org/cookies")

print(info_resp.json())

# 用完记得关闭（也可以用 with 语句）
session.close()
```

更优雅的写法——用 `with` 自动关闭：

```python
import requests

with requests.Session() as session:
    # 设置通用请求头（所有请求都会带上）
    session.headers.update({
        "User-Agent": "Mozilla/5.0",
        "Accept-Language": "zh-CN"
    })

    # 每次请求自动带 headers 和 cookies
    response = session.get("https://httpbin.org/get")
    print(response.json())
```

---

## 九、综合实战：天气查询工具

把今天学的知识串起来，做一个简单的天气查询工具。这里使用免费的公开 API：

```python
import requests
from requests.exceptions import RequestException


def get_weather(city: str) -> dict:
    """根据城市名查询天气信息"""
    url = "https://wttr.in/{city}"

    params = {
        "format": "j1",      # 返回 JSON 格式
        "lang": "zh"         # 中文
    }

    headers = {
        "User-Agent": "curl/7.68.0"   # wttr.in 需要这个 UA
    }

    try:
        response = requests.get(url, params=params, headers=headers, timeout=10)
        response.raise_for_status()
        return response.json()

    except Timeout:
        return {"error": "请求超时，请稍后重试"}
    except ConnectionError:
        return {"error": "网络连接失败"}
    except RequestException as e:
        return {"error": f"请求失败: {e}"}


def display_weather(data: dict) -> None:
    """格式化显示天气信息"""
    if "error" in data:
        print(f"查询失败: {data['error']}")
        return

    try:
        # 提取当前天气
        current = data["current_condition"][0]
        area = data["nearest_area"][0]

        print("=" * 40)
        print(f"地点: {area['areaName'][0]['value']}, {area['country'][0]['value']}")
        print(f"当前温度: {current['temp_C']}°C")
        print(f"体感温度: {current['FeelsLikeC']}°C")
        print(f"天气状况: {current['lang_zh'][0]['value']}")
        print(f"湿度: {current['humidity']}%")
        print(f"风速: {current['windspeedKmph']} km/h")
        print(f"能见度: {current['visibility']} km")
        print("=" * 40)

    except (KeyError, IndexError) as e:
        print(f"解析天气数据失败: {e}")


def main():
    """主函数：交互式天气查询"""
    print("🌤️  简易天气查询工具（输入 q 退出）")
    print("-" * 40)

    while True:
        city = input("\n请输入城市名: ").strip()
        if city.lower() == "q":
            print("再见！")
            break

        if not city:
            print("城市名不能为空")
            continue

        print(f"正在查询 {city} 的天气...")
        weather_data = get_weather(city)
        display_weather(weather_data)


if __name__ == "__main__":
    main()
```

运行效果：

```
🌤️  简易天气查询工具（输入 q 退出）
----------------------------------------

请输入城市名: Beijing
正在查询 Beijing 的天气...
========================================
地点: Beijing, China
当前温度: 32°C
体感温度: 35°C
天气状况: 多云
湿度: 55%
风速: 15 km/h
能见度: 10 km
========================================
```

---

## 十、练习题

### 练习 1：获取 GitHub 用户信息

编写一个函数，输入 GitHub 用户名，输出该用户的公开信息：

```python
import requests

def get_github_user(username):
    """获取 GitHub 用户信息"""
    url = f"https://api.github.com/users/{username}"
    response = requests.get(url)

    if response.ok:
        data = response.json()
        print(f"用户名: {data['name']}")
        print(f"简介: {data['bio']}")
        print(f"公开仓库数: {data['public_repos']}")
        print(f"粉丝数: {data['followers']}")
        print(f"个人网站: {data.get('blog', '无')}")
    else:
        print(f"查询失败，状态码: {response.status_code}")

# 测试
get_github_user("python")
```

### 练习 2：翻译工具

利用免费的翻译 API 实现一个简单的翻译函数（提示：可以用 `https://api.mymemory.translated.net/get` 这个免费 API）：

```python
import requests

def translate(text, lang_pair="zh-CN|en"):
    """
    翻译文本
    :param text: 要翻译的文本
    :param lang_pair: 语言对，如 "zh-CN|en" 表示中译英
    :return: 翻译结果
    """
    url = "https://api.mymemory.translated.net/get"
    params = {
        "q": text,
        "langpair": lang_pair
    }

    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        data = response.json()
        return data["responseData"]["translatedText"]
    except RequestException as e:
        return f"翻译失败: {e}"

# 测试
print(translate("你好，世界"))
print(translate("Hello, World", "en|zh-CN"))
```

### 练习 3：批量请求（扩展）

使用 `Session` 和循环，批量查询多个城市的天气，并把结果保存到一个列表中：

```python
import requests

def batch_weather(cities):
    """批量查询多个城市的天气"""
    results = []
    with requests.Session() as session:
        session.headers.update({"User-Agent": "curl/7.68.0"})
        for city in cities:
            try:
                resp = session.get(
                    f"https://wttr.in/{city}",
                    params={"format": "j1", "lang": "zh"},
                    timeout=10
                )
                resp.raise_for_status()
                data = resp.json()
                temp = data["current_condition"][0]["temp_C"]
                results.append({"city": city, "temp": temp, "status": "ok"})
            except Exception as e:
                results.append({"city": city, "temp": None, "status": str(e)})
    return results

# 测试
cities = ["Beijing", "Shanghai", "Guangzhou", "Tokyo", "London"]
for r in batch_weather(cities):
    if r["status"] == "ok":
        print(f"{r['city']}: {r['temp']}°C")
    else:
        print(f"{r['city']}: 查询失败 - {r['status']}")
```

---

## 十一、常见问题

### Q1：安装 requests 时报错怎么办？

```
ERROR: Could not find a version that satisfies the requirement requests
```

**原因**：pip 版本太旧或网络问题。

**解决**：
```bash
# 升级 pip
python -m pip install --upgrade pip

# 如果网络不好，使用国内镜像
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### Q2：中文响应出现乱码？

`requests` 自动检测编码，有时会检测错。手动设置：

```python
response = requests.get(url)
# 查看自动检测的编码
print(response.encoding)   # 可能是 ISO-8859-1（错误的）

# 手动改为正确的编码
response.encoding = "utf-8"
print(response.text)        # 中文正常显示
```

### Q3：请求被服务器拒绝（403 Forbidden）？

很多网站会检查 `User-Agent`。添加浏览器 UA 通常能解决：

```python
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
}
response = requests.get(url, headers=headers)
```

### Q4：requests 和 urllib 有什么区别？

| 特性 | `requests` | `urllib` |
|-----|-----------|---------|
| 来源 | 第三方库（需安装） | Python 内置 |
| 易用性 | 简单直观 | 比较繁琐 |
| JSON 处理 | `response.json()` 直接转换 | 需要手动 `json.loads()` |
| 编码处理 | 自动检测 | 手动处理 |
| 超时设置 | 简单 | 较复杂 |
| 推荐场景 | 日常使用、API 调用 | 不能安装第三方库时 |

**结论**：能用 requests 就用 requests，urllib 只在受限环境（无法安装第三方库）时使用。

### Q5：如何调试请求？

打印请求和响应的详细信息：

```python
import requests

response = requests.get("https://httpbin.org/get", params={"key": "value"})

# 打印实际请求的 URL
print("请求 URL:", response.request.url)
print("请求方法:", response.request.method)
print("请求头:", dict(response.request.headers))

# 打印响应信息
print("状态码:", response.status_code)
print("响应头:", dict(response.headers))
print("耗时:", response.elapsed.total_seconds(), "秒")
```

---

## 十二、免费学习资源

- [requests 官方文档（中文版）](https://requests.readthedocs.io/zh-cn/latest/) —— 最权威的参考
- [廖雪峰 - requests 入门](https://www.liaoxuefeng.com/wiki/1016959663602400/1183249758853888) —— 简洁易懂
- [菜鸟教程 - HTTP 请求](https://www.runoob.com/http/http-tutorial.html) —— 了解 HTTP 协议基础
- [HTTPBIN](https://httpbin.org/) —— 免费的 HTTP 请求测试服务

---

## 今日小结

| 知识点 | 掌握程度 |
|-------|---------|
| GET/POST 请求基本用法 | ⭐⭐⭐⭐⭐ |
| params / json 参数传递 | ⭐⭐⭐⭐⭐ |
| 请求头与 User-Agent | ⭐⭐⭐⭐ |
| 响应对象（text/json/headers） | ⭐⭐⭐⭐⭐ |
| 超时设置与异常处理 | ⭐⭐⭐⭐ |
| Session 会话管理 | ⭐⭐⭐ |

> **下一站预告**：Day 37 —— API 调用实战。我们将调用真实的公开 API（天气、新闻、汇率等），构建更完整的应用，学习 API 认证、分页查询等高级技巧。
