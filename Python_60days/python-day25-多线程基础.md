# Python Day 25：多线程基础

> **学习目标**：理解线程与进程的区别，掌握 Python `threading` 模块的基本用法，能够编写简单的多线程程序来提高 I/O 密集型任务的效率。

---

## 一、为什么需要多线程？

想象一下这个场景：你要下载 10 张图片，每张需要 5 秒。如果**一张一张下载**，总共需要 50 秒。但如果能**同时下载**，也许只需要 5 秒！

这就是多线程的核心思想——**让程序同时做多件事**。

### 现实类比

| 场景 | 单线程 | 多线程 |
|------|--------|--------|
| 做饭 | 一个人洗菜→切菜→炒菜→洗碗 | 一个人炒菜的同时，另一个人洗碗 |
| 写作业 | 先做完数学，再做英语 | 做完数学题等老师批改时，先写英语 |
| 下载文件 | 下载完 A 再下载 B | 同时下载 A 和 B |

### 线程 vs 进程

这是初学者最容易混淆的概念，我们用一张图来理解：

```
进程（Process）= 一个运行中的程序
├── 线程1（Thread 1）= 程序中的一条执行路径
├── 线程2（Thread 2）= 另一条执行路径
└── 线程3（Thread 3）= 又一条执行路径
```

| 比较项 | 进程 | 线程 |
|--------|------|------|
| 定义 | 操作系统分配资源的最小单位 | CPU 调度的最小单位 |
| 内存 | 每个进程有独立的内存空间 | 同一进程内的线程**共享内存** |
| 创建开销 | 大（需要分配独立内存） | 小（共享进程内存） |
| 通信 | 需要特殊机制（管道、队列等） | 可以直接访问共享变量 |
| 安全性 | 互相独立，一个崩溃不影响其他 | 一个线程出错可能导致整个进程崩溃 |

> **简单记忆**：进程像**独立的房子**，线程像房子里的**房间**。房子之间不能随便串门，但房间之间共享大门和走廊。

---

## 二、Python 的 threading 模块

Python 标准库提供了 `threading` 模块来处理多线程编程。下面我们从最简单的例子开始。

### 2.1 创建线程的两种方式

**方式一：直接创建 Thread 对象**

```python
import threading
import time

def say_hello(name):
    """打招呼函数，模拟耗时操作"""
    for i in range(3):
        print(f"[{name}] 第 {i+1} 次打招呼")
        time.sleep(1)  # 模拟耗时 1 秒

# 创建两个线程
t1 = threading.Thread(target=say_hello, args=("线程A",))
t2 = threading.Thread(target=say_hello, args=("线程B",))

# 启动线程
t1.start()
t2.start()

# 等待所有线程完成
t1.join()
t2.join()

print("所有线程执行完毕！")
```

**运行结果**（每次可能不同）：
```
[线程A] 第 1 次打招呼
[线程B] 第 1 次打招呼
[线程A] 第 2 次打招呼
[线程B] 第 2 次打招呼
[线程A] 第 3 次打招呼
[线程B] 第 3 次打招呼
所有线程执行完毕！
```

> **注意**：两个线程的输出是**交替**出现的，说明它们在**同时运行**。如果是单线程，要先执行完线程 A 的 3 次循环，再执行线程 B 的。

**方式二：继承 Thread 类（面向对象风格）**

```python
import threading
import time

class MyThread(threading.Thread):
    """自定义线程类"""

    def __init__(self, name):
        super().__init__()      # 调用父类的初始化方法
        self.name = name        # 保存线程名称

    def run(self):
        """线程启动后自动执行的方法"""
        for i in range(3):
            print(f"[{self.name}] 正在工作... 第 {i+1} 次")
            time.sleep(0.5)

# 创建并启动线程
t1 = MyThread("下载线程")
t2 = MyThread("上传线程")

t1.start()
t2.start()

t1.join()
t2.join()

print("全部任务完成！")
```

### 2.2 Thread 对象的常用方法与属性

| 方法/属性 | 说明 | 示例 |
|-----------|------|------|
| `start()` | 启动线程 | `t.start()` |
| `join(timeout=None)` | 等待线程结束 | `t.join()` |
| `is_alive()` | 判断线程是否在运行 | `t.is_alive()` |
| `getName()` / `name` | 获取线程名 | `t.name` |
| `daemon` | 设置为守护线程 | `t.daemon = True` |

```python
import threading
import time

def worker():
    print(f"线程 {threading.current_thread().name} 开始工作")
    time.sleep(2)
    print(f"线程 {threading.current_thread().name} 工作结束")

t = threading.Thread(target=worker, name="Worker-1")
print(f"启动前: is_alive = {t.is_alive()}")

t.start()
print(f"启动后: is_alive = {t.is_alive()}")

t.join()  # 等待线程结束
print(f"join后: is_alive = {t.is_alive()}")
```

---

## 三、守护线程（Daemon Thread）

守护线程是一种**后台线程**，它的特点是：当所有**非守护线程**结束后，守护线程会**自动被终止**，即使它还没执行完。

```python
import threading
import time

def daemon_thread():
    """守护线程：后台默默运行"""
    for i in range(5):
        print(f"守护线程: 第 {i+1} 次")
        time.sleep(1)

def normal_thread():
    """普通线程：主线程会等它结束"""
    for i in range(3):
        print(f"普通线程: 第 {i+1} 次")
        time.sleep(1)

d = threading.Thread(target=daemon_thread, name="守护线程")
d.daemon = True  # 设置为守护线程

n = threading.Thread(target=normal_thread, name="普通线程")

d.start()
n.start()

n.join()  # 只等待普通线程
print("主线程结束，守护线程自动终止！")
```

**输出**：
```
守护线程: 第 1 次
普通线程: 第 1 次
守护线程: 第 2 次
普通线程: 第 2 次
守护线程: 第 3 次
普通线程: 第 3 次
主线程结束，守护线程自动终止！
```

> 守护线程只打印了 3 次就被强制结束了（本来应该打印 5 次），因为普通线程已经执行完毕。

**守护线程的典型用途**：日志记录、垃圾回收、心跳检测等后台任务。

---

## 四、线程安全与 GIL

### 4.1 什么是 GIL？

Python 有一个著名的设计叫 **GIL（全局解释器锁，Global Interpreter Lock）**。简单来说：

- 同一时刻，**只有一个线程**能在 Python 解释器中执行字节码
- 这意味着 Python 的多线程**不能真正利用多核 CPU 进行并行计算**

> **类比**：GIL 像一把钥匙，多个线程排队轮流使用它。虽然看起来同时在运行，但实际上在任意瞬间只有一个人拿着钥匙在工作。

### 4.2 那多线程还有用吗？

**当然有用！** 关键要看任务的类型：

| 任务类型 | 特点 | 多线程效果 | 推荐方案 |
|----------|------|------------|----------|
| **I/O 密集型** | 大量时间花在等待（网络请求、文件读写、数据库查询） | ✅ 非常有效 | 多线程/协程 |
| **CPU 密集型** | 大量时间花在计算（数学运算、图像处理） | ❌ 几乎无加速 | 多进程 |

> **重点**：对于我们初学者来说，多线程最实用的场景就是 **I/O 密集型任务**——比如同时下载多个文件、同时发起多个网络请求、同时读写多个文件。

### 4.3 线程安全问题

当多个线程**同时修改**同一个变量时，可能出现数据不一致的问题：

```python
import threading

counter = 0  # 共享变量

def increment():
    """每个线程让 counter 加 100000 次"""
    global counter
    for _ in range(100000):
        counter += 1

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)

t1.start()
t2.start()

t1.join()
t2.join()

print(f"期望结果: 200000")
print(f"实际结果: {counter}")  # 大概率不是 200000！
```

**为什么？** 因为 `counter += 1` 不是原子操作，它实际上分三步：
1. 读取 counter 的值
2. 加 1
3. 写回 counter

两个线程可能同时读到相同的值，各自加 1 后写回，导致只加了 1 而不是 2。

### 4.4 使用锁（Lock）解决线程安全问题

`threading.Lock()` 是最常用的同步工具：

```python
import threading

counter = 0
lock = threading.Lock()  # 创建一把锁

def increment():
    global counter
    for _ in range(100000):
        # 获取锁 —— 只有一个线程能同时持有这把锁
        lock.acquire()
        try:
            counter += 1  # 这段代码同一时刻只有一个线程能执行
        finally:
            lock.release()  # 释放锁，让其他线程可以进入

# 更简洁的写法：使用 with 语句（推荐）
def increment_safe():
    global counter
    for _ in range(100000):
        with lock:        # 自动获取和释放锁
            counter += 1

t1 = threading.Thread(target=increment_safe)
t2 = threading.Thread(target=increment_safe)

t1.start()
t2.start()
t1.join()
t2.join()

print(f"结果: {counter}")  # 这次一定是 200000！
```

> **注意**：`with lock` 是推荐写法，它保证即使代码出错，锁也会被正确释放，避免死锁。

---

## 五、实战：多线程下载模拟器

下面用一个完整的例子来体验多线程的威力：

```python
import threading
import time
import random

def download_file(filename, duration):
    """模拟下载文件"""
    print(f"[{threading.current_thread().name}] 开始下载: {filename}")
    time.sleep(duration)  # 模拟下载耗时
    print(f"[{threading.current_thread().name}] 下载完成: {filename} (耗时{duration}秒)")

def main():
    files = [
        ("报告.pdf", 3),
        ("照片.jpg", 2),
        ("视频.mp4", 4),
        ("音乐.mp3", 1),
        ("数据.xlsx", 2),
    ]

    print("===== 单线程下载 =====")
    start = time.time()
    for name, duration in files:
        download_file(name, duration)
    single_time = time.time() - start
    print(f"单线程总耗时: {single_time:.1f}秒\n")

    print("===== 多线程下载 =====")
    start = time.time()
    threads = []
    for name, duration in files:
        t = threading.Thread(
            target=download_file,
            args=(name, duration),
            name=f"DL-{name}"
        )
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    multi_time = time.time() - start
    print(f"多线程总耗时: {multi_time:.1f}秒")
    print(f"\n提速了 {single_time / multi_time:.1f} 倍！")

if __name__ == "__main__":
    main()
```

**运行结果**（大致如下）：
```
===== 单线程下载 =====
[MainThread] 开始下载: 报告.pdf
[MainThread] 下载完成: 报告.pdf (耗时3秒)
[MainThread] 开始下载: 照片.jpg
...
单线程总耗时: 12.0秒

===== 多线程下载 =====
[DL-报告.pdf] 开始下载: 报告.pdf
[DL-照片.jpg] 开始下载: 照片.jpg
[DL-视频.mp4] 开始下载: 视频.mp4
[DL-音乐.mp3] 开始下载: 音乐.mp3
[DL-数据.xlsx] 开始下载: 数据.xlsx
[DL-音乐.mp3] 下载完成: 音乐.mp3 (耗时1秒)
...
多线程总耗时: 4.0秒

提速了 3.0 倍！
```

> 多线程几乎只用了最慢那个文件的时间（4秒），而单线程需要累加所有时间（12秒）。这就是多线程在 I/O 场景下的巨大优势！

---

## 六、其他常用同步工具

除了 Lock，`threading` 模块还提供了其他同步原语：

### 6.1 RLock（可重入锁）

允许同一个线程多次获取同一把锁，不会死锁：

```python
import threading

lock = threading.RLock()  # 可重入锁

def outer():
    with lock:
        print("外层函数获取锁")
        inner()  # 同一个线程再次获取同一把锁，不会死锁

def inner():
    with lock:
        print("内层函数获取锁")

t = threading.Thread(target=outer)
t.start()
t.join()
```

### 6.2 Event（事件通知）

线程间的一种简单通信机制：

```python
import threading
import time

event = threading.Event()  # 创建事件对象

def waiter(name):
    """等待事件的线程"""
    print(f"[{name}] 等待通知中...")
    event.wait()  # 阻塞，直到 event.set() 被调用
    print(f"[{name}] 收到通知，开始工作！")

def notifier():
    """发送通知的线程"""
    time.sleep(2)
    print("[通知者] 准备发送通知...")
    event.set()  # 唤醒所有等待的线程
    print("[通知者] 通知已发送！")

# 创建多个等待线程
for i in range(3):
    t = threading.Thread(target=waiter, args=(f"Worker-{i+1}",))
    t.start()

# 创建通知线程
t_notify = threading.Thread(target=notifier)
t_notify.start()

# 等待所有线程
for t in threading.enumerate():
    if t is not threading.current_thread():
        t.join()
```

### 6.3 Semaphore（信号量）

限制同时运行的线程数量：

```python
import threading
import time

# 信号量：最多允许 2 个线程同时运行
semaphore = threading.Semaphore(2)

def access_resource(name):
    with semaphore:
        print(f"[{name}] 获得访问权限")
        time.sleep(2)
        print(f"[{name}] 释放访问权限")

threads = []
for i in range(5):
    t = threading.Thread(target=access_resource, args=(f"Thread-{i+1}",))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

> 这个例子中，5 个线程争抢 2 个"许可证"，同一时刻最多只有 2 个线程在执行。适合控制数据库连接数、API 请求并发数等场景。

---

## 七、练习题

### 练习 1：基础巩固

编写一个程序，创建 3 个线程，分别打印数字 1-5、字母 a-e、符号 ★-★★★★★，观察多线程的交替执行效果。

```python
# 提示框架
import threading
import time

def print_numbers():
    for i in range(1, 6):
        print(f"数字: {i}")
        time.sleep(0.3)

def print_letters():
    # 补充代码
    pass

def print_stars():
    # 补充代码
    pass

# 补充：创建线程、启动、等待
```

### 练习 2：计时器

创建一个倒计时程序：主线程接收用户输入的秒数，一个后台线程每秒更新倒计时，倒计时结束后通知主线程。

```python
# 提示：使用 Event 来实现线程间通信
import threading
import time

class CountdownTimer:
    def __init__(self, seconds):
        self.seconds = seconds
        self.event = threading.Event()

    def run(self):
        # 在后台线程中实现倒计时
        pass
```

### 练习 3：线程安全的计数器

使用 Lock 编写一个线程安全的计数器类，支持 `increment()`、`decrement()` 和 `get_value()` 方法，用 10 个线程同时操作验证正确性。

```python
# 目标：Counter 类的 increment 被 10 个线程各调用 10000 次
# 最终 get_value() 应该返回 100000
class Counter:
    def __init__(self):
        # 补充代码
        pass

    def increment(self):
        # 补充代码
        pass

    def get_value(self):
        # 补充代码
        pass
```

---

## 八、常见问题

### Q1：多线程和多进程怎么选？

**A**：简单的判断标准：
- 任务主要是**等待**（网络、磁盘、用户输入）→ 用**多线程**
- 任务主要是**计算**（数学运算、数据处理）→ 用**多进程**（`multiprocessing` 模块）
- 不确定 → 先用多线程，简单易用

### Q2：join() 和 daemon 有什么区别？

**A**：
- `join()` = 主线程**主动等待**子线程结束
- `daemon = True` = 主线程**不等待**这个线程，主线程结束时守护线程自动被杀掉

### Q3：为什么会死锁？怎么避免？

**A**：死锁是两个线程互相等待对方释放锁，导致永远卡住。常见原因和避免方法：

```python
# ❌ 可能死锁的写法
lock_a = threading.Lock()
lock_b = threading.Lock()

def task1():
    lock_a.acquire()
    lock_b.acquire()  # 如果 task2 先拿了 lock_b，这里就卡住了
    # ...
    lock_b.release()
    lock_a.release()

def task2():
    lock_b.acquire()
    lock_a.acquire()  # 如果 task1 先拿了 lock_a，这里也卡住了
    # ...
    lock_a.release()
    lock_b.release()
```

**避免方法**：
1. 所有线程按**相同顺序**获取锁
2. 使用 `with lock` 语法（推荐）
3. 设置超时：`lock.acquire(timeout=5)`
4. 尽量减少锁的持有时间

### Q4： threading.current_thread() 有什么用？

**A**：在多线程代码中，打印日志或调试时非常有用，可以知道当前是哪个线程在执行：

```python
t = threading.current_thread()
print(f"线程名: {t.name}")
print(f"线程ID: {t.ident}")
print(f"是否存活: {t.is_alive()}")
```

### Q5：Python 多线程能利用多核 CPU 吗？

**A**：由于 GIL 的存在，Python 多线程在 **CPU 密集型**任务中无法真正并行利用多核。但如果你用 C 扩展、NumPy 等库，它们在执行底层计算时会释放 GIL，此时可以真正利用多核。对于 I/O 密集型任务（我们最常用的场景），GIL 在等待 I/O 时会被释放，所以多线程完全有效。

---

## 九、免费学习资源

| 资源 | 链接 | 说明 |
|------|------|------|
| 廖雪峰 - 多线程编程 | https://www.liaoxuefeng.com/wiki/1016959663602400/1017628749097152 | 讲解清晰，配有代码示例 |
| 菜鸟教程 - Python 多线程 | https://www.runoob.com/python3/python3-multithreading.html | 适合快速查阅 |
| Python 官方文档 - threading | https://docs.python.org/zh-cn/3/library/threading.html | 最权威的参考 |
| Real Python - Concurrency | https://realpython.com/python-concurrency/ | 英文，深入讲解 |

---

## 十、本节小结

| 知识点 | 要点 |
|--------|------|
| 线程 vs 进程 | 线程轻量、共享内存；进程独立、开销大 |
| 创建线程 | `Thread(target=fn)` 或继承 `Thread` 类 |
| `start()` / `join()` | 启动线程 / 等待线程结束 |
| 守护线程 | `daemon=True`，主线程结束时自动终止 |
| GIL | Python 同一时刻只有一个线程执行字节码，I/O 等待时释放 |
| 线程安全 | 多线程修改共享变量时需要用 `Lock` |
| `with lock` | 推荐的加锁写法，自动获取/释放 |
| Event | 线程间简单通知机制 |
| Semaphore | 限制并发线程数量 |

> **下一节**：Day 26 — 异常处理进阶，我们将学习自定义异常、异常链、try 的完整语法等高级异常处理技巧。
