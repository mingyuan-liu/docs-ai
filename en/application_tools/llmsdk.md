<!--
 * Copyright 2022-2023 SPACEMIT. All rights reserved.
 * Use of this source code is governed by a BSD-style license
 * that can be found in the LICENSE file.
 * 
 * @Author: David(qiang.fu@spacemit.com)
 * @Date: 2026-04-03 17:56:08
 * @LastEditTime: 2026-05-13 14:14:53
 * @FilePath: \doc\docs-ai\zh\application_tools\llmsdk.md
 * @Description: 
-->
---
sidebar_position: 2
---

# LLM SDK (临时版本，后续统一到AI SDK)

## 平台支持情况

|      平台 & 系统       |       是否支持     |
|-----------------------|-----------------------|
| K1 Buildroot          | ❌ 不支持               |
| K1 OpenHarmony     | ❌ 不支持              |
| K1 Bianbu LXQT/GNOME    | ❌ 不支持             |
| K3 Buildroot          | ❌ 不支持              |
| K3 OpenHarmony     | ❌ 不支持              |
| K3 Bianbu LXQT/GNOME  | ✅ 支持                |

## 安装服务

```bash
sudo apt update
sudo apt install llm-sdk
```


**Base URL:** `http://localhost:8050`

---

##  API目录

- [System](#system)
- [Models - 模型管理](#models---模型管理)
- [Chat - 聊天](#chat---聊天)
- [Sessions - 会话管理](#sessions---会话管理)
- [Knowledge Bases - 知识库管理](#knowledge-bases---知识库管理)
- [Config - 配置管理](#config---配置管理)
- [Assets - 静态资源](#assets---静态资源)

---

## System

### GET `/`
根路径，返回 API 基本信息。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/
```

**响应示例：**
```json
{"message": "llm-sdk LLM Chat API"}
```

---

### GET `/api/health`
健康检查。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/health
```

**响应示例：**
```json
{"status": "healthy", "app": "llm-sdk"}
```

---

## Models - 模型管理

**前缀：** `/api/models`

模型类型（`model_type`）可选值：`llm` | `embed` | `rerank`

---

### GET `/api/models/list`
获取指定类型的所有模型列表（从数据库读取）。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| model_type | string | `llm` | 模型类型 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/list
```

**响应示例：**
```json
{
  "models": [
    {
      "id": 1,
      "name": "Qwen2.5-7B-Q4",
      "path": "/home/user/.cache/llm-sdk/models/llm/model.gguf",
      "download_url": "https://example.com/model.gguf",
      "is_downloaded": true,
      "is_current": true,
      "model_type": "llm"
    }
  ]
}
```

---

### GET `/api/models/current`
获取当前运行的模型信息。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| model_type | string | `llm` | 模型类型 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/current
```

**响应示例：**
```json
{
  "mode": "llm",
  "model_name": "Qwen2.5-7B-Q4",
  "model_path": "/home/user/.cache/llm-sdk/models/llm/model.gguf",
  "status": "running"
}
```

---

### POST `/api/models/download`
触发模型下载（后台异步，立即返回）。

**请求体（JSON）：**
```json
{"model_id": 1}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/download \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**响应示例：**
```json
{"success": true, "message": "Download started for Qwen2.5-7B-Q4"}
```

若已下载：
```json
{"success": true, "already_downloaded": true, "message": "Model already downloaded"}
```

---

### POST `/api/models/download/cancel`
取消正在进行的模型下载。

**请求体（JSON）：**
```json
{"model_id": 1}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/download/cancel \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**响应示例：**
```json
{"success": true, "message": "Download cancelled for Qwen2.5-7B-Q4"}
```

---

### GET `/api/models/download/status`
获取指定模型的下载状态。

**查询参数：**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| model_id | int | 否 | 模型 ID |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/download/status
```

**响应示例：**
```json
{
  "status": "downloading",
  "downloaded": 1048576,
  "total": 4294967296,
  "progress": 0.024,
  "error": null
}
```

`status` 可选值：`not_started` | `downloading` | `completed` | `error`

---

### GET `/api/models/download/status/all` (SSE)
SSE 流式推送所有模型的下载状态。

**查询参数：**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| model_type | string | 否 | 筛选指定类型 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/download/status/all
```

**SSE 事件格式：**
```
data: {"tasks": {"1": {"status": "downloading", "progress": 0.5}}, "timestamp": 12345.0}
```

---

### POST `/api/models/start`
启动指定模型的 llama-server。

**请求体（JSON）：**
```json
{"model_id": 1}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/start \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**响应示例：**
```json
{"success": true, "message": "Server started"}
```

---

### POST `/api/models/start_all`
在后台启动所有模式（llm/embed/rerank）的当前模型服务器。

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/start_all
```

**响应示例：**
```json
{"success": true, "message": "Starting all model servers in background"}
```

---

### POST `/api/models/stop_all`
停止所有模式（llm/embed/rerank）的模型服务器。

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/stop_all
```

**响应示例：**
```json
{"success": true, "message": "All model servers stopped"}
```

---

### POST `/api/models/stop`
停止指定模型对应类型的服务器。

**请求体（JSON）：**
```json
{"model_id": 1}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/stop \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**响应示例：**
```json
{"success": true, "message": "Server stopped"}
```

---

### POST `/api/models/set_current`
将指定模型设为对应类型的当前模型（仅更新数据库，不启动服务器）。

**请求体（JSON）：**
```json
{"model_id": 1}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/set_current \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**响应示例：**
```json
{"success": true, "message": "Set Qwen2.5-7B-Q4 as current llm model"}
```

---

### GET `/api/models/server_status`
查询指定模式服务器的当前状态（单次）。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| model_type | string | `llm` | 模型类型 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/server_status
```

**响应示例：**
```json
{
  "status": "running",
  "model_name": "Qwen2.5-7B-Q4",
  "model_path": "/home/user/.cache/llm-sdk/models/llm/model.gguf",
  "error_message": null
}
```

`status` 可选值：`not_started` | `starting` | `running` | `error` | `stopped`

---

### GET `/api/models/server_status/stream` (SSE)
SSE 流式推送服务器状态，频率根据状态自动调整（启动中 0.5s，运行中 3s，其他 2s）。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| model_type | string | `llm` | 模型类型 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/server_status/stream
```

**SSE 事件格式：**
```
data: {"status": "running", "model_name": "...", "model_path": "...", "error_message": null, "timestamp": 12345.0}
```

---

### GET `/api/models/all/stream` (SSE)
SSE 统一流：同时推送 llm、embed、rerank 三种模式的服务器状态和下载进度。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/all/stream
```

**SSE 事件格式：**
```json
{
  "modes": {
    "llm": {
      "server_status": {"status": "running", "model_name": "...", "model_path": "...", "error_message": null},
      "download_tasks": {}
    },
    "embed": { "..." : "..." },
    "rerank": { "..." : "..." }
  },
  "timestamp": 12345.0
}
```

---

### GET `/api/models/get_param`
获取指定类型的模型参数（服务器参数 + 客户端参数）。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| model_type | string | `llm` | 模型类型 |

**响应示例（llm）：**
```json
{
  "context_size": 4096,
  "threads": 4,
  "gpu_layers": 0,
  "batch_size": 512,
  "temperature": 0.7,
  "repeat_penalty": 1.1,
  "max_tokens": 2048
}
```

embed 额外字段：`normalize`, `truncate`；rerank 额外字段：`top_n`, `return_documents`

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/models/get_param
```

---

### POST `/api/models/update_param`
更新模型参数并保存到数据库。服务器参数变化会自动重启服务器；客户端参数立即生效。

**查询参数：**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| model_type | string | 是 | 模型类型 |

**请求体（JSON，所有字段可选）：**
```json
{
  "context_size": 4096,
  "threads": 8,
  "gpu_layers": 0,
  "batch_size": 512,
  "temperature": 0.8,
  "repeat_penalty": 1.1,
  "max_tokens": 2048,
  "normalize": true,
  "truncate": true,
  "top_n": 10,
  "return_documents": true
}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/update_param
```

**响应示例：**
```json
{
  "success": true,
  "message": "Server parameters updated (server auto-restarted if running)",
  "server_params_changed": true,
  "client_params_changed": false
}
```

---

### POST `/api/models/reset_param`
重置指定类型的模型参数为默认值，若服务器正在运行则自动重启。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| model_type | string | `llm` | 模型类型 |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/models/reset_param
```

**响应：** 返回重置后的完整参数（同 `get_param` 格式）

---

## Chat - 聊天

**前缀：** `/api`

---

### POST `/api/chat`
发送聊天消息，支持流式（SSE）和非流式两种响应。

支持两种模式：
- **普通对话**：ConversationChain（带历史上下文）
- **RAG 对话**：提供 `kb_ids` 时自动切换为 RAGChain（检索增强生成）

**请求体（JSON）：**
```json
{
  "message": {
    "role": "user",
    "content": "你好，请介绍一下自己"
  },
  "session_id": 1,
  "stream": true,
  "kb_ids": null,
  "temperature": null,
  "repeat_penalty": null,
  "max_tokens": null,
  "model_type": "llm"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| message | object | 是 | 用户消息，包含 role 和 content |
| session_id | int | 是 | 会话 ID |
| stream | bool | 否 | 是否流式响应，默认 `true` |
| kb_ids | string[] | 否 | 知识库 ID 列表，提供则使用 RAG |
| temperature | float | 否 | 覆盖默认温度参数 |
| repeat_penalty | float | 否 | 覆盖默认重复惩罚参数 |
| max_tokens | int | 否 | 覆盖最大生成 token 数 |

**流式响应（SSE）格式：**
```
data: {"data": "你好", "done_flag": false}
data: {"data": "！", "done_flag": false}
data: {"data": "", "done_flag": true}
```

**非流式响应：**
```json
{"response": "你好！我是一个AI助手..."}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/chat \
  -H "Content-Type: application/json" \
  -d "{  \"message\": {    \"role\": \"user\",    \"content\": \"你好，请介绍一下自己\"  },  \"session_id\": 1,  \"stream\": true,  \"kb_ids\": null,  \"temperature\": null,  \"repeat_penalty\": null,  \"max_tokens\": null,  \"model_type\": \"llm\"}"
```

---

## Sessions - 会话管理

**前缀：** `/api/sessions`

---

### POST `/api/sessions`
创建新会话，根据第一条消息自动生成会话名称（最多 50 字符）。

**请求体（JSON）：**
```json
{"first_message": "帮我写一首诗"}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/sessions \
  -H "Content-Type: application/json" \
  -d "{\"first_message\": \"帮我写一首诗\"}"
```

**响应示例：**
```json
{"session_id": 1, "session_name": "帮我写一首诗"}
```

---

### GET `/api/sessions`
获取会话列表，按 `updated_at` 降序排列。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| limit | int | 50 | 返回数量上限 |
| offset | int | 0 | 偏移量 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/sessions
```

**响应示例：**
```json
{
  "sessions": [
    {
      "id": 1,
      "session_name": "帮我写一首诗",
      "created_at": "2026-03-24T10:00:00",
      "updated_at": "2026-03-24T10:05:00",
      "message_count": 4,
      "total_tokens": 512
    }
  ],
  "total": 1
}
```

---

### GET `/api/sessions/{session_id}`
获取指定会话的详情。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/sessions/1
```

**响应：** 同会话列表中单条数据格式。

**错误：** `404` 会话不存在

---

### PUT `/api/sessions/{session_id}/name`
更新会话名称。

**请求体（JSON）：**
```json
{"new_name": "新的会话名称"}
```

**请求示例 (curl)：**
```bash
curl -X PUT http://localhost:8050/api/sessions/1/name \
  -H "Content-Type: application/json" \
  -d "{\"new_name\": \"新的会话名称\"}"
```

**响应示例：**
```json
{"success": true, "message": "Session name updated successfully"}
```

---

### DELETE `/api/sessions/{session_id}`
删除会话（级联删除所有消息）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/sessions/1
```

**响应示例：**
```json
{"success": true, "message": "Session deleted successfully"}
```

---

### GET `/api/sessions/{session_id}/messages`
获取会话的所有消息。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/sessions/1/messages
```

**响应示例：**
```json
{
  "messages": [
    {
      "id": 1,
      "session_id": 1,
      "role": "user",
      "content": "帮我写一首诗",
      "token_count": 8,
      "created_at": "2026-03-24T10:00:00",
      "metadata": null
    }
  ],
  "session_id": 1,
  "total_tokens": 8
}
```

---

### DELETE `/api/sessions/{session_id}/messages`
清空会话中的所有消息（保留会话本身）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/sessions/1/messages
```

**响应示例：**
```json
{"success": true, "message": "Messages cleared successfully"}
```

---

## Knowledge Bases - 知识库管理

**前缀：** `/api/knowledge-bases`

---

### GET `/api/knowledge-bases`
获取所有知识库列表。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases
```

**响应示例：**
```json
{
  "success": true,
  "knowledge_bases": [
    {
      "id": 1,
      "name": "技术文档库",
      "description": "存放技术相关文档",
      "avatar_url": "/api/assets/minio/avatars/kb_1.png",
      "file_count": 3,
      "created_at": "2026-03-24T10:00:00"
    }
  ],
  "count": 1
}
```

---

### POST `/api/knowledge-bases`
创建新知识库，支持上传头像。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 知识库名称 |
| description | string | 否 | 描述 |
| avatar | file | 否 | 头像图片（PNG/JPEG/GIF/WEBP，最大 5MB） |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/knowledge-bases \
  -F "name=技术文档库" \
  -F "description=测试描述"
```

**响应示例：**
```json
{
  "success": true,
  "message": "Knowledge base '技术文档库' created successfully",
  "kb_id": 1,
  "knowledge_base": { "..." : "..." }
}
```

**错误：** `409` 知识库名称已存在

---

### GET `/api/knowledge-bases/{kb_id}`
获取指定知识库的详情。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1
```

**响应示例：**
```json
{"success": true, "knowledge_base": { "..." : "..." }}
```

---

### PUT `/api/knowledge-bases/{kb_id}`
更新知识库信息（名称、描述、头像）。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 新名称 |
| description | string | 否 | 新描述 |
| avatar | file | 否 | 新头像（PNG/JPEG/GIF/WEBP，最大 5MB） |

**请求示例 (curl)：**
```bash
curl -X PUT http://localhost:8050/api/knowledge-bases/1 \
  -F "name=新名称"
```

**响应示例：**
```json
{"success": true, "message": "Knowledge base updated successfully", "knowledge_base": { "..." : "..." }}
```

---

### DELETE `/api/knowledge-bases/{kb_id}`
删除知识库（级联删除文件、向量数据、头像）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/knowledge-bases/1
```

**响应示例：**
```json
{"success": true, "message": "Knowledge base '技术文档库' deleted successfully"}
```

---

### POST `/api/knowledge-bases/{kb_id}/files`
上传单个文件到知识库（同步处理，完成后返回）。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | file | 是 | 要上传的文档文件 |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/knowledge-bases/1/files \
  -F "file=@/path/to/document.pdf"
```

**响应示例：**
```json
{
  "success": true,
  "message": "File 'document.pdf' uploaded and processed successfully",
  "file_id": 42,
  "kb_id": 1
}
```

---

### POST `/api/knowledge-bases/{kb_id}/files/batch`
批量上传多个文件，异步处理（并发数上限为 3）。可使用进度流端点监控处理状态。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| files | file[] | 是 | 多个文件 |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/knowledge-bases/1/files/batch \
  -F "files=@/path/to/doc1.pdf" \
  -F "files=@/path/to/doc2.pdf"
```

**响应示例：**
```json
{
  "success": true,
  "files": [
    {"file_id": 42, "filename": "doc1.pdf", "status": "processing"},
    {"filename": "bad.xyz", "status": "error", "error": "不支持的文件类型: .xyz"}
  ],
  "message": "Uploaded 2 files"
}
```

---

### GET `/api/knowledge-bases/{kb_id}/files`
获取知识库中的所有文件列表。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1/files
```

**响应示例：**
```json
{
  "success": true,
  "files": [
    {
      "id": 42,
      "filename": "document.pdf",
      "status": "completed",
      "created_at": "2026-03-24T10:00:00"
    }
  ],
  "count": 1
}
```

---

### DELETE `/api/knowledge-bases/{kb_id}/files/{file_id}`
从知识库中删除指定文件（同时删除 MinIO 存储、向量数据和数据库记录）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/knowledge-bases/1/files/42
```

**响应示例：**
```json
{"success": true, "message": "File 'document.pdf' deleted successfully"}
```

---

### GET `/api/knowledge-bases/{kb_id}/files/progress/stream` (SSE)
SSE 流：实时推送知识库中所有文件的处理进度（处理中 0.5s，空闲时 2s）。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1/files/progress/stream
```

**SSE 事件格式：**
```
event: progress
data: {"files": [...], "timestamp": "2026-03-24T10:00:00"}
```

---

### GET `/api/knowledge-bases/{kb_id}/files/{file_id}/chunks`
获取文件在 ChromaDB 中的所有 chunk 信息（调试用途）。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1/files/42/chunks
```

**响应示例：**
```json
{
  "kb_id": 1,
  "file_id": 42,
  "chunk_count": 15,
  "chunks": [
    {"document": "...", "metadata": {"source": "document.pdf", "chunk_index": 0}}
  ]
}
```

---

## Config - 配置管理

**前缀：** `/api/config`

---

### GET `/api/config/rag`
获取当前 RAG 配置（检索参数 + 提示词）。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/config/rag
```

**响应示例：**
```json
{
  "retrieval": {
    "top_k": 3,
    "initial_k": 50,
    "intermediate_k": 10,
    "embed_weight": 0.5,
    "bm25_weight": 0.5,
    "rerank_enable": false
  },
  "prompts": {
    "conversation_system_prompt": "你是一个有用的助手...",
    "rag_system_prompt_template": "根据以下资料回答问题：\n\n{context}\n\n问题：{question}"
  }
}
```

---

### PUT `/api/config/rag`
更新 RAG 配置，保存到用户配置文件并立即热重载生效。

**请求体（JSON）：**
```json
{
  "retrieval": {
    "top_k": 5,
    "initial_k": 50,
    "intermediate_k": 10,
    "embed_weight": 0.6,
    "bm25_weight": 0.4,
    "rerank_enable": false
  },
  "prompts": {
    "conversation_system_prompt": "你是一个有用的助手...",
    "rag_system_prompt_template": "根据以下资料回答：\n\n{context}\n\n{question}"
  }
}
```

参数约束：
- `top_k`：1–20，且须 ≤ `intermediate_k`
- `intermediate_k`：5–50，且须 ≤ `initial_k`
- `initial_k`：10–200
- `embed_weight` / `bm25_weight`：0.0–1.0
- `rag_system_prompt_template` 必须包含 `{context}` 占位符

**请求示例 (curl)：**
```bash
curl -X PUT http://localhost:8050/api/config/rag \
  -H "Content-Type: application/json" \
  -d "{  \"retrieval\": {    \"top_k\": 5,    \"initial_k\": 50,    \"intermediate_k\": 10,    \"embed_weight\": 0.6,    \"bm25_weight\": 0.4,    \"rerank_enable\": false  },  \"prompts\": {    \"conversation_system_prompt\": \"你是一个有用的助手...\",    \"rag_system_prompt_template\": \"根据以下资料回答：\n\n{context}\n\n{question}\"  }}"
```

**响应示例：**
```json
{"message": "RAG配置已保存并立即生效"}
```

---

### POST `/api/config/rag/reset`
重置 RAG 配置为系统默认值，立即热重载生效。

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/config/rag/reset
```

**响应：** 返回重置后的完整 RAG 配置（同 `GET /api/config/rag` 格式）

---

## Assets - 静态资源

**前缀：** `/api/assets`

---

### GET `/api/assets/minio/{file_path}`
从 MinIO 代理提供静态文件（图片、PDF、文档等）。

**路径参数：**
| 参数 | 说明 |
|------|------|
| file_path | MinIO 中的文件路径，如 `kb_1/file_123.pdf` |

**特殊行为：**
- PPTX/PPT 文件会自动尝试提供对应的 PDF 预览版（`xxx_preview.pdf`），不存在时回退为原始文件

**支持的内容类型：**
- 图片：PNG、JPEG、GIF、WEBP、SVG
- 文档：PDF、TXT、MD
- Office：PPTX/PPT、DOCX/DOC、XLSX/XLS

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/assets/minio/kb_1/file_123.pdf
```

**响应：** 文件流（`StreamingResponse`），缓存 1 天

**错误：**
| 状态码 | 说明 |
|--------|------|
| 400 | 路径包含 `..` 或以 `/` 开头（目录穿越攻击防护） |
| 404 | 文件不存在 |
| 503 | MinIO 客户端未初始化 |
