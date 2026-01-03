# API参考

<cite>
**本文档引用的文件**
- [http_api_reference.md](file://docs/references/http_api_reference.md)
- [document_app.py](file://api/apps/document_app.py)
- [kb_app.py](file://api/apps/kb_app.py)
- [conversation_app.py](file://api/apps/conversation_app.py)
- [dialog_app.py](file://api/apps/dialog_app.py)
- [user_app.py](file://api/apps/user_app.py)
- [system_app.py](file://api/apps/system_app.py)
- [tenant_app.py](file://api/apps/tenant_app.py)
- [file_app.py](file://api/apps/file_app.py)
- [search_app.py](file://api/apps/search_app.py)
- [ragflow.py](file://sdk/python/ragflow_sdk/ragflow.py)
</cite>

## 目录
1. [简介](#简介)
2. [知识库管理](#知识库管理)
3. [文档管理](#文档管理)
4. [对话管理](#对话管理)
5. [会话管理](#会话管理)
6. [用户与租户管理](#用户与租户管理)
7. [系统与健康检查](#系统与健康检查)
8. [Python SDK使用指南](#python-sdk使用指南)

## 简介
RAGFlow提供了一套全面的RESTful API，用于管理和操作知识库、文档、对话等核心功能。所有API端点都需要通过API密钥进行身份验证。API密钥可以通过系统管理端点生成，并在请求头中以`Authorization: Bearer <YOUR_API_KEY>`的形式提供。本文档详细介绍了`/api/apps/`目录下各个应用提供的API端点，包括HTTP方法、URL路径、请求头、请求体结构、响应状态码及响应体结构。

## 知识库管理
知识库（Knowledge Base）是RAGFlow中组织和管理文档的核心单元。以下端点用于创建、更新、删除和查询知识库。

### 创建知识库
创建一个新的知识库。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/kbs/create`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "name": "string",
  "parser_id": "string",
  "parser_config": "object",
  "embedding_model": "string",
  "permission": "string",
  "chunk_method": "string",
  "avatar": "string",
  "description": "string",
  "language": "string",
  "embedding_model_id": "string",
  "tenant_id": "string"
}
```

**响应状态码**
- `200`: 成功创建知识库。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "kb_id": "string"
  }
}
```

**Section sources**
- [kb_app.py](file://api/apps/kb_app.py#L48-L68)

### 更新知识库
更新指定知识库的配置。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/kbs/update`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "kb_id": "string",
  "name": "string",
  "description": "string",
  "parser_id": "string",
  "parser_config": "object",
  "embedding_model": "string",
  "permission": "string",
  "chunk_method": "string",
  "avatar": "string",
  "language": "string",
  "embedding_model_id": "string"
}
```

**响应状态码**
- `200`: 成功更新知识库。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "tenant_id": "string",
    "name": "string",
    "description": "string",
    "language": "string",
    "avatar": "string",
    "parser_id": "string",
    "parser_config": "object",
    "embedding_model": "string",
    "permission": "string",
    "chunk_method": "string",
    "document_count": 0,
    "chunk_count": 0,
    "token_num": 0,
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string",
    "status": "string"
  }
}
```

**Section sources**
- [kb_app.py](file://api/apps/kb_app.py#L71-L149)

### 获取知识库详情
获取指定知识库的详细信息。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/kbs/detail`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**查询参数**
- `kb_id` (*必需*): 知识库的ID。

**响应状态码**
- `200`: 成功获取知识库详情。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "tenant_id": "string",
    "name": "string",
    "description": "string",
    "language": "string",
    "avatar": "string",
    "parser_id": "string",
    "parser_config": "object",
    "embedding_model": "string",
    "permission": "string",
    "chunk_method": "string",
    "document_count": 0,
    "chunk_count": 0,
    "token_num": 0,
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string",
    "status": "string",
    "size": 0,
    "connectors": []
  }
}
```

**Section sources**
- [kb_app.py](file://api/apps/kb_app.py#L153-L178)

### 列出知识库
列出用户拥有的或有权访问的知识库。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/kbs/list`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**查询参数**
- `keywords` (*可选*): 搜索关键词。
- `page` (*可选*): 页码。
- `page_size` (*可选*): 每页数量。
- `orderby` (*可选*): 排序字段。
- `desc` (*可选*): 是否降序排列。
- `parser_id` (*可选*): 解析器ID。

**请求体 (JSON Schema)**
```json
{
  "owner_ids": ["string"]
}
```

**响应状态码**
- `200`: 成功列出知识库。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "kbs": [
      {
        "id": "string",
        "tenant_id": "string",
        "name": "string",
        "description": "string",
        "language": "string",
        "avatar": "string",
        "parser_id": "string",
        "parser_config": "object",
        "embedding_model": "string",
        "permission": "string",
        "chunk_method": "string",
        "document_count": 0,
        "chunk_count": 0,
        "token_num": 0,
        "create_time": 0,
        "create_date": "string",
        "update_time": 0,
        "update_date": "string",
        "status": "string"
      }
    ],
    "total": 0
  }
}
```

**Section sources**
- [kb_app.py](file://api/apps/kb_app.py#L182-L214)

### 删除知识库
删除指定的知识库。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/kbs/rm`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "kb_id": "string"
}
```

**响应状态码**
- `200`: 成功删除知识库。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [kb_app.py](file://api/apps/kb_app.py#L219-L258)

## 文档管理
文档管理API用于上传、解析、更新和删除知识库中的文档。

### 上传文档
将文件上传到指定的知识库。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/upload`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**表单数据**
- `kb_id` (*必需*): 知识库ID。
- `file` (*必需*): 要上传的文件。

**响应状态码**
- `200`: 成功上传文档。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": [
    {
      "id": "string",
      "kb_id": "string",
      "parser_id": "string",
      "parser_config": "object",
      "created_by": "string",
      "type": "string",
      "name": "string",
      "location": "string",
      "size": 0,
      "thumbnail": "string",
      "suffix": "string",
      "run": "string",
      "progress": 0,
      "progress_msg": "string",
      "chunk_num": 0,
      "token_num": 0,
      "create_time": 0,
      "create_date": "string",
      "update_time": 0,
      "update_date": "string",
      "status": "string"
    }
  ]
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L51-L84)

### 创建文档
在知识库中创建一个虚拟文档（如URL链接）。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/create`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "name": "string",
  "kb_id": "string"
}
```

**响应状态码**
- `200`: 成功创建文档。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "kb_id": "string",
    "parser_id": "string",
    "parser_config": "object",
    "created_by": "string",
    "type": "string",
    "name": "string",
    "location": "string",
    "size": 0,
    "thumbnail": "string",
    "suffix": "string",
    "run": "string",
    "progress": 0,
    "progress_msg": "string",
    "chunk_num": 0,
    "token_num": 0,
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string",
    "status": "string"
  }
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L153-L207)

### 列出文档
列出指定知识库中的所有文档。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/list`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**查询参数**
- `kb_id` (*必需*): 知识库ID。
- `keywords` (*可选*): 搜索关键词。
- `page` (*可选*): 页码。
- `page_size` (*可选*): 每页数量。
- `orderby` (*可选*): 排序字段。
- `desc` (*可选*): 是否降序排列。
- `create_time_from` (*可选*): 创建时间起始。
- `create_time_to` (*可选*): 创建时间结束。

**请求体 (JSON Schema)**
```json
{
  "run_status": ["string"],
  "types": ["string"],
  "suffix": ["string"]
}
```

**响应状态码**
- `200`: 成功列出文档。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "total": 0,
    "docs": [
      {
        "id": "string",
        "kb_id": "string",
        "parser_id": "string",
        "parser_config": "object",
        "created_by": "string",
        "type": "string",
        "name": "string",
        "location": "string",
        "size": 0,
        "thumbnail": "string",
        "suffix": "string",
        "run": "string",
        "progress": 0,
        "progress_msg": "string",
        "chunk_num": 0,
        "token_num": 0,
        "create_time": 0,
        "create_date": "string",
        "update_time": 0,
        "update_date": "string",
        "status": "string"
      }
    ]
  }
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L210-L267)

### 更改文档状态
更改一个或多个文档的处理状态（启用/禁用）。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/change_status`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "doc_ids": ["string"],
  "status": "string"
}
```

**响应状态码**
- `200`: 成功更改文档状态。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "doc_id": {
      "status": "string"
    }
  }
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L341-L378)

### 删除文档
删除一个或多个文档。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/rm`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "doc_id": "string" | ["string"]
}
```

**响应状态码**
- `200`: 成功删除文档。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L381-L398)

### 运行文档解析
启动或停止文档的解析任务。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/run`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "doc_ids": ["string"],
  "run": "string",
  "delete": "boolean"
}
```

**响应状态码**
- `200`: 成功运行文档解析。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L402-L447)

### 重命名文档
更改文档的名称。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/rename`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "doc_id": "string",
  "name": "string"
}
```

**响应状态码**
- `200`: 成功重命名文档。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L453-L498)

### 更改文档解析器
更改文档的解析方法。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/documents/change_parser`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "doc_id": "string",
  "parser_id": "string",
  "parser_config": "object",
  "pipeline_id": "string"
}
```

**响应状态码**
- `200`: 成功更改文档解析器。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [document_app.py](file://api/apps/document_app.py#L546-L596)

## 对话管理
对话管理API用于创建、更新、删除和查询对话（Chat）。

### 创建对话
创建一个新的对话。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/dialogs/set`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "dialog_id": "string",
  "name": "string",
  "description": "string",
  "icon": "string",
  "top_n": "integer",
  "top_k": "integer",
  "rerank_id": "string",
  "similarity_threshold": "number",
  "vector_similarity_weight": "number",
  "llm_setting": "object",
  "meta_data_filter": "object",
  "prompt_config": "object",
  "kb_ids": ["string"]
}
```

**响应状态码**
- `200`: 成功创建对话。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "tenant_id": "string",
    "name": "string",
    "description": "string",
    "icon": "string",
    "top_n": 0,
    "top_k": 0,
    "rerank_id": "string",
    "similarity_threshold": 0,
    "vector_similarity_weight": 0,
    "llm_id": "string",
    "llm_setting": "object",
    "prompt_config": "object",
    "meta_data_filter": "object",
    "kb_ids": ["string"],
    "kb_names": ["string"],
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string",
    "status": "string"
  }
}
```

**Section sources**
- [dialog_app.py](file://api/apps/dialog_app.py#L30-L108)

### 获取对话详情
获取指定对话的详细信息。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/dialogs/get`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**查询参数**
- `dialog_id` (*必需*): 对话的ID。

**响应状态码**
- `200`: 成功获取对话详情。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "tenant_id": "string",
    "name": "string",
    "description": "string",
    "icon": "string",
    "top_n": 0,
    "top_k": 0,
    "rerank_id": "string",
    "similarity_threshold": 0,
    "vector_similarity_weight": 0,
    "llm_id": "string",
    "llm_setting": "object",
    "prompt_config": "object",
    "meta_data_filter": "object",
    "kb_ids": ["string"],
    "kb_names": ["string"],
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string",
    "status": "string"
  }
}
```

**Section sources**
- [dialog_app.py](file://api/apps/dialog_app.py#L126-L137)

### 列出对话
列出用户拥有的所有对话。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/dialogs/list`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功列出对话。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": [
    {
      "id": "string",
      "tenant_id": "string",
      "name": "string",
      "description": "string",
      "icon": "string",
      "top_n": 0,
      "top_k": 0,
      "rerank_id": "string",
      "similarity_threshold": 0,
      "vector_similarity_weight": 0,
      "llm_id": "string",
      "llm_setting": "object",
      "prompt_config": "object",
      "meta_data_filter": "object",
      "kb_ids": ["string"],
      "kb_names": ["string"],
      "create_time": 0,
      "create_date": "string",
      "update_time": 0,
      "update_date": "string",
      "status": "string"
    }
  ]
}
```

**Section sources**
- [dialog_app.py](file://api/apps/dialog_app.py#L152-L164)

### 删除对话
删除一个或多个对话。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/dialogs/rm`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "dialog_ids": ["string"]
}
```

**响应状态码**
- `200`: 成功删除对话。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [dialog_app.py](file://api/apps/dialog_app.py#L207-L225)

## 会话管理
会话管理API用于与对话进行交互，包括发送消息、获取回答等。

### 设置会话
创建或更新一个会话。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/conversations/set`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "conversation_id": "string",
  "is_new": "boolean",
  "name": "string",
  "dialog_id": "string"
}
```

**响应状态码**
- `200`: 成功设置会话。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "dialog_id": "string",
    "name": "string",
    "message": [
      {
        "role": "string",
        "content": "string"
      }
    ],
    "reference": [],
    "user_id": "string",
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string"
  }
}
```

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L37-L76)

### 获取会话
获取指定会话的详细信息。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/conversations/get`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**查询参数**
- `conversation_id` (*必需*): 会话的ID。

**响应状态码**
- `200`: 成功获取会话。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "dialog_id": "string",
    "name": "string",
    "message": [
      {
        "role": "string",
        "content": "string"
      }
    ],
    "reference": [],
    "user_id": "string",
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string",
    "avatar": "string"
  }
}
```

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L81-L105)

### 完成会话
向对话发送消息并获取回答。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/conversations/completion`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "conversation_id": "string",
  "messages": [
    {
      "role": "string",
      "content": "string"
    }
  ],
  "llm_id": "string",
  "temperature": "number",
  "top_p": "number",
  "frequency_penalty": "number",
  "presence_penalty": "number",
  "max_tokens": "integer",
  "stream": "boolean"
}
```

**响应状态码**
- `200`: 成功完成会话。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构 (流式)**
```json
{
  "code": 0,
  "message": "",
  "data": {
    "answer": "string",
    "reference": []
  }
}
```

**响应体结构 (非流式)**
```json
{
  "code": 0,
  "data": {
    "answer": "string",
    "reference": []
  }
}
```

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L168-L249)

### 删除消息
删除会话中的消息。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/conversations/delete_msg`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "conversation_id": "string",
  "message_id": "string"
}
```

**响应状态码**
- `200`: 成功删除消息。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "dialog_id": "string",
    "name": "string",
    "message": [
      {
        "role": "string",
        "content": "string"
      }
    ],
    "reference": [],
    "user_id": "string",
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string"
  }
}
```

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L344-L364)

### 点赞/点踩
对回答进行点赞或点踩。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/conversations/thumbup`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "conversation_id": "string",
  "message_id": "string",
  "thumbup": "boolean",
  "feedback": "string"
}
```

**响应状态码**
- `200`: 成功点赞/点踩。
- `400`: 请求参数错误。
- `401`: 未授权访问。
- `403`: 访问被拒绝。
- `500`: 服务器内部错误。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "dialog_id": "string",
    "name": "string",
    "message": [
      {
        "role": "string",
        "content": "string",
        "thumbup": "boolean",
        "feedback": "string"
      }
    ],
    "reference": [],
    "user_id": "string",
    "create_time": 0,
    "create_date": "string",
    "update_time": 0,
    "update_date": "string"
  }
}
```

**Section sources**
- [conversation_app.py](file://api/apps/conversation_app.py#L367-L391)

## 用户与租户管理
这些API用于用户注册、登录、信息管理和租户成员管理。

### 用户登录
用户使用邮箱和密码登录。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/user/login`

**请求头**
- `Content-Type: application/json`

**请求体 (JSON Schema)**
```json
{
  "email": "string",
  "password": "string"
}
```

**响应状态码**
- `200`: 登录成功。
- `401`: 认证失败。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "email": "string",
    "nickname": "string",
    "avatar": "string",
    "access_token": "string",
    "last_login_time": "string",
    "login_channel": "string",
    "is_superuser": "boolean",
    "is_active": "string"
  },
  "message": "string"
}
```

**Section sources**
- [user_app.py](file://api/apps/user_app.py#L64-L137)

### 获取用户信息
获取当前登录用户的信息。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/user/info`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功获取用户信息。
- `401`: 未授权访问。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "email": "string",
    "nickname": "string",
    "avatar": "string",
    "access_token": "string",
    "last_login_time": "string",
    "login_channel": "string",
    "is_superuser": "boolean",
    "is_active": "string"
  }
}
```

**Section sources**
- [user_app.py](file://api/apps/user_app.py#L576-L602)

### 更新用户设置
更新用户的昵称、头像等信息。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/user/setting`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "nickname": "string",
  "avatar": "string"
}
```

**响应状态码**
- `200`: 成功更新用户设置。
- `401`: 未授权访问。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [user_app.py](file://api/apps/user_app.py#L508-L573)

### 列出团队成员
列出指定租户（团队）的所有成员。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/tenants/<tenant_id>/user/list`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功列出团队成员。
- `401`: 未授权访问。

**响应体结构**
```json
{
  "code": 0,
  "data": [
    {
      "id": "string",
      "tenant_id": "string",
      "user_id": "string",
      "invited_by": "string",
      "role": "string",
      "status": "string",
      "create_time": 0,
      "create_date": "string",
      "update_time": 0,
      "update_date": "string",
      "delta_seconds": 0
    }
  ]
}
```

**Section sources**
- [tenant_app.py](file://api/apps/tenant_app.py#L30-L44)

### 邀请用户加入团队
邀请用户加入指定的租户（团队）。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/tenants/<tenant_id>/user`

**请求头**
- `Content-Type: application/json`
- `Authorization: Bearer <YOUR_API_KEY>`

**请求体 (JSON Schema)**
```json
{
  "email": "string"
}
```

**响应状态码**
- `200`: 成功邀请用户。
- `401`: 未授权访问。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "id": "string",
    "avatar": "string",
    "email": "string",
    "nickname": "string"
  }
}
```

**Section sources**
- [tenant_app.py](file://api/apps/tenant_app.py#L48-L100)

### 删除团队成员
从团队中移除成员。

**HTTP方法**: `DELETE`
**URL路径**: `/api/v1/tenants/<tenant_id>/user/<user_id>`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功删除团队成员。
- `401`: 未授权访问。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [tenant_app.py](file://api/apps/tenant_app.py#L103-L115)

## 系统与健康检查
这些API用于获取系统信息、健康状态和生成API密钥。

### 获取系统版本
获取RAGFlow的当前版本。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/system/version`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功获取版本信息。

**响应体结构**
```json
{
  "code": 0,
  "data": "string"
}
```

**Section sources**
- [system_app.py](file://api/apps/system_app.py#L42-L62)

### 获取系统状态
获取系统的健康状态，包括文档引擎、存储、数据库和Redis。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/system/status`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功获取系统状态。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "doc_engine": {
      "type": "string",
      "status": "string",
      "elapsed": "string"
    },
    "storage": {
      "storage": "string",
      "status": "string",
      "elapsed": "string"
    },
    "database": {
      "database": "string",
      "status": "string",
      "elapsed": "string"
    },
    "redis": {
      "status": "string",
      "elapsed": "string"
    },
    "task_executor_heartbeats": {}
  }
}
```

**Section sources**
- [system_app.py](file://api/apps/system_app.py#L65-L171)

### 生成API密钥
为当前用户生成一个新的API密钥。

**HTTP方法**: `POST`
**URL路径**: `/api/v1/system/new_token`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功生成API密钥。

**响应体结构**
```json
{
  "code": 0,
  "data": {
    "tenant_id": "string",
    "token": "string",
    "beta": "string",
    "create_time": 0,
    "create_date": "string",
    "update_time": null,
    "update_date": null
  }
}
```

**Section sources**
- [system_app.py](file://api/apps/system_app.py#L185-L231)

### 列出API密钥
列出当前用户拥有的所有API密钥。

**HTTP方法**: `GET`
**URL路径**: `/api/v1/system/token_list`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功列出API密钥。

**响应体结构**
```json
{
  "code": 0,
  "data": [
    {
      "tenant_id": "string",
      "token": "string",
      "beta": "string",
      "create_time": 0,
      "create_date": "string",
      "update_time": null,
      "update_date": null
    }
  ]
}
```

**Section sources**
- [system_app.py](file://api/apps/system_app.py#L235-L279)

### 删除API密钥
删除指定的API密钥。

**HTTP方法**: `DELETE`
**URL路径**: `/api/v1/system/token/<token>`

**请求头**
- `Authorization: Bearer <YOUR_API_KEY>`

**响应状态码**
- `200`: 成功删除API密钥。

**响应体结构**
```json
{
  "code": 0,
  "data": true
}
```

**Section sources**
- [system_app.py](file://api/apps/system_app.py#L284-L313)

## Python SDK使用指南
RAGFlow提供了Python SDK，可以更方便地调用其API。以下是使用SDK的基本示例。

### 安装
```bash
pip install ragflow_sdk
```

### 初始化
```python
from ragflow_sdk import RAGFlow

# 使用API密钥和基础URL初始化RAGFlow客户端
api_key = "your_api_key_here"
base_url = "http://localhost:8080"
ragflow = RAGFlow(api_key, base_url)
```

### 创建知识库
```python
# 创建一个名为"my_dataset"的知识库
dataset = ragflow.create_dataset(
    name="my_dataset",
    description="My first dataset",
    embedding_model="BAAI/bge-large-zh-v1.5@BAAI"
)
print(f"Created dataset: {dataset.id}")
```

### 上传文档
```python
# 将文件上传到知识库
with open("path/to/your/document.pdf", "rb") as file:
    files = {"file": file}
    response = ragflow.post(f"/datasets/{dataset.id}/documents", files=files)
    if response.status_code == 200:
        print("Document uploaded successfully!")
```

### 与对话交互
```python
# 创建一个对话
chat = ragflow.create_chat(
    name="My Chat",
    dataset_ids=[dataset.id]
)

# 发送消息并获取回答
messages = [{"role": "user", "content": "What is RAGFlow?"}]
response = ragflow.post(f"/chats/{chat.id}/completion", json={"messages": messages})
if response.status_code == 200:
    answer = response.json()["data"]["answer"]
    print(f"Answer: {answer}")
```

### 列出知识库
```python
# 列出所有知识库
datasets = ragflow.list_datasets()
for ds in datasets:
    print(f"Dataset: {ds.name} (ID: {ds.id})")
```

### 删除知识库
```python
# 删除一个知识库
ragflow.delete_datasets(ids=[dataset.id])
print("Dataset deleted.")
```

**Section sources**
- [ragflow.py](file://sdk/python/ragflow_sdk/ragflow.py)