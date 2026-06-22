**完整语法格式：**
```cpp
[capture](parameters) -> return_type { function_body }
```

**例子：**
1. 比较函数
```cpp
sort(points.begin(),points.end(),[](const vector<int>& a,const vector<int>& b){
    return a[0] < b[0];
});
```
- `[]` - **捕获列表**：空的方括号表示不捕获任何外部变量
- `(const vector<int>& a, const vector<int>& b)` - **参数列表**：接收两个常量引用参数
- `{ return a[0] < b[0]; }` - **函数体**：比较逻辑，返回第一个点的x坐标是否小于第二个点的x坐标

这个lambda表达式等价于定义一个比较函数：
```cpp
bool compare(const vector<int>& a, const vector<int>& b) {
    return a[0] < b[0];
}
// 然后使用：sort(points.begin(), points.end(), compare());
```

2. 判断函数
```cpp
auto inside=[&](int x,int y){ return x>=0&&x<n&&y>=0&&y<m;};
```
- `[&]` : 代表将外部变量以**引用**的方式捕获
- `return x>=0&&x<n&&y>=0&&y<m;`: 用于在图结构中判断点位置是否合法
