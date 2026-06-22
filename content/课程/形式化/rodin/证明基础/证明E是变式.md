```
A1; 
while P do 
A2 
invariant 
Q
variant
E
end
```
需要证明两个点: 

1. **NAT**: E是一个自然数
	- Pre, P, Q  $\vdash$ E $\in$ N
2. **VAR:** E随着A2执行而逐渐减小, 最终使程序退出循环
	- Pre, P, Q  $\vdash$ ([A2]E) < E