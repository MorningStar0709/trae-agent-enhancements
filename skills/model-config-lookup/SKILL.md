---
name: model-config-lookup
description: Look up AI model specs from official sources and output a configuration table for Trae. Use when the user asks to add/configure a model or look up context window, including Chinese requests such as 添加模型, 模型怎么填, 上下文窗口是多少, 模型参数, or 帮我查模型信息. Also covers requests mentioning specific model names like DeepSeek, Claude, GPT, Gemini, or Qwen.
---

# Model Config Lookup

## Do Not Use

Do not use this Skill when:
- The user only wants to discuss model performance or compare models qualitatively.
- The user wants to install or deploy a model locally (e.g., Ollama, vLLM).
- The user asks about API pricing only, without needing configuration fields.

## Workflow

### Step 1: Identify the Model

Extract from the user's request:
- Provider name (e.g., DeepSeek, OpenAI, Anthropic, Google)
- Model name or ID (e.g., deepseek-v4-flash, claude-sonnet-4-20250514, gpt-4.1)
- If the user only gives a vague description (e.g., "最新的 Claude"), ask which specific model.

### Step 2: Fetch Official Specs — MUST Visit Official Source

**铁律：所有数值必须来自官方网站或官方 API 文档，不得仅凭博客、教程或第三方聚合站填写。**

Priority order for data sources:
1. **Official API documentation** (highest priority)
2. **Official developer platform / console**
3. **Official blog / announcement post**
4. **Hugging Face model card** (if open-sourced)
5. **Third-party aggregator** (lowest, must note source)

#### Step 2a: Visit the Official Docs Page

Use `WebFetch` to directly fetch the provider's official documentation page. Known official doc URLs:

| Provider | Official Docs URL |
|----------|-------------------|
| DeepSeek | `https://api-docs.deepseek.com/zh-cn/` |
| OpenAI | `https://platform.openai.com/docs/models` |
| Anthropic | `https://docs.anthropic.com/en/docs/about-claude/models` |
| Google Gemini | `https://ai.google.dev/gemini-api/docs/models` |
| Xiaomi MiMo | `https://platform.xiaomimimo.com/#/docs` |
| Aliyun Qwen | `https://help.aliyun.com/zh/model-studio/getting-started/models` |
| Silicon Flow | `https://docs.siliconflow.cn/` |
| Zhipu (GLM) | `https://open.bigmodel.cn/dev/howuse/model` |
| Baidu (ERNIE) | `https://cloud.baidu.com/doc/WENXINWORKSHOP/s/clntwmv7t` |
| ByteDance (Doubao) | `https://www.volcengine.com/docs/82379/1399008` |

> 上述 URL 可能随官方站点更新而变更，若 `WebFetch` 无法获取有效内容，跳转 Step 2c 使用 `WebSearch` 重新查找。

If the provider is not in the list above, use `WebSearch` to find their official docs:
```
{provider} API documentation model list
{provider} 官方 API 文档 模型列表
```

#### Step 2b: Extract These Fields from Official Source

Always look for these specific fields:
- **Model ID**: the exact string used in API calls (e.g., `deepseek-v4-flash`)
- **Context Window**: total input context in tokens
- **Max Output Tokens**: maximum single-response output in tokens
- **API Base URL**: the provider's OpenAI-compatible endpoint
- **Multimodal support**: whether the model accepts image/vision input
- **Function/Tool Calling**: whether supported

#### Step 2c: Cross-Verify

If the official site does not clearly state a value, check secondary sources in priority order. Do NOT guess — if uncertain, state the uncertainty and provide the best-known value with a source note. When values conflict, prefer the official source and note the discrepancy.

### Step 3: Output the Configuration Table

Output a Markdown table with exactly these fields. The user copies this into a form or config file:

| 字段 | 填入值 |
|------|--------|
| API格式 | `{api_format}` |
| 自定义请求地址 | `{api_base_url}` |
| 模型ID | `{model_id}` |
| 是否开启多模态 | `{multimodal}` |
| API密钥 | `{key_hint}` |
| 模型展示名称 | `{display_name}` |
| 上下文窗口（输入） | `{context_window}` |
| 上下文窗口（输出） | `{max_output}` |
| 工具调用轮次 | `{tool_rounds}` |

Field rules:
- **API格式**: `OpenAI Chat Completions 格式` for most providers (DeepSeek, OpenAI, Google Gemini, Qwen, MiMo, most Chinese providers). `Anthropic Messages 格式` for Claude models via Anthropic's native API.
- **自定义请求地址**: The base URL. The tool automatically appends `/chat/completions` to this address, so do NOT include `/chat/completions`, `/v1/chat/completions`, or any trailing slash. Common values:
  - DeepSeek: `https://api.deepseek.com`
  - OpenAI: `https://api.openai.com/v1`
  - Anthropic (OpenAI-compatible): `https://api.anthropic.com/v1` — 注意原生端点是 `/messages` 而非 `/chat/completions`，选 `Anthropic Messages 格式` 以使用正确端点
  - Google Gemini: `https://generativelanguage.googleapis.com/v1beta/openai`
  - Aliyun Qwen: `https://dashscope.aliyuncs.com/compatible-mode/v1`
  - Silicon Flow: `https://api.siliconflow.cn/v1`
  - For other providers, look up their OpenAI-compatible endpoint.
- **模型ID**: Use the exact Model ID from the API docs, lowercase with hyphens (e.g., `deepseek-v4-flash`). This is case-sensitive.
- **是否开启多模态**: `是` if the model supports vision/image input (e.g., GPT-4o, Claude Sonnet 4, Gemini 2.5). `否` for text-only models. If uncertain, default to `否` with a note.
- **API密钥**: Provide a contextual hint based on the user's situation. If the user already has a key configured for this provider, say "同上，共用同一个 Key". Otherwise, provide the URL to the key management page.
- **模型展示名称**: Human-readable name for display in the model list (e.g., `DeepSeek V4 Flash`, `Claude Sonnet 4`).
- **上下文窗口（输入）**: Total input context window in tokens. Use the numeric value only (e.g., `1048576`, not "1M tokens").
- **上下文窗口（输出）**: Maximum output tokens in a single response. Use the numeric value only (e.g., `384000`).
- **工具调用轮次**: 根据供应商文档填写。如果文档未标注，留空或标注 "N/A"。

### Step 4: Supplementary Notes

After the table, add a brief section if relevant:
- **Thinking/reasoning models**: If the model supports a thinking mode (e.g., DeepSeek V4, Claude with extended thinking), note how to disable it if needed for simple tasks.
- **Cost reminder**: If pricing info was found, add a one-line cost summary.
- **Known caveats**: Any gotchas (e.g., "V4-Flash requires `extra_body={"thinking":{"type":"disabled"}}` to avoid empty content on some prompts").

## Multi-Model Requests

If the user asks for multiple models at once (e.g., "帮我查一下 Flash 和 Pro"), output one table per model, separated by a horizontal rule. Search for all models in parallel if possible.

## Error Handling

- **Model not found**: If the model name does not match any known provider, ask the user to clarify the provider and model name.
- **Specs unavailable**: If official docs are unreachable, use the most authoritative secondary source (e.g., model cards on Hugging Face, official blog posts). Note the source.
- **Conflicting numbers**: If multiple sources give different context window sizes, prefer the official API docs > official blog > Hugging Face model card > third-party aggregator. State the source.
