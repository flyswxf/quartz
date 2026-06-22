考试会给出完整程序
### Context
- constant: pre中的元素的声明
	- 如果pre中有$\forall$, 则其中元素不需要额外在constant中声明
- axiom: pre中的约束
	- 函数的定义: $f\in 0..n-1 \rightarrow N$, 需要自己写
	- 函数的性质: 会给出

## 第一步
### Machine
不管程序执行的具体内容, 只描述需求,
- variable
	- post中的元素的声明
- invariant
	- post中元素的约束, 比如i$\in$N
- event
	1. final事件, 即程序结束
		- guard设置为post, post需要自己写
			- 一旦满足guard, 则意味着程序可以实现post
	2. progress事件, 代表程序执行
		- 设置为anticipated, 代表它会成立, 但是现在不知道具体是怎么做的
		- action中包含程序内容的抽象实现, 比如$r:\in Z$
	3. init
		- 赋值, 可以用  i :$\in$ N的形式, 表示赋了某一个自然数

## 第二步
### refined Machine
- invariant
	- 不变式中的内容, 要自己编
- final
	- guard改成$\neg$P
- event
	- init
		- 变量的初值
	- progress
		- anticipated 改成convergent, 意味着它是实现的功能
		- action 程序中的内容, **最好符合执行顺序,否则可能会出现证明错误**
		- guard 加上P

convergent: rodin会检查变式, 确保progress会终止

## 第三步
- Removing non-determinacy: 将非确定赋值:$\in$替换为确定的赋值:=
	- ![[assets/进度精化算法示例.png]]
	- ![[assets/算法精化步骤.png]]