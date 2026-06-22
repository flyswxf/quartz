双端队列
## 核心特性
- 普通单调栈只能取`频繁比较处`, 也就是`最弱`的元素,无法访问栈底`最强`的元素 
- 而双端队列可以从队头和队尾添加和移除元素


## 使用场景
单调队列

## 例子: 
```cpp
deque<int> lar;//维护最大值lar.front(). front最大, back最小, 新的数从back添加进去
for(int i=0;i<n;i++){
	while(!lar.emtpy()&&nums[lar.back()]<nums[i]) //前面的是比我大的第一个数
		lar.pop_back();
	lar.push_back(i);
	while(lar.front()<i-k)// 双端队列特性: 可以取front, 当front已经太旧/不满足某些条件, 就移除
		lar.pop_front();
}
```