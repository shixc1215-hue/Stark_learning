# Python Day 43 — NumPy 基础

> **学习目标**：掌握 NumPy 的核心概念 ndarray，学会创建、索引、运算数组，理解广播机制，为科学计算和数据分析打下基础。

---

## 一、什么是 NumPy？

NumPy（Numerical Python）是 Python 科学计算的基础库。它的核心是 **ndarray**（N-dimensional array，N维数组）对象——一个高效的多维容器，可以存储同类型的元素。

### 为什么要学 NumPy？

| 对比项 | Python 原生 list | NumPy ndarray |
|--------|----------------|---------------|
| 数据类型 | 可以混合类型 | 必须同类型 |
| 运算方式 | 逐个循环 | **向量化**（C底层实现） |
| 内存占用 | 较大（每个元素是独立对象） | 紧凑（连续内存块） |
| 运算速度 | 慢 | **快 10-100 倍** |
| 多维支持 | 需要嵌套 list | 原生支持 |

```python
import numpy as np

# 一个简单的性能对比
import time

# 用 Python list
py_list = list(range(1000000))
start = time.time()
py_result = [x * 2 for x in py_list]
print(f"list 耗时: {time.time() - start:.4f} 秒")

# 用 NumPy
np_array = np.arange(1000000)
start = time.time()
np_result = np_array * 2  # 向量化运算，无需循环
print(f"numpy 耗时: {time.time() - start:.4f} 秒")
```

输出大致如下：
```
list 耗时: 0.0800 秒
numpy 耗时: 0.0020 秒
```

速度差距非常明显！这就是为什么几乎所有 Python 数据分析库（pandas、scikit-learn 等）都基于 NumPy。

### 安装

```bash
pip install numpy
```

```python
import numpy as np  # 约定俗成的别名
print(np.__version__)  # 查看版本
```

---

## 二、创建 ndarray

### 2.1 从 Python 数据创建

```python
import numpy as np

# 1. 从 list 创建
arr1 = np.array([1, 2, 3, 4, 5])
print(arr1)          # [1 2 3 4 5]
print(type(arr1))     # <class 'numpy.ndarray'>

# 2. 从嵌套 list 创建二维数组
arr2 = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2)
# [[1 2 3]
#  [4 5 6]]

# 3. 指定数据类型（dtype）
arr3 = np.array([1, 2, 3], dtype=float)
print(arr3)          # [1. 2. 3.]
print(arr3.dtype)    # float64

# 常见 dtype 类型
print(np.array([True, False]).dtype)        # bool
print(np.array([1, 2, 3]).dtype)            # int64（系统默认）
print(np.array([1.0, 2.0]).dtype)           # float64
print(np.array([1, 2, 3], dtype=np.int32))  # int32
print(np.array(['a', 'b']).dtype)           # <U1（Unicode 字符串）
```

### 2.2 NumPy 内置创建函数

```python
import numpy as np

# --- 全零、全一、全指定值 ---
print(np.zeros(5))            # [0. 0. 0. 0. 0.]          一维，5个0
print(np.zeros((2, 3)))       # 2行3列的全零矩阵
# [[0. 0. 0.]
#  [0. 0. 0.]]

print(np.ones((3, 3)))        # 3x3 全一矩阵
print(np.full((2, 2), 7))     # 2x2 矩阵，所有元素为7

# --- 等差数列 ---
print(np.arange(0, 10, 2))    # [0 2 4 6 8]   起点、终点(不含)、步长
print(np.arange(5))           # [0 1 2 3 4]   单参数：从0开始

# --- 等分序列 ---
print(np.linspace(0, 1, 5))   # [0.   0.25 0.5  0.75 1.  ]  在[0,1]均匀取5个点
print(np.linspace(0, 100, 11))  # [  0.  10.  20.  30.  40.  50.  60.  70.  80.  90. 100.]

# --- 随机数组 ---
np.random.seed(42)            # 固定随机种子，保证可复现

print(np.random.rand(3))       # [0.3745 0.9507 0.7320]  3个[0,1)均匀分布随机数
print(np.random.randint(1, 10, size=5))  # [3 7 8 5 4]    5个[1,10)随机整数
print(np.random.randn(3))      # 标准正态分布随机数

# --- 单位矩阵 ---
print(np.eye(3))
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]
```

> **💡 小贴士**：`np.arange` 类似 Python 内置的 `range()`，但返回的是 ndarray；`np.linspace` 更适合需要**固定数量**的等分点（比如画图时的 x 坐标）。

---

## 三、ndarray 的核心属性

```python
import numpy as np

arr = np.array([[1, 2, 3, 4],
                [5, 6, 7, 8],
                [9, 10, 11, 12]])

print(arr.ndim)     # 2    维度数（几维数组）
print(arr.shape)    # (3, 4)  形状（3行4列）
print(arr.size)     # 12   元素总数
print(arr.dtype)    # int64  数据类型
print(arr.itemsize) # 8    每个元素占的字节数（int64 = 8字节）
print(arr.nbytes)   # 96   总字节数 = size × itemsize

# 一维数组
arr1d = np.array([10, 20, 30])
print(arr1d.shape)  # (3,)   注意逗号，表示一维有3个元素
```

> **💡 小贴士**：`.shape` 返回一个元组。一维数组的形状是 `(n,)`，注意那个逗号——它和 `(n)` 含义不同（后者只是括号包裹的整数）。

---

## 四、数组索引与切片

### 4.1 一维数组索引

```python
import numpy as np

arr = np.array([10, 20, 30, 40, 50, 60])

print(arr[0])       # 10     第一个元素
print(arr[-1])      # 60     最后一个元素
print(arr[2:5])     # [30 40 50]  切片 [起始:终止:步长]（不含终止）
print(arr[::2])     # [10 30 50]  每隔一个取一个
print(arr[::-1])    # [60 50 40 30 20 10]  反转
```

### 4.2 二维数组索引

```python
import numpy as np

arr = np.array([[1, 2, 3],
                [4, 5, 6],
                [7, 8, 9]])

# 基本索引 —— arr[行, 列]
print(arr[0, 1])    # 2     第0行第1列
print(arr[2, 2])    # 9     第2行第2列
print(arr[0])       # [1 2 3]  取第0行（所有列）
print(arr[:, 1])    # [2 5 8]  取第1列（所有行）

# 切片
print(arr[0:2])          # 前两行
# [[1 2 3]
#  [4 5 6]]

print(arr[:, 1:])       # 所有行，从第1列开始
# [[2 3]
#  [5 6]
#  [8 9]]

print(arr[0:2, 1:3])    # 前两行，第1~2列
# [[2 3]
#  [5 6]]

# 花式索引（用列表指定多个位置）
print(arr[[0, 2]])       # 第0行和第2行
# [[1 2 3]
#  [7 8 9]]

print(arr[:, [0, 2]])    # 第0列和第2列
# [[1 3]
#  [4 6]
#  [7 9]]
```

### 4.3 布尔索引（非常重要）

```python
import numpy as np

arr = np.array([15, 25, 35, 45, 55, 65])

# 1. 生成条件布尔数组
print(arr > 30)           # [False False  True  True  True  True]

# 2. 用布尔数组筛选元素
print(arr[arr > 30])      # [35 45 55 65]

# 3. 组合条件（用 & | ~，不要用 and/or/not）
print(arr[(arr > 20) & (arr < 50)])  # [25 35 45]
print(arr[(arr < 20) | (arr > 60)])  # [15 65]

# 4. 二维数组中的布尔索引
scores = np.array([[80, 90, 70],
                   [60, 85, 95],
                   [40, 75, 88]])

# 筛选出大于80分的成绩
print(scores[scores > 80])     # [90 85 95 88]

# 找到不及格的分数（< 60）
print(scores[scores < 60])     # [40]
```

> **⚠️ 常见错误**：布尔索引组合条件时，必须用 `&`（和）、`|`（或）、`~`（非），且每个条件要用括号包裹。`and`/`or`/`not` 不可用于 ndarray 的逐元素比较。

---

## 五、数组的形状变换

```python
import numpy as np

# reshape —— 改变形状（元素总数不变）
arr = np.arange(12)           # [0 1 2 3 4 5 6 7 8 9 10 11]
print(arr.reshape(3, 4))
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

print(arr.reshape(4, 3))
# [[ 0  1  2]
#  [ 3  4  5]
#  [ 6  7  8]
#  [ 9 10 11]]

# -1 表示自动推算该维度
print(arr.reshape(3, -1))     # 自动算出每行4列 → (3, 4)
print(arr.reshape(-1, 4))     # 自动算出行数 → (3, 4)
print(arr.reshape(-1))         # 展平为一维

# flatten vs ravel —— 展平为一维
arr2d = np.array([[1, 2], [3, 4]])
print(arr2d.flatten())    # [1 2 3 4]  返回副本
print(arr2d.ravel())      # [1 2 3 4]  返回视图（可能影响原数组）

# T —— 转置（行列互换）
arr3x4 = np.arange(12).reshape(3, 4)
print(arr3x4.T.shape)     # (4, 3)

# expand_dims —— 增加维度
arr1d = np.array([1, 2, 3])
print(np.expand_dims(arr1d, axis=0))  # [[1 2 3]]   形状 (1, 3)
print(np.expand_dims(arr1d, axis=1))  # [[1]
                                      #  [2]        形状 (3, 1)
                                      #  [3]]

# squeeze —— 去除长度为1的维度
arr_squeeze = np.array([[[1, 2, 3]]])  # 形状 (1, 1, 3)
print(np.squeeze(arr_squeeze).shape)    # (3,)
```

---

## 六、数组运算（向量化）

NumPy 的核心优势——**向量化运算**：对整个数组进行操作，无需写循环。

### 6.1 算术运算

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

# 四则运算 —— 逐元素计算
print(a + b)     # [11 22 33 34]
print(a - b)     # [ -9 -18 -27 -36]
print(a * b)     # [ 10  40  90 160]
print(b / a)     # [10. 10. 10. 10.]

# 标量运算 —— 广播
print(a + 100)   # [101 102 103 104]
print(a * 2)     # [2 4 6 8]
print(a ** 2)    # [ 1  4  9 16]   平方
print(a % 3)     # [1 2 0 1]      取模

# 二维数组运算
m1 = np.array([[1, 2], [3, 4]])
m2 = np.array([[5, 6], [7, 8]])

print(m1 + m2)
# [[ 6  8]
#  [10 12]]

print(m1 * m2)    # 逐元素相乘（注意：不是矩阵乘法！）
# [[ 5 12]
#  [21 32]]

print(m1 @ m2)    # 矩阵乘法（点积）
# [[19 22]
#  [43 50]]
```

### 6.2 通用函数（ufunc）

NumPy 提供了大量逐元素操作的数学函数：

```python
import numpy as np

arr = np.array([1, 4, 9, 16, 25])

# 数学函数
print(np.sqrt(arr))     # [1. 2. 3. 4. 5.]     平方根
print(np.abs(arr - 10)) # [9. 6. 1. 6. 15.]    绝对值
print(np.exp(np.array([0, 1, 2])))  # [1.    2.718 7.389]  e的n次方
print(np.log(np.array([1, np.e, np.e**2])))  # [0. 1. 2.]  自然对数

# 取整函数
print(np.floor(np.array([1.2, 2.8, -1.5])))   # [ 1.  2. -2.]  向下取整
print(np.ceil(np.array([1.2, 2.8, -1.5])))    # [ 2.  3. -1.]  向上取整
print(np.round(np.array([1.45, 2.55, 3.65]), 1))  # [1.4 2.6 3.6] 四舍五入

# 比较函数
a = np.array([1, 3, 5, 7])
b = np.array([2, 3, 4, 7])
print(np.maximum(a, b))   # [2 3 5 7]  逐元素取较大值
print(np.minimum(a, b))   # [1 3 4 7]  逐元素取较小值
```

---

## 七、广播机制（Broadcasting）

广播是 NumPy 最强大的特性之一：当两个数组形状不同时，NumPy 会自动"扩展"较小的数组，使它们可以进行运算。

### 广播规则

1. 如果两个数组的维度数不同，较小的数组会在**左侧**补 1 维
2. 如果在某一维度上，两个数组大小不同（其中一个为 1），则较小者会被**复制扩展**
3. 如果某维度上大小不同且都不为 1 → **报错**

```python
import numpy as np

# 规则1示例：标量与数组
a = np.array([[1, 2, 3],    # 形状 (2, 3)
              [4, 5, 6]])
print(a + 10)
# 标量 10 → 广播为 [[10,10,10],[10,10,10]] → 逐元素相加
# [[11 12 13]
#  [14 15 16]]

# 规则2示例：一维数组与二维数组
row = np.array([10, 20, 30])   # 形状 (3,)
a_row = a + row
# row → (1, 3) → (2, 3)  沿行方向复制
# [[11 22 33]
#  [14 25 36]]

col = np.array([[10], [20]])   # 形状 (2, 1)
a_col = a + col
# col → (2, 1) → (2, 3)  沿列方向复制
# [[11 12 13]
#  [24 25 26]]

# 实际应用：标准化数据
data = np.array([[80, 90, 70],
                 [60, 85, 95],
                 [40, 75, 88]])
mean = data.mean(axis=0)  # 每列均值 [60.  83.33 84.33]
std = data.std(axis=0)    # 每列标准差

normalized = (data - mean) / std  # 广播实现 Z-score 标准化
print(normalized)
```

> **💡 小贴士**：广播让你不用写循环就能对整个数组做运算，但要注意理解"谁被扩展了"。画个形状对照表更容易理解。

---

## 八、聚合与统计

```python
import numpy as np

arr = np.array([[80, 90, 70],
                [60, 85, 95],
                [40, 75, 88]])

# 基本统计
print(arr.sum())       # 583    所有元素求和
print(arr.mean())      # 64.78  平均值
print(arr.max())       # 95     最大值
print(arr.min())       # 40     最小值
print(arr.std())       # 16.29  标准差
print(arr.var())       # 265.61 方差

# axis 参数 —— 按轴聚合
print(arr.sum(axis=0))   # [180 250 253]  每列求和
print(arr.sum(axis=1))   # [240 240 203]  每行求和
print(arr.mean(axis=0))  # [60.  83.33 84.33]  每列平均
print(arr.max(axis=1))  # [90 95 88]  每行最大值

# 累计运算
print(np.cumsum(arr))    # 累计求和（返回一维）
print(np.cumprod(np.array([1, 2, 3, 4])))  # [1 2 6 24] 累计乘积

# 排序
arr_unsorted = np.array([30, 10, 50, 20, 40])
print(np.sort(arr_unsorted))     # [10 20 30 40 50]  返回排序后副本
arr_unsorted.sort()              # 原地排序
print(arr_unsorted)              # [10 20 30 40 50]

# 找索引
arr2 = np.array([30, 10, 50, 20, 40])
print(np.argmax(arr2))   # 2     最大值的索引
print(np.argmin(arr2))   # 1     最小值的索引
print(np.argsort(arr2))  # [1 3 0 4 2]  排序后的索引顺序

# 唯一值与计数
arr3 = np.array(['apple', 'banana', 'apple', 'cherry', 'banana'])
print(np.unique(arr3))              # ['apple' 'banana' 'cherry']
print(np.unique(arr3, return_counts=True))  # (array([...]), array([2, 2, 1]))
```

> **💡 小贴士**：`axis=0` 是沿行方向（跨行）聚合 → 结果是每列的值；`axis=1` 是沿列方向（跨列）聚合 → 结果是每行的值。记住"沿着某个轴压缩"。

---

## 九、数组的拷贝与视图

```python
import numpy as np

# 视图（View）—— 共享数据，修改会互相影响
original = np.array([1, 2, 3, 4, 5])
view = original[1:4]      # 切片返回视图
view[0] = 99
print(original)            # [ 1 99  3  4  5]  ← 原数组也被改了！

# 副本（Copy）—— 独立数据
original = np.array([1, 2, 3, 4, 5])
copy = original[1:4].copy()  # 显式复制
copy[0] = 99
print(original)            # [1 2 3 4 5]  ← 原数组不受影响

# 如何判断？
a = np.array([1, 2, 3])
b = a[:2]
print(b.base is a)         # True  —— b 是 a 的视图

c = a[:2].copy()
print(c.base is a)         # False —— c 是独立副本
```

> **⚠️ 常见坑**：切片操作返回的是**视图**（view），不是副本！如果你修改切片，原数组也会变。需要独立副本时记得用 `.copy()`。

---

## 十、综合实战：学生成绩分析系统

```python
import numpy as np

# 1. 生成模拟数据：5名学生，4门课程
np.random.seed(42)
subjects = ['语文', '数学', '英语', '物理']
students = ['张三', '李四', '王五', '赵六', '孙七']

scores = np.random.randint(50, 100, size=(5, 4))
print("=== 成绩表 ===")
print(scores)

# 2. 每位学生的总分和平均分
total_per_student = scores.sum(axis=1)
avg_per_student = scores.mean(axis=1)
print("\n=== 学生成绩汇总 ===")
for i, name in enumerate(students):
    print(f"{name}: 总分={total_per_student[i]}, 平均分={avg_per_student[i]:.1f}")

# 3. 每门课程的统计
print("\n=== 各科目统计 ===")
for j, subj in enumerate(subjects):
    col = scores[:, j]
    print(f"{subj}: 最高={col.max()}, 最低={col.min()}, "
          f"平均={col.mean():.1f}, 标准差={col.std():.1f}")

# 4. 找出每科的最高分学生
print("\n=== 各科最高分学生 ===")
for j, subj in enumerate(subjects):
    best_idx = np.argmax(scores[:, j])
    print(f"{subj}: {students[best_idx]}（{scores[best_idx, j]}分）")

# 5. 及格率（>= 60 分的比例）
pass_rate = (scores >= 60).sum() / scores.size * 100
print(f"\n总体及格率: {pass_rate:.1f}%")

# 6. 找出需要补考的学生（有科目 < 60）
failed = (scores < 60).any(axis=1)  # any: 只要有一科不及格
failed_students = [students[i] for i in range(len(students)) if failed[i]]
print(f"需要补考的学生: {', '.join(failed_students) if failed_students else '无'}")

# 7. Z-score 标准化（消除量纲）
mean = scores.mean(axis=0)
std = scores.std(axis=0)
z_scores = (scores - mean) / std
print("\n=== 标准化后的成绩（Z-score）===")
print(np.round(z_scores, 2))

# 8. 综合排名（按平均分降序）
ranking = np.argsort(-avg_per_student)  # 降序排列的索引
print("\n=== 综合排名 ===")
for rank, idx in enumerate(ranking, 1):
    print(f"第{rank}名: {students[idx]}（平均分={avg_per_student[idx]:.1f}）")
```

---

## 练习题

### 练习1：基础操作

创建一个 5×5 的单位矩阵，将其对角线元素替换为 `[1, 2, 3, 4, 5]`，然后计算每行的和。

```python
# 参考答案
matrix = np.eye(5)
np.fill_diagonal(matrix, [1, 2, 3, 4, 5])
row_sums = matrix.sum(axis=1)
print(matrix)
print("每行和:", row_sums)
```

### 练习2：布尔索引

给定一组温度数据（一周七天），筛选出温度在 20°C 以上且湿度低于 60% 的天数。

```python
# 参考答案
days = np.array(['周一', '周二', '周三', '周四', '周五', '周六', '周日'])
temperature = np.array([18, 25, 22, 30, 28, 19, 24])
humidity = np.array([55, 70, 50, 65, 45, 80, 55])

mask = (temperature > 20) & (humidity < 60)
print("适合出门的日子:", days[mask])
```

### 练习3：广播实战

用广播机制，为下面的成绩矩阵的每一列减去该列的平均值，实现去中心化。

```python
# 参考答案
scores = np.array([[80, 90, 70],
                  [60, 85, 95],
                  [40, 75, 88]])
col_mean = scores.mean(axis=0)       # 形状 (3,)
centered = scores - col_mean          # 广播 (3,3) - (3,) → 自动扩展
print(centered)
```

---

## 常见问题 FAQ

### Q1：np.array([1, 2, 3]) 和 Python 的 [1, 2, 3] 有什么区别？

最大的区别是**运算方式**。Python list 的 `+` 是拼接，`*` 是重复；而 NumPy 的 `+` 是逐元素相加，`*` 是逐元素相乘。此外 NumPy 数组要求所有元素同类型，内存效率更高。

### Q2：什么时候用 reshape，什么时候用 resize？

`reshape` 返回视图或副本（取决于内存布局），**不改变原数组**；`resize` 直接修改原数组本身。日常使用中 `reshape` 更常用更安全。

### Q3：广播失败报错怎么办？

检查两个数组在对应维度上是否兼容——要么大小相同，要么其中一方大小为 1。如果完全不兼容，可以先用 `reshape` 或 `np.newaxis` 调整形状。

### Q4：为什么有时候 sum() 返回的值跟预期不一样？

注意 `axis` 参数：不传 `axis` 是对**所有元素**求和；`axis=0` 是按列方向压缩（跨行）；`axis=1` 是按行方向压缩（跨列）。搞混了会导致结果完全不同。

### Q5：NumPy 能处理缺失值吗？

原生 NumPy 不能很好地处理 NaN（Not a Number）。对于含有缺失值的数据分析，推荐使用 `numpy.ma`（掩码数组）或 pandas（下一阶段会学）。但可以用 `np.nanmean()`、`np.nanmax()` 等函数忽略 NaN 进行计算。

---

## 免费学习资源

- **NumPy 官方文档（中文）**：https://numpy.org.cn/ — 最权威的参考，含完整函数说明
- **廖雪峰 NumPy 教程**：https://www.liaoxuefeng.com/wiki/1016959663602400 — 简洁易懂的中文教程
- **菜鸟教程 NumPy**：https://www.runoob.com/numpy/numpy-tutorial.html — 适合入门速查
- **NumPy 官方快速入门**：https://numpy.org/doc/stable/user/quickstart.html — 官方英文指南
- **YouTube: Corey Schafer NumPy**：https://www.youtube.com/playlist?list=PL-osiE809TBEo9AgV5r03UzYJx4QYkVpQ — 视频教程

---

## 今日小结

| 概念 | 要点 |
|------|------|
| ndarray | NumPy 核心，高效同类型多维数组 |
| 创建数组 | `np.array`, `zeros`, `ones`, `arange`, `linspace`, `random` |
| 属性 | `ndim`, `shape`, `size`, `dtype` |
| 索引切片 | 一维/二维索引、布尔索引、花式索引 |
| 形状变换 | `reshape`, `flatten`, `T`, `expand_dims` |
| 向量化运算 | 逐元素算术、ufunc 函数、避免循环 |
| 广播 | 形状不同时自动扩展对齐 |
| 聚合统计 | `sum/mean/max/min` + `axis` 参数 |
| 视图与副本 | 切片返回视图，`.copy()` 获得独立副本 |

> **下期预告**：Day 44 — NumPy 进阶，将学习线性代数运算、数组拼接与分割、文件读写以及高级索引技巧。
