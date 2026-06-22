openMP控制并行的编译指令, 格式如下:
```c
#pragma omp construct [clause[, clause]]
```
例如: `#pragma omp parallel num_threads(4)`
- \#pragma omp
	- 指令前缀, 必须携带
- construct
	- 编译指令, 通常是parallel
- clause
	- 子句
## 常用编译指令
### 线程编号相关
- 获取并行线程的编号
	- `int ID = omp_get_thread_num();`
- 获取当前[[并行区域]]的线程数量
	- `int nthrds = omp_get_num_threads();`
- **指定一个[[并行区域]]的线程数量**
	- `#pragma omp parallel num_threads(4)`

### 变量作用域
- 设定并行区域外(默认为共享的)变量为私有
	- `#pragma omp private(var)`
	- 该语句只会声明一个私有var, 但不会有初始值(哪怕并行区域外的var有初值)


### 临界区
- 指定一个并行区域中的一段代码只能串行访问
	- **\#pragma omp critical**
	- 保证重名的critical只有一个在执行
	- 如果需要两组critical, 需要添加critical区域名称
		- \#pragma omp critical(queue_pop)
		- \#pragma omp critical(queue_push)
- 指定一个变量的修改是原子的(**只有一条语句**)
	- \#pragma omp atomic
	- 比如`X+=tmp;
- 指定只能有一个线程执行该代码, 其他进程跳过该代码
	- `#pragma omp single`: 任意线程, 该代码结束**有自动路障**
	- `#pragma omp master`: 只有主线程才能执行, 该代码结束**无自动路障**

### 同步
- 规定[[并行区域]]内线程必须全部执行到该语句再继续
	- `#pragma omp barrier`
- 一般(嵌套)[[并行区域]]结束会自动添加**路障**, 可以用`#pragma omp nowait`**取消自动路障**

### 循环
- **指定一个for循环并行执行**
	- `#pragma omp parallel for`, 或`#pragma omp parallel \n #pragma omp for`
		- **parallel和for必须一起**
	- 循环下标通常在并行区域外部定义(是共享的), 但是for指令会自动转换为私有的(相当于`private(i)`)
	- for自动做了任务分解, 如图
	- 通过`schedule(type, chunk_size)`规定任务分配的规则
	- ![[assets/并行循环实现对比.png]]
- 当for循环内有一个共享变量, 需要显式声明, reduction会自动完成临界区域控制
	- `#pragma omp parallel for reduction (+:sum)`
		- 对共享变量使用的计算(+, -,\*)
		- 共享变量名称