
`std::bitset` 是 C++ 标准库中一个非常有用的类，用于高效地处理固定大小的位序列。

### `bitset` 是如何实现的？

`std::bitset` 本质上是一个模板类，它将位（bits）打包存储在一个 `unsigned long` 或 `unsigned long long` 类型的数组中。它不是一个动态容器（如 `std::vector`），其大小在编译时就必须确定，并且之后不能更改。

例如，一个 `bitset<100>` 可能会在内部使用两个 `unsigned long long`（每个64位）来存储这100个位。通过位运算（如与 `&`、或 `|`、异或 `^`、移位 `<<` `>>`）来访问和修改特定的位，这使得操作非常快速。

### `bitset` 的属性 (模板参数)

`bitset` 的主要属性是它的大小，这个大小是在创建时通过模板参数指定的：

```cpp
template<size_t N> class bitset;
```

- `N`: 一个 `size_t` 类型的值，表示 `bitset` 中位的数量。这是一个编译时常量。

**示例：**
```cpp
// 创建一个可以存储 32 位的 bitset
std::bitset<32> myBits;

// 创建一个可以存储 1000 位的 bitset
std::bitset<1000> largeBits;
```

### `bitset` 的常用方法

以下是 `bitset` 提供的一些最重要和最常用的方法：

#### 1. 构造和初始化

你可以用多种方式创建和初始化 `bitset`。

```cpp
#include <bitset>
#include <string>
#include <iostream>

int main() {
    // 1. 默认构造，所有位都为 0
    std::bitset<8> b1; // 00000000
    std::cout << "b1: " << b1 << std::endl;

    // 2. 用一个整数初始化 (仅低位有效)
    std::bitset<8> b2(42); // 42 的二进制是 00101010
    std::cout << "b2: " << b2 << std::endl;

    // 3. 用一个字符串初始化
    std::string bit_string = "110101";
    std::bitset<8> b3(bit_string); // 00110101 (前面补0)
    std::cout << "b3: " << b3 << std::endl;

    // 4. 用字符串的一部分初始化
    // 从索引 2 开始，取 3 个字符 "010"
    std::bitset<8> b4(bit_string, 2, 3); // 00000010
    std::cout << "b4: " << b4 << std::endl;
}
```

#### 2. 访问和修改位

你可以像操作数组一样访问和修改 `bitset` 中的位。

- `operator[]`: 访问指定位置的位（返回一个特殊的引用类型，可以赋值）。
- `test(pos)`: 检查指定位置的位是否为1（`true`）或0（`false`），功能类似 `[]` 但更安全，会进行边界检查。
- `set(pos, value)`: 将指定位置的位设置为1（或指定的 `value`）。
- `reset(pos)`: 将指定位置的位清零。
- `flip(pos)`: 翻转指定位置的位（0变1，1变0）。

```cpp
std::bitset<8> bits("10101010");

// 访问
std::cout << "Position 1: " << bits[1] << std::endl; // 输出 1
std::cout << "Position 2: " << bits.test(2) << std::endl; // 输出 0

// 修改
bits[0] = 1;       // 设置最低位为 1 -> 10101011
bits.set(7);       // 设置最高位为 1 -> 10101011 (已经是1)
bits.reset(1);     // 将位置 1 的位清零 -> 10101001
bits.flip(2);      // 翻转位置 2 的位 -> 10101101

std::cout << "Modified bits: " << bits << std::endl;
```

#### 3. 整体操作

- `set()`: 将所有位都设置为1。
- `reset()`: 将所有位都清零。
- `flip()`: 翻转所有位。

```cpp
std::bitset<8> bits("00001111");
bits.flip(); // 变成 11110000
std::cout << "Flipped all: " << bits << std::endl;
```

#### 4. 查询信息

- `size()`: 返回 `bitset` 的大小（即位数 `N`）。
- `count()`: 返回值为1的位的数量。
- `any()`: 检查是否存在任何一个位为1。
- `none()`: 检查是否所有位都为0。
- `all()`: 检查是否所有位都为1。

```cpp
std::bitset<8> bits("00010010");
std::cout << "Size: " << bits.size() << std::endl;     // 8
std::cout << "Count of 1s: " << bits.count() << std::endl; // 2
std::cout << "Any bit is 1? " << bits.any() << std::endl;   // true
std::cout << "No bit is 1? " << bits.none() << std::endl;  // false
```

#### 5. 位运算

`bitset` 重载了所有位运算符，可以方便地进行集合操作。

- `&` (与), `|` (或), `^` (异或)
- `~` (非)
- `<<` (左移), `>>` (右移)
- `&=`, `|=`, `^=` , `<<=`, `>>=`

```cpp
std::bitset<8> a("01010101");
std::bitset<8> b("00110011");

std::cout << "a & b: " << (a & b) << std::endl; // 00010001
std::cout << "a | b: " << (a | b) << std::endl; // 01110111
std::cout << "a ^ b: " << (a ^ b) << std::endl; // 01100110
std::cout << "~a: " << (~a) << std::endl;       // 10101010
std::cout << "a << 2: " << (a << 2) << std::endl; // 01010100
```

#### 6. 转换

- `to_string()`: 将 `bitset` 转换为 `std::string`。
- `to_ulong()`: 将 `bitset` 转换为 `unsigned long`。
- `to_ullong()`: 将 `bitset` 转换为 `unsigned long long`。

**注意**: 如果 `bitset` 的值超出了 `unsigned long` 或 `unsigned long long` 的表示范围，`to_ulong` 和 `to_ullong` 会抛出 `std::overflow_error` 异常。

```cpp
std::bitset<8> bits("11000011");

std::string s = bits.to_string();
unsigned long ul = bits.to_ulong();

std::cout << "String: " << s << std::endl; // "11000011"
std::cout << "Unsigned long: " << ul << std::endl; // 195
```


### 常见操作的时间复杂度

| 操作 (Method)                                                   | 时间复杂度  | 说明                                                                                                                         |
| ------------------------------------------------------------- | :----: | -------------------------------------------------------------------------------------------------------------------------- |
| **访问和单点修改**                                                   |        |                                                                                                                            |
| `operator[]`, `test()`, `set(pos)`, `reset(pos)`, `flip(pos)` | `O(1)` | 直接通过位运算定位到内部数组的特定整数和特定位，操作极快。                                                                                              |
| **整体操作**                                                      |        |                                                                                                                            |
| `set()`, `reset()`, `flip()`                                  | `O(N)` | **(反常识点)** 这些操作需要遍历 `bitset` 内部存储的所有整数，并对每个整数执行操作。因此，它们的成本与 `bitset` 的大小 `N` 成正比，而不是 `O(1)`。                               |
| **查询操作**                                                      |        |                                                                                                                            |
| `count()`                                                     | `O(N)` | **(反常识点)** 需要遍历所有位来统计1的数量。虽然现代CPU有 `popcount` 指令可以极大地加速这个过程，但其复杂度仍然是线性的。                                                   |
| `any()`, `none()`, `all()`                                    | `O(N)` | 在最坏情况下，这些函数需要检查所有位才能得出结论。例如，`any()` 在一个全为0的 `bitset` 上必须检查到最后。它们可以提前退出，但大O表示法关心最坏情况。                                       |
| `size()`                                                      | `O(1)` | `N` 是一个编译时常量，所以 `size()` 只是返回这个存储的值。                                                                                       |
| **位运算**                                                       |        |                                                                                                                            |
| `&`,, `^`, `~`, `<<`, `>>`                                    | `O(N)` | **(反常识点)** 这是最需要注意的一点！与原生整数类型的 `O(1)` 移位不同，`bitset` 的移位和逻辑运算需要遍历其内部的整个整数数组。移位操作尤其复杂，因为它可能需要将位从一个内部整数移动到下一个，这无法通过单个CPU指令完成。 |
| **转换操作**                                                      |        |                                                                                                                            |
| `to_string()`                                                 | `O(N)` | 需要遍历所有 `N` 位来构建一个长度为 `N` 的字符串。                                                                                             |
| `to_ulong()`, `to_ullong()`                                   | `O(N)` | **(反常识点)** 如果 `N` 大于 `unsigned long` 的位数（通常是64），该函数必须检查所有超出范围的位是否为0，以确保转换不溢出。如果 `N` 小于等于64，则可以认为是 `O(1)`。                  |

### 总结：需要特别注意的反常识点

1.  **整体操作不是 `O(1)`**: 对整个 `bitset` 进行的 `set()`, `reset()`, `flip()` 操作，其成本与 `bitset` 的大小成正比 (`O(N)`)。
2.  **位移运算不是 `O(1)`**: 这是最大的“陷阱”。`bitset << 2` 看起来像一个单一、快速的操作，但实际上它是一个 `O(N)` 的操作，因为 `bitset` 内部是分块存储的，移位需要跨越这些块。
3.  **`count()` 不是 `O(1)`**: 即使有硬件加速，`count()` 的复杂度理论上也是 `O(N)`。
4.  **转换可能不是 `O(1)`**: `to_ulong()` 和 `to_ullong()` 在 `bitset` 尺寸 `N` 较大时，为了进行溢出检查，其复杂度也是 `O(N)`。

理解这些 `O(N)` 的操作对于在性能敏感的代码中正确使用 `std::bitset` 至关重要。虽然 `bitset` 在空间和单点访问上极为高效，但在进行涉及所有位的操作时，其成本不应被低估。
