# LLM服务配置

<cite>
**本文档引用的文件**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [llm_factories.json](file://conf/llm_factories.json)
- [settings.py](file://common/settings.py)
- [llm_app.py](file://api/apps/llm_app.py)
- [tenant_llm_service.py](file://api/db/services/tenant_llm_service.py)
- [chat_model.py](file://rag/llm/chat_model.py)
- [embedding_model.py](file://rag/llm/embedding_model.py)
- [rerank_model.py](file://rag/llm/rerank_model.py)
</cite>

## 目录
1. [简介](#简介)
2. [LLM服务配置概述](#llm服务配置概述)
3. [核心配置文件分析](#核心配置文件分析)
4. [LLM工厂配置详解](#llm工厂配置详解)
5. [API密钥与安全配置](#api密钥与安全配置)
6. [请求速率限制与模型回退机制](#请求速率限制与模型回退机制)
7. [最佳实践与配置建议](#最佳实践与配置建议)

## 简介
本文档系统性地介绍了RAGFlow中LLM服务的配置方法，重点阐述了如何配置各种大语言模型服务，包括聊天模型、嵌入模型和重排序模型。文档详细解释了`/conf/service_conf.yaml`中`user_default_llm`配置块的factory、api_key、base_url以及default_models的设置方法，说明了如何通过配置文件或环境变量集成不同厂商的API（如OpenAI、Anthropic、阿里云通义千问）。同时，文档指导用户如何在`llm_factories.json`中定义自定义LLM工厂，并提供了API密钥安全管理、请求速率限制处理和模型回退（fallback）机制的配置最佳实践。

## LLM服务配置概述
RAGFlow的LLM服务配置主要通过两个核心文件进行管理：`conf/service_conf.yaml`和`conf/llm_factories.json`。`service_conf.yaml`文件定义了系统级别的LLM配置，包括默认的LLM工厂、API密钥、基础URL和默认模型。`llm_factories.json`文件则定义了所有可用的LLM工厂及其支持的模型列表。系统通过这些配置文件来确定如何与不同的LLM提供商进行通信，并为用户提供一个统一的接口来访问各种LLM服务。

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [llm_factories.json](file://conf/llm_factories.json)

## 核心配置文件分析
### service_conf.yaml 配置详解
`service_conf.yaml`文件中的`user_default_llm`配置块是LLM服务的核心配置部分。该配置块包含以下关键参数：

- **factory**: 指定默认的LLM工厂，如"OpenAI"、"Tongyi-Qianwen"等。
- **api_key**: 用于访问LLM服务的API密钥。
- **base_url**: LLM服务的基础URL，用于构建API请求的完整URL。
- **default_models**: 定义了各种类型模型的默认配置，包括聊天模型、嵌入模型和重排序模型。

```yaml
user_default_llm:
  factory: 'OpenAI'
  api_key: 'your-api-key'
  base_url: 'https://api.openai.com/v1'
  default_models:
    chat_model:
      name: 'gpt-4-turbo'
    embedding_model:
      name: 'text-embedding-ada-002'
    rerank_model: 'bge-reranker-v2'
```

**Diagram sources**
- [service_conf.yaml](file://conf/service_conf.yaml#L46-L104)

### llm_factories.json 结构解析
`llm_factories.json`文件定义了所有可用的LLM工厂及其支持的模型。每个工厂包含以下信息：

- **name**: 工厂名称，如"OpenAI"、"Tongyi-Qianwen"。
- **logo**: 工厂的logo（可选）。
- **tags**: 工厂支持的模型类型标签，如"LLM,TEXT EMBEDDING,TTS"。
- **status**: 工厂的状态（1表示有效，0表示无效）。
- **rank**: 工厂的排序权重。
- **llm**: 工厂支持的具体模型列表，每个模型包含模型名称、标签、最大token数、模型类型等信息。

```json
{
  "factory_llm_infos": [
    {
      "name": "OpenAI",
      "tags": "LLM,TEXT EMBEDDING,TTS",
      "status": "1",
      "rank": "999",
      "llm": [
        {
          "llm_name": "gpt-4-turbo",
          "tags": "LLM,CHAT,128K",
          "max_tokens": 128000,
          "model_type": "chat"
        }
      ]
    }
  ]
}
```

**Diagram sources**
- [llm_factories.json](file://conf/llm_factories.json#L1-L800)

**Section sources**
- [llm_factories.json](file://conf/llm_factories.json)

## LLM工厂配置详解
### 配置不同厂商的API
RAGFlow支持通过配置文件或环境变量集成多种LLM提供商的API。配置方法如下：

1. **OpenAI**: 在`service_conf.yaml`中设置`factory`为"OpenAI"，并提供相应的`api_key`和`base_url`。
2. **阿里云通义千问**: 设置`factory`为"Tongyi-Qianwen"，并配置相应的API密钥和基础URL。
3. **Anthropic**: 设置`factory`为"Anthropic"，并提供API密钥和基础URL。

```yaml
user_default_llm:
  factory: 'Tongyi-Qianwen'
  api_key: 'your-tongyi-api-key'
  base_url: 'https://dashscope.aliyuncs.com/api/v1'
  default_models:
    chat_model:
      name: 'qwen-plus'
    embedding_model:
      name: 'text-embedding-v2'
```

### 自定义LLM工厂
用户可以通过修改`llm_factories.json`文件来定义自定义的LLM工厂。添加新的工厂需要提供工厂名称、标签、状态、排序权重以及支持的模型列表。例如，添加一个名为"MyCustomLLM"的工厂：

```json
{
  "name": "MyCustomLLM",
  "tags": "LLM,CHAT",
  "status": "1",
  "rank": "500",
  "llm": [
    {
      "llm_name": "my-custom-model",
      "tags": "LLM,CHAT,64K",
      "max_tokens": 64000,
      "model_type": "chat"
    }
  ]
}
```

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [llm_factories.json](file://conf/llm_factories.json)

## API密钥与安全配置
### API密钥管理
API密钥是访问LLM服务的关键凭证，必须妥善管理。建议采取以下措施：

1. **使用环境变量**: 将API密钥存储在环境变量中，而不是直接写入配置文件。
2. **定期轮换**: 定期更换API密钥，以降低密钥泄露的风险。
3. **最小权限原则**: 为每个应用分配最小必要的权限，避免使用具有过高权限的密钥。

### 安全最佳实践
- **避免硬编码**: 不要在代码或配置文件中硬编码API密钥。
- **使用密钥管理服务**: 考虑使用专门的密钥管理服务来存储和管理API密钥。
- **监控密钥使用**: 定期检查API密钥的使用情况，及时发现异常行为。

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [settings.py](file://common/settings.py)

## 请求速率限制与模型回退机制
### 速率限制处理
RAGFlow内置了请求速率限制处理机制，以应对LLM提供商的速率限制。当遇到429状态码（Too Many Requests）时，系统会自动进行重试，重试间隔采用指数退避策略。

```python
def _handle_http_error(e: requests.HTTPError, attempt: int) -> int:
    if e.response.status_code == 429:
        retry_after = e.response.headers.get("Retry-After")
        if retry_after:
            return int(retry_after)
        return 2 ** attempt
    raise e
```

### 模型回退机制
当首选模型不可用时，RAGFlow支持配置模型回退机制。系统会尝试使用备用模型来完成请求，确保服务的连续性。回退策略可以在`service_conf.yaml`中配置，指定备用模型的名称和优先级。

**Section sources**
- [utils.py](file://common/data_source/utils.py)
- [firecrawl_connector.py](file://intergrations/firecrawl/firecrawl_connector.py)

## 最佳实践与配置建议
1. **合理配置默认模型**: 根据应用场景选择合适的默认模型，平衡性能和成本。
2. **定期更新LLM工厂配置**: 及时更新`llm_factories.json`文件，添加新的模型支持。
3. **监控API使用情况**: 定期检查API调用情况，优化模型选择和配置。
4. **测试配置变更**: 在生产环境应用配置变更前，先在测试环境中进行充分测试。

**Section sources**
- [service_conf.yaml](file://conf/service_conf.yaml)
- [llm_factories.json](file://conf/llm_factories.json)
- [settings.py](file://common/settings.py)