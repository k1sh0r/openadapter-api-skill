---
name: openadapter-api
description: OpenAdapter API reference and integration guide. Use this skill whenever the user mentions OpenAdapter, asks about using the OpenAdapter API, wants to set up or configure an OpenAI-compatible client to point at OpenAdapter, needs help with chat completions, embeddings, image generation, audio transcription, text-to-speech, web tools (search/scrape/crawl), function/tool calling, JSON mode, streaming, or Anthropic Messages API compatibility. Also use when the user asks about switching from OpenAI or Anthropic to OpenAdapter, configuring Claude Code or Claude Desktop with OpenAdapter, or troubleshooting OpenAdapter API errors (401, 403, 429, 502, 503). Also covers OCR, document parsing, text-to-document generation, hybrid embeddings, vector DB (collections, upsert, search, RAG), MCP Store setup, LiveKit speech-to-speech integration, burst capacity, and rate limit retry patterns.
---

# OpenAdapter API

OpenAdapter is a multi-provider AI gateway. One API key (`sk-cv-...`) gives you access to 40+ models from multiple providers. Point any OpenAI-compatible client at the gateway and it just works. Built-in vector DB, utility tools, and MCP servers included.

## Quick Reference

- **Base URL:** `https://api.openadapter.in`
- **API Key:** `sk-cv-...` (user provides their own key from [Dashboard → API Keys](https://dashboard.openadapter.in/keys))
- **Auth header:** `Authorization: Bearer sk-cv-...`

## Two Surfaces

- **OpenAI-compatible** — `https://api.openadapter.in/v1` — works with Cursor, Cline, Aider, Zed, Continue.dev, the OpenAI SDK, and anything else that speaks the OpenAI API.
- **Anthropic Messages** — `https://api.openadapter.in/v1/messages` — works with Claude Code, Claude Desktop, and the Anthropic SDK.

One API key works for both.

## Endpoints Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v1/models` | List available models |
| GET | `/v1/models/:id` | Get model details |
| POST | `/v1/chat/completions` | Chat (OpenAI format) |
| POST | `/v1/messages` | Chat (Anthropic format) |
| POST | `/v1/images/generations` | Generate images |
| POST | `/v1/embeddings` | Create vector embeddings |
| POST | `/v1/audio/transcriptions` | Transcribe audio (Whisper) |
| POST | `/v1/audio/speech` | Text-to-speech (Kokoro TTS) |
| POST | `/v1/tools/search` | Web search |
| POST | `/v1/tools/search/images` | Image search |
| POST | `/v1/tools/search/news` | News search |
| POST | `/v1/tools/search/videos` | Video search |
| POST | `/v1/tools/scrape` | Scrape a URL (fast/stealth/dynamic modes) |
| POST | `/v1/tools/scrape/markdown` | Scrape URL to markdown |
| POST | `/v1/tools/scrape/extract` | Extract structured data (CSS/XPath/regex/text selectors) |
| POST | `/v1/tools/crawl` | Multi-page crawl (depth control, URL patterns) |
| POST | `/v1/edge/ocr` | OCR — image to text |
| POST | `/v1/edge/ocr/base64` | OCR from base64 image |
| POST | `/v1/edge/docparse` | Extract text from PDF/DOCX/PPTX/XLSX/HTML |
| POST | `/v1/edge/text2doc` | Convert text to PDF/DOCX/HTML/MD/TXT |
| POST | `/v1/edge/embeddings/hybrid` | Dense + sparse + ColBERT embeddings (BGE-M3) |
| GET | `/v1/vectors/collections` | List vector collections |
| POST | `/v1/vectors/collections` | Create a vector collection |
| GET | `/v1/vectors/collections/{name}` | Get collection metadata |
| DELETE | `/v1/vectors/collections/{name}` | Delete a collection |
| POST | `/v1/vectors/collections/{name}/points` | Upsert vectors |
| POST | `/v1/vectors/collections/{name}/search` | Vector similarity search |

## Common Workflows

### Setting up a client

The pattern is always the same: change the base URL and API key on any OpenAI-compatible SDK.

**Python:**
```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-cv-...",
    base_url="https://api.openadapter.in/v1"
)
```

**JavaScript:**
```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'sk-cv-...',
  baseURL: 'https://api.openadapter.in/v1',
});
```

**Go:**
```go
import openai "github.com/sashabaranov/go-openai"

config := openai.DefaultConfig("sk-cv-...")
config.BaseURL = "https://api.openadapter.in/v1"
client := openai.NewClientWithConfig(config)
```

**PHP:**
```php
use OpenAI;

$client = OpenAI::factory()
    ->withApiKey('sk-cv-...')
    ->withBaseUri('https://api.openadapter.in/v1')
    ->make();
```

**Rust:**
```rust
use async_openai::{Client, config::OpenAIConfig};

let config = OpenAIConfig::new()
    .with_api_key("sk-cv-...")
    .with_api_base("https://api.openadapter.in/v1");
let client = Client::with_config(config);
```

**Ruby:**
```ruby
require "openai"

client = OpenAI::Client.new(
  access_token: "sk-cv-...",
  uri_base: "https://api.openadapter.in/v1"
)
```

### Claude Code / Claude Desktop setup

**One-liner (recommended):**
```bash
curl -sL "https://api.openadapter.in/api/setup/claude-code?key=YOUR_API_KEY" | bash
```
Writes `~/.claude/settings.json` with your key and role → model mapping. Override defaults: append `&sonnet=<alias>&haiku=<alias>&opus=<alias>`.

**Manual setup — set environment variables:**
```
ANTHROPIC_BASE_URL=https://api.openadapter.in
ANTHROPIC_AUTH_TOKEN=sk-cv-...
```

**Default role → model mapping:**

| Claude Code role | Gateway alias |
|-----------------|---------------|
| sonnet | Kimi-K2.5 |
| haiku | MiniMax-M2.7 |
| opus | GLM-5 |

### Anthropic SDK setup

**Python:**
```python
import anthropic

client = anthropic.Anthropic(
    api_key="sk-cv-...",
    base_url="https://api.openadapter.in"
)

message = client.messages.create(
    model="GLM-4.7",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What is the capital of France?"}]
)
```

**JavaScript:**
```javascript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: 'sk-cv-...',
  baseURL: 'https://api.openadapter.in',
});
```

### Supported Integrations

OpenAdapter works with: **Claude Code**, **Cursor**, **Continue.dev**, **Cline**, **Aider**, **Zed**, **Windsurf**, **OpenCode**, **Pi**, **OpenClaw**, **Roo Code**, **Kilo Code**, **Void**, **PearAI**, and any other OpenAI- or Anthropic-compatible client.

For detailed per-client setup instructions (one-liner installers, config files, env vars), read `references/integrations.md`.

### MCP Store

Pre-built MCP servers for Claude Code and OpenCode — one-liner install:

| Server | Purpose | Command |
|--------|---------|---------|
| Web Search | SearXNG search (requires Docker) | `curl -sL "https://api.openadapter.in/api/setup/mcp-search?key=KEY&tool=claude-code" \| bash` |
| Vision | Image analysis via QwenVL | `curl -sL "https://api.openadapter.in/api/setup/mcp-vision?key=KEY&tool=claude-code" \| bash` |
| Terminal Monitor | Background process management | `curl -sL "https://api.openadapter.in/api/setup/mcp-terminal?key=KEY&tool=claude-code" \| bash` |
| Image Generate | Text-to-image with auto-failover | `curl -sL "https://api.openadapter.in/api/setup/mcp-image-gen?key=KEY&tool=claude-code" \| bash` |
| Image Edit | Modify images with text prompt | `curl -sL "https://api.openadapter.in/api/setup/mcp-image-edit?key=KEY&tool=claude-code" \| bash` |

For OpenCode, swap `tool=claude-code` with `tool=opencode`. Windows PowerShell equivalents use `irm "..." | iex`.

### Speech to Speech (LiveKit)

Wire OpenAdapter's STT (Parakeet), LLM, and TTS into a LiveKit voice agent for real-time voice interaction. Uses `wss://edge.openadapter.in/parakeet/v1/audio/stream` for streaming STT with sub-second transcription.

## Pricing

- **LLM / embeddings calls:** Each counts as one request against your plan quota.
- **Tool calls:** Each costs 0.15 credits.
- **Vector DB calls:** Each costs 0.05 of one request.
- **Failed calls:** Not billed.

## Rate Limits

Three rolling windows: 5-hour (burst), 7-day (sustained), 30-day (monthly cap). On 429, honor the `Retry-After` header. Don't retry 4xx errors — only 429 and 5xx are retry-safe.

## API Details

For detailed API parameters, request/response formats, and advanced usage patterns (streaming, tool calling, JSON mode, embeddings models, image generation, audio, web tools, OCR, doc parsing, text-to-doc, hybrid embeddings, vector DB, RAG pattern, LiveKit integration, rate limits, error handling), read `references/api-details.md`.
