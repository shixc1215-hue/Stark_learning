# Python Day 44 — NumPy 进阶

> **学习目标**：在 Day 43 基础上，掌握线性代数运算、数组拼接与分割、条件函数、文件读写、结构化数组以及性能优化技巧，全面提升 NumPy 实战能力。

---

## 一、线性代数运算

NumPy 的 `np.linalg` 子模块提供了丰富的线性代数工具，是机器学习和科学计算的基石。

### 1.1 矩阵乘法（回顾与扩展）

```python
import numpy as np

A = np.array([[1, 2],
              [3, 4]])
B = np.array([[5, 6],
              [7, 8]])

# 三种写法，结果相同
print(A @ B)           # 推荐：Python 3.5+ 运算符
print(np.dot(A, B))    # 经典写法
print(np.matmul(A, B)) # 完整函数名

# 结果:
# [[19 22]
#  [43 50]]

# ⚠️ 注意区分：* 是逐元素相乘，@ 是矩阵乘法
print(A * B)
# [[ 5 12]
#  [21 32]]
```

### 1.2 行列式与逆矩阵

```python
import numpy as np

A = np.array([[1, 2],
              [3, 4]])

# 行列式
det = np.linalg.det(A)
print(f"行列式 = {det:.2f}")  # 行列式 = -2.00

# 逆矩阵（矩阵必须可逆，即行列式 ≠ 0）
A_inv = np.linalg.inv(A)
print("A 的逆矩阵:")
print(A_inv)
# [[-2.   1. ]
#  [ 1.5 -0.5]]

# 验证：A × A_inv ≈ 单位矩阵
print(A @ A_inv)
# [[ 1.0000000e+00  0.0000000e+00]
#  [ 4.4408921e-16  1.0000000e+00]]  (浮点误差，近似 0)

# 伪逆矩阵（适用于不可逆矩阵或非方阵）
B = np.array([[1, 2, 3],
              [4, 5, 6]])  # 2×3 矩阵，无逆矩阵
B_pinv = np.linalg.pinv(B)
print("伪逆矩阵形状:", B_pinv.shape)  # (3, 2)
```

### 1.3 解线性方程组

求解 `Ax = b` 是科学计算中最常见的任务之一：

```python
import numpy as np

# 解方程组：
# 2x + y = 5
# 3x + 4y = 10

A = np.array([[2, 1],
              [3, 4]])
b = np.array([5, 10])

# 方法1：直接求解（推荐）
x = np.linalg.solve(A, b)
print(f"x = {x[0]:.2f}, y = {x[1]:.2f}")  # x = 1.43, y = 2.14

# 方法2：用逆矩阵（理解原理用）
x2 = np.linalg.inv(A) @ b
print(x2)  # 结果相同

# 验证
print("A @ x =", A @ x)  # [ 5. 10.] ✓
```

### 1.4 特征值与特征向量

```python
import numpy as np

# 对称矩阵的特征值分解（常用于 PCA 降维）
matrix = np.array([[4, 2],
                   [2, 3]])

eigenvalues, eigenvectors = np.linalg.eig(matrix)
print("特征值:", eigenvalues)      # [5.56155281 1.43844719]
print("特征向量矩阵:")
print(eigenvectors)

# 验证：A @ v = λ * v
idx = 0
v = eigenvectors[:, idx]
lam = eigenvalues[idx]
print("验证 Av = λv:", np.allclose(matrix @ v, lam * v))  # True

# 矩阵的迹（trace = 特征值之和）
print("迹:", np.trace(matrix))              # 7
print("特征值之和:", eigenvalues.sum())     # 7.0 ✓
```

### 1.5 范数与秩

```python
import numpy as np

A = np.array([[1, 2, 3],
              [4, 5, 6]])

# 矩阵的秩（线性无关的行/列数）
print("秩:", np.linalg.matrix_rank(A))  # 2

# 向量范数
v = np.array([3, 4])
print("L2 范数（长度）:", np.linalg.norm(v))      # 5.0  (√(3²+4²))
print("L1 范数:", np.linalg.norm(v, ord=1))       # 7    (|3|+|4|)
print("无穷范数:", np.linalg.norm(v, ord=np.inf)) # 4    (max(|3|,|4|))

# 矩阵范数
print("Frobenius 范数:", np.linalg.norm(A))  # 9.54
print("Frobenius 范数:", np.sqrt((A ** 2).sum()))  # 等价写法
```

> **💡 小贴士**：线性代数在机器学习中无处不在——最小二乘法、PCA、SVD、神经网络的反向传播都依赖这些运算。不必死记公式，但要知道"用什么函数"。

---

## 二、数组拼接与分割

### 2.1 拼接（concatenate / stack）

```python
import numpy as np

a = np.array([[1, 2],
              [3, 4]])
b = np.array([[5, 6],
              [7, 8]])

# np.concatenate —— 沿指定轴拼接
print(np.concatenate([a, b], axis=0))  # 纵向拼接（上下）
# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]

print(np.concatenate([a, b], axis=1))  # 横向拼接（左右）
# [[1 2 5 6]
#  [3 4 7 8]]

# 对于一维数组（axis=0 拼接，无 axis=1）
c = np.array([1, 2, 3])
d = np.array([4, 5, 6])
print(np.concatenate([c, d]))  # [1 2 3 4 5 6]

# np.stack —— 堆叠，增加新维度
print(np.stack([a, b], axis=0).shape)  # (2, 2, 2)  增加了第0维
print(np.stack([a, b], axis=1).shape)  # (2, 2, 2)  增加了第1维
print(np.stack([a, b], axis=2).shape)  # (2, 2, 2)  增加了第2维

# np.vstack / np.hstack —— 更直观的别名
print(np.vstack([a, b]))  # 等价于 concatenate(axis=0)
print(np.hstack([a, b]))  # 等价于 concatenate(axis=1)
print(np.dstack([a, b]))  # 沿第三维堆叠（深度方向）

# np.column_stack —— 按列水平拼接一维数组为二维
print(np.column_stack([np.array([1, 2]), np.array([3, 4])]))
# [[1 3]
#  [2 4]]
```

### 2.2 分割（split / hsplit / vsplit）

```python
import numpy as np

arr = np.arange(12).reshape(3, 4)
print(arr)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# np.split —— 按位置分割
upper, lower = np.split(arr, 2, axis=0)  # 沿行方向分成两半
print("上半部分:")
print(upper)
# [[0 1 2 3]
#  [4 5 6 7]]

# 按指定索引位置分割（在第2列和第3列处切开）
left, middle, right = np.split(arr, [2, 3], axis=1)
print("中间列:", middle)  # [[2], [6], [10]]

# np.hsplit / np.vsplit —— 更直观的别名
left_half, right_half = np.hsplit(arr, 2)  # 水平切成两半
top_half, bottom_half = np.vsplit(arr, 2)  # 垂直切成两半（3行无法等分2份会报错）

# np.array_split —— 不等分时不会报错（自动调整）
parts = np.array_split(arr, 4, axis=0)  # 3行分4份
print(len(parts))  # 4份（2, 1, 1, 1行）
```

---

## 三、条件函数（np.where / np.select）

### 3.1 np.where — 三元条件

```python
import numpy as np

scores = np.array([45, 78, 92, 63, 55, 88, 30, 71])

# np.where(条件, 真值, 假值) —— 逐元素三目运算
result = np.where(scores >= 60, "及格", "不及格")
print(result)  # ['不及格' '及格' '及格' '及格' '不及格' '及格' '不及格' '及格']

# 只传条件参数 → 返回满足条件的索引
pass_idx = np.where(scores >= 60)
print("及格的同学索引:", pass_idx[0])  # [1 2 3 5 7]

# 二维数组的 np.where
arr = np.array([[1, -2, 3],
                [-4, 5, -6]])

# 将负数替换为 0（常见操作）
arr_nonneg = np.where(arr > 0, arr, 0)
print(arr_nonneg)
# [[1 0 3]
#  [0 5 0]]

# 等价写法：np.clip 也可以处理
print(np.clip(arr, 0, None))  # 同上，将值裁剪到 [0, ∞)
```

### 3.2 np.select — 多条件分支

```python
import numpy as np

scores = np.array([45, 78, 92, 63, 55, 88, 30, 71])

# 多条件分支（类似 if-elif-else）
conditions = [
    scores >= 90,   # 优秀
    scores >= 80,   # 良好
    scores >= 60,   # 及格
]
choices = ["优秀", "良好", "及格"]

grade = np.select(conditions, choices, default="不及格")
print(grade)
# ['不及格' '良好' '优秀' '及格' '不及格' '良好' '不及格' '及格']

# 注意：条件要按优先级排列（从严格到宽松）
# 如果先判断 scores >= 60，那么 92 分也会匹配"及格"
```

### 3.3 np.piecewise — 分段函数

```python
import numpy as np

x = np.linspace(-5, 5, 11)

# 分段函数：f(x) = x²   if x < 0
#               = 0     if x == 0
#               = x     if x > 0
y = np.piecewise(x,
    [x < 0, x == 0, x > 0],
    [lambda t: t**2, 0, lambda t: t]
)
print(x)
print(y)
# x:  [-5 -4 -3 -2 -1  0  1  2  3  4  5]
# y:  [25 16  9  4  1  0  1  2  3  4  5]
```

---

## 四、文件读写

### 4.1 二进制格式（.npy / .npz）

```python
import numpy as np

# 生成数据
arr = np.random.rand(1000, 100)

# 保存单个数组
np.save("temp_data.npy", arr)

# 加载
loaded = np.load("temp_data.npy")
print(np.array_equal(arr, loaded))  # True

# 保存多个数组到 .npz 压缩包
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
np.savez("temp_archive.npz", alpha=a, beta=b)

# 加载 .npz（返回 NpzFile 对象，按名称访问）
data = np.load("temp_archive.npz")
print(data['alpha'])  # [1 2 3]
print(data['beta'])   # [4 5 6]

# 压缩保存（更小文件）
np.savez_compressed("temp_archive_compressed.npz", alpha=a, beta=b)
```

### 4.2 文本格式（CSV 风格）

```python
import numpy as np

arr = np.array([[80, 90, 70],
                [60, 85, 95],
                [40, 75, 88]])

# 保存为文本文件
np.savetxt("temp_scores.csv", arr, delimiter=",", fmt="%d")
# 参数说明：delimiter=分隔符，fmt=格式（%d整数，%.2f两位小数）

# 加载文本文件
loaded = np.loadtxt("temp_scores.csv", delimiter=",", dtype=int)
print(loaded)

# 带表头
np.savetxt("temp_scores_header.csv", arr, delimiter=",", fmt="%d",
           header="语文,数学,英语", comments="")

# 跳过表头加载
loaded2 = np.genfromtxt("temp_scores_header.csv", delimiter=",", skip_header=1)
print(loaded2)
```

### 4.3 文件格式选择指南

| 格式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| `.npy` | 加载最快、保留 dtype | 二进制不可读 | 临时数据、中间结果 |
| `.npz` | 多数组打包、压缩 | 不可读 | 模型参数、批量数据 |
| `.csv` (txt) | 人类可读、跨语言 | 加载较慢、精度损失 | 数据交换、小数据 |
| `.hdf5` | 超大数据、分层存储 | 需额外库 | 科学数据、深度学习 |

---

## 五、高级索引技巧

### 5.1 np.ix_ — 构造开放网格索引

```python
import numpy as np

arr = np.arange(20).reshape(5, 4)
print(arr)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]
#  [12 13 14 15]
#  [16 17 18 19]]

# 需求：取第 0、2 行和第 1、3 列交叉的 4 个元素
rows = [0, 2]
cols = [1, 3]

# ❌ 错误写法（花式索引取的是两个独立子集）
print(arr[rows, cols])  # [1, 11] — 第(0,1)和(2,3)位置的元素

# ✅ 正确写法（取 2×2 的子矩阵）
idx = np.ix_(rows, cols)
print(arr[idx])
# [[ 1  3]
#  [ 9 11]]
```

### 5.2 np.newaxis — 增加维度

```python
import numpy as np

arr = np.array([1, 2, 3])  # 形状 (3,)

# 在第 0 维增加 → 变成行向量 (1, 3)
row_vec = arr[np.newaxis, :]
print(row_vec.shape)  # (1, 3)

# 在第 1 维增加 → 变成列向量 (3, 1)
col_vec = arr[:, np.newaxis]
print(col_vec.shape)  # (3, 1)

# None 和 np.newaxis 等价
print(arr[None, :].shape)  # (1, 3)

# 实际应用：用广播计算每对数据的距离
x = np.array([1, 2, 3])[:, np.newaxis]  # (3, 1)
y = np.array([10, 20, 30])[np.newaxis, :]  # (1, 3)
diff = y - x  # (3, 3) 广播
print(diff)
# [[ 9 19 29]
#  [ 8 18 28]
#  [ 7 17 27]]
```

### 5.3 搜索与查找

```python
import numpy as np

arr = np.array([30, 10, 50, 20, 40, 10, 50])

# 搜索 —— 满足条件的索引
idx = np.searchsorted(np.sort(arr), 25)
print("25 应该插入的位置:", idx)  # 在排序数组中的位置

# 唯一值与计数
values, counts = np.unique(arr, return_counts=True)
print("值:", values)      # [10 20 30 40 50]
print("次数:", counts)    # [ 2  1  1  1  2]

# 找到第一个满足条件的位置
print(np.argmax(arr >= 40))  # 2  （50 是第一个 >= 40 的）
print(np.argmax(arr > 100))  # 0  ⚠️ 全都不满足时返回 0！
```

> **⚠️ 常见坑**：`np.argmax` 在所有值都为 False 时会返回 0（第一个最大值的索引），这不是"没找到"的意思。更安全的做法是先检查是否有满足条件的值：
> ```python
> mask = arr > 100
> if mask.any():
>     print(np.argmax(mask))
> else:
>     print("没找到")
> ```

---

## 六、结构化数组（Structured Arrays）

结构化数组类似 C 语言的 struct，允许在一个数组中存储不同类型的"字段"，适合表格数据。

```python
import numpy as np

# 定义结构化数据类型
dtype = [
    ('name', 'U10'),    # 最多10个字符的 Unicode 字符串
    ('age', 'i4'),       # 4字节整数
    ('height', 'f4'),    # 4字节浮点
    ('passed', '?'),     # 布尔类型
]

# 创建结构化数组
students = np.array([
    ('张三', 20, 1.75, True),
    ('李四', 22, 1.68, True),
    ('王五', 19, 1.82, False),
], dtype=dtype)

print(students)
# [('张三', 20, 1.75,  True) ('李四', 22, 1.68,  True) ('王五', 19, 1.82, False)]

# 按字段名访问
print(students['name'])       # ['张三' '李四' '王五']
print(students['age'])        # [20 22 19]
print(students['height'].mean())  # 1.75

# 按条件筛选
print(students[students['age'] > 20])
# [('李四', 22, 1.68, True)]

# 排序（按指定字段）
print(np.sort(students, order='age'))
# [('王五', 19, 1.82, False) ('张三', 20, 1.75,  True) ('李四', 22, 1.68,  True)]
```

> **💡 小贴士**：结构化数组比纯 Python list of dict 更节省内存，适合大规模表格数据。但对于复杂的表格操作，pandas 的 DataFrame 更方便——结构化数组更多用于底层性能敏感场景。

---

## 七、性能优化技巧

### 7.1 向量化 vs 循环

```python
import numpy as np
import time

data = np.random.rand(1000000)

# ❌ 慢：Python 循环
start = time.time()
result_loop = [max(0, x - 0.5) for x in data]
print(f"循环耗时: {time.time() - start:.4f} 秒")  # 约 0.15 秒

# ✅ 快：NumPy 向量化
start = time.time()
result_vec = np.maximum(data - 0.5, 0)
print(f"向量化耗时: {time.time() - start:.4f} 秒")  # 约 0.005 秒
```

### 7.2 常用性能优化策略

```python
import numpy as np

# 1. 预分配数组（避免不断扩容）
size = 10000
# ❌ 慢：动态 append
result = []
for i in range(size):
    result.append(i ** 2)

# ✅ 快：预分配后填充
result = np.empty(size)
for i in range(size):
    result[i] = i ** 2

# ✅ 更快：纯向量化
result = np.arange(size) ** 2

# 2. 就地操作（节省内存）
arr = np.random.rand(1000, 1000)

# ❌ 创建新数组
arr2 = arr * 2  # 新数组

# ✅ 就地修改
arr *= 2  # 不创建新数组，直接修改 arr

# 3. 使用 dtype 减少内存
# 大数组用 float32 而不是 float64（精度够用时）
arr_f32 = np.zeros(10000000, dtype=np.float32)  # 约 40 MB
arr_f64 = np.zeros(10000000, dtype=np.float64)  # 约 80 MB

# 4. 布尔掩码比 where 更快
large = np.random.rand(1000000)
# 两种写法效果相同，np.where 更通用
filtered1 = large[large > 0.5]  # 布尔索引
filtered2 = np.where(large > 0.5, large, np.nan)  # 保留原形状

# 5. 避免在循环中不断调用 NumPy 函数
# ❌ 每次循环都做一次函数调用
# for i in range(1000):
#     result[i] = np.sqrt(data[i])

# ✅ 一次性向量化
result = np.sqrt(data[:1000])
```

### 7.3 内存布局（C order vs Fortran order）

```python
import numpy as np

# C order（行优先，默认）—— 按行存储
arr_c = np.array([[1, 2, 3],
                   [4, 5, 6]], order='C')
# 内存中: 1, 2, 3, 4, 5, 6

# Fortran order（列优先）—— 按列存储
arr_f = np.array([[1, 2, 3],
                   [4, 5, 6]], order='F')
# 内存中: 1, 4, 2, 5, 3, 6

# 判断内存布局
print(arr_c.flags['C_CONTIGUOUS'])  # True  按行连续
print(arr_f.flags['F_CONTIGUOUS'])  # True  按列连续

# 建议：按行遍历用 C order，按列遍历用 F order
# 通常默认 C order 就好，不需要特别关心
```

---

## 八、综合实战：图像数据基础处理

用 NumPy 模拟简单的图像处理操作（真实图像处理通常用 Pillow/OpenCV，但底层都是 NumPy 数组）。

```python
import numpy as np

# 模拟一张 6×6 的灰度图像（值 0-255）
np.random.seed(42)
image = np.random.randint(50, 200, size=(6, 6), dtype=np.uint8)
print("原始图像:")
print(image)

# 1. 亮度调整：增加 50（裁剪到 0-255 范围）
brighter = np.clip(image.astype(np.int16) + 50, 0, 255).astype(np.uint8)
print("\n调亮后:")
print(brighter)

# 2. 翻转操作
flipped_h = np.fliplr(image)  # 水平翻转（左右镜像）
flipped_v = np.flipud(image)  # 垂直翻转（上下镜像）
print("\n水平翻转:")
print(flipped_h)

# 3. 旋转 90 度
rotated = np.rot90(image, k=1)  # k=1 逆时针 90°
print("\n旋转 90°:")
print(rotated)

# 4. 简单滤波（3×3 均值模糊）
kernel_size = 3
pad = kernel_size // 2
padded = np.pad(image, pad, mode='edge')  # 边缘填充
blurred = np.zeros_like(image, dtype=np.float64)

for i in range(image.shape[0]):
    for j in range(image.shape[1]):
        window = padded[i:i+kernel_size, j:j+kernel_size]
        blurred[i, j] = window.mean()

print("\n模糊后:")
print(blurred.astype(np.uint8))

# 5. 阈值二值化
threshold = 120
binary = np.where(image > threshold, 255, 0).astype(np.uint8)
print("\n二值化（阈值=120）:")
print(binary)

# 6. 统计信息
print(f"\n图像统计:")
print(f"  大小: {image.shape}")
print(f"  平均亮度: {image.mean():.1f}")
print(f"  最暗: {image.min()}, 最亮: {image.max()}")
print(f"  dtype: {image.dtype}")
```

---

## 练习题

### 练习1：矩阵运算

创建两个 3×3 随机矩阵 A 和 B，计算：
- 矩阵乘法 C = A × B
- C 的行列式和迹
- 验证 A × A⁻¹ ≈ 单位矩阵

```python
# 参考答案
import numpy as np

np.random.seed(42)
A = np.random.randint(1, 10, (3, 3)).astype(float)
B = np.random.randint(1, 10, (3, 3)).astype(float)

C = A @ B
print("矩阵乘积 C:")
print(C)
print(f"C 的行列式: {np.linalg.det(C):.2f}")
print(f"C 的迹: {np.trace(C):.2f}")

identity = A @ np.linalg.inv(A)
print("A × A⁻¹:")
print(np.round(identity, 10))
```

### 练习2：np.where 条件处理

给定一组学生的成绩（数组），用 `np.where` 和 `np.select` 实现：
- 低于 60 分标记为"不及格"
- 60-79 分标记为"中等"
- 80-89 分标记为"良好"
- 90 分及以上标记为"优秀"

```python
# 参考答案
import numpy as np

scores = np.array([45, 78, 92, 63, 55, 88, 30, 71, 95, 82])

conditions = [scores >= 90, scores >= 80, scores >= 60]
choices = ["优秀", "良好", "中等"]
grades = np.select(conditions, choices, default="不及格")
print("成绩:", scores)
print("等级:", grades)
```

### 练习3：数组拼接与排序

创建两组学生数据（姓名和分数），拼接后按分数降序排列，输出排名前三的学生。

```python
# 参考答案
import numpy as np

# 第一组
names1 = np.array(['Alice', 'Bob', 'Charlie'])
scores1 = np.array([85, 92, 78])

# 第二组
names2 = np.array(['David', 'Eve'])
scores2 = np.array([95, 88])

# 拼接
all_names = np.concatenate([names1, names2])
all_scores = np.concatenate([scores1, scores2])

# 按分数降序排名
ranking = np.argsort(-all_scores)

print("前三名:")
for rank, idx in enumerate(ranking[:3], 1):
    print(f"  第{rank}名: {all_names[idx]}（{all_scores[idx]}分）")
```

---

## 常见问题 FAQ

### Q1：np.dot、np.matmul 和 @ 有什么区别？

- `@` 运算符和 `np.matmul` 是等价的，专门做矩阵乘法
- `np.dot` 在二维数组上也等价于矩阵乘法，但在高维数组上行为不同（沿最后维做点积）
- 日常使用推荐 `@`，最直观

### Q2：solve 和 inv + @ 哪个好？

`np.linalg.solve(A, b)` 比先求逆再相乘（`inv(A) @ b`）更快且数值更稳定。在科学计算中，应尽量避免显式求逆矩阵。

### Q3：np.where 返回值和条件索引有什么区别？

`np.where(condition)` 只传一个参数时，返回满足条件的索引元组（用于高级索引）。`np.where(condition, x, y)` 传三个参数时，返回按条件选值的新数组。两个功能完全不同！

### Q4：结构化数组和 pandas DataFrame 怎么选？

- 数据量小、操作简单 → 结构化数组即可
- 需要分组、透视、缺失值处理等 → 用 pandas DataFrame
- 结构化数组更适合底层高性能场景（如 C 扩展交互）

### Q5：什么时候需要手动指定 dtype？

默认 NumPy 会自动推断类型（整数 → int64，浮点 → float64）。但当数据量大时，降级用 float32/int32/int16 可以**减半**内存占用。布尔运算结果用 bool 类型。必要时显式指定。

---

## 免费学习资源

- **NumPy 官方线性代数文档**：https://numpy.org/doc/stable/reference/routines.linalg.html — 所有 linalg 函数详解
- **菜鸟教程 NumPy 进阶**：https://www.runoob.com/numpy/numpy-linear-algebra.html — 线性代数部分
- **SciPy Lecture Notes**：https://scipy-lectures.org/ — NumPy + SciPy 完整教程（英文）
- **NumPy 100 练习**：https://github.com/rougier/numpy-100 — 100 道 NumPy 题目，从入门到精通
- **《Python 数据科学手册》**：https://jakevdp.github.io/PythonDataScienceHandbook/ — 免费在线书，NumPy 章节非常好

---

## 今日小结

| 概念 | 要点 |
|------|------|
| 线性代数 | `@` 矩阵乘法、`det` 行列式、`inv` 逆矩阵、`solve` 解方程组、`eig` 特征值 |
| 拼接分割 | `concatenate`、`stack`、`vstack/hstack`、`split`、`hsplit/vsplit` |
| 条件函数 | `np.where` 三元运算、`np.select` 多分支、`np.piecewise` 分段函数 |
| 文件读写 | `save/load`（.npy）、`savez`（.npz）、`savetxt/loadtxt`（文本） |
| 高级索引 | `np.ix_` 网格索引、`np.newaxis` 增维、`searchsorted`、`unique` |
| 结构化数组 | 多字段异构类型，类似 C struct，适合表格数据 |
| 性能优化 | 向量化优于循环、预分配、就地操作、选合适 dtype |

> **下期预告**：Day 45 — 综合数据处理项目，将综合运用 pandas + NumPy + matplotlib 完成一个完整的数据分析项目，检验前几天的学习成果。
