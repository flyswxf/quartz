## 已知参数
M,m: 关系表R有M个页,m条记录
N,n: 关系表S有N个页,n条记录
**选择小表作为外表, 能提高查询效率**
### Simple Nested Loop Join
- Cost: $M+(m\times N)$
	- 用数据块的I/O次数来衡量开销, 因为每次读写数据块的时间是由硬件固定的.
	- M: 搜索全部M个页
	- $m\times N$: R中每个项都要搜索S中全部N个页
- Seek: $M+m$

### Block Nested Loop Join
- Cost: $M+(M\times N)$
	- M: 搜索全部M个页
	- $M\times N$: R中每个页都要搜索S中全部N个页, 其实就是将R中的页切割成Block(这样占用的空间变小,可以缓存)缓存下来了. 这样一个表的内容就可以多次使用.
- Seek: 

### Parallel BLock Nested Loop Join
使用B-2个内存块存放R(外循环的表), 1个内存块存放S(内循环的表), 1个内存块存放输出结果
- Cost:  $M+(⌈M/(B-2)⌉\times N)$
	- 最好情况下, B能完全容纳M, 即$B>M+2$, Cost = M+N

### Index Nested Loop Join
对于R中每个记录, 利用S上索引搜索join key相等的项
- Cost: $M+m\times C$
	- 假设根据索引查询一个项的代价是常量:C

### Sort-Merge Join
先排序, 再遍历
- Cost: 二者之和
	- Sort: $2M\times (1+⌈log_{B-1}^{⌈M/B⌉} ⌉)$和$2N\times (1+⌈log_{B-1}^{⌈N/B⌉} ⌉)$
		- 如果已经关系表已经有序, 则不需要考虑Sort的开销
	- Merge: $M+N$

### Hash Join
给定B, 最大可以对$B\times (B-1)$个块大小的表进行哈希
给定N, 至少需要sqrt(N)个内存块

- Cost: 3(M+N)
	- Partitioning Phase: 2(M+N)
	- Probing Phase: M+N