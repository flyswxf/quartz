# lower_bound 与 upper_bound 用法总结

`lower_bound` 和 `upper_bound` 是 C++ 标准库中的二分查找函数，定义在 `<algorithm>` 头文件中。它们能在有序区间内以 $O(\log N)$ 的时间复杂度进行查找。

## 核心用法表格

以下总结了不同排序规则下的查找目标及对应的函数用法：

| 数组排序方式 | 查找目标 | 函数调用 | 返回值 | 示例结论 (针对数组 `[1, 2, 2, 3]`) |
| :--- | :--- | :--- | :--- | :--- |
| **升序 (从小到大)** | 第一个 $\ge x$ 的元素 | `lower_bound(begin, end, x)` | 迭代器 | 找 $\ge 2$，返回第一个 `2` 的迭代器 |
| **升序 (从小到大)** | 第一个 $> x$ 的元素 | `upper_bound(begin, end, x)` | 迭代器 | 找 $> 2$，返回 `3` 的迭代器 |
| **降序 (从大到小)** | 第一个 $\le x$ 的元素 | `lower_bound(begin, end, x, greater<int>())` | 迭代器 | (针对 `[3, 2, 2, 1]`) 找 $\le 2$，返回第一个 `2` |
| **降序 (从大到小)** | 第一个 $< x$ 的元素 | `upper_bound(begin, end, x, greater<int>())` | 迭代器 | (针对 `[3, 2, 2, 1]`) 找 $< 2$，返回 `1` |

> **注意**：如果找不到符合条件的元素，函数均返回尾迭代器（即 `end()`）。
> **核心易错点**：使用 `greater<int>()` 时，**原数组必须已经是降序排列**，否则会导致未定义行为。

---

## 针对 vector 或数组的用法示例

对于支持随机访问的容器（如 `vector`、数组），可以直接使用全局的 `std::lower_bound` 和 `std::upper_bound`。因为支持随机访问，可以通过减去 `begin()` 快速获取索引下标，此操作的时间复杂度为 $O(1)$。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
    // 1. 升序数组
    vector<int> v_asc = {1, 2, 4, 4, 5};
    
    // 找第一个 >= 4 的数
    auto it1 = lower_bound(v_asc.begin(), v_asc.end(), 4);
    int index1 = it1 - v_asc.begin(); // index1 = 2
    
    // 找第一个 > 4 的数
    auto it2 = upper_bound(v_asc.begin(), v_asc.end(), 4);
    int index2 = it2 - v_asc.begin(); // index2 = 4

    // 2. 降序数组
    vector<int> v_desc = {5, 4, 4, 2, 1};
    
    // 找第一个 <= 4 的数
    auto it3 = lower_bound(v_desc.begin(), v_desc.end(), 4, greater<int>());
    int index3 = it3 - v_desc.begin(); // index3 = 1
    
    return 0;
}
```

---

## 针对 set / map 等关联容器的用法

对于 `set`、`map`、`multiset` 等不支持随机访问（底层为红黑树）的容器，**必须使用容器自身的成员函数**。如果对它们使用全局的 `std::lower_bound`，虽然也能编译通过，但因为迭代器只能逐个移动（`std::advance`），时间复杂度会从 $O(\log N)$ 严重退化为 $O(N)$。

```cpp
#include <iostream>
#include <set>

using namespace std;

int main() {
    set<int> s = {1, 2, 4, 5};
    
    // 正确用法：调用 set 的成员函数，时间复杂度 O(log N)
    auto it_correct = s.lower_bound(4); 
    
    // 错误用法：调用全局算法，时间复杂度退化为 O(N)
    // auto it_wrong = lower_bound(s.begin(), s.end(), 4); 
    
    if (it_correct != s.end()) {
        cout << "找到了 >= 4 的元素: " << *it_correct << endl;
    }
    
    return 0;
}
```

> **提示**：关联容器的迭代器不支持减法运算（即不能使用 `it - s.begin()`），因此无法直接获取元素的下标（逻辑上也没有下标概念）。如果一定要算距离，可以使用 `std::distance(s.begin(), it)`，但这需要 $O(N)$ 的时间复杂度。
