---
name: openadapter-api
description: OpenAdapter API reference and integration guide. Use this skill whenever the user mentions OpenAdapter, asks about using the OpenAdapter API, wants to set up or configure an OpenAI-compatible client to point at OpenAdapter, needs help with chat completions, embeddings, image generation, audio transcription, text-to-speech, web tools (search/scrape/crawl), function/tool calling, JSON mode, streaming, or Anthropic Messages API compatibility. Also use when the user asks about switching from OpenAI or Anthropic to OpenAdapter, configuring Claude Code or Claude Desktop with OpenAdapter, or troubleshooting OpenAdapter API errors (401, 403, 429, 502, 503).
---

# OpenAdapter API

OpenAdapter is a drop-in replacement for the OpenAI API. One API key gives access to 40+ models from multiple providers. Point any OpenAI-compatible client at the gateway and it just works.

## Quick Reference

- **Base URL:** `https://api.openadapter.in`
- **API Key:** `OPENADAPTER_KEY` (user provides their own key)
- **Auth header:** `Authorization: Bearer OPENADAPTER_KEY`

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
| POST | `/v1/tools/scrape` | Scrape a URL |
| POST | `/v1/tools/scrape/markdown` | Scrape URL to markdown |
| POST | `/v1/tools/scrape/extract` | Extract structured data |
| POST | `/v1/tools/crawl` | Multi-page crawl |

## Common Workflows

### Setting up a client

The pattern is always the same: change the base URL and API key on any OpenAI-compatible SDK.

**Python:**
```python
from openai import OpenAI

client = OpenAI(
    api_key="OPENADAPTER_KEY",
    base_url="https://api.openadapter.in/v1"
)
```

**JavaScript:**
```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'OPENADAPTER_KEY',
  baseURL: 'https://api.openadapter.in/v1',
});
```

**Go:**
```go
import openai "github.com/sashabaranov/go-openai"

config := openai.DefaultConfig("OPENADAPTER_KEY")
config.BaseURL = "https://api.openadapter.in/v1"
client := openai.NewClientWithConfig(config)
```

**PHP:**
```php
use OpenAI;

$client = OpenAI::factory()
    ->withApiKey('OPENADAPTER_KEY')
    ->withBaseUri('https://api.openadapter.in/v1')
    ->make();
```

**Rust:**
```rust
use async_openai::{Client, config::OpenAIConfig};

let config = OpenAIConfig::new()
    .with_api_key("OPENADAPTER_KEY")
    .with_api_base("https://api.openadapter.in/v1");
let client = Client::with_config(config);
```

**Ruby:**
```ruby
require "openai"

client = OpenAI::Client.new(
  access_token: "OPENADAPTER_KEY",
  uri_base: "https://api.openadapter.in/v1"
)
```

### Claude Code / Claude Desktop setup

Set environment variables:
```
ANTHROPIC_BASE_URL=https://api.openadapter.in
ANTHROPIC_AUTH_TOKEN=OPENADAPTER_KEY
```
This uses the Anthropic Messages format on `/v1/messages`.

### Anthropic SDK setup

**Python:**
```python
import anthropic

client = anthropic.Anthropic(
    api_key="OPENADAPTER_KEY",
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
  apiKey: 'OPENADAPTER_KEY',
  baseURL: 'https://api.openadapter.in',
});
```

## API Details

For detailed API parameters, request/response formats, and advanced usage patterns (streaming, tool calling, JSON mode, embeddings models, image generation, audio, web tools, rate limits, error handling), read `references/api-details.md`.
