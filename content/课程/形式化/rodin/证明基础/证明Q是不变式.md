### 不变式是在循环中始终满足的条件

**证明不变式**(定义域为**自然数**)需要使用**数学归纳法**
1. Q(0)成立
2. Q(k)成立则Q(k+1)成立

```
A1; 
while P do 
A2 
invariant 
Q 
end
```

使用形式化语言表示为
1. **INI**: Event **init** establishes the invariant
	- Pre  $\vdash$ \[A1]Q
	- A1: 循环的初始状态, 对应数学归纳法中的最初值
	- \[A1]Q: 以A1进入循环可以满足Q
2. **INV**: Event **progress** maintains the invariant
	- Pre, P, Q  $\vdash$ \[A2]Q 
	- A2: 循环每次更新的状态
	- Q: 执行A2前满足Q
	- \[A2]Q: 执行完A2仍然满足Q
	- P: 循环需要始终满足的条件
