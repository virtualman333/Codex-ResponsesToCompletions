# Responses Chat Proxy

English | [中文](#中文)

A small, standalone proxy service that accepts OpenAI Responses API requests, converts them to Chat Completions requests, forwards them to a Chat-compatible upstream, and converts the upstream response back to Responses API format.

It is designed to be easy to self-host and publish as an independent open-source project. It does not depend on Django, databases, billing, or user management.

## Features

- `POST /v1/responses`: Responses API compatible endpoint.
- Converts Responses `input` and `instructions` to Chat `messages`.
- Converts Responses tool definitions to Chat Completions function tools.
- Converts non-streaming Chat Completions responses back to Responses objects.
- Converts streaming Chat Completions SSE chunks to Responses API SSE events.
- Optional `POST /v1/chat/completions` passthrough endpoint.
- Optional proxy-side bearer token authentication.
- Configured entirely with environment variables.

## Quick Start

**Linux/macOS:**

```bash
cd responses-chat-proxy
python -m venv .venv
source .venv/bin/activate
pip install -e .
cp .env.example .env
```

**Windows Command Prompt:**

```cmd
cd responses-chat-proxy
python -m venv .venv
.venv\Scripts\activate
pip install -e .
copy .env.example .env
```

**Windows PowerShell:**

```powershell
cd responses-chat-proxy
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e .
Copy-Item .env.example .env
```

Edit `.env`:

```env
UPSTREAM_BASE_URL=https://api.openai.com/v1
UPSTREAM_API_KEY=sk-your-upstream-key
PROXY_API_KEY=
```

Run the service:

```bash
python -m responses_chat_proxy
```

Or:

```bash
uvicorn responses_chat_proxy.main:app --host 0.0.0.0 --port 8000
```

## Usage

Non-streaming Responses request:

```bash
curl http://localhost:8000/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "instructions": "You are concise.",
    "input": "Say hello in one sentence."
  }'
```

Streaming Responses request:

```bash
curl -N http://localhost:8000/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "input": "Count to three.",
    "stream": true
  }'
```

If `PROXY_API_KEY` is set, clients must send:

```text
Authorization: Bearer your-proxy-key
```

The proxy always uses `UPSTREAM_API_KEY` when forwarding to the upstream provider.

## Configuration

| Variable | Default | Description |
| --- | --- | --- |
| `UPSTREAM_BASE_URL` | `https://api.openai.com/v1` | Chat-compatible upstream base URL. Include `/v1` if required by the provider. |
| `UPSTREAM_API_KEY` | empty | Bearer token used for upstream requests. |
| `PROXY_API_KEY` | empty | Optional bearer token required from proxy clients. Empty disables proxy auth. |
| `HOST` | `0.0.0.0` | Host used by `python -m responses_chat_proxy`. |
| `PORT` | `8000` | Port used by `python -m responses_chat_proxy`. |
| `REQUEST_TIMEOUT_SECONDS` | `120` | Timeout for non-streaming upstream requests. |
| `STREAM_TIMEOUT_SECONDS` | `300` | Timeout for streaming upstream requests. |
| `VERIFY_SSL` | `true` | Whether httpx verifies upstream TLS certificates. |
| `LOG_LEVEL` | `info` | Uvicorn log level. |

## API Mapping

Responses request fields:

- `instructions` becomes a leading Chat `system` message.
- string `input` becomes a Chat `user` message.
- list `input` items with `type: "message"` become Chat messages.
- `developer` role is mapped to Chat `system`.
- `input_text` and `output_text` content parts become Chat `text` parts.
- `input_image` content parts become Chat `image_url` parts.
- `max_output_tokens` becomes `max_tokens`.
- Responses flat function tools become Chat nested `{"type":"function","function":...}` tools.
- `reasoning.effort` becomes `reasoning_effort`.

Chat response fields:

- `chatcmpl-*` IDs are mapped to `resp-*`.
- assistant `message.content` becomes `output_text`.
- Chat `usage.prompt_tokens` becomes Responses `usage.input_tokens`.
- Chat `usage.completion_tokens` becomes Responses `usage.output_tokens`.
- streaming Chat SSE events are converted to Responses SSE events such as `response.created`, `response.output_text.delta`, and `response.completed`.

## Development

```bash
pip install -e ".[dev]"
pytest
```

## License

MIT License. See [LICENSE](LICENSE).

---

## 中文

一个轻量、独立的代理服务：接收 OpenAI Responses API 请求，将其转换为 Chat Completions 请求并转发到兼容 Chat 协议的上游，再把上游响应转换回 Responses API 格式。

该项目适合单独开源和自部署，不依赖 Django、数据库、计费系统或用户管理系统。

## 功能

- `POST /v1/responses`：兼容 Responses API 的入口。
- 将 Responses 的 `input` 和 `instructions` 转换为 Chat 的 `messages`。
- 将 Responses 工具定义转换为 Chat Completions function tools。
- 将非流式 Chat Completions 响应转换回 Responses object。
- 将流式 Chat Completions SSE chunk 转换为 Responses API SSE 事件。
- 可选 `POST /v1/chat/completions` 原样透传端点。
- 可选代理侧 Bearer Token 鉴权。
- 完全通过环境变量配置。

## 快速开始

**Linux/macOS:**

```bash
cd responses-chat-proxy
python -m venv .venv
source .venv/bin/activate
pip install -e .
cp .env.example .env
```

**Windows 命令提示符 (Command Prompt):**

```cmd
cd responses-chat-proxy
python -m venv .venv
.venv\Scripts\activate
pip install -e .
copy .env.example .env
```

**Windows PowerShell:**

```powershell
cd responses-chat-proxy
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e .
Copy-Item .env.example .env
```

编辑 `.env`：

```env
UPSTREAM_BASE_URL=https://api.openai.com/v1
UPSTREAM_API_KEY=sk-your-upstream-key
PROXY_API_KEY=
```

启动服务：

```bash
python -m responses_chat_proxy
```

或者：

```bash
uvicorn responses_chat_proxy.main:app --host 0.0.0.0 --port 8000
```

## 使用示例

非流式 Responses 请求：

```bash
curl http://localhost:8000/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "instructions": "You are concise.",
    "input": "Say hello in one sentence."
  }'
```

流式 Responses 请求：

```bash
curl -N http://localhost:8000/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "input": "Count to three.",
    "stream": true
  }'
```

如果设置了 `PROXY_API_KEY`，客户端请求需要携带：

```text
Authorization: Bearer your-proxy-key
```

代理转发到上游时始终使用 `UPSTREAM_API_KEY`。

## 配置项

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `UPSTREAM_BASE_URL` | `https://api.openai.com/v1` | 兼容 Chat 协议的上游 base URL。如果上游需要 `/v1`，这里要包含 `/v1`。 |
| `UPSTREAM_API_KEY` | 空 | 转发到上游时使用的 Bearer Token。 |
| `PROXY_API_KEY` | 空 | 代理对客户端要求的 Bearer Token。留空表示关闭代理侧鉴权。 |
| `HOST` | `0.0.0.0` | 使用 `python -m responses_chat_proxy` 启动时监听的地址。 |
| `PORT` | `8000` | 使用 `python -m responses_chat_proxy` 启动时监听的端口。 |
| `REQUEST_TIMEOUT_SECONDS` | `120` | 非流式上游请求超时时间。 |
| `STREAM_TIMEOUT_SECONDS` | `300` | 流式上游请求超时时间。 |
| `VERIFY_SSL` | `true` | httpx 是否校验上游 TLS 证书。 |
| `LOG_LEVEL` | `info` | Uvicorn 日志等级。 |

## 协议映射

Responses 请求字段：

- `instructions` 转为 Chat 的首条 `system` message。
- 字符串 `input` 转为 Chat 的 `user` message。
- 列表形式 `input` 中 `type: "message"` 的元素转为 Chat messages。
- `developer` role 映射为 Chat 的 `system` role。
- `input_text` 和 `output_text` 内容块转为 Chat `text` 内容块。
- `input_image` 内容块转为 Chat `image_url` 内容块。
- `max_output_tokens` 转为 `max_tokens`。
- Responses 扁平 function tool 转为 Chat 嵌套格式 `{"type":"function","function":...}`。
- `reasoning.effort` 转为 `reasoning_effort`。

Chat 响应字段：

- `chatcmpl-*` ID 映射为 `resp-*`。
- assistant `message.content` 转为 `output_text`。
- Chat `usage.prompt_tokens` 转为 Responses `usage.input_tokens`。
- Chat `usage.completion_tokens` 转为 Responses `usage.output_tokens`。
- 流式 Chat SSE 事件会转换为 Responses SSE 事件，例如 `response.created`、`response.output_text.delta`、`response.completed`。

## 开发

```bash
pip install -e ".[dev]"
pytest
```

## 开源协议

MIT License。详见 [LICENSE](LICENSE)。
