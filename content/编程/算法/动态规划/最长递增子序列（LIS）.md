## 一、问题定义
给定整数数组 `nums`，求**最长严格递增子序列**的长度。
- **子序列**：不要求连续，保持相对顺序
- **示例**：`[10,9,2,5,3,7,101,18]` → 输出`4`（如`[2,3,7,101]`）

---

## 二、基础解法：动态规划（O(n²)）
### 核心思想
- `dp[i]`：以第`i`个元素结尾的LIS长度
- 状态转移：`dp[i] = max(dp[i], dp[j] + 1)`（当`nums[j] < nums[i]`且`j < i`）
- 初始值：`dp[i] = 1`
- 结果：`max(dp)`

### C++代码
```cpp
int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;
    vector<int> dp(n, 1);
    int max_len = 1;
    
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        max_len = max(max_len, dp[i]);
    }
    
    return max_len;
}
```

---

## 三、优化解法：贪心+二分查找（O(n log n)）
### 核心思想
维护数组`tails`，其中`tails[k]`表示**长度为k+1的递增子序列的最小末尾元素**。

**贪心策略**：相同长度的子序列，末尾元素越小越好，越容易接后续更大元素。

### 关键规律（必须保留）
以`nums = [10,9,2,5,3,7,101,18]`为例：

| 子序列长度k | 所有长度为k的子序列的末尾元素 | 其中最小的末尾元素 |
|-------------|--------------------------------|--------------------|
| 1           | 10, 9, 2                       | 2                  |
| 2           | 5, 3                           | 3                  |
| 3           | 7                              | 7                  |
| 4           | 101, 18                        | 18                 |

**重要性质**：`tails`数组**严格递增**（可通过反证法证明）。

### 算法步骤
1. 初始化`tails`为空
2. 遍历每个元素`num`：
   - 若`num > tails.back()`，添加到末尾
   - 否则，用二分查找找到第一个≥`num`的元素，替换它
3. 结果：`tails.size()`

### C++代码
```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int num : nums) {
        // 二分查找第一个大于等于num的元素
        auto it = lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end()) {
            tails.push_back(num);
        } else {
            *it = num;
        }
    }
    return tails.size();
}
```

---

## 四、扩展问题
### 1. 最长非递减子序列
将`lower_bound`改为`upper_bound`：
```cpp
auto it = upper_bound(tails.begin(), tails.end(), num);
```
