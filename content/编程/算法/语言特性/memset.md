# memset 详解

`memset` 是 C/C++ 中非常高效的内存初始化函数，在算法竞赛中被广泛使用。它定义在 `<cstring>` (C++) 或 `<string.h>` (C) 头文件中。

## 1. 函数原型与参数

```cpp
void *memset(void *ptr, int value, size_t num);
```

- **`ptr`**: 指向要填充的内存块的指针（通常是数组名）。
- **`value`**: 要设置的值。
    - **注意**：虽然参数类型是 `int`，但 `memset` 是**按字节 (Byte)** 进行赋值的。它取 `value` 的低 8 位，将内存块中的每个字节都设置为这个值。
- **`num`**: 要设置的字节数。通常使用 `sizeof(数组名)` 或 `sizeof(类型) * 数量`。

## 2. 常见用法

### 2.1 初始化为 0 或 -1
这是最安全且最常用的两种情况，因为 0 的二进制是全 0，-1 的二进制是全 1。
```cpp
int arr[100];
memset(arr, 0, sizeof(arr));  // 所有元素变为 0
memset(arr, -1, sizeof(arr)); // 所有元素变为 -1
```

### 2.2 初始化为"无穷大"
在图论（如 Dijkstra, Floyd）或 DP 中，常需要将数组初始化为一个很大的值。
- **`0x3f`**: 推荐使用。
    ```cpp
    memset(arr, 0x3f, sizeof(arr)); 
    ```
    - 每个 int 变为 `0x3f3f3f3f` (十进制约 $1.06 \times 10^9$)。
    - **优点**: 两个 `0x3f3f3f3f` 相加不会溢出 `int` (约 $2 \times 10^9 < 2.14 \times 10^9$)，非常适合做“无穷大”。
- **`0x7f`**:
    ```cpp
    memset(arr, 0x7f, sizeof(arr));
    ```
    - 每个 int 变为 `0x7f7f7f7f` (十进制约 $2.13 \times 10^9$)。
    - **缺点**: 两个这样的值相加会溢出 `int`。

### 2.3 易错陷阱
不要尝试用 `memset` 将 `int` 数组初始化为 1。
```cpp
memset(arr, 1, sizeof(arr)); 
// 结果：每个元素变为 0x01010101 (十进制 16843009)，而不是 1。
```

## 3. 算法竞赛中的作用

### 3.1 多组测试数据 (Multi-testcase)
竞赛题目常包含 $T$ 组测试数据。在处理每组数据前，必须清空上一组的状态。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int MAXN = 1005;
int a[MAXN];

void solve() {
    // 【重要】每组数据开始前重置数组
    memset(a, 0, sizeof(a));
    
    // ... 执行当前组的逻辑 ...
}

int main() {
    int T;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```
> **优化提示**: 如果 `MAXN` 很大（如 $10^5$）且 $T$ 很大，但每组数据实际只用到很少一部分（如 $n$），直接 `memset` 整个大数组可能会导致 **TLE (超时)**。此时应该用 `for` 循环只清空 `0` 到 `n` 的部分。

### 3.2 局部变量与变长数组 (VLA) 的初始化
在函数内部定义的数组（栈上分配），默认值是**随机的垃圾值**，必须手动初始化。

特别是对于 GCC 编译器支持的**变长数组 (Variable Length Array, VLA)**（即数组长度由变量 `n` 决定），标准 C++ 不允许使用 `{0}` 初始化列表。此时 `memset` 是唯一的救星。

```cpp
void func(int n) {
    // int arr[n] = {0}; // ❌ 错误！变长数组不能这样初始化 (编译报错)
    
    int arr[n];          // GCC 扩展支持变长数组
    memset(arr, 0, sizeof(arr)); // ✅ 正确！使用 memset 清零
    
    // 现在可以使用 arr 了
}
```
*注：虽然标准 C++ (C++11/14/17/20) 不支持 VLA，但在算法竞赛常用的 GCC 环境下是默认支持的。*
