### 处理对象
数据库崩溃前, 已经执行完成(commit)的事务
在[[ARIES算法]]中, 与[[Undo]]配合, 需要Redo**全部事务**
### 目的
恢复数据库崩溃前的内存状态

### 算法步骤
从[[ARIES算法#^eae45a|脏页表]]中页的最小的[[ARIES算法#^c38f7c|recLSN]](最老的修改)开始
**重做所有操作**(能执行的内容, 比如update, CLR, abort\commit不用也不能执行), 除非
- 操作的页不在[[ARIES算法#^eae45a|脏页表]]中
- 操作的页在[[ARIES算法#^eae45a|脏页表]]中, 但是
	- 该页的[[ARIES算法#^c38f7c|recLSN]]>LSN(当前操作的修改已经被写入硬盘)
	- 或, 磁盘上的[[ARIES算法#^2e9bb6|pageLSN]]>=LSN(当前操作的修改将来会被写入硬盘)
重做结束后,将[[ARIES算法#^0bea86|活动事务表]]中所有状态为C(committed)的事务移除表, 并写入\<Ti End>(已经commit但是没来得及写end的事务)
除此以外, **redo不会写入任何Log**
