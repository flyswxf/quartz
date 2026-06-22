## 概述
规划是 Agent 将复杂目标分解为可执行步骤的能力。面对复杂任务时，LLM 需要通过系统性的思考路径来避免盲目尝试。

在工程系统中，规划通常不是孤立执行，而是由 [[Harness Engineering/02_运行时循环与状态机#运行时循环的最小闭环|运行时循环]] 驱动，并通过 [[Harness Engineering/03_上下文工程(Context Engineering)#上下文装配的目标|上下文工程]] 持续接收最新的工具观察结果与记忆上下文。

## 任务分解 (Task Decomposition)
将大问题拆解为小问题。

### 1. 链式思考 (Chain of Thought, CoT)
引导模型逐步输出推理过程，而不仅仅是最终答案。
- **原理**：提示模型“Think step by step”。
- **优势**：显著提高数学、逻辑推理任务的准确性。

### 2. 思维树 (Tree of Thoughts, ToT)
在 CoT 的基础上，在每一步探索多个可能的分支，并评估这些分支的可行性。
- **机制**：结合搜索算法（如 BFS、DFS）和启发式评估。

### 3. Plan-and-Solve
在执行任何动作之前，先显式地生成一个完整的计划，然后按计划执行。

## 反思与自我修正 (Reflection & Self-Correction)
Agent 需要评估自己的执行结果，并在出错时进行修正。

### ReAct (Reason + Act)
最经典的 Agent 模式之一，交替进行推理和动作。
- **工作流**：`Thought` $\rightarrow$ `Action` $\rightarrow$ `Observation` $\rightarrow$ `Thought` ...

### Reflexion
通过语言反馈来强化 Agent，而不是更新模型权重。Agent 会反思之前失败的尝试，生成改进建议（自我反思），并在下一次尝试中应用。

## 规范的 ReAct 实现示例
以下是一个标准的、带有安全限制机制（防止无限循环）的 ReAct 基础实现逻辑：

```python
import re
import logging
from typing import Callable, Dict

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 模拟的大语言模型调用函数
def llm_generate(prompt: str) -> str:
    # 生产环境中此处为 openai.chat.completions.create(...)
    pass

class ReActAgent:
    def __init__(self, tools: Dict[str, Callable], max_steps: int = 5):
        """
        初始化 ReAct Agent
        :param tools: 可用工具的字典映射 {工具名: 执行函数}
        :param max_steps: 最大思考与执行步数，防止无限循环
        """
        self.tools = tools
        self.max_steps = max_steps
        self.system_prompt = """
You run in a loop of Thought, Action, PAUSE, Observation.
At the end of the loop you output an Answer.

Use Thought to describe your thoughts about the question you have been asked.
Use Action to run one of the actions available to you - then return PAUSE.
Observation will be the result of running those actions.

Available actions:
{tool_descriptions}

Example session:
Question: What is the capital of France?
Thought: I should look up France on Wikipedia
Action: wikipedia: France
PAUSE
Observation: France is a country. The capital is Paris.
Answer: The capital of France is Paris
"""

    def _get_tool_descriptions(self) -> str:
        return "\n".join([f"- {name}: {func.__doc__}" for name, func in self.tools.items()])

    def run(self, query: str) -> str:
        """执行 ReAct 循环"""
        prompt = self.system_prompt.replace("{tool_descriptions}", self._get_tool_descriptions())
        prompt += f"\n\nQuestion: {query}"
        
        # Action 的正则匹配模式
        action_re = re.compile(r"^Action: (\w+): (.*)$", re.MULTILINE)

        for step in range(self.max_steps):
            logger.info(f"--- Step {step + 1} ---")
            
            # 1. 触发大模型进行 Thought 和 Action
            response = llm_generate(prompt)
            logger.info(f"LLM Output:\n{response}")
            
            prompt += f"\n{response}"

            # 2. 检查是否已经得出最终答案
            if "Answer:" in response:
                return response.split("Answer:")[-1].strip()

            # 3. 提取 Action
            match = action_re.search(response)
            if not match:
                logger.warning("No Action found or invalid format. Forcing a retry.")
                prompt += "\nObservation: Invalid format. Please provide an Action or Answer."
                continue
                
            action_name, action_input = match.groups()
            action_name = action_name.strip()
            action_input = action_input.strip()
            
            # 4. 执行工具逻辑，确保异常安全
            if action_name not in self.tools:
                observation = f"Error: Tool {action_name} not found."
            else:
                try:
                    observation = self.tools[action_name](action_input)
                except Exception as e:
                    logger.error(f"Tool execution failed: {e}")
                    observation = f"Error during execution: {e}"
            
            logger.info(f"Observation: {observation}")
            
            # 5. 注入结果进入下一轮
            prompt += f"\nObservation: {observation}"

        return "Error: Agent reached maximum steps without finding an answer."

# 测试用例
# def calculate(expr: str) -> str:
#     """Evaluate a math expression."""
#     return str(eval(expr))  # 注意：生产环境中禁止直接使用 eval，应使用安全的数学解析库
#
# agent = ReActAgent(tools={"calculate": calculate})
# print(agent.run("What is 15 * 7?"))
```
