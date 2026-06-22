## 1. 技术原理复盘

### 1.1 模块目标与要解决的问题

`node_program("X"*k)` 系列 pass 属于 Shrinker 的核心收缩策略，目标是通过批量删除节点来快速减少测试用例的长度。核心问题是：单个节点的删除虽然安全，但效率较低，而批量删除可以显著提高收缩效率，但需要解决如何确定批量操作的范围和次数的问题。

`_node_program` 实现了自适应逻辑，通过二分查找确定可以连续应用节点程序的最大次数，将复杂度从 O(k) 降低到 O(log(k))，解决了批量操作的效率问题。

### 1.2 输入与输出

输入是当前 Shrinker 的 `shrink_target`，其核心内容是 `nodes` 作为 ChoiceNode 序列。

输出是更新后的 `shrink_target`，表现为更短的 ChoiceNode 序列，仍满足谓词并保持 `INTERESTING`。

### 1.3 基本流程与架构

整体从 `greedy_shrink` 阶段进入，`node_program("X"*k)` 系列 pass 作为 `shrink_passes` 列表的前几个成员被优先执行。每个 `node_program("X"*k)` 都会创建一个 ShrinkPass，其执行函数是 `_node_program`，并传入相应的 "X" 序列作为 program。

`_node_program` 的核心流程是：
1. 随机选择一个起始位置
2. 尝试在该位置执行节点程序
3. 如果成功，向左移动到该区域的起始位置
4. 然后尝试连续多次执行节点程序

通过这种方式，可以高效地批量删除节点，快速减少测试用例的长度。

## 2. 源码阅读与逻辑拆解

### 2.1 输入 → 处理 → 输出的完整逻辑

输入阶段以 `shrink_target.nodes` 为主线数据，`_node_program` 首先获取 program 的长度 n，然后通过 `chooser.choose` 随机选择一个起始位置 i，确保从 i 开始有足够的节点可以应用 program。

处理阶段分为三个步骤：
1. **单次尝试**：先在随机选择的位置 i 尝试执行节点程序，如果失败则直接返回
2. **向左扩展**：如果单次尝试成功，使用 `find_integer` 向左移动，找到可以执行节点程序的最左边界
3. **批量执行**：在找到的最左边界，再次使用 `find_integer` 尝试连续多次执行节点程序，确定最大的执行次数

输出阶段由 `run_node_program` 内部调用 `consider_new_nodes` 触发执行与比较，通过 `incorporate_test_data` 把更小且满足谓词的候选更新为新的 `shrink_target`。

### 2.2 执行流程图

#### 2.2.1 `node_program` 系列 pass 的创建

```mermaid
flowchart TD
  A["Shrinker.__init__"] --> B["创建 shrink_passes 列表"]
  B --> C["node_program('X'*5)"]
  B --> D["node_program('X'*4)"]
  B --> E["node_program('X'*3)"]
  B --> F["node_program('X'*2)"]
  B --> G["node_program('X'*1)"]
  C --> H["创建 ShrinkPass"]
  D --> H
  E --> H
  F --> H
  G --> H
  H --> I["设置执行函数为 _node_program"]
  I --> J["设置 name 为 node_program_XXX"]
  J --> K["添加到 shrink_passes 列表"]
```

#### 2.2.2 `_node_program` 的自适应逻辑

```mermaid
flowchart TD
  A["_node_program(chooser, program)"] --> B["计算 program 长度 n"]
  B --> C["随机选择起始位置 i"]
  C --> D["run_node_program(i, program)"]
  D --> E{是否成功?}
  E -->|否| F["返回"]
  E -->|是| G["定义 offset_left(k) 函数"]
  G --> H["find_integer 寻找最左边界"]
  H --> I["更新 i 为最左边界"]
  I --> J["original = shrink_target"]
  J --> K["find_integer 寻找最大连续执行次数"]
  K --> L["run_node_program(i, program, repeats=k)"]
  L --> M["返回"]
  
  D -.数据.-> N["输入: shrink_target.nodes"]
  L -.数据.-> O["输出: 更新后的 shrink_target"]
```

#### 2.2.3 `run_node_program` 的执行逻辑

```mermaid
flowchart TD
  A["run_node_program(i, program)"] --> B["创建 nodes 副本 attempt"]
  B --> C["遍历 program 中的每个命令"]
  C --> D{"命令类型?"}
  D -->|"X"| E["删除 attempt[i+j] 节点"]
  D -->|其他| F["抛出 NotImplementedError"]
  E --> G["继续处理下一个命令"]
  G --> H{"所有命令处理完毕?"}
  H -->|否| C
  H -->|是| I["consider_new_nodes(attempt)"]
  I --> J{"是否满足谓词且更小?"}
  J -->|是| K["返回 True"]
  J -->|否| L["返回 False"]
  
  A -.数据.-> M["输入: shrink_target.nodes"]
  K -.数据.-> N["输出: 更新后的 shrink_target"]
```

#### 2.2.4 `find_integer` 的二分查找逻辑

```mermaid
flowchart TD
  A["find_integer(f)"] --> B["假设 f(0)=True"]
  B --> C["线性扫描 [1,4]"]
  C --> D{"找到 f(n)=False?"}
  D -->|是| E["返回 n-1"]
  D -->|否| F["指数探测找到 False 的上界"]
  F --> G["lo = 4, hi = 5"]
  G --> H{"hi < 1e6 且 f(hi)=True?"}
  H -->|是| I["lo = hi, hi *= 2"]
  I --> H
  H -->|否| J["二分搜索精确定位边界"]
  J --> K{"lo + 1 < hi?"}
  K -->|是| L["mid = (lo + hi) // 2"]
  L --> M{"f(mid)=True?"}
  M -->|是| N["lo = mid"]
  M -->|否| O["hi = mid"]
  N --> K
  O --> K
  K -->|否| P["返回 lo"]
  
  A -.数据.-> Q["输入: 测试函数 f"]
  P -.数据.-> R["输出: 最大的 n 使得 f(n)=True"]
```

### 2.3 关键实现细节

#### 2.3.1 批量删除的顺序

`node_program("X"*k)` 系列 pass 按照 k 从大到小的顺序执行：
```python
self.shrink_passes: list[ShrinkPass] = [
    ShrinkPass(self.try_trivial_spans),
    self.node_program("X" * 5),  # 先尝试删除5个连续节点
    self.node_program("X" * 4),  # 然后尝试删除4个连续节点
    self.node_program("X" * 3),  # 依此类推
    self.node_program("X" * 2),
    self.node_program("X" * 1),
    # ... 其他 pass
]
```

这种顺序设计可以快速减少测试用例的长度，因为较大的批量删除通常能带来更大的收缩效果。

#### 2.3.2 自适应逻辑的效率

`_node_program` 使用 `find_integer` 函数实现自适应逻辑，将复杂度从 O(k) 降低到 O(log(k))：
```python
def find_integer(f: Callable[[int], bool]) -> int:
    # 假设 f(0)=True，找到最大的 n 使得 f(n)=True 且 f(n+1)=False
    # 算法:
    # 1. 线性扫描 [1,4] —— 小结果时避免浪费
    # 2. 指数探测找到 False 的上界
    # 3. 二分搜索精确定位边界
    # 复杂度: O(log n)
    # ...
```

通过这种方式，即使需要连续删除很多节点，也只需要少数几次测试调用。

#### 2.3.3 与 ChoiceTree 的集成

`node_program` 系列 pass 与 `ChoiceTree` 紧密集成，通过 `chooser.choose` 选择起始位置，避免重复尝试相同的位置：
```python
i = chooser.choose(range(len(self.nodes) - n + 1))
```

`ChoiceTree` 记录了选择序列，确保每个可能的起始位置都被尝试过，直到整个搜索空间被耗尽。
