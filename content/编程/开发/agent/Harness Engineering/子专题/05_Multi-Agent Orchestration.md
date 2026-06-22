## 概述
Multi-Agent Orchestration 指的是如何把多个 Agent 组织成一个协同系统，使它们在共享目标下分工、通信、同步状态并汇总结果。它不是简单地“多开几个 Agent”，而是要解决调度、路由、共享状态、冲突控制和失败恢复问题。

它与 [[多智能体系统(Multi-Agent)#典型协作模式|多智能体系统]] 的关系是：多智能体系统更偏概念与协作范式，而 Multi-Agent Orchestration 更偏工程实现与运行机制。

## 为什么单个 Agent 不够
随着任务复杂度上升，单个 Agent 往往会暴露以下问题：
- 上下文过长，导致注意力分散。
- 工具种类过多，选择空间过大。
- 一个 Agent 同时负责规划、执行、审查、汇总，角色冲突严重。
- 错误发生后，无法定位究竟是哪一部分能力失效。

通过编排多个角色化 Agent，可以将系统拆成更清晰的职责单元。

## 编排的核心问题

### 1. 谁负责分配任务
需要一个 `supervisor`、`router` 或 `manager` 判断当前子任务应该发给哪个 Agent。

### 2. Agent 之间如何通信
是通过共享黑板（blackboard）、消息队列，还是点对点对话。

### 3. 谁维护全局状态
如果每个 Agent 只维护自己的局部上下文，那么谁来保存总体目标、已完成步骤和最终结论。

### 4. 如何终止
多智能体系统比单 Agent 更容易进入循环，因此必须有更严格的退出条件。

## 常见编排模式

### 1. Supervisor-Worker
一个总控 Agent 负责拆解任务并调用多个执行 Agent。
- 优点：结构清晰，便于控制。
- 缺点：Supervisor 成为瓶颈。

### 2. Blackboard
多个 Agent 通过共享状态板读写中间结果。
- 优点：适合异步协作和复杂信息整合。
- 缺点：需要很好地处理并发和冲突。

### 3. Debate
多个 Agent 围绕同一问题给出不同答案，再由裁判 Agent 汇总。
- 优点：适合降低偏见、提高鲁棒性。
- 缺点：开销大，且可能产生冗余争论。

### 4. Pipeline
不同 Agent 依次执行固定阶段，例如 `Planner -> Retriever -> Coder -> Reviewer -> Summarizer`。
- 优点：最接近工程流水线。
- 缺点：灵活性有限。

## 推荐的工程抽象
在 harness 里，较稳定的抽象通常包括：
- `AgentRole`：角色定义。
- `TaskEnvelope`：子任务载体。
- `SharedState`：共享状态。
- `Orchestrator`：编排器。
- `TerminationPolicy`：终止策略。

## 共享状态的设计
共享状态至少应包含：
- `goal`：总体目标。
- `artifacts`：中间产物，如检索结果、代码片段、摘要。
- `completed_tasks`：已完成子任务。
- `pending_tasks`：待处理任务。
- `messages`：关键交互日志。

关键原则是：共享状态保存“系统需要共享的事实”，而不是每个 Agent 的完整上下文副本。

## 终止条件
多智能体编排必须定义更严格的退出条件，例如：
- 所有必需子任务完成。
- 达到最大编排轮数。
- 连续若干轮没有新产物产生。
- Supervisor 明确给出 `final_answer`。
- 命中策略层熔断。

## 规范实现示例
下面给出一个 Supervisor-Worker 风格的最小编排器。目标是演示角色路由、共享状态和终止控制，而不是依赖某个具体框架。

```python
from __future__ import annotations

from dataclasses import dataclass, field
from enum import Enum
from typing import Protocol


class AgentRole(str, Enum):
    PLANNER = "planner"
    RESEARCHER = "researcher"
    CODER = "coder"
    REVIEWER = "reviewer"


@dataclass(slots=True)
class TaskEnvelope:
    task_id: str
    role: AgentRole
    content: str
    depends_on: list[str] = field(default_factory=list)


@dataclass(slots=True)
class SharedState:
    goal: str
    pending_tasks: list[TaskEnvelope] = field(default_factory=list)
    completed_tasks: list[str] = field(default_factory=list)
    artifacts: dict[str, str] = field(default_factory=dict)
    final_answer: str | None = None


class RoleAgent(Protocol):
    def run(self, task: TaskEnvelope, shared_state: SharedState) -> str:
        ...


class PlannerAgent:
    def run(self, task: TaskEnvelope, shared_state: SharedState) -> str:
        # 真实系统中这里应由 LLM 生成分解结果
        shared_state.pending_tasks.extend(
            [
                TaskEnvelope(task_id="research-1", role=AgentRole.RESEARCHER, content="检索相关资料"),
                TaskEnvelope(task_id="code-1", role=AgentRole.CODER, content="根据资料生成代码"),
                TaskEnvelope(task_id="review-1", role=AgentRole.REVIEWER, content="审查代码与结论"),
            ]
        )
        return "task_decomposed"


class ResearcherAgent:
    def run(self, task: TaskEnvelope, shared_state: SharedState) -> str:
        shared_state.artifacts["research_notes"] = "这是检索得到的背景资料摘要。"
        return "research_completed"


class CoderAgent:
    def run(self, task: TaskEnvelope, shared_state: SharedState) -> str:
        notes = shared_state.artifacts.get("research_notes", "")
        shared_state.artifacts["draft_code"] = f"# generated from notes\n# {notes}\nprint('hello')"
        return "code_completed"


class ReviewerAgent:
    def run(self, task: TaskEnvelope, shared_state: SharedState) -> str:
        code = shared_state.artifacts.get("draft_code", "")
        shared_state.artifacts["review_report"] = f"代码审查完成，长度={len(code)}"
        shared_state.final_answer = "任务已完成，附带研究摘要、代码草稿和审查结论。"
        return "review_completed"


class MultiAgentOrchestrator:
    def __init__(self, max_rounds: int = 10) -> None:
        self.max_rounds = max_rounds
        self.agents: dict[AgentRole, RoleAgent] = {
            AgentRole.PLANNER: PlannerAgent(),
            AgentRole.RESEARCHER: ResearcherAgent(),
            AgentRole.CODER: CoderAgent(),
            AgentRole.REVIEWER: ReviewerAgent(),
        }

    def run(self, goal: str) -> SharedState:
        state = SharedState(goal=goal)
        state.pending_tasks.append(
            TaskEnvelope(task_id="plan-1", role=AgentRole.PLANNER, content=goal)
        )

        rounds = 0
        while state.pending_tasks and rounds < self.max_rounds and state.final_answer is None:
            rounds += 1
            task = state.pending_tasks.pop(0)

            agent = self.agents[task.role]
            result = agent.run(task, state)

            state.completed_tasks.append(task.task_id)
            state.artifacts[f"result::{task.task_id}"] = result

        if state.final_answer is None and rounds >= self.max_rounds:
            state.final_answer = "编排提前终止：达到最大轮数。"

        return state
```

## 这段实现体现了什么

### 1. 编排器只负责调度，不负责执行所有细节
`MultiAgentOrchestrator` 只关心：
- 当前有哪些任务。
- 该把任务交给谁。
- 是否达到退出条件。

### 2. 共享状态是系统总线
各个角色 Agent 不直接互相依赖，而是通过 `SharedState` 交换中间产物。

### 3. 终止条件显式存在
最大轮数和 `final_answer` 都被显式编码，避免无界对话。

## 从最小实现走向真实系统
若进入真实系统，通常还需要继续补以下能力：

### 1. 路由器
不再写死 `task.role`，而是由模型或规则引擎动态选择下一个 Agent。

### 2. 任务依赖检查
只有前置任务完成后，后续任务才能执行。

### 3. 冲突消解
多个 Agent 可能对同一 artifact 给出不一致结论，需要引入 reviewer 或 judge。

### 4. 可观测性
每个 Agent 的输入、输出、耗时、工具调用都要落入 [[Harness Engineering/06_评测与可观测性#观测数据通常包括什么|观测数据]]。

### 5. 安全边界
不同角色 Agent 不应默认共享同等工具权限。例如研究 Agent 只读，执行 Agent 才能写。

## 与 HITL 的结合
在复杂企业流程中，经常不是“多 Agent 自动完成一切”，而是：
- Supervisor 负责任务拆解。
- Worker Agent 负责检索、分析、执行。
- Reviewer Agent 负责审查。
- 人工审批节点负责最终确认高风险动作。

因此，Multi-Agent Orchestration 往往与 [[Harness Engineering/子专题/04_Human-in-the-Loop 与 Approval Workflow#Approval Workflow 的最小状态机|Human-in-the-Loop 与 Approval Workflow]] 联合使用。

## 工程实践建议
- 先从 `Supervisor-Worker` 做起，不要一开始就上自由对话式多 Agent。
- 共享状态里只存“可复用事实”，不要复制完整历史对话。
- 给不同角色分配不同工具集和不同预算。
- 对每个角色记录独立的成功率和失败模式。

## 与其他模块的关系
- 概念层面的协作模式，详见 [[多智能体系统(Multi-Agent)#典型协作模式|多智能体系统]]。
- 单 Agent 主循环如何构成基础执行单元，详见 [[Harness Engineering/02_运行时循环与状态机#规范实现示例|运行时循环与状态机]]。
- 端到端骨架如何作为 orchestrator 的单元执行器，详见 [[Harness Engineering/07_端到端最小可用实现#架构目标|端到端最小可用实现]]。
