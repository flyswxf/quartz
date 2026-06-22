## 概述
Human-in-the-Loop（HITL）指的是在 Agent 执行链路中引入人工决策节点，使系统在关键步骤上不是“全自动执行”，而是“自动推进 + 人工把关”。Approval Workflow 则是这种人工介入机制在工程系统中的具体流程设计。

在真实系统中，很多失败并不是因为模型不会回答，而是因为系统让模型在不该自动决策的地方自动决策了。例如：
- 删除文件。
- 修改数据库。
- 发送邮件。
- 执行交易。
- 调用高成本 API。

因此，HITL 的核心价值不是“让人来替模型思考”，而是“把高风险决策从模型自主执行中剥离出来”。

## 为什么它重要

### 1. 降低副作用风险
对读操作来说，错误通常只是答案错误；对写操作来说，错误可能直接产生不可逆副作用。

### 2. 降低责任不清
审批链条可以明确记录：谁批准了什么动作、基于什么 observation 批准、审批发生在什么时间。

### 3. 支持渐进式自动化
一个系统可以先对高风险动作启用人工审批，后续在评测充分后再逐步自动化。

## 哪些动作应进入审批

### 1. 强副作用操作
- 删除或覆盖文件。
- 写数据库。
- 调用支付、通知、部署类接口。

### 2. 高不确定性操作
- 模型置信度低但仍建议执行的动作。
- 工具 observation 含糊不清的情况。
- 上下文中存在明显冲突的情况。

### 3. 高敏感数据操作
- 读取用户隐私数据。
- 导出内部文档。
- 显示或转发受限信息。

## Approval Workflow 的最小状态机
审批流通常可抽象为以下状态：

1. `proposed`：Agent 提出动作建议。
2. `pending_approval`：系统冻结执行，等待人工审批。
3. `approved`：审批通过，允许进入执行层。
4. `rejected`：审批拒绝，动作终止或回退。
5. `expired`：超时未审批，自动失效。

可以写成：
$$Proposal \rightarrow Pending \rightarrow \{Approved,\ Rejected,\ Expired\}$$

## 设计原则

### 1. 审批对象必须结构化
不能只把一段自然语言交给审批人，而应包含：
- 工具名称。
- 参数。
- 风险等级。
- 触发原因。
- 相关 observation。
- 预期副作用。

### 2. 审批应早于执行
审批不是事后审计，而是执行前门控。

### 3. 审批结果要进入主状态
审批通过或拒绝都应回写到 harness 状态中，供后续回答和审计使用。

### 4. 要有超时与默认策略
审批请求不能无限挂起，应支持超时失效、自动拒绝或转人工接管队列。

## 与主循环的关系
HITL 并不是独立系统，而是 [[Harness Engineering/02_运行时循环与状态机#运行时循环的最小闭环|运行时循环]] 中的一条条件分支：

1. 模型提出工具调用请求。
2. 策略层判断是否需要审批。
3. 若需要，则生成 proposal 并暂停主循环。
4. 审批通过后恢复执行。
5. 审批拒绝则将 rejection observation 回填给模型或终止任务。

## 规范实现示例
下面给出一个简化但工程上合理的审批工作流骨架。重点是结构化 proposal、状态持久化接口和恢复执行的设计。

```python
from __future__ import annotations

import time
import uuid
from dataclasses import dataclass
from enum import Enum
from typing import Any, Protocol


class ApprovalStatus(str, Enum):
    PROPOSED = "proposed"
    PENDING = "pending_approval"
    APPROVED = "approved"
    REJECTED = "rejected"
    EXPIRED = "expired"


@dataclass(slots=True)
class ApprovalProposal:
    proposal_id: str
    run_id: str
    tool_name: str
    arguments: dict[str, Any]
    reason: str
    risk_level: str
    created_at: float
    status: ApprovalStatus


class ApprovalBackend(Protocol):
    def save(self, proposal: ApprovalProposal) -> None:
        ...

    def get(self, proposal_id: str) -> ApprovalProposal | None:
        ...

    def update_status(self, proposal_id: str, status: ApprovalStatus) -> None:
        ...


class InMemoryApprovalBackend:
    def __init__(self) -> None:
        self._store: dict[str, ApprovalProposal] = {}

    def save(self, proposal: ApprovalProposal) -> None:
        self._store[proposal.proposal_id] = proposal

    def get(self, proposal_id: str) -> ApprovalProposal | None:
        return self._store.get(proposal_id)

    def update_status(self, proposal_id: str, status: ApprovalStatus) -> None:
        if proposal_id in self._store:
            self._store[proposal_id].status = status


class ApprovalManager:
    def __init__(self, backend: ApprovalBackend, ttl_seconds: int = 600) -> None:
        self.backend = backend
        self.ttl_seconds = ttl_seconds

    def create_proposal(
        self,
        *,
        run_id: str,
        tool_name: str,
        arguments: dict[str, Any],
        reason: str,
        risk_level: str,
    ) -> ApprovalProposal:
        proposal = ApprovalProposal(
            proposal_id=str(uuid.uuid4()),
            run_id=run_id,
            tool_name=tool_name,
            arguments=arguments,
            reason=reason,
            risk_level=risk_level,
            created_at=time.time(),
            status=ApprovalStatus.PENDING,
        )
        self.backend.save(proposal)
        return proposal

    def resolve(self, proposal_id: str) -> ApprovalStatus:
        proposal = self.backend.get(proposal_id)
        if proposal is None:
            raise ValueError("proposal_not_found")

        if proposal.status == ApprovalStatus.PENDING:
            if time.time() - proposal.created_at > self.ttl_seconds:
                self.backend.update_status(proposal_id, ApprovalStatus.EXPIRED)

        latest = self.backend.get(proposal_id)
        if latest is None:
            raise ValueError("proposal_not_found_after_update")
        return latest.status


def requires_approval(tool_name: str, risk_level: str) -> bool:
    high_risk_tools = {"delete_file", "write_file", "send_email", "update_database"}
    return tool_name in high_risk_tools or risk_level in {"high", "critical"}
```

## 如何接进主 harness
若要接入 [[Harness Engineering/07_端到端最小可用实现#完整示例代码|端到端最小可用实现]]，可以在真正执行工具之前插入审批判断：

```python
if requires_approval(tool_name, risk_level):
    proposal = approval_manager.create_proposal(
        run_id=state.run_id,
        tool_name=tool_name,
        arguments=arguments,
        reason="high_risk_side_effect",
        risk_level=risk_level,
    )
    state.error = f"pending_approval:{proposal.proposal_id}"
    return state
```

审批通过后，再恢复运行：

```python
status = approval_manager.resolve(proposal_id)
if status == ApprovalStatus.APPROVED:
    observation = tool_registry.execute(tool_name, raw_arguments)
elif status == ApprovalStatus.REJECTED:
    observation = '{"ok": false, "error": "approval_rejected"}'
else:
    observation = '{"ok": false, "error": "approval_not_ready"}'
```

## 人工接管模式
HITL 不只有“点同意/点拒绝”这一种形态，还可以扩展为：

### 1. 审批模式
人工只决定“能不能执行”。

### 2. 修改模式
人工可以修改参数，例如把错误路径改成正确目录。

### 3. 接管模式
人工直接接手当前会话，后续不再由 Agent 自动推进。

## 工程实践建议
- 对所有高风险 proposal 生成唯一 `proposal_id`，便于审计。
- 审批页上显示工具参数 diff，而不是原始 JSON 大段文本。
- 拒绝时尽量回写结构化原因，让模型知道是“策略拒绝”还是“参数不合理”。
- 对长时间无响应的审批请求设置超时失效和提醒机制。

## 与其他模块的关系
- 风险分级与策略判断，详见 [[Harness Engineering/05_可靠性与安全#高风险动作的防护|可靠性与安全]]。
- 主循环的暂停与恢复，详见 [[Harness Engineering/02_运行时循环与状态机#退出条件|运行时循环与状态机]]。
- 端到端接入方式，详见 [[Harness Engineering/07_端到端最小可用实现#为什么这是“最小可用”|端到端最小可用实现]]。
