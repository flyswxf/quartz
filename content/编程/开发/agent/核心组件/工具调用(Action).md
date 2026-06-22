## 概述
工具调用（Tool Use / Function Calling）能力使 Agent 突破了单纯的文本生成，能够与外部环境交互。这是大语言模型从“静态知识库”走向“动态智能体”的核心机制。

在工程实现中，工具调用通常由 [[Harness Engineering/04_工具注册与执行层#工具执行层的职责|工具注册与执行层]] 负责参数校验、超时控制、审批和 observation 标准化，并由 [[Harness Engineering/02_运行时循环与状态机#运行时循环的最小闭环|运行时循环]] 将其编排进完整任务流程。

## 完整执行生命周期
当 Agent 收到一个需要外部信息的复杂问题时，完整的工具调用流程通常包含以下六个阶段：

### 1. 接收问题与上下文注入
系统接收用户的自然语言输入。此时，应用层会将**可用工具的定义**（包含工具名称、功能描述、参数 Schema）与用户问题一起封装成特定格式，发送给大模型。

### 2. 意图识别与工具触发 (Thought & Action)
大模型分析问题，发现自身知识无法解决，或者需要执行特定动作。模型决定挂起正常的文本生成，转而输出一个“工具调用请求”（包含目标工具的名称及严格符合 Schema 的参数，通常是 JSON 格式）。

### 3. 应用层接管 (Execution Pause)
应用层代码捕获到模型的“工具调用请求”，暂停与大模型的对话。此时模型处于等待状态。

### 4. 工具逻辑执行 (Tool Execution)
应用层根据模型输出的参数，在本地环境或云端安全地执行对应的真实函数（如发起 HTTP 请求、查询数据库、执行沙箱代码）。这一步必须包含**严格的错误处理**（如捕获超时、异常输入），以防崩溃。

### 5. 结果反馈 (Observation Injection)
工具执行完毕后，应用层将执行结果（通常转为字符串或 JSON）封装为特殊的消息角色（如 `role="tool"`），追加到对话历史中，再次发送给大模型。

### 6. 最终回答合成 (Final Response)
大模型读取到工具返回的结果，综合这些新信息进行推理，最终生成人类可读的自然语言回答。如果信息仍然不足，它可能再次触发步骤 2（即多次工具调用循环）。

## 规范与安全的实现示例
在生产环境中，工具调用需要处理异常、校验参数，并管理消息流状态。以下是基于 OpenAI 官方 `v1.x` SDK 的标准实现方式，展示了完整的闭环：

```python
import json
import logging
from typing import Dict, Any
from openai import OpenAI

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

client = OpenAI()

# 1. 定义工具的实际执行逻辑 (需包含错误处理)
def get_current_weather(location: str, unit: str = "celsius") -> str:
    """模拟调用外部天气 API"""
    try:
        # 生产环境中这里是真实的 HTTP 请求
        # response = requests.get(f"https://api.weather.com/v1?q={location}", timeout=5)
        # response.raise_for_status()
        
        # 模拟返回逻辑
        mock_data = {
            "Beijing": {"temp": 22, "condition": "Sunny"},
            "London": {"temp": 15, "condition": "Rainy"}
        }
        weather = mock_data.get(location, {"temp": 20, "condition": "Unknown"})
        return json.dumps({"location": location, "temperature": weather["temp"], "unit": unit})
    except Exception as e:
        logger.error(f"Weather API Error: {e}")
        return json.dumps({"error": "Failed to fetch weather data."})

# 工具映射表，便于应用层动态路由
AVAILABLE_TOOLS = {
    "get_current_weather": get_current_weather
}

# 2. 定义工具的 Schema (推荐参考 JSON Schema 规范)
TOOLS_SCHEMA = [
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "获取指定城市的当前天气信息。",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市名称，例如：Beijing, London",
                    },
                    "unit": {
                        "type": "string", 
                        "enum": ["celsius", "fahrenheit"],
                        "description": "温度单位"
                    },
                },
                "required": ["location"],
            },
        }
    }
]

def run_agent_conversation(user_query: str) -> str:
    """完整的 Agent 工具调用生命周期"""
    # 步骤 1: 构造初始消息
    messages = [{"role": "user", "content": user_query}]
    
    # 步骤 2: 发送请求，模型决定是否调用工具
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=TOOLS_SCHEMA,
        tool_choice="auto"  # 允许模型自主决策
    )
    
    response_message = response.choices[0].message
    
    # 分支 A: 模型认为无需工具，直接返回了回答
    if not response_message.tool_calls:
        return response_message.content
        
    # 分支 B: 模型发起了工具调用请求
    # 步骤 3: 必须将模型的调用请求完整加入历史记录，以保持对话连贯性
    messages.append(response_message)
    
    # 步骤 4: 遍历并执行所有工具调用请求 (支持并行调用)
    for tool_call in response_message.tool_calls:
        function_name = tool_call.function.name
        try:
            # 安全解析参数
            function_args = json.loads(tool_call.function.arguments)
        except json.JSONDecodeError:
            logger.error("Failed to parse tool arguments.")
            function_args = {}
            
        logger.info(f"Agent calling tool: {function_name} with args: {function_args}")
        
        # 路由到真实函数
        function_to_call = AVAILABLE_TOOLS.get(function_name)
        if function_to_call:
            function_response = function_to_call(**function_args)
        else:
            function_response = json.dumps({"error": f"Tool {function_name} not found."})
            
        # 步骤 5: 将工具执行结果作为 'tool' 角色注入对话历史
        messages.append({
            "tool_call_id": tool_call.id,
            "role": "tool",
            "name": function_name,
            "content": function_response,
        })
        
    # 步骤 6: 携带工具反馈再次请求模型，生成最终回答
    final_response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages
    )
    
    return final_response.choices[0].message.content

# 测试执行
# print(run_agent_conversation("What is the weather like in London today?"))
```
