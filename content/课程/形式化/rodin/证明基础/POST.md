```
A1; 
while P do 
A2 
invariant 
Q 
end
```

### Event finale refine its abstractions

当Q已经证明是不变体后, 
- Pre ^ $\neg P$: **循环结束时的状态**
- Pre , $\neg P$, Q: 在循环结束时的状态下, **Q得到的结果**
**证明Q得到的结果是Post, 则程序部分正确**