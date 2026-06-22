使用[[内存管理协议#^f7c2c6|STEAL+NO-FORCE]]
分为三个阶段: [[分析]], [[Redo]],[[Undo]]

### 数据结构
- 日志
	- 为每一条日志分配一个**唯一递增**的**LSN**(log sequence number)
- 脏页表 ^eae45a
	- 数据库崩溃时, (可能)还没写入硬盘的脏页
		- 因为[[分析]]阶段只将Page加入脏页表而不会将Page移除表, 可能存在修改过但之后写入硬盘的正常页
	- 存放
		- Page-id: 页的唯一标识
		- **recLSN**: **第一个对该页修改的LSN**(将该页变为脏页的第一条操作) ^c38f7c
	- 在[[分析]]阶段获取
- 活动事务表 ^0bea86
	- **数据库崩溃时, 还没结束的事务**
	- 存放
		- Txn-id: 事务的唯一标识
		- **lastLSN**: **Ti所做的最新一条操作的LSN** ^a10895
		- 状态: U(uncommitted)/C(committed)
- 磁盘页: 含有**PageLSN**, 存放对该page的最新一次修改的LSN(**写入硬盘前的最后一个LSN**) ^2e9bb6
- 内存页: 含有**PageLSN**, 维护**对该page的最新一次修改的LSN**
- flushedLSN: **磁盘上的log的最后一个LSN**(**最后一个写入硬盘的页的最后一个LSN**)
	- [[基于日志的恢复#^957bdd|WAL]]的逻辑依靠$pageLSN\le flushedLSN$来确定
- MasterRecord: 存放上一个[[Checkpoint]]的LSN

### 算法

