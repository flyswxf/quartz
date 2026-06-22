## 概述
单体 Agent 在处理复杂任务时，容易受限于单一视角或 Prompt 长度限制导致效果不佳。多智能体系统（Multi-Agent Systems, MAS）通过引入多个相互协作的 Agent 来解决这一问题。

从工程角度看，多智能体并不是单体 Agent 的简单复制，而是需要在 [[Harness Engineering/01_概述与设计目标#分层结构|Harness Engineering]] 之上增加角色路由、会话隔离、共享状态和跨 Agent 观测能力。

如果重点是“多个 Agent 如何被系统编排起来”，可以继续看 [[Harness Engineering/子专题/05_Multi-Agent Orchestration#概述|Multi-Agent Orchestration]]。

## 核心优势
1. **分工明确**：不同的 Agent 可以扮演不同的角色（Persona），拥有不同的系统提示（System Prompt）和专用工具。
2. **相互监督与评审**：一个 Agent 生成结果，另一个 Agent 负责评审（Critique），从而减少幻觉和错误。
3. **状态隔离**：每个 Agent 维护自己的记忆和上下文，避免单一 Agent 上下文过载。

## 典型协作模式

### 1. 协作式 (Collaborative)
Agent 之间共同努力实现同一个目标，例如软件开发。
- **角色设定**：如 MetaGPT 中的 Product Manager, Architect, Engineer。
- **流程**：PM 负责需求分析 $\rightarrow$ Architect 负责系统设计 $\rightarrow$ Engineer 负责编写代码。

### 2. 辩论/竞争式 (Debate/Competitive)
Agent 之间持有不同观点或目标，通过辩论得出更优的结论。
- **优势**：强制模型从多个角度思考，有效降低确认偏误（Confirmation Bias）。

### 3. 层级式 (Hierarchical)
引入“主从”架构。
- **Manager Agent**：负责理解总体意图、任务拆解和任务分发。
- **Worker Agent**：负责执行具体的子任务，并将结果汇报给 Manager。

## 规范的 AutoGen 多智能体代码示例
以下是一个使用微软 AutoGen 框架构建的双智能体（助手 + 执行者）交互示例。为了安全起见，代码执行环境被显式地限制在指定的本地目录，并禁用了危险的全局沙箱操作：

```python
import os
import autogen

# 1. 严格配置 LLM 接口
config_list = [
    {
        "model": "gpt-4o",
        "api_key": os.environ.get("OPENAI_API_KEY")
    }
]

# 2. 创建 Assistant Agent (大脑/规划者)
# 负责编写代码或提供解决方案，但不执行代码。
assistant = autogen.AssistantAgent(
    name="Data_Analyst_Assistant",
    llm_config={
        "config_list": config_list,
        "temperature": 0.2, # 降低温度以获得更稳定的代码输出
    },
    system_message="你是一个专业的数据分析师。编写 Python 代码解决用户的问题。当任务完全解决且验证无误时，回复 'TERMINATE'。"
)

# 3. 创建 UserProxy Agent (执行者/环境交互者)
# 负责代替人类执行 Assistant 编写的代码，并将运行结果返回给 Assistant。
user_proxy = autogen.UserProxyAgent(
    name="Human_Proxy",
    human_input_mode="NEVER", # 完全自动模式，无需人工干预
    max_consecutive_auto_reply=10, # 设置最大交互轮数防止死循环
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
    code_execution_config={
        "work_dir": "safe_workspace", # 【安全约束】强制在独立子目录中执行代码
        "use_docker": False # 生产环境强烈建议设为 True 以隔离运行环境
    }
)

# 4. 启动多智能体协作流程
def main():
    print("Initiating multi-agent conversation...")
    user_proxy.initiate_chat(
        assistant,
        message="获取苹果公司(AAPL)最近一个月的股票价格，并绘制折线图保存为 aapl_stock.png。"
    )

# if __name__ == "__main__":
#     main()
```
