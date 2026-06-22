「消息传递接口」（Message Passing Interface, MPI）

- MPI是基于消息传递模型
	- **默认阻塞通信**
		- 尽管数据已经被送达到 B 了，但是进程 B 依然需要确认它想要接收 A 的数据(**不用接受, 比如缓存起来, 但是需要确认收到消息**)。一旦它确定了这点，数据就被传输成功了。**进程 A 会接收到数据传递成功的信息，然后去干其他事情。
	- **缓存消息**
		- 有时候 A 需要传递很多不同的消息给 B。为了让 B 能比较方便地区分不同的消息，MPI 运行发送者和接受者额外地指定一些信息 ID (正式名称是_标签_, **_tags_**)。当 B 只要求接收某种特定标签的信息的时候，其他的不是这个标签的信息会**先被缓存起来**，等到 B 需要的时候才会给 B。


## 编程
- 引入头文件`#include <mpi.h>`
- 与[[OpenMP]]不同, MPI的赋值语句通过传递变量指针来实现
- 开局格式基本固定
	- MPI_Init(NULL,NULL);
	- int rank, size;
	- MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	- MPI_Comm_size(MPI_COMM_WORLD, &size);
- 0号进程收集/发放数组
	- int\* sorted = nullptr;
	- if (rank == 0)
        sorted = new int[n];


### 启动/终止
- `MPI_Init(NULL, NULL);`
	- 表示初始化MPI环境, 不用传参数
	- **初始化后, 后续代码都是并行运行**
- `MPI_Finalize()`
	- 退出MPI环境

### 进程数量
- 通讯域中进程编号
	- `MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);`
	- 编号从0开始
- 通讯域进程数量
	- `MPI_Comm_size(MPI_COMM_WORLD, &world_size);`
		- `MPI_COMM_WORLD`: MPI自带的默认通讯域
		- `world_size`: 返回值, 通讯域的大小将会传入world_size变量

### 通讯
#### 发送消息
- `MPI_Send(void* data,int count,MPI_Datatype datatype,int destination,int tag,MPI_Comm communicator)
	- 发送信息
		- `data`: 发送的数据地址
		- `count`: 发送多少个数据
		- `datatype`: 数据类型, 使用MPI定义的数据类型
			- `MPI_INT`
			- `MPI_DOUBLE`
			- `MPI_FLOAT`
			- `MPI_BYTE`: 传递char类型数据
	- 唯一确定一条消息
		- `destination`: 接受方进程编号
		- `tag`: 消息的编号
		- `communicator`: 通讯域
#### 例子
0号进程发送一个整型到1号进程
```
int number;
if (world_rank == 0) {
    number = -1;
    MPI_Send(&number, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
}
```

#### 广播
1. 向通讯域中所有进程**发送消息**
```
MPI_Bcast(
	void* data,
    int count,
    MPI_Datatype datatype,
    int root,
    MPI_Comm communicator)
```
- root: 发送消息的进程号
	- 进程号与root相同的进程执行MPI_Bcast()会发送消息
	- 进程号与root不同的进程执行MPI_Bcast()会接收消息
#### 例子
```
MPI_Bcast(data, num_elements, MPI_INT, 0, MPI_COMM_WORLD);
```

2. 向通讯域中所有进程**分配消息**
```
MPI_Scatter(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    int root,
    MPI_Comm communicator)
```
- send_count: **分配个每个进程的数据数量**
- recv_count: 缓冲区大小, 应该设置等于send_count

3. 收集通讯域中所有进程的消息
**Gather和Scatter是相反的作用**
```
MPI_Gather(
    void* send_data,
    int send_count,
    MPI_Datatype send_datatype,
    void* recv_data,
    int recv_count,
    MPI_Datatype recv_datatype,
    int root,
    MPI_Comm communicator)
```
- recv_count: 从每个进程收集的数据数量

4. **收集通讯域中进程的数据并做简单计算**
```
MPI_Reduce(
    void* send_data,
    void* recv_data,
    int count,
    MPI_Datatype datatype,
    MPI_Op op,
    int root,
    MPI_Comm communicator)
```
- recv_data: 比如求和计算, **此时recv_data只需要一个数, 不需要数组**
- count: 每个进程发送数据的数量
- op: 对收集到的数据做什么计算
	- `MPI_MAX` - 返回最大元素。
	- `MPI_MIN` - 返回最小元素。
	- `MPI_SUM` - 对元素求和。
	- `MPI_PROD` - 将所有元素相乘。
- root：进程号是root的进程收集数据, 其他进程发送数据

#### 接收消息
- `MPI_Recv(void* data,int count,MPI_Datatype datatype,int source,int tag,MPI_Comm communicator,MPI_Status* status)`
	- 接收消息
		- `data`: 接受数据的存放地址
		- `count`: **最多接受**该消息的多少个数据
		- `datatype`: 数据类型, 使用MPI定义的数据类型
			- `MPI_INT`
			- `MPI_DOUBLE`
			- `MPI_FLOAT`
			- `MPI_BYTE`: 传递char类型数据
	- 唯一确定一条消息
		- `source`: 发送方进程编号
		- `tag`: 消息的编号
		- `communicator`: 通讯域
	- `status`: 接受到的信息的状态
		- 是**MPI_Status**类型变量, 用`MPI_Status status;`声明
		- `status.MPI_SOURCE`: 发送方进程编号
		- `status.MPI_TAG`: 消息的编号
		- 如果不需要status, 用`MPI_STATUS_IGNORE`忽略

#### 例子
world_rank号进程发送一个整型到world_rank-1号进程, 不需要记录status
```
MPI_Recv(&token, 1, MPI_INT, world_rank - 1, 0,MPI_COMM_WORLD, MPI_STATUS_IGNORE);
```


### 同步
阻塞通讯域中所有进程直到它们都到达路障
- `MPI_Barrier(MPI_COMM_WORLD);`
