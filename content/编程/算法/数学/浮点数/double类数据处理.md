# 浮点数 (double) 特殊处理

## 核心思想
浮点数（如 `double`）在计算机中以二进制分数形式存储，因此会产生精度丢失问题。例如 `0.1 + 0.2` 可能并不严格等于 `0.3`。
在算法竞赛中，直接使用 `==` 判断两个浮点数是否相等是非常危险的，极易导致 WA (Wrong Answer)。

**解决策略**：引入一个极小的正数 `eps` (Epsilon，通常取 $10^{-8}$ 或 $10^{-9}$)。
如果两个浮点数的差的绝对值小于 `eps`，我们就认为这两个浮点数是相等的。我们将这种比较方式封装成一个**符号函数 `sgn(x)`**，所有的浮点数比较操作都应基于这个函数来进行。

## C++ 模板

```cpp
#include <iostream>
#include <cmath>
using namespace std;

// 定义精度 eps，一般取 1e-8 到 1e-10 之间
const double eps = 1e-8;

// 定义圆周率 PI，使用 acos(-1.0) 保证最高精度
const double PI = acos(-1.0);

/*
 * 核心符号函数 sgn(x)
 * 返回值：
 *   0 : 当 x 在 [-eps, eps] 之间，即 x == 0
 *   1 : 当 x > eps，即 x > 0
 *  -1 : 当 x < -eps，即 x < 0
 */
inline int sgn(double x) {
    if (fabs(x) < eps) return 0;
    if (x < 0) return -1;
    return 1;
}

/*
 * 浮点数比较包装函数 (可选，通常直接用 sgn 即可)
 * 用于比较两个浮点数 a 和 b 的大小关系
 */
inline int cmp(double a, double b) {
    return sgn(a - b);
}

// ---------------- 常用比较操作封装 ----------------
inline bool eq(double a, double b) { return sgn(a - b) == 0; }  // a == b
inline bool ls(double a, double b) { return sgn(a - b) < 0; }   // a < b
inline bool le(double a, double b) { return sgn(a - b) <= 0; }  // a <= b
inline bool gt(double a, double b) { return sgn(a - b) > 0; }   // a > b
inline bool ge(double a, double b) { return sgn(a - b) >= 0; }  // a >= b
```

## 关键要点说明

1. **`eps` 的取值与精度选择**
   - 算法竞赛中一律推荐使用 `double` 而不是 `float`（`float` 精度仅约 7 位有效数字，极易产生误差）。
   - `eps` 一般取 `1e-8`。如果题目精度要求较高，可以取 `1e-10` 到 `1e-12`。
   - **不要把 `eps` 设置得太小**（例如 `1e-15`）。`double` 的极限精度大约是 15~17 位十进制有效数字，设置太小会导致 `fabs(x) < eps` 误判，反而失去容错作用。

2. **输出 `-0.00` 的问题与避免方法**
   - **现象**：在 C++ 中，如果一个负数极小（如 `-0.0000001`），直接 `printf("%.2f\n", x)` 时，可能会输出尴尬的 `-0.00`，这在大部分在线评测系统 (OJ) 中会被判为格式错误。
   - **解决**：在输出前，利用 `sgn` 函数过滤掉极小负数。
     ```cpp
     if (sgn(x) == 0) printf("0.00\n");
     else printf("%.2f\n", x);
     ```

3. **常量 $\pi$ 的定义**
   - 强烈建议使用 `const double PI = acos(-1.0);`，避免手动敲击 `3.1415926535...` 导致位数不足或打字错误。

4. **除以零与 `NaN` / `Inf`**
   - 在计算几何或数学公式中，除数可能是浮点数。由于精度问题，除数不能直接写 `if (divisor == 0)`，而必须写 `if (sgn(divisor) == 0)`，否则极小值作为除数会产生极其巨大的结果，甚至是 `Inf` (Infinity)。
   - 出现 `NaN` (Not a Number) 通常是因为进行了非法的数学运算，例如 `sqrt(-1.0)` 或 `acos(1.0000001)`。特别是在使用 `acos` 时，传入的参数由于精度误差可能略微超过 1（如 `1.0 + 1e-16`），这会导致 `acos` 返回 `NaN`。
   - **防御式编程**：在传参给反三角函数前，务必对范围进行截断约束：
     ```cpp
     double safe_acos(double x) {
         if (x > 1.0) x = 1.0;
         if (x < -1.0) x = -1.0;
         return acos(x);
     }
     ```
