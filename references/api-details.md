# OpenAdapter API Detailed Reference

## Table of Contents

- [Chat Completions](#chat-completions)
- [Streaming](#streaming)
- [Function / Tool Calling](#function--tool-calling)
- [JSON Mode / Structured Output](#json-mode--structured-output)
- [Anthropic Messages API](#anthropic-messages-api)
- [Embeddings](#embeddings)
- [Image Generation](#image-generation)
- [Audio Transcription](#audio-transcription)
- [Text-to-Speech](#text-to-speech)
- [Speech to Speech (LiveKit)](#speech-to-speech-livekit)
- [Web Tools](#web-tools)
  - [Web Search](#web-search)
  - [Image Search](#image-search)
  - [News Search](#news-search)
  - [Video Search](#video-search)
  - [Scrape URL](#scrape-url)
  - [Page to Markdown](#page-to-markdown)
  - [Extract Structured Data](#extract-structured-data)
  - [Multi-page Crawl](#multi-page-crawl)
- [Edge Tools](#edge-tools)
  - [OCR — Image to Text](#ocr--image-to-text)
  - [Doc Parse](#doc-parse)
  - [Text to Document](#text-to-document)
  - [Hybrid Embeddings](#hybrid-embeddings)
- [Vector DB](#vector-db)
  - [Collections](#collections)
  - [Upsert & Embeddings](#upsert--embeddings)
  - [Search](#vector-search)
  - [RAG Pattern](#rag-pattern)
- [MCP Store](#mcp-store)
- [Rate Limits & Burst Capacity](#rate-limits--burst-capacity)
- [Error Codes](#error-codes)
- [Compatible Libraries](#compatible-libraries)

---

## Chat Completions

`POST https://api.openadapter.in/v1/chat/completions`

The primary endpoint. Accepts OpenAI-format requests and routes to the best available provider automatically.

### Supported Parameters

`model`, `messages`, `temperature`, `max_tokens`, `top_p`, `frequency_penalty`, `presence_penalty`, `stop`, `n`, `stream`, `response_format`, `tools`, `tool_choice`, `seed`, `logprobs`, `top_logprobs`

### Basic Example

**Python:**
```python
response = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "system", "content": "You are a helpful coding assistant."},
        {"role": "user", "content": "Write a Python function to reverse a linked list."}
    ],
    temperature=0.7,
    max_tokens=2048
)

print(response.choices[0].message.content)
print(f"Tokens used: {response.usage.total_tokens}")
```

**JavaScript:**
```javascript
const response = await client.chat.completions.create({
  model: 'glm-4.7',
  messages: [
    { role: 'system', content: 'You are a helpful coding assistant.' },
    { role: 'user', content: 'Write a function to reverse a linked list.' },
  ],
  temperature: 0.7,
  max_tokens: 2048,
});

console.log(response.choices[0].message.content);
console.log(`Tokens used: ${response.usage.total_tokens}`);
```

**curl:**
```bash
curl https://api.openadapter.in/v1/chat/completions \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-4.7",
    "messages": [
      {"role": "system", "content": "You are a helpful coding assistant."},
      {"role": "user", "content": "Write a Python function to reverse a linked list."}
    ],
    "temperature": 0.7,
    "max_tokens": 2048
  }'
```

## Streaming

Add `stream: true` to get Server-Sent Events. Tokens arrive as they're generated.

**Python:**
```python
stream = client.chat.completions.create(
    model="DeepSeek-V3",
    messages=[{"role": "user", "content": "Explain quantum computing simply."}],
    stream=True
)

for chunk in stream:
    delta = chunk.choices[0].delta
    if delta.content:
        print(delta.content, end="", flush=True)
```

**JavaScript:**
```javascript
const stream = await client.chat.completions.create({
  model: 'DeepSeek-V3',
  messages: [{ role: 'user', content: 'Explain quantum computing simply.' }],
  stream: true,
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content;
  if (content) process.stdout.write(content);
}
```

**curl:**
```bash
curl https://api.openadapter.in/v1/chat/completions \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -N \
  -d '{
    "model": "DeepSeek-V3",
    "messages": [{"role": "user", "content": "Explain quantum computing simply."}],
    "stream": true
  }'
# Each line: data: {"choices":[{"delta":{"content":"..."}}]}
# Stream ends with: data: [DONE]
```

## Function / Tool Calling

Define tools in OpenAI format. The gateway handles format conversion between providers — define tools once in OpenAI format and it works everywhere.

**Python:**
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["city"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)

msg = response.choices[0].message
if msg.tool_calls:
    for call in msg.tool_calls:
        print(f"Call: {call.function.name}({call.function.arguments})")
```

**JavaScript:**
```javascript
const tools = [{
  type: 'function',
  function: {
    name: 'get_weather',
    description: 'Get weather for a city',
    parameters: {
      type: 'object',
      properties: {
        city: { type: 'string', description: 'City name' },
        unit: { type: 'string', enum: ['celsius', 'fahrenheit'] },
      },
      required: ['city'],
    },
  },
}];

const response = await client.chat.completions.create({
  model: 'glm-4.7',
  messages: [{ role: 'user', content: "What's the weather in Tokyo?" }],
  tools,
  tool_choice: 'auto',
});

const msg = response.choices[0].message;
if (msg.tool_calls) {
  for (const call of msg.tool_calls) {
    console.log(`Call: ${call.function.name}(${call.function.arguments})`);
  }
}
```

**curl:**
```bash
curl https://api.openadapter.in/v1/chat/completions \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-4.7",
    "messages": [{"role": "user", "content": "What is the weather in Tokyo?"}],
    "tools": [{
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get weather for a city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string"},
            "unit": {"type": "string", "enum": ["celsius","fahrenheit"]}
          },
          "required": ["city"]
        }
      }
    }],
    "tool_choice": "auto"
  }'
```

## JSON Mode / Structured Output

Force the model to respond in valid JSON using `response_format`.

**Python:**
```python
response = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "system", "content": "Extract contact info as JSON."},
        {"role": "user", "content": "John Smith, email john@example.com, phone 555-0123"}
    ],
    response_format={"type": "json_object"}
)

import json
data = json.loads(response.choices[0].message.content)
```

**JavaScript:**
```javascript
const response = await client.chat.completions.create({
  model: 'glm-4.7',
  messages: [
    { role: 'system', content: 'Extract contact info as JSON.' },
    { role: 'user', content: 'John Smith, email john@example.com, phone 555-0123' },
  ],
  response_format: { type: 'json_object' },
});

const data = JSON.parse(response.choices[0].message.content);
```

**curl:**
```bash
curl https://api.openadapter.in/v1/chat/completions \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-4.7",
    "messages": [
      {"role": "system", "content": "Extract contact info as JSON."},
      {"role": "user", "content": "John Smith, email john@example.com, phone 555-0123"}
    ],
    "response_format": {"type": "json_object"}
  }'
```

Tip: Always include "JSON" in the system prompt when using JSON mode. Most models need this hint to produce valid JSON reliably.

## Anthropic Messages API

OpenAdapter supports the native Anthropic Messages format on `/v1/messages`. Same API key works for both endpoints. This is what Claude Code, Claude Desktop, and the Anthropic SDK use.

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
print(message.content[0].text)
```

**JavaScript:**
```javascript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: 'OPENADAPTER_KEY',
  baseURL: 'https://api.openadapter.in',
});

const message = await client.messages.create({
  model: 'GLM-4.7',
  max_tokens: 1024,
  messages: [
    { role: 'user', content: 'What is the capital of France?' }
  ],
});
console.log(message.content[0].text);
```

**curl:**
```bash
curl https://api.openadapter.in/v1/messages \
  -H "x-api-key: OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "GLM-4.7",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
```

## Embeddings

`POST https://api.openadapter.in/v1/embeddings`

Create vector embeddings for search, RAG, clustering, and semantic similarity. Standard OpenAI Embeddings API format.

### Discovering Embedding Models

Call `GET /v1/models` and use entries where `model_type` is `"embedding"`. Your plan may restrict which models appear (for example, curated lists on specific plans).

Aliases are case-sensitive — use the exact id from the models list.

Some aliases (for example `qwen3-embedding`) may show `model_type: "chat"` in the models list for historical routing reasons but still work with `POST /v1/embeddings`. If the embeddings call succeeds, the alias is valid.

### Response Shape

Matches OpenAI: `object` with `data[]` containing `embedding`, `index`, `model`, and usually `usage` with `total_tokens`.

### Request Body

| Field | Required | Notes |
|-------|----------|-------|
| model | Yes | Embedding alias, e.g. `qwen3-embedding-small` |
| input | Yes | A string or array of strings |
| encoding_format | No | `float` (default) |
| dimensions | No | Reduces vector size only if upstream model supports it |
| user | No | Optional end-user id for logging |

### Examples

**curl:**
```bash
curl https://api.openadapter.in/v1/embeddings \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3-embedding-small",
    "input": ["hello world", "second line"]
  }'
```

**Python:**
```python
resp = client.embeddings.create(
    model="qwen3-embedding-small",
    input=["Hello from OpenAdapter", "Batch item two"],
)
for item in resp.data:
    print(len(item.embedding), "dimensions")
```

**JavaScript:**
```javascript
const resp = await client.embeddings.create({
  model: 'jina-embeddings-v5',
  input: 'Single string input is also valid.',
});
console.log(resp.data[0].embedding.length);
```

### Embedding Models

Use one model per index or vector store. Mixing embeddings from different models breaks similarity search.

| Model alias | Vector size | Notes |
|-------------|-------------|-------|
| qwen3-embedding-small | 1024 | General text (default workhorse), alias for Qwen3 0.6B on OpenAdapter Edge |
| Qwen3-Embedding-0.6B | 1024 | Alternative display name for tooling that expects this id |
| qwen3-embedding | 1024 | Convenience alias |
| jina-embeddings-v5 | 1024 | Strong multilingual / general retrieval, hosted on OpenAdapter Edge (Jina v5 text small) |
| bge-m3 | 1024 | Dense retrieval, popular for RAG, OpenAdapter Edge (BAAI bge-m3) |
| nv-embedqa-e5-v5 | 1024 | NVIDIA embedding model, routed via NVIDIA |

Sizes above are what the gateway returns today; always verify with a test call if you rely on exact dimensionality for your DB schema.

### RAG and the Dashboard

The Vector DB page in the dashboard uses embeddings for ingest and query. Pick an embedding model in the collection settings and keep it consistent for that collection.

### Limits and Behavior

- **Quota**: Each call consumes request quota (and token-based accounting where applicable), similar to other `/v1` routes.
- **Rate limits**: Your plan RPM and per-model tier limits apply unless your plan explicitly disables them.
- **Routing**: The gateway picks a healthy provider key for the alias; you do not pass provider credentials.
- **Input size**: Very long inputs may be truncated or rejected by the upstream model — stay within reasonable chunk sizes for your use case.
- **Model restrictions**: If a model is not allowed for your plan, the API returns an error listing allowed models when your plan uses model restrictions.

## Image Generation

`POST https://api.openadapter.in/v1/images/generations`

DALL-E compatible parameters. Routes to the best available image model.

**Python:**
```python
response = client.images.generate(
    model="qwen-image-2512",
    prompt="A serene mountain lake at sunset with vibrant colors",
    n=1,
    size="1024x1024"
)

image_url = response.data[0].url
# or response.data[0].b64_json if response_format="b64_json"
```

**JavaScript:**
```javascript
const response = await client.images.generate({
  model: 'qwen-image-2512',
  prompt: 'A serene mountain lake at sunset with vibrant colors',
  n: 1,
  size: '1024x1024',
});

console.log(response.data[0].url);
```

**curl:**
```bash
curl https://api.openadapter.in/v1/images/generations \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen-image-2512",
    "prompt": "A serene mountain lake at sunset with vibrant colors",
    "n": 1,
    "size": "1024x1024"
  }'
```

## Audio Transcription

`POST https://api.openadapter.in/v1/audio/transcriptions`

Whisper-compatible. Supports mp3, mp4, wav, webm, ogg. Uses multipart/form-data upload.

**curl (basic):**
```bash
curl https://api.openadapter.in/v1/audio/transcriptions \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -F model="whisper-large-v3" \
  -F file="@recording.mp3"
```

**curl (Telegram .ogg with verbose JSON):**
```bash
curl https://api.openadapter.in/v1/audio/transcriptions \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -F model="whisper-large-v3-turbo" \
  -F response_format="verbose_json" \
  -F file="@voice-message.ogg;type=audio/ogg"
```

**Python:**
```python
audio_file = open("recording.mp3", "rb")
transcript = client.audio.transcriptions.create(
    model="whisper-large-v3",
    file=audio_file
)
print(transcript.text)
```

**JavaScript:**
```javascript
import fs from 'fs';

const transcript = await client.audio.transcriptions.create({
  model: 'whisper-large-v3',
  file: fs.createReadStream('recording.mp3'),
});

console.log(transcript.text);
```

Telegram voice messages (.ogg) work with `whisper-large-v3-turbo`, `whisper-large-v3`, or `whisper-1`.

## Text-to-Speech

`POST https://api.openadapter.in/v1/audio/speech`

Kokoro TTS, OpenAI-compatible drop-in.

**Python:**
```python
with client.audio.speech.with_streaming_response.create(
    model="tts-1",
    voice="alloy",
    input="Hello from OpenAdapter!"
) as response:
    response.stream_to_file("speech.mp3")
```

**JavaScript:**
```javascript
import fs from 'fs';

const response = await client.audio.speech.create({
  model: 'tts-1',
  voice: 'alloy',
  input: 'Hello from OpenAdapter!',
});

const buffer = Buffer.from(await response.arrayBuffer());
fs.writeFileSync('speech.mp3', buffer);
```

**curl:**
```bash
curl https://api.openadapter.in/v1/audio/speech \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "tts-1", "voice": "alloy", "input": "Hello world!"}' \
  --output speech.mp3
```

### Voices

Standard OpenAI voices: `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer`. Also Kokoro-native: `af_heart`, `am_adam`.

### Output Formats

Set `response_format`: `mp3` (default), `wav`, `opus`, `flac`, `pcm`.

## Speech to Speech (LiveKit)

Wire OpenAdapter's STT (Parakeet), LLM, and TTS into a LiveKit voice agent for real-time, low-latency voice interaction.

Key benefits:
- **Fast STT** — BBPC Parakeet for sub-second transcription
- **Unified gateway** — multiple LLMs via a single OpenAI-compatible endpoint
- **Low latency** — optimized for real-time voice

### Prerequisites

- A LiveKit project (Cloud or self-hosted)
- An OpenAdapter API Key (`sk-cv-...`)
- A BBPC Parakeet API Key (for STT)
- Python 3.10+

### Environment Setup

```
# LiveKit
LIVEKIT_URL=wss://your-livekit-project.livekit.cloud
LIVEKIT_API_KEY=your_livekit_key
LIVEKIT_API_SECRET=your_livekit_secret

# OpenAdapter
OPENADAPTER_API_KEY=sk-cv-your-key
OPENADAPTER_BASE_URL=https://api.openadapter.in/v1

# Parakeet STT
PARAKEET_API_KEY=your_parakeet_key
PARAKEET_WS_URL=wss://edge.openadapter.in/parakeet/v1/audio/stream

# Optional: model selection
OPENADAPTER_LLM_MODEL=Kimi-K2.5
OPENADAPTER_TTS_MODEL=tts-1
OPENADAPTER_TTS_VOICE=alloy
```

### Install Dependencies

```bash
pip install livekit-agents livekit-plugins-openai python-dotenv websockets numpy
```

### Agent Configuration

```python
import os
from livekit.agents import Agent, AutoSubscribe, JobContext, WorkerOptions, cli
from livekit.agents.voice import AgentSession
from livekit.plugins import openai as lk_openai
from parakeet_stt import ParakeetSTT

async def entrypoint(ctx: JobContext) -> None:
    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)

    stt = ParakeetSTT(
        ws_url=os.environ["PARAKEET_WS_URL"],
        api_key=os.environ["PARAKEET_API_KEY"],
    )

    llm = lk_openai.LLM(
        model=os.getenv("OPENADAPTER_LLM_MODEL", "Kimi-K2.5"),
        api_key=os.environ["OPENADAPTER_API_KEY"],
        base_url=os.environ["OPENADAPTER_BASE_URL"],
    )

    tts = lk_openai.TTS(
        model=os.getenv("OPENADAPTER_TTS_MODEL", "tts-1"),
        voice=os.getenv("OPENADAPTER_TTS_VOICE", "alloy"),
        api_key=os.environ["OPENADAPTER_API_KEY"],
        base_url=os.environ["OPENADAPTER_BASE_URL"],
    )

    agent = Agent(instructions="You are a helpful voice assistant.")
    session = AgentSession(stt=stt, llm=llm, tts=tts)

    await session.start(agent, room=ctx.room)

if __name__ == "__main__":
    cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint))
```

### Running

```bash
# Dev mode (hot-reload)
python agent.py dev

# Production
python agent.py start
```

The Parakeet STT plugin (`parakeet_stt.py`) is provided by OpenAdapter — see the Speech to Speech docs page for the full plugin source code.

## Web Tools

Each tool call costs 0.15 credits. Available on the `free` plan and above. Failed calls are not billed.

### Web Search

`POST /v1/tools/search`

Search the web with multiple engines. Get titles, URLs, snippets, and suggestions.

```bash
curl https://api.openadapter.in/v1/tools/search \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "latest AI news", "num_results": 10, "language": "en"}'
```

### Image Search

`POST /v1/tools/search/images`

Search for images across multiple engines. Returns image URLs, thumbnails, sources, and resolutions.

```bash
curl https://api.openadapter.in/v1/tools/search/images \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "mountain sunset", "num_results": 10}'
```

### News Search

`POST /v1/tools/search/news`

Search recent news articles. Headlines, URLs, snippets, and publication dates.

```bash
curl https://api.openadapter.in/v1/tools/search/news \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "OpenAI funding", "num_results": 10}'
```

### Video Search

`POST /v1/tools/search/videos`

Search for videos. Titles, URLs, thumbnails, duration, and channel info.

```bash
curl https://api.openadapter.in/v1/tools/search/videos \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "react tutorial", "num_results": 10}'
```

### Scrape URL

`POST /v1/tools/scrape` — Scrape any webpage. Extract text, links, images, metadata.

Supports `mode` parameter: `fast`, `stealth`, or `dynamic` (JS-rendered).

```bash
curl https://api.openadapter.in/v1/tools/scrape \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "mode": "fast"}'
```

### Page to Markdown

`POST /v1/tools/scrape/markdown` — Convert any webpage to clean Markdown. Ideal for feeding web content into LLMs.

```bash
curl https://api.openadapter.in/v1/tools/scrape/markdown \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/article"}'
```

### Extract Structured Data

`POST /v1/tools/scrape/extract` — Extract structured data using CSS, XPath, regex, or text selectors. Returns clean JSON keyed by selector `name`.

`selectors` is an array of selector objects:

| Field | Notes |
|-------|-------|
| name | Output key for the value(s) found |
| type | `css`, `xpath`, `regex`, or `text` |
| query | The selector / pattern itself |
| attribute | Optional — attribute name to extract instead of inner text (e.g. `href`) |
| all | Optional — set `true` to return an array of all matches instead of just the first |

`mode` accepts `fast`, `stealth`, or `dynamic` (JS-rendered).

```bash
curl https://api.openadapter.in/v1/tools/scrape/extract \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "selectors": [
      {"name": "title", "type": "css", "query": "h1"},
      {"name": "links", "type": "css", "query": "a", "attribute": "href", "all": true}
    ],
    "mode": "fast"
  }'
```

### Multi-page Crawl

`POST /v1/tools/crawl` — Crawl multiple pages on a site. Depth control, URL patterns, and domain filtering.

```bash
curl https://api.openadapter.in/v1/tools/crawl \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "max_pages": 5, "max_depth": 2}'
```

## Edge Tools

Edge tools live under `/v1/edge/*`. Each call costs 0.15 credits. Available on the `free` plan and above.

### OCR — Image to Text

`POST /v1/edge/ocr` — Extract text from images using OCR. Upload an image file or provide a base64-encoded image.

```bash
curl https://api.openadapter.in/v1/edge/ocr \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -F "file=@image.png"
```

Base64 variant:

```bash
curl https://api.openadapter.in/v1/edge/ocr/base64 \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"image": "<base64>"}'
```

### Doc Parse

`POST /v1/edge/docparse` — Extract text from documents: PDF, DOCX, PPTX, XLSX, HTML. Upload a file and get clean text back.

```bash
curl https://api.openadapter.in/v1/edge/docparse \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -F "file=@document.pdf"
```

### Text to Document

`POST /v1/edge/text2doc` — Convert text into a downloadable document. Supports PDF, DOCX, HTML, Markdown, and TXT formats.

```bash
curl https://api.openadapter.in/v1/edge/text2doc \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "# Hello\n\nWorld", "format": "pdf"}' \
  --output document.pdf
```

### Hybrid Embeddings

`POST /v1/edge/embeddings/hybrid` — Generate dense + sparse + ColBERT embeddings using BGE-M3. Ideal for hybrid search combining BM25 and vector similarity.

```bash
curl https://api.openadapter.in/v1/edge/embeddings/hybrid \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"input": "Search query", "model": "bge-m3"}'
```

## Vector DB

Built-in vector storage for embeddings, search, and RAG — uses your existing API key. The gateway proxies `/v1/vectors/*` calls through to the upstream vector service after auto-provisioning a tenant for your account on the first call.

Each vector API call costs **0.05 of one request** against your plan quota.

### Quick Map

| Endpoint | Purpose |
|----------|---------|
| `GET /v1/vectors/collections` | List your collections |
| `POST /v1/vectors/collections` | Create a collection |
| `GET /v1/vectors/collections/{name}` | Collection metadata |
| `DELETE /v1/vectors/collections/{name}` | Drop a collection |
| `POST /v1/vectors/collections/{name}/points` | Upsert vectors |
| `POST /v1/vectors/collections/{name}/search` | Vector similarity search |

### Collections

**Create:**
```bash
curl -X POST https://api.openadapter.in/v1/vectors/collections \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "docs",
    "vector_size": 1024,
    "distance": "Cosine"
  }'
```

| Field | Notes |
|-------|-------|
| name | Unique within your account |
| vector_size | Must match your embedding model's output dimension |
| distance | `Cosine` is standard. Other values depend on the upstream service. |

**List:**
```bash
curl https://api.openadapter.in/v1/vectors/collections \
  -H "Authorization: Bearer OPENADAPTER_KEY"
```

**Inspect:**
```bash
curl https://api.openadapter.in/v1/vectors/collections/docs \
  -H "Authorization: Bearer OPENADAPTER_KEY"
```

**Delete:**
```bash
curl -X DELETE https://api.openadapter.in/v1/vectors/collections/docs \
  -H "Authorization: Bearer OPENADAPTER_KEY"
```

Collections are isolated per API key — two users with the same collection name don't see each other's data.

### Upsert & Embeddings

**Generate embeddings:**
```bash
curl -X POST https://api.openadapter.in/v1/embeddings \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen3-embedding-small", "input": ["chunk 1 text", "chunk 2 text"]}'
```

**Upsert points:**
```bash
curl -X POST https://api.openadapter.in/v1/vectors/collections/docs/points \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "points": [
      {"id": 1, "vector": [0.01, -0.02, ...], "payload": {"text": "chunk 1 text", "source": "intro.md"}},
      {"id": 2, "vector": [0.03, 0.04, ...], "payload": {"text": "chunk 2 text", "source": "intro.md"}}
    ]
  }'
```

Re-upserting an existing `id` overwrites the point.

**Python end-to-end:**
```python
import requests

API_KEY = "sk-cv-..."
BASE = "https://api.openadapter.in"
headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

chunks = ["chunk 1 text", "chunk 2 text"]

resp = requests.post(f"{BASE}/v1/embeddings", json={
    "model": "qwen3-embedding-small", "input": chunks
}, headers=headers)
vectors = [d["embedding"] for d in resp.json()["data"]]

requests.post(f"{BASE}/v1/vectors/collections/docs/points", json={
    "points": [
        {"id": i + 1, "vector": v, "payload": {"text": t}}
        for i, (v, t) in enumerate(zip(vectors, chunks))
    ]
}, headers=headers)
```

### Vector Search

```bash
curl -X POST https://api.openadapter.in/v1/vectors/collections/docs/search \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"vector": [0.01, -0.02, ...], "limit": 5}'
```

Response:
```json
{
  "results": [
    {"id": 7, "score": 0.91, "payload": {"text": "..."}}
  ]
}
```

`score` is the similarity from whatever distance metric the collection was created with.

### RAG Pattern

Three calls — embed, search, chat — and you have retrieval-augmented generation.

**Python:**
```python
import requests

API_KEY = "sk-cv-..."
BASE = "https://api.openadapter.in"
headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}

def rag_query(question, collection="docs", top_k=5):
    vec = requests.post(f"{BASE}/v1/embeddings", json={
        "model": "qwen3-embedding-small", "input": [question]
    }, headers=headers).json()["data"][0]["embedding"]

    results = requests.post(
        f"{BASE}/v1/vectors/collections/{collection}/search",
        json={"vector": vec, "limit": top_k},
        headers=headers,
    ).json()["results"]
    context = "\n\n".join(f"[{i+1}] {r['payload']['text']}" for i, r in enumerate(results))

    resp = requests.post(f"{BASE}/v1/chat/completions", json={
        "model": "GLM-4.7",
        "messages": [
            {"role": "system", "content": f"Answer using context:\n{context}"},
            {"role": "user", "content": question},
        ],
    }, headers=headers).json()
    return resp["choices"][0]["message"]["content"]

print(rag_query("How do I create a collection?"))
```

Tips:
- Use the same embedding model at ingest and query time — different dimensions / spaces won't compare.
- Top-K of 3–8 is usually enough for chat-style RAG; more chunks dilute the signal.
- Include payload metadata (filenames, headings) so the model can cite sources.

## MCP Store

Drop-in MCP servers that plug OpenAdapter features into Claude Code, OpenCode, and other MCP-aware tools. All use your OpenAdapter API key and bill against your normal plan quota.

### Available Servers

**Web Search** — Real-time web search via SearXNG. Requires Docker.
```bash
curl -sL "https://api.openadapter.in/api/setup/mcp-search?key=YOUR_API_KEY&tool=claude-code" | bash
```

**Vision** — Image analysis using QwenVL. No local GPU needed.
```bash
curl -sL "https://api.openadapter.in/api/setup/mcp-vision?key=YOUR_API_KEY&tool=claude-code" | bash
```

**Terminal Monitor** — Run and monitor background processes from your agent.
```bash
curl -sL "https://api.openadapter.in/api/setup/mcp-terminal?key=YOUR_API_KEY&tool=claude-code" | bash
```

**Image Generate** — Text-to-image with auto-failover across models.
```bash
curl -sL "https://api.openadapter.in/api/setup/mcp-image-gen?key=YOUR_API_KEY&tool=claude-code" | bash
```

**Image Edit** — Modify existing images with a text prompt.
```bash
curl -sL "https://api.openadapter.in/api/setup/mcp-image-edit?key=YOUR_API_KEY&tool=claude-code" | bash
```

For OpenCode, swap `tool=claude-code` with `tool=opencode`. Windows PowerShell equivalents use `irm "..." | iex`.

## Rate Limits & Burst Capacity

Plan defines request limits across three rolling windows:

- **Burst** — Refills within a couple of minutes. Lets you fire parallel requests without hitting 429.
- **Sustained** — After the burst is spent, requests flow at the plan's steady rate.
- **5-hour window** — short-term limit
- **7-day window** — medium-term usage cap
- **Monthly window** — overall quota

Windows reset automatically on a rolling basis. When a limit is reached, the API returns `429 Too Many Requests` with a `Retry-After` header.

### Handling 429

Always honor the `Retry-After` header. Don't aggressive-retry without backoff.

**Python:**
```python
import time, requests

def call_api(body, max_retries=3):
    for _ in range(max_retries):
        res = requests.post(
            'https://api.openadapter.in/v1/chat/completions',
            headers={
                'Authorization': 'Bearer sk-cv-...',
                'Content-Type': 'application/json',
            },
            json=body,
        )
        if res.status_code == 429:
            time.sleep(int(res.headers.get('Retry-After', 5)))
            continue
        res.raise_for_status()
        return res.json()
    raise Exception("Max retries exceeded")
```

**JavaScript:**
```javascript
async function callAPI(body, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const res = await fetch('https://api.openadapter.in/v1/chat/completions', {
      method: 'POST',
      headers: {
        Authorization: 'Bearer sk-cv-...',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(body),
    });

    if (res.status === 429) {
      const wait = parseInt(res.headers.get('Retry-After') || '5', 10);
      await new Promise((r) => setTimeout(r, wait * 1000));
      continue;
    }

    if (!res.ok) throw new Error(`API error: ${res.status}`);
    return res.json();
  }
  throw new Error('Max retries exceeded');
}
```

### Best Practices

1. Use streaming for long completions — keeps connections warm.
2. Fire parallel requests up to ~8 concurrent for typical plans.
3. Always read `Retry-After` instead of guessing a sleep duration.
4. Don't retry 4xx (auth, validation) errors. Only 429 and 5xx are retry-safe.
5. Track per-window usage from the dashboard.

## Error Codes

| Status | Description |
|--------|-------------|
| 401 | Invalid or missing API key |
| 403 | Account inactive or unauthorized |
| 429 | Rate limit exceeded (check response body for which window; includes `Retry-After` header) |
| 502 | All upstream providers failed (gateway retried automatically) |
| 503 | Service temporarily unavailable |

### Error Response Format

```json
{
  "error": {
    "message": "Rate limit exceeded: monthly window (1500/1500 requests used)",
    "type": "rate_limit_error",
    "code": 429
  }
}
```

### Error Handling Pattern (Python)

```python
import openai

try:
    response = client.chat.completions.create(
        model="glm-4.7",
        messages=[{"role": "user", "content": "Hello!"}]
    )
except openai.RateLimitError as e:
    print(f"Rate limited: {e}")
    # Wait and retry, or check your Usage page
except openai.AuthenticationError as e:
    print(f"Auth error: {e}")
    # Check your API key
except openai.APIError as e:
    print(f"API error: {e}")
    # Transient error — safe to retry
```

## Compatible Libraries

| Language | Package | Install |
|----------|---------|---------|
| Python | openai | `pip install openai` |
| JavaScript | openai | `npm install openai` |
| Go | sashabaranov/go-openai | `go get github.com/sashabaranov/go-openai` |
| PHP | openai-php/client | `composer require openai-php/client` |
| Rust | async-openai | `cargo add async-openai` |
| Ruby | ruby-openai | `gem install ruby-openai` |
| C# / .NET | OpenAI | `dotnet add package OpenAI` |
| Java | openai-java | See Maven Central |

## Integrations

For detailed per-client setup instructions (Claude Code, Cursor, Cline, Aider, Zed, Windsurf, OpenCode, Pi, OpenClaw, Roo Code, Kilo Code, Void, PearAI, Continue.dev) — including one-liner installers, config files, env vars, and role → model mappings — read `references/integrations.md`.
