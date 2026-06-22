```maude
  ---- Implement funcItion revert
  ---- which reverts the order of the natural numbers in a given list.
  ---- e.g. revert(3,2,7,1)=1,7,2,3

  fmod REVERT is
    pr NAT .
    sort List .
    subsorts Nat < List .
    op empty : -> List [ctor].
    op revert : List -> List .
    op _,_ : List List -> List [ctor assoc id: empty].

    var N : Nat .
    var L : List .
    eq revert(empty) = empty .
    eq revert(N , L) = revert(L) , N .
  endfm
```
- pr: protecting的缩写, 代表引用 #todo 
- empty: 定义空列表常量
- revert: 不加下划线默认为[[操作符 Operator#^d1eec5|前缀表达式]]
- eq: 用递归形式表达函数语义
- assoc(连接性) id: empty(同一性) comm(幂等性): 同一性能令结果不显示empty #todo

问题: 
常量解析的时候去哪了?
and or, /\\, \\/
 assoc id: nil是什么作用


