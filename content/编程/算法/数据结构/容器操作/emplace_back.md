- 场景：`ans` 是 `vector<vector<int>>`。
- 当你写：
  ```cpp
  ans.emplace_back(x, y);
  ```
  `emplace_back` 会把这两个 `int` 直接“就地转发”给被存放的类型 `vector<int>` 的构造函数。
  而 `vector<int>` 有一个构造：`vector(size_type n, const T& value)`，
  所以它会被匹配为“构造一个长度为 `x`、元素值都为 `y` 的 `vector<int>`”。这就会出现：
  - `x == 0` → 存入M空数组 `[]`
  - `x == 1` → 存入 `[y]`
  - `x == 2` → 存入 `[y, y]`

- 当你写：
  ```cpp
  ans.push_back({x, y});
  ```
  `push_back` 只接受一个参数，这里传入的是一个初始化列表 `initializer_list<int>`，
  会先用 `{x, y}` 构造出一个临时的 `vector<int>{x, y}`，再移动到 `ans` 里，
  所以得到的就是你期望的 `[x, y]`。

---

## 正确写法（任选其一）

在保持 `vector<vector<int>>` 的前提下，以下三种写法都正确：

1) 最直观：
```cpp
ans.push_back({x, y});
```

2) 显式构造 `vector<int>` 再就地放入：
```cpp
ans.emplace_back(vector<int>{x, y});
```

3) 直接以初始化列表构造元素类型：
```cpp
ans.emplace_back(initializer_list<int>{x, y});
```

---

## 小结
- `emplace_back` 会把参数转发给元素类型的构造函数；当元素类型是 `vector<int>` 时，两个 `int` 会匹配到 `(n, value)` 构造器，容易造成“长度为 x，元素全为 y”的误用。
- 想得到坐标 `[x, y]` 这样的二元素数组，用初始化列表去构造 `vector<int>` 即可：`push_back({x, y})` / `emplace_back(vector<int>{x, y})`。