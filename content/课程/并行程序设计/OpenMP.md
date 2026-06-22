

- OpenMP is a multi-threading, shared address model.
	- Threads communicate by **sharing variables**. 
- Unintended sharing of data causes race conditions
	- **race condition**: when the program’s outcome changes as the threads are scheduled differently. 
- To control race conditions:
	-  Use synchronization to protect data conflicts. 
- Synchronization is expensive so:
	- Change how data is accessed to minimize the need for synchronization.


- OpenMP使用[[directive]]来设置[[并行区域]].
- OpenMP使用Fork-join模型



### 编程
- 引入头文件`#include <omp.h>`
- 共享变量定义在并行区域外
- 