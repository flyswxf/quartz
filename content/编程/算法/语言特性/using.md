## 简化类型别名
这种写法可以节省打字并提升可读性

```cpp
using P = pair<long long, int>; // {到达时间, 节点}
priority_queue<P, vector<P>, greater<P>> pq;
```
只是为 `pair<long long, int>` 起了别名 `P`，逻辑不变。
之后依然可用常见操作，例如：

```cpp
auto [t, u] = pq.top();     // 结构化绑定，直接解包
pq.emplace(nt, v);          // 原地构造并入队
```

## 限制
```cpp
using ll = long long;          //允许
using int = long long;         //不允许, 因为int已经有重名
```
- 需要有`=`和`;`符号
- 不能重申类型名
