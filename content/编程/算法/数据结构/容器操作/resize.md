在全局定义数组,但在函数内部动态分布空间
使用resize函数
## 即使是二维数组也可以
```cpp
vector<vector<bool>> visit;
int func(vector<vector<int>>& grid){
	int n=grid.size(),m=grid[0].size();
	visit.resize(n,vector<bool>(m));
}
```