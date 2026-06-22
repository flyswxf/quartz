- 目标：在数组 `A[p..q]` 中找到第 `i` 小的元素。
- 思想：随机选主元，原地划分，只在更小的一侧继续查找。

#### 时间复杂度
$$T(n)= \Theta(n)$$
- 最坏 $O(n^2)$
- 期望递归深度 `O(log n)`，最坏 `O(n)`
- 额外空间 `O(1)`（原地分区）。

## 伪代码
```
Rand-Select(A, p, q, i)
    if p = q then return A[p]
    r ← Rand-Partition(A, p, q)      // 随机主元，返回最终位置 r
    k ← r - p + 1                     // A[r] 在 A[p..q] 中的位次
    if i = k then return A[r]
    if i < k then
        return Rand-Select(A, p, r-1, i)
    else
        return Rand-Select(A, r+1, q, i - k)
```

### Rand-Partition（随机划分）
- 随机选择一个 `pivot = A[t]`。
- 将区间 `A[p..q]` 原地重排为 `<= pivot | pivot | >= pivot`。
- 返回 `pivot` 的最终下标 `r`。

#### 时间复杂度
$$T(n)= \Theta(n)$$
其中`n`为数组长度
```python
def partition(a, l, r):
    pivot = a[l]
    i, j = l, r
    while True:
        while a[i] < pivot: i += 1
        while a[j] > pivot: j -= 1
        if i >= j:
            return j
        a[i], a[j] = a[j], a[i]
        i += 1
        j -= 1
```

## 优化算法: 确定性选择（Median-of-Medians, BFPRT）
- 目标：在最坏情况下也保证线性复杂度的第 `i` 小选择。
- 思路：将元素分组求“中位数的中位数”作主元，确保每次划分后至多舍弃常数比例的元素，从而得到线性递归。

### 算法步骤
1. 将 `n` 个元素分成每组 5 个。
2. 对每组 5 个元素求中位数，得到约 `⌊n/5⌋` 个中位数(如果最后一组不足5个, 则不算该组的中位数)。
3. **递归地**在这些中位数中选择它们的中位数 `x`，作为主元。
4. 以 `x` 为主元对原数组做分区，令 `k = rank(x)`。
5. 若 `i = k` 返回 `x`；否则在相应一侧递归选择。

### 伪代码
```text
Det-Select(A, p, q, i):
  if p = q then return A[p]
  x <- MedianOfMedians(A, p, q)        // 确定性主元
  r <- PartitionByPivot(A, p, q, x)
  k <- r - p + 1                       // 以下代码与随机选择一样
  if i = k then return A[r]
  if i < k then
    return Det-Select(A, p, r-1, i)
  else
    return Det-Select(A, r+1, q, i - k)

MedianOfMedians(A, p, q):
  将 A[p..q] 划分为若干组，每组 5 个
  对每组求中位数，形成数组 M
  返回 Det-Select(M, 1, |M|, ceil(|M|/2))
```

### 关键性质
- 所选主元 `x` 至少不小于一半组的中位数，且不大于另一半组的中位数。
- 可证明每次划分后，较差的一侧元素数量至多为 `3n/4`（更紧的上界为 `7n/10`），保证规模按常数比例缩减。

### 复杂度推导
分组与求组内中位数、分区等线性工作记为 $\Theta(n)$；递归选择组中位数耗时 $T(n/5)$；进入较差一侧递归规模至多 $\frac{3}{4}n$。于是有递推：
$$
T(n) = T\!\left(\tfrac{n}{5}\right) + T\!\left(\tfrac{3}{4}n\right) + \Theta(n)
$$
用代换法证明线性界：设对充分大的 $n$，有 $T(n) \le c n$，则
$$
T(n) \le \tfrac{1}{5}cn + \tfrac{3}{4}cn + a n
= \Big(\tfrac{19}{20}c\Big) n + a n
\le c n
$$
当选择常数 $c \ge 20a$ 即成立。因此
$$
T(n) = \Theta(n)
$$

### 说明与实践
- 常数因子较大，实际工程中常用随机选择以获得更好的常数性能；但 BFPRT 提供了严格的最坏线性时间保证。
