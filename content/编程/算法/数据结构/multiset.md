# multiset 详解

## 概述
`multiset` 是 C++ STL 中的关联容器，允许存储重复元素，并且元素会自动按照升序排列。

## 主要特点

### 1. 允许重复元素
- 与 `set` 不同，`multiset` 可以存储多个相同的元素
- 相同元素会被存储多次

### 2. 自动排序
- 元素插入后会自动按照升序排列
- 默认使用 `<` 操作符进行比较
- 可以自定义比较函数

### 3. 基于红黑树实现
- 内部使用平衡二叉搜索树（红黑树）
- 保证了插入、删除、查找的时间复杂度为 O(log n)

## 头文件和声明
```cpp
#include <set>
using namespace std;

multiset<int> ms;                    // 默认升序
multiset<int, greater<int>> ms_desc; // 降序
```

## 常用方法

### 插入操作
```cpp
// insert() - 插入元素
ms.insert(5);
ms.insert(3);
ms.insert(5);  // 可以插入重复元素

// emplace() - 原地构造插入
ms.emplace(7);
```

### 删除操作
```cpp
// erase() - 删除元素
ms.erase(5);        // 删除所有值为5的元素
ms.erase(ms.find(3)); // 删除迭代器指向的单个元素,该元素是随机选择的,因为相同元素之间没有区别

// clear() - 清空容器
ms.clear();
```

### 查找操作
```cpp
// find() - 查找元素
auto it = ms.find(5);
if (it != ms.end()) {
    cout << "找到元素: " << *it << endl;
}

// count() - 统计元素个数
int cnt = ms.count(5);  // 返回值为5的元素个数

// lower_bound() - 返回第一个不小于给定值的迭代器
auto lower = ms.lower_bound(5);

// upper_bound() - 返回第一个大于给定值的迭代器
auto upper = ms.upper_bound(5);

// equal_range() - 返回等于给定值的元素范围
auto range = ms.equal_range(5);
```

### 容量相关
```cpp
// size() - 返回元素个数
cout << ms.size() << endl;

// empty() - 判断是否为空
if (ms.empty()) {
    cout << "容器为空" << endl;
}

// max_size() - 返回最大容量
cout << ms.max_size() << endl;
```

### 迭代器
```cpp
// begin() / end() - 正向迭代器
for (auto it = ms.begin(); it != ms.end(); ++it) {
    cout << *it << " ";
}

// rbegin() / rend() - 反向迭代器
for (auto it = ms.rbegin(); it != ms.rend(); ++it) {
    cout << *it << " ";
}

// 范围for循环
for (const auto& elem : ms) {
    cout << elem << " ";
}
```

## multiset vs set 区别对比

| 特性 | multiset | set |
|------|----------|-----|
| **重复元素** | ✅ 允许重复 | ❌ 不允许重复 |
| **排序** | ✅ 自动排序 | ✅ 自动排序 |
| **插入** | 总是成功 | 重复元素插入失败 |
| **删除** | erase(value)删除所有相同元素 | erase(value)删除唯一元素 |
| **查找** | find()返回任意一个匹配元素 | find()返回唯一匹配元素 |
| **计数** | count()可能>1 | count()只能是0或1 |
| **内存占用** | 相对较大（存储重复元素） | 相对较小 |

## 使用场景

### multiset 适用场景：
- 需要存储重复元素并保持有序
- 需要统计元素出现次数
- 实现优先队列的变种
- 处理有重复值的排序问题

### set 适用场景：
- 需要去重并保持有序
- 快速判断元素是否存在
- 集合运算（并集、交集等）

## 实际应用示例

### 1. 统计元素频次
```cpp
multiset<int> ms = {1, 2, 2, 3, 3, 3};
for (int val : {1, 2, 3}) {
    cout << val << " 出现 " << ms.count(val) << " 次" << endl;
}
```

### 2. 维护有序的重复数据
```cpp
multiset<int> scores = {85, 92, 78, 92, 88, 85};
cout << "最高分: " << *scores.rbegin() << endl;
cout << "最低分: " << *scores.begin() << endl;
```

### 3. 滑动窗口中位数
```cpp
// 使用两个multiset维护较小和较大的一半元素
multiset<int> small, large;
// ... 实现滑动窗口中位数逻辑
```

## 注意事项

1. **删除操作要小心**：`erase(value)` 会删除所有相同的元素
2. **迭代器失效**：删除元素后，指向被删除元素的迭代器会失效
3. **自定义比较函数**：确保比较函数满足严格弱序关系
4. **性能考虑**：虽然操作复杂度是 O(log n)，但常数因子比 unordered_set 大

## 时间复杂度总结

| 操作 | 时间复杂度 |
|------|------------|
| 插入 | O(log n) |
| 删除 | O(log n) |
| 查找 | O(log n) |
| 遍历 | O(n) |
| 计数 | O(log n + k)，k为重复元素个数 |