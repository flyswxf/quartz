## 概述
Evaluation Harness 指的是用于系统性评估 Agent 的测试与比较框架。它不是单纯的“跑几个问题看看效果”，而是把样例、执行过程、评分逻辑、统计结果和回放能力组织起来，使不同模型、不同 prompt、不同 harness 策略可以被稳定比较。

它与 [[Harness Engineering/06_评测与可观测性#评测的目标|评测与可观测性]] 的关系是：可观测性负责记录系统做了什么，Evaluation Harness 负责判断系统做得好不好。

## 为什么单看最终答案不够
Agent 与普通问答模型不同，它的中间过程本身就是质量的一部分：
- 是否调用了正确工具。
- 是否多调用了不必要的工具。
- 是否在错误 observation 下及时恢复。
- 是否遵守了安全和预算约束。

因此，Agent 评测通常至少要覆盖：
- 最终答案质量。
- 过程质量。
- 策略合规性。
- 资源效率。

## 离线评测的基本结构
一个可维护的离线评测框架通常包含五部分：

### 1. 数据集
每条样本至少包含：
- `input`：用户问题。
- `expected_answer` 或 `reference_facts`。
- `allowed_tools`。
- `risk_tags`。
- `metadata`：难度、领域、语言、是否需要检索等。

### 2. 执行器
给定同一份样本，分别运行不同 harness 配置，例如：
- 不同模型。
- 不同系统提示。
- 不同记忆策略。
- 不同工具选择策略。

### 3. 评分器
评分器可以分为：
- 基于规则的评分。
- 基于 LLM-as-a-Judge 的评分。
- 混合评分。

### 4. 结果存储
保存每个样本的：
- 最终答案。
- 工具调用序列。
- 步数。
- 延迟。
- 错误码。

### 5. 聚合分析
按任务类型、风险等级、工具种类、语言类型等维度做切片统计。

## 关键指标
- **Answer Correctness**：答案是否满足任务要求。
- **Tool Selection Accuracy**：工具选择是否正确。
- **Budget Efficiency**：是否在合理步数和调用次数内完成。
- **Recovery Capability**：发生工具错误后能否恢复。
- **Policy Compliance**：是否违反安全或审批策略。

## 常见评测陷阱
- **只看均值**：平均分高不代表尾部失败少。
- **只看 final answer**：可能掩盖了大量越权工具调用。
- **评测集过小**：无法发现稳定性问题。
- **Judge 泄漏偏好**：LLM 评分器可能偏爱特定表述风格。

## 规范实现示例
下面给出一个最小版 Evaluation Harness 示例。它的目标是批量执行样本，并同时统计答案分数和工具调用合规性。

```python
from __future__ import annotations

from dataclasses import dataclass
from statistics import mean
from typing import Any


@dataclass(slots=True)
class EvalSample:
    sample_id: str
    user_input: str
    reference_keywords: list[str]
    allowed_tools: set[str]


@dataclass(slots=True)
class EvalResult:
    sample_id: str
    final_answer: str
    tool_names: list[str]
    step_count: int
    passed: bool
    answer_score: float
    policy_score: float


def score_answer(final_answer: str, reference_keywords: list[str]) -> float:
    if not reference_keywords:
        return 1.0
    matched = sum(1 for keyword in reference_keywords if keyword.lower() in final_answer.lower())
    return matched / len(reference_keywords)


def score_policy(tool_names: list[str], allowed_tools: set[str]) -> float:
    if not tool_names:
        return 1.0
    illegal_calls = sum(1 for name in tool_names if name not in allowed_tools)
    return 1.0 - illegal_calls / len(tool_names)


def evaluate_samples(harness, samples: list[EvalSample]) -> dict[str, Any]:
    results: list[EvalResult] = []

    for sample in samples:
        state = harness.run(sample.user_input)
        tool_names = [
            msg.get("name", "")
            for msg in state.history
            if msg.get("role") == "tool"
        ]
        answer = state.final_answer or ""
        answer_score = score_answer(answer, sample.reference_keywords)
        policy_score = score_policy(tool_names, sample.allowed_tools)
        passed = answer_score >= 0.6 and policy_score == 1.0 and state.error is None

        results.append(
            EvalResult(
                sample_id=sample.sample_id,
                final_answer=answer,
                tool_names=tool_names,
                step_count=state.step_index,
                passed=passed,
                answer_score=answer_score,
                policy_score=policy_score,
            )
        )

    summary = {
        "pass_rate": mean(1.0 if item.passed else 0.0 for item in results) if results else 0.0,
        "avg_answer_score": mean(item.answer_score for item in results) if results else 0.0,
        "avg_policy_score": mean(item.policy_score for item in results) if results else 0.0,
        "avg_step_count": mean(item.step_count for item in results) if results else 0.0,
        "results": results,
    }
    return summary
```

## 更稳妥的实践方式
- 对不同任务类型分别建子测试集，例如检索型、工具型、规划型、长上下文型。
- 同时保存“最终答案”和“完整运行 trace”，便于失败后回放。
- 对高风险场景单独统计 `policy violation`，不要被总体平均分稀释。

## 与其他模块的关系
- 端到端执行入口，详见 [[Harness Engineering/07_端到端最小可用实现#完整示例代码|端到端最小可用实现]]。
- 结构化 trace 的采集方式，详见 [[Harness Engineering/06_评测与可观测性#规范实现示例|评测与可观测性]]。
- 高风险策略判定，详见 [[Harness Engineering/05_可靠性与安全#高风险动作的防护|可靠性与安全]]。
