### 处理对象
在数据库崩溃前, 未执行完成(commit)的事务, 即[[ARIES算法#^0bea86|活动事务表]]中在[[Redo]]后剩下的状态为U(uncommitted)的事务

### 目的
撤销未提交事务的修改

### 算法步骤
**ToUndo**: {[[ARIES算法#^0bea86|活动事务表]]中的所有lastLSN}
**重复执行以下步骤**, 直到ToUndo为空
1. 选择ToUndo中最大的LSN(最近的操作)
2. 如果该LSN是[[基于日志的恢复#^fea3c9|CLR]]
	- 如果CLR的undonextLSN**为空**(这是最后的CLR,Ti的undo已经完成了), Log记录\<Ti End>
	- 如果CLR的undonextLSN**不为空**, 将CLR的undonextLSN加入ToUndo(继续Ti未完成的undo)
3. 如果该LSN是更新操作
	- Undo该操作
	- Log记录CLR
	- 将[[基于日志的恢复#^798544|prevLSN]]加入ToUndo
	- 如果没有prevLSN（该LSN的上一条是begin），则再Log写一条\<Ti end>
4. 在ToUndo中移除该LSN
