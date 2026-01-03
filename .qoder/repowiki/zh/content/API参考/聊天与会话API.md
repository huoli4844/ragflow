# 聊天与会话API

<cite>
**本文档引用的文件**   
- [conversation_app.py](file://api/apps/conversation_app.py)
- [dialog_app.py](file://api/apps/dialog_app.py)
- [chat.py](file://api/apps/sdk/chat.py)
- [session.py](file://api/apps/sdk/session.py)
- [conversation_service.py](file://api/db/services/conversation_service.py)
- [dialog_service.py](file://api/db/services/dialog_service.py)
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py)
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py)
</cite>

## 目录
1. [简介](#简介)
2. [会话管理API](#会话管理api)
3. [聊天助手管理API](#聊天助手管理api)
4. [会话交互API](#会话交互api)
5. [流式响应处理](#流式响应处理)
6. [对话状态管理](#对话状态管理)
7. [与Agent和知识库集成](#与agent和知识库集成)
8. [SDK使用示例](#sdk使用示例)
9. [错误处理](#错误处理)

## 简介
RAGFlow系统提供了一套完整的聊天和会话管理API，支持创建和管理聊天会话、发送消息、查询对话历史以及管理会话状态。该API支持与知识库和Agent的集成，能够实现多轮对话和上下文保持。API提供了HTTP和Python SDK两种访问方式，支持流式响应，适用于各种应用场景。

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L1-L479)
- [dialog_app.py](file://api/apps/dialog_app.py#L1-L228)

## 会话管理API
会话管理API允许用户创建、查询、更新和删除聊天会话。每个会话都与一个聊天助手（chat assistant）关联，用于保持对话的上下文。

### 创建会话
通过POST请求创建新的会话，需要提供会话名称和关联的聊天助手ID。

**HTTP方法**: POST  
**URL路径**: `/api/v1/chats/{chat_id}/sessions`  
**请求体**:
```json
{
  "name": "New session"
}
```

**响应**:
```json
{
  "code": 0,
  "data": {
    "id": "session_id",
    "name": "New session",
    "messages": [
      {
        "role": "assistant",
        "content": "Hi! I'm your assistant. What can I do for you?"
      }
    ],
    "chat_id": "chat_assistant_id"
  }
}
```

### 查询会话列表
通过GET请求获取指定聊天助手下的所有会话列表。

**HTTP方法**: GET  
**URL路径**: `/api/v1/chats/{chat_id}/sessions`  
**查询参数**:
- `page`: 页码
- `page_size`: 每页大小
- `orderby`: 排序字段
- `desc`: 是否降序

**响应**:
```json
{
  "code": 0,
  "data": [
    {
      "id": "session_id",
      "name": "New session",
      "messages": [
        {
          "role": "assistant",
          "content": "Hi! I'm your assistant. What can I do for you?"
        }
      ],
      "chat_id": "chat_assistant_id"
    }
  ]
}
```

### 更新会话
通过PUT请求更新会话信息，如会话名称。

**HTTP方法**: PUT  
**URL路径**: `/api/v1/chats/{chat_id}/sessions/{session_id}`  
**请求体**:
```json
{
  "name": "Updated session name"
}
```

### 删除会话
通过DELETE请求删除一个或多个会话。

**HTTP方法**: DELETE  
**URL路径**: `/api/v1/chats/{chat_id}/sessions`  
**请求体**:
```json
{
  "ids": ["session_id1", "session_id2"]
}
```

**Section sources**
- [session.py](file://api/apps/sdk/session.py#L47-L690)
- [conversation_service.py](file://api/db/services/conversation_service.py#L32-L49)

## 聊天助手管理API
聊天助手管理API允许用户创建、查询、更新和删除聊天助手。聊天助手是会话的容器，定义了对话的上下文和行为。

### 创建聊天助手
通过POST请求创建新的聊天助手。

**HTTP方法**: POST  
**URL路径**: `/api/v1/chats`  
**请求体**:
```json
{
  "name": "My Assistant",
  "llm": {
    "model_name": "gpt-3.5-turbo"
  },
  "prompt": {
    "system": "You are a helpful assistant.",
    "opener": "Hi! How can I help you?",
    "show_quote": true
  },
  "dataset_ids": ["dataset_id1", "dataset_id2"]
}
```

### 查询聊天助手列表
通过GET请求获取用户的所有聊天助手列表。

**HTTP方法**: GET  
**URL路径**: `/api/v1/chats`  
**查询参数**:
- `page`: 页码
- `page_size`: 每页大小
- `orderby`: 排序字段
- `desc`: 是否降序

### 更新聊天助手
通过PUT请求更新聊天助手的配置。

**HTTP方法**: PUT  
**URL路径**: `/api/v1/chats/{chat_id}`  
**请求体**:
```json
{
  "name": "Updated Assistant Name",
  "llm": {
    "model_name": "gpt-4"
  }
}
```

### 删除聊天助手
通过DELETE请求删除一个或多个聊天助手。

**HTTP方法**: DELETE  
**URL路径**: `/api/v1/chats`  
**请求体**:
```json
{
  "ids": ["chat_id1", "chat_id2"]
}
```

**Section sources**
- [chat.py](file://api/apps/sdk/chat.py#L27-L267)
- [dialog_app.py](file://api/apps/dialog_app.py#L30-L228)

## 会话交互API
会话交互API允许用户向会话发送消息并接收响应。API支持流式响应，可以实时获取模型的输出。

### 发送消息
通过POST请求向会话发送消息。

**HTTP方法**: POST  
**URL路径**: `/api/v1/chats/{chat_id}/completions`  
**请求体**:
```json
{
  "question": "What is the weather like today?",
  "session_id": "session_id",
  "stream": true
}
```

**请求参数**:
- `question`: 用户的问题
- `session_id`: 会话ID
- `stream`: 是否使用流式响应
- `llm_id`: 指定使用的LLM模型ID
- `temperature`, `top_p`, `frequency_penalty`, `presence_penalty`, `max_tokens`: LLM模型参数

### 流式响应格式
当`stream`参数为`true`时，API返回流式响应，每个数据块包含部分响应内容。

```text
data:{"code":0,"message":"","data":{"answer":"Hello","reference":{},"audio_binary":null,"id":null,"session_id":"session_id"}}

data:{"code":0,"message":"","data":{"answer":"Hello world","reference":{},"audio_binary":null,"id":null,"session_id":"session_id"}}

data:{"code":0,"message":"","data":{"answer":"Hello world!","reference":{"chunks":[],"doc_aggs":[]},"audio_binary":null,"id":"message_id","session_id":"session_id"}}

data:{"code":0,"message":"","data":true}
```

**Section sources**
- [session.py](file://api/apps/sdk/session.py#L122-L149)
- [conversation_app.py](file://api/apps/conversation_app.py#L168-L252)

## 流式响应处理
流式响应允许客户端实时接收模型的输出，提供更好的用户体验。客户端需要处理SSE（Server-Sent Events）格式的响应。

### 流式响应处理示例
```python
import requests

def handle_streaming_response(response):
    for line in response.iter_lines():
        if line:
            line = line.decode('utf-8')
            if line.startswith('data:'):
                data = line[5:]
                if data == '[DONE]':
                    break
                try:
                    json_data = json.loads(data)
                    if 'data' in json_data and json_data['data'] is not True:
                        print(json_data['data']['answer'])
                except json.JSONDecodeError:
                    continue
```

### 响应头设置
服务器在流式响应中设置了以下响应头：
- `Cache-control: no-cache`
- `Connection: keep-alive`
- `X-Accel-Buffering: no`
- `Content-Type: text/event-stream; charset=utf-8`

这些设置确保了流式响应能够正确传输，避免了代理服务器的缓冲问题。

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L234-L240)
- [session.py](file://api/apps/sdk/session.py#L136-L141)

## 对话状态管理
系统通过会话（session）机制管理对话状态，每个会话独立保存对话历史和上下文。

### 会话状态结构
每个会话包含以下信息：
- `id`: 会话唯一标识
- `name`: 会话名称
- `messages`: 消息列表，包含用户和助手的对话历史
- `chat_id`: 关联的聊天助手ID
- `reference`: 引用信息，包含知识库检索结果

### 上下文保持机制
系统通过以下方式保持对话上下文：
1. 在发送请求时，将完整的对话历史（除系统消息外）包含在请求中
2. 系统自动过滤系统消息和无效的助手消息
3. 将最新的用户消息作为当前问题处理
4. 在响应中返回更新后的完整对话历史

### 消息过滤规则
系统在处理消息时应用以下过滤规则：
- 忽略角色为"system"的消息
- 忽略第一个角色为"assistant"且前面没有用户消息的消息
- 保留所有其他消息以保持对话上下文

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L173-L179)
- [conversation_service.py](file://api/db/services/conversation_service.py#L125-L137)

## 与Agent和知识库集成
系统支持与Agent和知识库的深度集成，提供增强的聊天功能。

### Agent集成
通过Agent集成，可以实现更复杂的对话流程和功能扩展。

**HTTP方法**: POST  
**URL路径**: `/api/v1/agents/{agent_id}/completions`  
**请求体**:
```json
{
  "question": "What is the weather like today?",
  "session_id": "session_id",
  "stream": true
}
```

### 知识库集成
聊天助手可以关联一个或多个知识库，系统会自动从知识库中检索相关信息来回答用户问题。

#### 检索流程
1. 用户发送问题
2. 系统使用嵌入模型将问题转换为向量
3. 在知识库中进行向量相似度搜索
4. 结合关键词匹配和重排序模型对结果进行排序
5. 将检索到的相关内容作为上下文提供给LLM
6. LLM生成最终回答

#### 检索参数
- `top_n`: 返回的最相关片段数量
- `similarity_threshold`: 相似度阈值，低于此值的片段将被过滤
- `vector_similarity_weight`: 向量相似度在混合评分中的权重
- `rerank_id`: 重排序模型ID

### OpenAI兼容API
系统提供了与OpenAI API兼容的接口，方便现有应用迁移。

**HTTP方法**: POST  
**URL路径**: `/api/v1/chats_openai/{chat_id}/chat/completions`  
**请求体**:
```json
{
  "model": "model",
  "messages": [
    {"role": "user", "content": "Say this is a test!"}
  ],
  "stream": true,
  "reference": true
}
```

**Section sources**
- [session.py](file://api/apps/sdk/session.py#L151-L343)
- [dialog_service.py](file://api/db/services/dialog_service.py#L352-L638)

## SDK使用示例
Python SDK提供了更便捷的API访问方式，封装了底层HTTP请求。

### 创建聊天助手
```python
from ragflow_sdk import RAGFlow

client = RAGFlow(api_key="your_api_key", base_url="http://your_ragflow_address")
chat = client.create_chat({
    "name": "My Assistant",
    "llm": {"model_name": "gpt-3.5-turbo"},
    "prompt": {
        "system": "You are a helpful assistant.",
        "opener": "Hi! How can I help you?"
    },
    "dataset_ids": ["dataset_id1"]
})
```

### 创建会话并发送消息
```python
# 创建会话
session = chat.create_session("My Session")

# 发送消息（流式）
for message in session.ask("What is the weather like today?", stream=True):
    print(message.content)

# 发送消息（非流式）
response = session.ask("What is the weather like today?", stream=False)
print(response.content)
```

### 查询会话列表
```python
sessions = chat.list_sessions()
for session in sessions:
    print(f"Session: {session.name}, ID: {session.id}")
```

**Section sources**
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py#L22-L88)
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py#L21-L129)

## 错误处理
API提供了详细的错误处理机制，帮助开发者快速定位和解决问题。

### 常见错误码
| 错误码 | 错误信息 | 说明 |
|-------|--------|------|
| 0 | Success | 成功 |
| 100 | Bad Request | 请求参数错误 |
| 102 | Invalid parameter | 无效参数 |
| 109 | Authentication error | 认证错误 |
| 500 | Internal Server Error | 服务器内部错误 |

### 错误响应格式
```json
{
  "code": 102,
  "message": "Conversation not found!"
}
```

### 错误处理最佳实践
1. 检查HTTP状态码和响应体中的错误码
2. 根据错误信息进行相应的处理
3. 对于认证错误，检查API密钥是否正确
4. 对于参数错误，验证请求参数是否符合要求
5. 对于服务器错误，可以尝试重试或联系技术支持

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L40-L61)
- [chat.py](file://api/apps/sdk/chat.py#L48-L55)