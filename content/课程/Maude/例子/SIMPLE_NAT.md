```maude
  fmod SIMPLE-NAT is
    sort Nat .
    op zero : -> Nat .
    op s_ : Nat -> Nat .
    op _+_ : Nat Nat -> Nat .
    vars N M : Nat .
    eq zero + N = N .
    eq s N + M = s (N + M) .
  endfm
```

- `sort Nat .`: 声明一个[[数据类型 Sort|数据类型]], 实际表示自然数
- `op zero : -> Nat .`:  [[操作符 Operator#^eff35b|声明一个常量]]为自然数
- `op s_ : Nat -> Nat .`:  后缀操作符, 代表`+1`.例如`s 0`代表1, `s s 0`代表2
- `vars N M : Nat .`: 声明两个[[变量 Variables|变量]]
- `eq s N + M = s (N + M) .`: 代表`N+1+M=N+M+1`, 之所以s N之间可以没有括号, 是因为`+`的运算优先级次于`s`(越小优先级越高), 参考[[运算优先级#^27e400|默认运算优先级]]