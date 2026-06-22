# std::map 使用笔记（实战版）

> 适用语言标准：C++11 起；部分接口标注 C++17/20

## 目录
- 一、概念与性质
- 二、头文件与别名
- 三、迭代与容量
- 四、元素访问（读/写/不插入）
- 五、插入与删除（修改器）
- 六、有序查找与区间操作
- 七、迭代器工具（prev/next 等）
- 八、观察器与比较器（含透明比较）
- 九、复杂度与性能要点
- 十、迭代器/引用失效规则
- 十一、常见坑与最佳实践
- 十二、常用代码片段（可直接粘贴）
- 十三、与 unordered_map 的取舍

---

## 一、概念与性质
- 底层：平衡二叉搜索树（通常红黑树），键按比较器有序（默认升序）。
- 键唯一（需要重复键用 std::multimap）。
- 遍历顺序恒为升序。
- 查找/插入/删除平均与最坏都是 O(log N)。

## 二、头文件与别名
```cpp
#include <map>
using std::map;
```

## 三、迭代与容量
- 大小与状态
  - `size()` O(1)，`empty()` O(1)，`clear()` O(N)
- 正向/反向/常量迭代
  - `begin()/end()`，`rbegin()/rend()`，`cbegin()/cend()`
- 遍历（C++17）
```cpp
for (auto& [k, v] : mp) { /* ... */ }
```

## 四、元素访问（读/写/不插入）
- `operator[](const Key&) -> T&`
  - 不存在会插入 `{key, T()}` 后返回引用（“读也插入”的典型坑）
  - O(log N)
- `at(const Key&) -> T& / const T&`
  - 不存在抛 `std::out_of_range`，不会插入
  - O(log N)
- `find(const Key&) -> iterator`
  - 不存在返回 `end()`，不会插入
  - O(log N)
- `contains(const Key&) -> bool`（C++20）
  - O(log N)

## 五、插入与删除（修改器）
- 插入
  - `insert({k, v})`：已存在则不变；O(log N)
  - `insert(hint, {k, v})`：合理 hint 可均摊近 O(1)，最坏 O(log N)
  - `emplace(args...)`：就地构造，避免拷贝
  - `try_emplace(k, args...)`（C++17）：只有键不存在才构造 value
  - `insert_or_assign(k, v)`（C++17）：存在则赋值，不存在则插入
- 删除
  - `erase(it)` -> 返回后继迭代器（C++11 起）；O(1) 调度 + O(log N) 结构调整
  - `erase(key)` -> 返回删除个数（0/1）；O(log N)
  - `erase(first, last)`：O(k + log N)
- 合并/节点操作（C++17）
  - `merge(other)`：移动不冲突节点，低代价
  - `extract(key/it)`：取出节点句柄，修改 key 后可插回

## 六、有序查找与区间操作
- `lower_bound(key)`：第一个 `>= key`
- `upper_bound(key)`：第一个 `> key`
- `equal_range(key)`：等价于 `{lower_bound, upper_bound}`

常见模式：
- 后继（>=x）：`auto it = mp.lower_bound(x);`
- 真后继（>x）：`auto it = mp.upper_bound(x);`
- 前驱（<x）：
```cpp
auto it = mp.lower_bound(x);
if (it != mp.begin()) { auto pred = std::prev(it); /* pred 是前驱 */ }
```
- 区间遍历 [L, R]：
```cpp
for (auto it = mp.lower_bound(L); it != mp.end() && it->first <= R; ++it) {
    // 处理 it
}
```

## 七、迭代器工具（prev/next 等）
- 头文件：`#include <iterator>`
- `std::next(it, n=1)`：向后 n 步；对 `end()` 调用 `next` 或越界是未定义行为（UB）
- `std::prev(it, n=1)`：向前 n 步；对 `begin()` 调用 `prev` 是 UB
- 使用前务必做边界判断：
```cpp
if (it != mp.begin()) { auto p = std::prev(it); /* 安全 */ }
if (it != mp.end())   { auto n = std::next(it); /* 安全 */ }
```

## 八、观察器与比较器（含透明比较）
- `key_comp()` / `value_comp()`：返回比较器对象
- 自定义比较器：
```cpp
struct Cmp { bool operator()(const Key& a, const Key& b) const { return a < b; } };
std::map<Key, T, Cmp> mp;
```
- 透明比较（异构查找，减少临时对象，C++14/17）：
```cpp
struct TransparentLess {
  using is_transparent = void;
  bool operator()(std::string_view a, std::string_view b) const { return a < b; }
  bool operator()(const std::string& a, std::string_view b) const { return a < b; }
  bool operator()(std::string_view a, const std::string& b) const { return a < b; }
};
std::map<std::string, int, TransparentLess> mp;
auto it = mp.find(std::string_view("key")); // 不构造 std::string
```

## 九、复杂度与性能要点
- 查找/插入/删除：O(log N)
- 有序遍历：O(N)
- 带合理 hint 的单调插入：均摊近 O(1)
- map 有序但指针/节点开销较大；若只需等值查找且大量操作，考虑 `unordered_map`

## 十、迭代器/引用失效规则
- 插入：不使其它迭代器失效
- 删除：仅被删元素的迭代器/引用失效
- 修改 value（`it->second`）安全；key（`it->first`）为 const，不能改
- 如需改 key：`auto nh = mp.extract(it); nh.key() = new_key; mp.insert(std::move(nh));`（C++17）

## 十一、常见坑与最佳实践
- 坑1：`mp[x]` 会隐式插入默认值（即使你只是“看一眼”）→ 用 `find/contains/at`
- 坑2：`prev(begin())` / `next(end())` 是 UB → 先判断边界
- 坑3：比较不同容器的迭代器是 UB
- 坑4：遍历中删除：用 `it = mp.erase(it)` 的返回值续遍历
- 建议：
  - 需要“前驱/后继/区间”的问题优先选 `map`
  - 高并发/大量散列访问用 `unordered_map`

## 十二、常用代码片段（可直接粘贴）

### 1) 前驱与后继（安全版本）
```cpp
auto it = mp.lower_bound(x); // 第一个 >= x

// 前驱 < x
if (it != mp.begin()) {
    auto pred = std::prev(it);
    // 使用 pred
}

// 后继 >= x
if (it != mp.end()) {
    auto succ_ge = it;
    // 使用 succ_ge
}

// 真后继 > x
auto it2 = mp.upper_bound(x);
if (it2 != mp.end()) {
    auto succ_gt = it2;
    // 使用 succ_gt
}
```

### 2) 最近键（离 x 最近）
```cpp
auto it = mp.lower_bound(x);
bool has = false;
int best = 0;
if (it != mp.end()) { best = it->first; has = true; }
if (it != mp.begin()) {
    auto p = std::prev(it);
    if (!has || std::abs(p->first - x) <= std::abs(best - x)) { best = p->first; has = true; }
}
if (has) { /* 使用 best */ }
```

### 3) 存在性与读取（不插入）
```cpp
if (auto it = mp.find(k); it != mp.end()) {
    auto& v = it->second; // 安全读取
}
// 或（C++20）
if (mp.contains(k)) { /* ... */ }
```

### 4) 高效插入与赋值（C++17）
```cpp
mp.try_emplace(k, arg1, arg2); // 仅在不存在时构造 value
mp.insert_or_assign(k, v);     // 存在则赋值，不存在则插入
```

### 5) 区间遍历 [L, R]
```cpp
for (auto it = mp.lower_bound(L); it != mp.end() && it->first <= R; ++it) {
    // ...
}
```

### 6) 单调键插入（hint 提速）
```cpp
auto hint = mp.end();
for (auto& [k, v] : data_sorted_by_k) {
    hint = mp.insert(hint, {k, v}); // 均摊近 O(1)
}
```

### 7) 节点修改键（C++17）
```cpp
if (auto it = mp.find(old_key); it != mp.end()) {
    auto nh = mp.extract(it);
    nh.key() = new_key;
    mp.insert(std::move(nh));
}
```

### 8) 滑动窗口里找近邻（避免隐式插入）
```cpp
// 删除窗口外元素时避免 mp[x]：要用 find
if (auto it = mp.find(x); it != mp.end()) {
    if (--it->second == 0) mp.erase(it);
}

// 查邻居时使用 lower_bound / upper_bound 并做边界检查
auto it = mp.lower_bound(x);
if (it != mp.end() && it->first - x <= valueDiff) return true;
if (it != mp.begin()) {
    auto p = std::prev(it);
    if (x - p->first <= valueDiff) return true;
}
```

## 十三、与 unordered_map 的取舍
- 选 map 的场景：需要“有序”“前驱/后继”“范围/区间查询”“最近邻”等
- 选 unordered_map 的场景：只需等值查找/插入/删除，且更关注均摊 O(1) 与更低常数

---

需要我把你当前的包含滑动窗口逻辑的函数改成“无隐式插入、prev/next 安全”的版本并替换到你的 d:\desktop\code\test\work.cpp 吗？我可以直接修改、运行并给你输出结果。
        