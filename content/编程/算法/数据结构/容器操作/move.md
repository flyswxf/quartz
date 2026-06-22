std::move()用于将变量移动到数组中
```
string s = 'key';
vector<int> v;
v.push_back(move(s));
```
会使得
```
s = unspecified(不可使用)
v[0]='key'
```
这个转移避免了不必要的copy