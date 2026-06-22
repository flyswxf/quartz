# KMP算法核心理解笔记

## 1. 核心思想
KMP算法的本质是：**利用已经部分匹配的有效信息，保持文本串指针不回溯，通过修改模式串指针，让模式串尽量地移动到有效的位置。**

实现这个过程的核心工具就是 `next` 数组（前缀函数）。

## 2. next数组的含义
`next[i]` 的定义是：在子串 `pattern[0...i]` 中，**最长相等前后缀的长度**。
*   **前缀**：包含首字母，不包含尾字母的所有子串。
*   **后缀**：包含尾字母，不包含首字母的所有子串。

> 举例：对于字符串 `ABA`，前缀有 `A`, `AB`；后缀有 `A`, `BA`。最长相等前后缀是 `A`，长度为1。所以 `next[2] = 1`。

## 3. 核心难点解析：为什么是 `j = next[j-1]`？

在构建 `next` 数组或匹配过程中，最让人困惑的代码是这一段：

```cpp
while (j > 0 && pattern[i] != pattern[j]) {
    j = next[j - 1]; // 为什么是跳到 next[j-1]？
}
```

### 3.1 表面现象
当 `pattern[i] != pattern[j]` 时，说明试图延长当前找到的“最长相等前后缀”失败了。此时我们需要“退而求其次”，找一个**次长**的相等前后缀来尝试接上 `pattern[i]`。

### 3.2 深刻本质（克隆体原理）

假设模式串为 `A B A C A B A B`，当计算最后一个 `'B'` 的 `next` 值时：
*   当前遍历到 `i = 7` (指向最后一个 `'B'`)
*   上一轮成功匹配的最长前后缀长度为3，即 `j = 3` (指向 `'C'`)
*   此时比较 `pattern[7]` (`'B'`) 和 `pattern[3]` (`'C'`)，发现**不匹配**。

**直觉疑问**：
明明我们想知道的是：字符串末尾的 `pattern[6]` 能不能跟开头的 `pattern[0]` 对齐？为什么代码却是通过查 `next[2]` (即 `next[j-1]`) 来获取信息的？

**等价替换推导**：
1.  **已知事实**：在上一轮（`i=6`），我们已经确认了 `pattern[4...6]` (`"ABA"`) **完全等于** `pattern[0...2]` (`"ABA"`)。
    *   `pattern[4...6]` 是刚匹配完的**后缀**。
    *   `pattern[0...2]` 是对应的**前缀**。
2.  **当前目标**：既然长度为3的前后缀接不上，我们想在刚才的后缀 `pattern[4...6]` 里找一个**更短的后缀**，看看它能不能跟开头的**前缀**对上。
3.  **等价转换**：因为 `pattern[4...6]` 是 `pattern[0...2]` 的**完美克隆体**，所以：
    *   找 `pattern[4...6]` 的后缀 
    *   **等价于**
    *   找 `pattern[0...2]` 的后缀！
4.  **得出结论**：我们要在 `pattern[0...2]` 中寻找“后缀等于前缀”的最大长度。而这，**恰恰就是 `next[2]` 的定义**！

### 3.3 结论
代码之所以不去查末尾的 `pattern[6]`，是因为末尾匹配成功的部分（克隆体），跟开头的部分（本体）一模一样。查本体的 `next` 值（即 `next[j-1]`），就等于查了末尾克隆体的内部结构。这就是动态规划复用历史信息的精髓。

## 4. 完整代码模板

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

// 计算next数组
vector<int> getNext(const string& pattern) {
    int m = pattern.size();
    vector<int> next(m, 0); // next[0]必然为0
    
    // i指向当前正在计算的字符，j指向前缀的末尾（也是已匹配的前后缀长度）
    for (int i = 1, j = 0; i < m; i++) {
        // 如果不匹配，就一直回退，直到匹配或者退回起点
        while (j > 0 && pattern[i] != pattern[j]) {
            j = next[j - 1]; 
        }
        // 如果匹配成功，前缀长度加1
        if (pattern[i] == pattern[j]) {
            j++;
        }
        next[i] = j;
    }
    return next;
}

// KMP搜索
vector<int> kmpSearch(const string& text, const string& pattern) {
    vector<int> result;
    int n = text.size(), m = pattern.size();
    if (m == 0) return result;
    
    vector<int> next = getNext(pattern);
    
    for (int i = 0, j = 0; i < n; i++) {
        // 匹配逻辑与构建next数组一模一样
        while (j > 0 && text[i] != pattern[j]) {
            j = next[j - 1];
        }
        if (text[i] == pattern[j]) {
            j++;
        }
        // 当j等于模式串长度时，说明找到了一个完整匹配
        if (j == m) {
            // i 是主串中匹配到的最后一个字符的索引
            // 模式串长度为 m，所以起始位置是 i - m + 1
            result.push_back(i - m + 1);
            // 匹配成功后，也要回退j，继续寻找下一个可能的匹配
            j = next[j - 1];
        }
    }
    return result;
}
```
