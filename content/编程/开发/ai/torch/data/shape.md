```python
A=np.ones((5,3,7))
A.sum(axis=0)# shape=(3,7)
# sum会将对应axis消掉，相当于5个（3，7）矩阵相加
A.sum(axis=0,keepdim=True)# shape=(1,3,7)
# 如果keepdim，那么对应axis变成1
```