# Python SDK参考

<cite>
**本文档中引用的文件**   
- [ragflow.py](file://sdk/python/ragflow_sdk/ragflow.py)
- [dataset.py](file://sdk/python/ragflow_sdk/modules/dataset.py)
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py)
- [document.py](file://sdk/python/ragflow_sdk/modules/document.py)
- [chunk.py](file://sdk/python/ragflow_sdk/modules/chunk.py)
- [agent.py](file://sdk/python/ragflow_sdk/modules/agent.py)
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py)
- [base.py](file://sdk/python/ragflow_sdk/modules/base.py)
- [hello_ragflow.py](file://sdk/python/hello_ragflow.py)
- [pyproject.toml](file://sdk/python/pyproject.toml)
- [python_api_reference.md](file://docs/references/python_api_reference.md)
</cite>

## 目录
1. [安装指南与版本兼容性](#安装指南与版本兼容性)
2. [RAGFlow类](#ragflow类)
3. [数据集模块](#数据集模块)
4. [聊天助手模块](#聊天助手模块)
5. [文档模块](#文档模块)
6. [分块模块](#分块模块)
7. [智能体模块](#智能体模块)
8. [会话模块](#会话模块)
9. [典型使用示例](#典型使用示例)
10. [异常处理](#异常处理)

## 安装指南与版本兼容性

要使用RAGFlow Python SDK，可以通过pip安装：

```bash
pip install ragflow-sdk
```

根据`pyproject.toml`文件中的定义，SDK需要Python版本在3.12到3.15之间。SDK的主要依赖是`requests>=2.30.0,<3.0.0`和`beartype>=0.20.0,<1.0.0`。

SDK的当前版本为0.22.1，可以通过以下方式查看：

```python
import ragflow_sdk
print(ragflow_sdk.__version__)
```

**Section sources**
- [pyproject.toml](file://sdk/python/pyproject.toml#L1-L32)
- [hello_ragflow.py](file://sdk/python/hello_ragflow.py#L1-L20)

## RAGFlow类

`RAGFlow`类是SDK的主入口点，用于初始化客户端并访问所有核心功能。它提供了创建和管理数据集、聊天助手、智能体等资源的方法。

### 初始化

```python
def __init__(self, api_key, base_url, version="v1")
```

初始化RAGFlow客户端。

**参数说明：**
- `api_key`: 用于身份验证的API密钥
- `base_url`: RAGFlow服务的基础URL（例如：`http://localhost:9380`）
- `version`: API版本，默认为"v1"

**返回值：** `RAGFlow`实例

**异常类型：** 无

### 创建数据集

```python
def create_dataset(
    self,
    name: str,
    avatar: Optional[str] = None,
    description: Optional[str] = None,
    embedding_model: Optional[str] = None,
    permission: str = "me",
    chunk_method: str = "naive",
    parser_config: Optional[DataSet.ParserConfig] = None,
) -> DataSet
```

创建一个新的数据集。

**参数说明：**
- `name`: 数据集名称，必需参数
- `avatar`: 头像的Base64编码
- `description`: 数据集描述
- `embedding_model`: 嵌入模型名称
- `permission`: 权限设置，可选值为"me"（仅自己）或"team"（团队）
- `chunk_method`: 分块方法，可选值包括"naive"、"qa"、"table"等
- `parser_config`: 解析器配置对象

**返回值：** `DataSet`对象

**异常类型：** 如果创建失败，抛出`Exception`

### 删除数据集

```python
def delete_datasets(self, ids: list[str] | None = None)
```

删除一个或多个数据集。

**参数说明：**
- `ids`: 要删除的数据集ID列表。如果为None，则删除所有数据集

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

### 获取数据集

```python
def get_dataset(self, name: str)
```

根据名称获取数据集。

**参数说明：**
- `name`: 数据集名称

**返回值：** `DataSet`对象

**异常类型：** 如果数据集不存在，抛出`Exception`

### 列出数据集

```python
def list_datasets(
    self,
    page: int = 1,
    page_size: int = 30,
    orderby: str = "create_time",
    desc: bool = True,
    id: str | None = None,
    name: str | None = None
) -> list[DataSet]
```

列出所有数据集。

**参数说明：**
- `page`: 页码，默认为1
- `page_size`: 每页大小，默认为30
- `orderby`: 排序字段，可选值为"create_time"或"update_time"
- `desc`: 是否降序排序，默认为True
- `id`: 特定数据集ID
- `name`: 数据集名称

**返回值：** `DataSet`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 创建聊天助手

```python
def create_chat(
    self,
    name: str,
    avatar: str = "",
    dataset_ids=None,
    llm: Chat.LLM | None = None,
    prompt: Chat.Prompt | None = None
) -> Chat
```

创建一个新的聊天助手。

**参数说明：**
- `name`: 聊天助手名称，必需参数
- `avatar`: 头像的Base64编码
- `dataset_ids`: 关联的数据集ID列表
- `llm`: LLM配置对象
- `prompt`: 提示词配置对象

**返回值：** `Chat`对象

**异常类型：** 如果创建失败，抛出`Exception`

### 删除聊天助手

```python
def delete_chats(self, ids: list[str] | None = None)
```

删除一个或多个聊天助手。

**参数说明：**
- `ids`: 要删除的聊天助手ID列表。如果为None，则删除所有聊天助手

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

### 列出聊天助手

```python
def list_chats(
    self,
    page: int = 1,
    page_size: int = 30,
    orderby: str = "create_time",
    desc: bool = True,
    id: str | None = None,
    name: str | None = None
) -> list[Chat]
```

列出所有聊天助手。

**参数说明：**
- `page`: 页码，默认为1
- `page_size`: 每页大小，默认为30
- `orderby`: 排序字段，可选值为"create_time"或"update_time"
- `desc`: 是否降序排序，默认为True
- `id`: 特定聊天助手ID
- `name`: 聊天助手名称

**返回值：** `Chat`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 检索分块

```python
def retrieve(
    self,
    dataset_ids,
    document_ids=None,
    question="",
    page=1,
    page_size=30,
    similarity_threshold=0.2,
    vector_similarity_weight=0.3,
    top_k=1024,
    rerank_id: str | None = None,
    keyword: bool = False,
    cross_languages: list[str]|None = None,
    metadata_condition: dict | None = None,
)
```

从指定数据集中检索分块。

**参数说明：**
- `dataset_ids`: 要搜索的数据集ID列表
- `document_ids`: 要搜索的文档ID列表
- `question`: 用户查询或关键词
- `page`: 起始索引，默认为1
- `page_size`: 最大返回分块数，默认为30
- `similarity_threshold`: 最小相似度分数，默认为0.2
- `vector_similarity_weight`: 向量余弦相似度权重，默认为0.3
- `top_k`: 参与向量余弦计算的分块数，默认为1024
- `rerank_id`: 重排序模型ID
- `keyword`: 是否启用关键词匹配
- `cross_languages`: 需要翻译成的语言列表
- `metadata_condition`: 元数据过滤条件

**返回值：** `Chunk`对象列表

**异常类型：** 如果检索失败，抛出`Exception`

### 列出智能体

```python
def list_agents(
    self,
    page: int = 1,
    page_size: int = 30,
    orderby: str = "update_time",
    desc: bool = True,
    id: str | None = None,
    title: str | None = None
) -> list[Agent]
```

列出所有智能体。

**参数说明：**
- `page`: 页码，默认为1
- `page_size`: 每页大小，默认为30
- `orderby`: 排序字段，可选值为"create_time"或"update_time"
- `desc`: 是否降序排序，默认为True
- `id`: 特定智能体ID
- `title`: 智能体标题

**返回值：** `Agent`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 创建智能体

```python
def create_agent(self, title: str, dsl: dict, description: str | None = None) -> None
```

创建一个新的智能体。

**参数说明：**
- `title`: 智能体标题
- `dsl`: 智能体的画布DSL
- `description`: 智能体描述

**返回值：** 无

**异常类型：** 如果创建失败，抛出`Exception`

### 更新智能体

```python
def update_agent(self, agent_id: str, title: str | None = None, description: str | None = None, dsl: dict | None = None) -> None
```

更新现有智能体。

**参数说明：**
- `agent_id`: 要更新的智能体ID
- `title`: 新的智能体标题
- `description`: 新的智能体描述
- `dsl`: 新的智能体DSL

**返回值：** 无

**异常类型：** 如果更新失败，抛出`Exception`

### 删除智能体

```python
def delete_agent(self, agent_id: str) -> None
```

删除指定的智能体。

**参数说明：**
- `agent_id`: 要删除的智能体ID

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

**Section sources**
- [ragflow.py](file://sdk/python/ragflow_sdk/ragflow.py#L26-L286)

## 数据集模块

`DataSet`类代表一个知识库数据集，包含管理文档和分块的方法。

### 属性

- `id`: 数据集ID
- `name`: 数据集名称
- `avatar`: 头像
- `tenant_id`: 租户ID
- `description`: 描述
- `embedding_model`: 嵌入模型
- `permission`: 权限设置
- `document_count`: 文档数量
- `chunk_count`: 分块数量
- `chunk_method`: 分块方法
- `parser_config`: 解析器配置
- `pagerank`: 页面排名

### 更新数据集

```python
def update(self, update_message: dict)
```

更新数据集配置。

**参数说明：**
- `update_message`: 包含要更新属性的字典

**返回值：** 更新后的`DataSet`对象

**异常类型：** 如果更新失败，抛出`Exception`

### 上传文档

```python
def upload_documents(self, document_list: list[dict])
```

向数据集上传文档。

**参数说明：**
- `document_list`: 文档列表，每个文档是一个包含"display_name"和"blob"的字典

**返回值：** `Document`对象列表

**异常类型：** 如果上传失败，抛出`Exception`

### 列出文档

```python
def list_documents(
    self,
    id: str | None = None,
    name: str | None = None,
    keywords: str | None = None,
    page: int = 1,
    page_size: int = 30,
    orderby: str = "create_time",
    desc: bool = True,
    create_time_from: int = 0,
    create_time_to: int = 0,
)
```

列出数据集中的文档。

**参数说明：**
- `id`: 文档ID
- `name`: 文档名称
- `keywords`: 用于匹配文档标题的关键词
- `page`: 页码
- `page_size`: 每页大小
- `orderby`: 排序字段
- `desc`: 是否降序排序
- `create_time_from`: 创建时间起始时间戳
- `create_time_to`: 创建时间结束时间戳

**返回值：** `Document`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 删除文档

```python
def delete_documents(self, ids: list[str] | None = None)
```

删除一个或多个文档。

**参数说明：**
- `ids`: 要删除的文档ID列表

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

### 异步解析文档

```python
def async_parse_documents(self, document_ids)
```

异步解析文档。

**参数说明：**
- `document_ids`: 要解析的文档ID列表

**返回值：** 无

**异常类型：** 如果解析失败，抛出`Exception`

### 解析文档

```python
def parse_documents(self, document_ids)
```

同步解析文档，并等待解析完成。

**参数说明：**
- `document_ids`: 要解析的文档ID列表

**返回值：** 包含解析结果的元组列表，每个元组包含(文档ID, 状态, 分块数, 令牌数)

**异常类型：** 如果解析失败，抛出`Exception`

### 异步取消解析文档

```python
def async_cancel_parse_documents(self, document_ids)
```

取消正在解析的文档。

**参数说明：**
- `document_ids`: 要取消解析的文档ID列表

**返回值：** 无

**异常类型：** 如果取消失败，抛出`Exception`

**Section sources**
- [dataset.py](file://sdk/python/ragflow_sdk/modules/dataset.py#L21-L154)

## 聊天助手模块

`Chat`类代表一个聊天助手，用于与用户进行对话。

### 属性

- `id`: 聊天助手ID
- `name`: 名称
- `avatar`: 头像
- `llm`: LLM配置
- `prompt`: 提示词配置

### LLM配置

`Chat.LLM`类包含LLM的配置参数：

- `model_name`: 模型名称
- `temperature`: 温度参数
- `top_p`: Top-p采样参数
- `presence_penalty`: 存在惩罚
- `frequency_penalty`: 频率惩罚
- `max_tokens`: 最大令牌数

### 提示词配置

`Chat.Prompt`类包含提示词的配置参数：

- `similarity_threshold`: 相似度阈值
- `keywords_similarity_weight`: 关键词相似度权重
- `top_n`: 顶部N个分块
- `top_k`: 顶部K个分块
- `variables`: 变量列表
- `rerank_model`: 重排序模型
- `empty_response`: 空响应
- `opener`: 开场白
- `show_quote`: 是否显示引用
- `prompt`: 提示词内容

### 更新聊天助手

```python
def update(self, update_message: dict)
```

更新聊天助手配置。

**参数说明：**
- `update_message`: 包含要更新属性的字典

**返回值：** 无

**异常类型：** 如果更新失败，抛出`Exception`

### 创建会话

```python
def create_session(self, name: str = "New session") -> Session
```

创建与聊天助手的新会话。

**参数说明：**
- `name`: 会话名称，默认为"New session"

**返回值：** `Session`对象

**异常类型：** 如果创建失败，抛出`Exception`

### 列出会话

```python
def list_sessions(
    self,
    page: int = 1,
    page_size: int = 30,
    orderby: str = "create_time",
    desc: bool = True,
    id: str = None,
    name: str = None
) -> list[Session]
```

列出与聊天助手相关的会话。

**参数说明：**
- `page`: 页码
- `page_size`: 每页大小
- `orderby`: 排序字段
- `desc`: 是否降序排序
- `id`: 会话ID
- `name`: 会话名称

**返回值：** `Session`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 删除会话

```python
def delete_sessions(self, ids: list[str] | None = None)
```

删除一个或多个会话。

**参数说明：**
- `ids`: 要删除的会话ID列表

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

**Section sources**
- [chat.py](file://sdk/python/ragflow_sdk/modules/chat.py#L22-L88)

## 文档模块

`Document`类代表数据集中的单个文档。

### 属性

- `id`: 文档ID
- `name`: 文档名称
- `thumbnail`: 缩略图
- `dataset_id`: 关联的数据集ID
- `chunk_method`: 分块方法
- `parser_config`: 解析器配置
- `source_type`: 源类型
- `type`: 类型
- `created_by`: 创建者
- `size`: 大小（字节）
- `token_count`: 令牌数
- `chunk_count`: 分块数
- `progress`: 处理进度
- `progress_msg`: 进度消息
- `process_begin_at`: 处理开始时间
- `process_duration`: 处理持续时间
- `run`: 运行状态
- `status`: 状态
- `meta_fields`: 元字段

### 更新文档

```python
def update(self, update_message: dict)
```

更新文档配置。

**参数说明：**
- `update_message`: 包含要更新属性的字典

**返回值：** 更新后的`Document`对象

**异常类型：** 如果更新失败，抛出`Exception`

### 下载文档

```python
def download(self)
```

下载文档。

**参数说明：** 无

**返回值：** 文档内容的字节数据

**异常类型：** 如果下载失败，抛出`Exception`

### 列出分块

```python
def list_chunks(self, page=1, page_size=30, keywords="", id="")
```

列出文档中的分块。

**参数说明：**
- `page`: 页码
- `page_size`: 每页大小
- `keywords`: 用于匹配分块内容的关键词
- `id`: 分块ID

**返回值：** `Chunk`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 添加分块

```python
def add_chunk(self, content: str, important_keywords: list[str] = [], questions: list[str] = [])
```

向文档添加分块。

**参数说明：**
- `content`: 分块内容
- `important_keywords`: 重要关键词列表
- `questions`: 相关问题列表

**返回值：** `Chunk`对象

**异常类型：** 如果添加失败，抛出`Exception`

### 删除分块

```python
def delete_chunks(self, ids: list[str] | None = None)
```

删除一个或多个分块。

**参数说明：**
- `ids`: 要删除的分块ID列表

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

**Section sources**
- [document.py](file://sdk/python/ragflow_sdk/modules/document.py#L23-L102)

## 分块模块

`Chunk`类代表文档中的一个文本分块。

### 属性

- `id`: 分块ID
- `content`: 内容
- `important_keywords`: 重要关键词
- `questions`: 相关问题
- `create_time`: 创建时间
- `create_timestamp`: 创建时间戳
- `dataset_id`: 关联的数据集ID
- `document_name`: 关联的文档名称
- `document_id`: 关联的文档ID
- `available`: 可用性状态
- `similarity`: 相似度分数
- `vector_similarity`: 向量相似度
- `term_similarity`: 术语相似度
- `positions`: 位置信息
- `doc_type`: 文档类型

### 更新分块

```python
def update(self, update_message: dict)
```

更新分块内容或配置。

**参数说明：**
- `update_message`: 包含要更新属性的字典

**返回值：** 无

**异常类型：** 如果更新失败，抛出`ChunkUpdateError`

**Section sources**
- [chunk.py](file://sdk/python/ragflow_sdk/modules/chunk.py#L26-L57)

## 智能体模块

`Agent`类代表一个智能体，可以执行复杂的任务。

### 属性

- `id`: 智能体ID
- `avatar`: 头像
- `canvas_type`: 画布类型
- `description`: 描述
- `dsl`: 智能体的DSL

### 创建会话

```python
def create_session(self, **kwargs) -> Session
```

创建与智能体的新会话。

**参数说明：**
- `**kwargs`: 开始组件的参数

**返回值：** `Session`对象

**异常类型：** 如果创建失败，抛出`Exception`

### 列出会话

```python
def list_sessions(
    self,
    page: int = 1,
    page_size: int = 30,
    orderby: str = "create_time",
    desc: bool = True,
    id: str = None
) -> list[Session]
```

列出与智能体相关的会话。

**参数说明：**
- `page`: 页码
- `page_size`: 每页大小
- `orderby`: 排序字段
- `desc`: 是否降序排序
- `id`: 会话ID

**返回值：** `Session`对象列表

**异常类型：** 如果获取失败，抛出`Exception`

### 删除会话

```python
def delete_sessions(self, ids: list[str] | None = None)
```

删除一个或多个会话。

**参数说明：**
- `ids`: 要删除的会话ID列表

**返回值：** 无

**异常类型：** 如果删除失败，抛出`Exception`

**Section sources**
- [agent.py](file://sdk/python/ragflow_sdk/modules/agent.py#L21-L94)

## 会话模块

`Session`类代表与聊天助手或智能体的对话会话。

### 属性

- `id`: 会话ID
- `name`: 会话名称
- `messages`: 消息列表
- `chat_id`: 关联的聊天助手ID
- `agent_id`: 关联的智能体ID

### 询问

```python
def ask(self, question="", stream=False, **kwargs)
```

向会话提问。

**参数说明：**
- `question`: 问题内容
- `stream`: 是否流式输出
- `**kwargs`: 提示词系统中的参数

**返回值：** 
- 如果`stream=False`，返回`Message`对象
- 如果`stream=True`，返回`Message`对象的迭代器

**异常类型：** 如果询问失败，抛出`Exception`

### 更新会话

```python
def update(self, update_message)
```

更新会话。

**参数说明：**
- `update_message`: 包含要更新属性的字典

**返回值：** 无

**异常类型：** 如果更新失败，抛出`Exception`

### 消息类

`Message`类代表会话中的单个消息。

**属性：**
- `content`: 消息内容
- `reference`: 引用的分块列表
- `role`: 角色
- `prompt`: 提示词
- `id`: 消息ID

**Section sources**
- [session.py](file://sdk/python/ragflow_sdk/modules/session.py#L21-L129)

## 典型使用示例

以下示例展示了如何在实际项目中使用RAGFlow SDK：

```python
from ragflow_sdk import RAGFlow

# 初始化客户端
rag_object = RAGFlow(api_key="<YOUR_API_KEY>", base_url="http://<YOUR_BASE_URL>:9380")

# 创建数据集
dataset = rag_object.create_dataset(name="kb_1")

# 上传文档
documents = [
    {'display_name': 'test1.txt', 'blob': open('./test_data/test1.txt',"rb").read()},
    {'display_name': 'test2.txt', 'blob': open('./test_data/test2.txt',"rb").read()}
]
dataset.upload_documents(documents)

# 解析文档
documents = dataset.list_documents(keywords="test")
ids = [doc.id for doc in documents]
finished = dataset.parse_documents(ids)
for doc_id, status, chunk_count, token_count in finished:
    print(f"Document {doc_id} parsing finished with status: {status}, chunks: {chunk_count}, tokens: {token_count}")

# 创建聊天助手
assistant = rag_object.create_chat("Miss R", dataset_ids=[dataset.id])

# 创建会话
session = assistant.create_session()

# 与聊天助手对话
print("\n==================== Miss R =====================\n")
print("Hello. What can I do for you?")

while True:
    question = input("\n==================== User =====================\n> ")
    print("\n==================== Miss R =====================\n")
    
    cont = ""
    for ans in session.ask(question, stream=True):
        print(ans.content[len(cont):], end='', flush=True)
        cont = ans.content
```

**Section sources**
- [python_api_reference.md](file://docs/references/python_api_reference.md#L195-L1557)

## 异常处理

RAGFlow SDK在各种情况下会抛出异常：

1. **通用异常**：大多数方法在失败时会抛出`Exception`，包含错误信息。
2. **分块更新异常**：`Chunk.update()`方法在失败时会抛出`ChunkUpdateError`，包含错误代码、消息和详细信息。
3. **网络异常**：底层的`requests`库可能会抛出网络相关的异常。

建议在使用SDK时使用try-catch块来处理这些异常：

```python
try:
    dataset = rag_object.create_dataset(name="kb_1")
except Exception as e:
    print(f"创建数据集失败: {e}")
```

**Section sources**
- [ragflow.py](file://sdk/python/ragflow_sdk/ragflow.py#L76-L77)
- [dataset.py](file://sdk/python/ragflow_sdk/modules/dataset.py#L48-L49)
- [chunk.py](file://sdk/python/ragflow_sdk/modules/chunk.py#L53-L57)