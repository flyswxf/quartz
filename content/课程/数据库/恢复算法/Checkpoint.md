数据库可以从检查点开始恢复
制作检查点需要等待所有事务完成再刷盘, 才能保证数据库的一致性.可以使用**模糊检查点**避免

## 模糊检查点
制作检查点时允许事务同时执行

#### Log
- \<CHECKPOINT-BEGIN>: 标志从此处开始制作检查点
- \<CHECKPOINT-END>
	- 标志检查点制作完成
	- **脏页表**: 记录\<CHECKPOINT-BEGIN>前的脏页
	- **事务表**: 记录\<CHECKPOINT-BEGIN>前的未结束的事务
	- **忽略\<CHECKPOINT-BEGIN>\<CHECKPOINT-END>之间的操作的影响**

