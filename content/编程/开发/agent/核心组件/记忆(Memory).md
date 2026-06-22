## 概述
记忆机制赋予 Agent 状态（State），使其能够理解历史对话和过去的经验，从而做出更连贯的决策。

在工程实现中，记忆系统通常不会直接暴露给模型，而是先经过 [[Harness Engineering/03_上下文工程(Context Engineering)#组织原则|上下文工程]] 的筛选、压缩与来源标记，再进入主上下文。

## 记忆类型与生命周期

### 1. 短期记忆 (Short-Term Memory)
指模型当前上下文窗口内的信息，主要用于维持单次会话的连贯性。
- **实现方式**：直接将历史对话记录（Message History）拼接到 Prompt 中发送给模型。
- **限制**：
  - 严格受限于大语言模型的 Context Window 长度（如 8K, 128K）。
  - 上下文过长会导致推理成本线性增加，并容易引发“中间迷失”（Lost in the Middle）现象。
- **优化策略**：
  - **滑动窗口（Sliding Window）**：仅保留最近 $N$ 轮对话。
  - **动态摘要（Summary Memory）**：当对话达到长度阈值时，使用 LLM 将早期的对话提炼为一段简短的摘要，再拼接到新的上下文中。

### 2. 长期记忆 (Long-Term Memory)
允许 Agent 跨越会话边界，在很长一段时间内保留和回忆海量信息（如知识库、用户画像）。
- **实现方式**：绝大多数长期记忆依赖于外部存储系统，最主流的实现是使用 **[[向量数据库(Vector DB)#概述|向量数据库]]**。
- **工作机制**：
  1. **写入**：将文本信息转化为高维向量（Embeddings），存入数据库。
  2. **检索**：将用户的当前 Query 同样转化为向量，通过近似最近邻（ANN）搜索，找出最相似的历史记忆片段。
  3. **注入**：将检索到的记忆作为上下文（Context）注入到 Prompt 中。

### 3. 实体与关系记忆 (Entity / Graph Memory)
基于知识图谱（Knowledge Graph）的记忆机制。
- 专门提取和存储关于特定实体（人、地点、概念）的关系。相比向量检索的模糊匹配，图谱记忆能提供更精确的逻辑关联。

## 规范的记忆存储与检索代码示例
以下是一个使用标准 `chromadb` 和 OpenAI Embeddings 进行长期记忆存储和检索的安全实现：

```python
import os
import chromadb
from chromadb.utils import embedding_functions
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class AgentMemory:
    def __init__(self, db_path: str = "./agent_memory_db"):
        """初始化向量数据库作为长期记忆载体"""
        try:
            # 建立本地持久化客户端
            self.client = chromadb.PersistentClient(path=db_path)
            
            # 使用 OpenAI 官方的 Embedding 函数
            self.embedding_func = embedding_functions.OpenAIEmbeddingFunction(
                api_key=os.environ.get("OPENAI_API_KEY"),
                model_name="text-embedding-3-small"
            )
            
            # 创建或获取集合 (Collection)
            self.collection = self.client.get_or_create_collection(
                name="user_interactions",
                embedding_function=self.embedding_func,
                metadata={"hnsw:space": "cosine"} # 使用余弦相似度
            )
            logger.info("Agent memory initialized successfully.")
        except Exception as e:
            logger.error(f"Failed to initialize memory: {e}")
            raise

    def add_memory(self, memory_id: str, content: str, metadata: dict = None):
        """安全地写入新记忆"""
        try:
            self.collection.add(
                documents=[content],
                metadatas=[metadata or {}],
                ids=[memory_id]
            )
            logger.info(f"Memory {memory_id} saved.")
        except Exception as e:
            logger.error(f"Error saving memory: {e}")

    def recall(self, query: str, top_k: int = 3) -> list:
        """检索相关的历史记忆"""
        try:
            results = self.collection.query(
                query_texts=[query],
                n_results=top_k
            )
            # 提取文档内容
            documents = results.get("documents", [[]])[0]
            return documents
        except Exception as e:
            logger.error(f"Error recalling memory: {e}")
            return []

# 使用示例
# memory_sys = AgentMemory()
# memory_sys.add_memory("mem_001", "用户是一名物理对抗攻击方向的研究员。", {"source": "user_profile"})
# context = memory_sys.recall("用户研究什么方向？")
# print("Retrieved Context:", context)
```
