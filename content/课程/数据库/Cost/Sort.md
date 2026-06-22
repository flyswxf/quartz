### 已知参数
- $b_{r}$: 数据表R包含多少个在磁盘中的数据块
- $b_{b}$: 一次从磁盘中读取多少个数据块到缓冲区(这样就只用seek一次)
- M: 内存中的可用数据块数量
- 常量
	- $T_{transfer}$: Block transfer time
	- $T_{seek}$: Disk seek cost


### 可计算变量
- sorting pass: 1
- merge pass: $⌈log_{⌊M/b_{b}⌋-1}^{⌈b_{r}/M⌉} ⌉$
- Block transfer: $b_{r}\times (2⌈log_{⌊M/b_{b}⌋-1}^{⌈b_{r}/M⌉}⌉+1)$
	- 写入和读出的数据块总数
	- $2b_{r}$: Pass #0, 全部读入读出
	- $2b_r-b_r$: 最后结果不用写回去
- seek: $2⌈b_{r}/M⌉+⌈b_r/b_b⌉\times (2⌈log_{⌊M/b_{b}⌋-1}^{⌈b_{r}/M⌉}⌉-1)$
	- $2⌈b_{r}/M⌉$: 每次读M个, 全部读入读出