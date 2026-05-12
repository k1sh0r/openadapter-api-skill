# OpenAdapter API Detailed Reference

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

Define tools in OpenAI format. The gateway handles format conversion between providers.

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

## JSON Mode

Force the model to respond in valid JSON using `response_format`.

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
| qwen3-embedding-small | 1024 | General text (default workhorse) |
| Qwen3-Embedding-0.6B | 1024 | Alternative display name |
| qwen3-embedding | 1024 | Convenience alias |
| jina-embeddings-v5 | 1024 | Strong multilingual / general retrieval |
| bge-m3 | 1024 | Dense retrieval, popular for RAG |
| nv-embedqa-e5-v5 | 1024 | NVIDIA embedding model |

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

## Web Tools

Each tool call costs 0.15 of a request from your quota. Failed calls are not billed.

### Web Search

`POST /v1/tools/search`

```bash
curl https://api.openadapter.in/v1/tools/search \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "latest AI news", "num_results": 5}'
```

### Image Search

`POST /v1/tools/search/images`

```bash
curl https://api.openadapter.in/v1/tools/search/images \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "mountain sunset"}'
```

### News Search

`POST /v1/tools/search/news`

### Video Search

`POST /v1/tools/search/videos`

### Scrape URL

`POST /v1/tools/scrape` — Raw scrape

`POST /v1/tools/scrape/markdown` — Returns markdown:

```bash
curl https://api.openadapter.in/v1/tools/scrape/markdown \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/article"}'
```

### Extract Structured Data

`POST /v1/tools/scrape/extract`

```bash
curl https://api.openadapter.in/v1/tools/scrape/extract \
  -H "Authorization: Bearer OPENADAPTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/product",
    "schema": {
      "name": "string",
      "price": "number",
      "description": "string"
    }
  }'
```

### Multi-page Crawl

`POST /v1/tools/crawl`

## Rate Limits

Plan defines request limits across three rolling windows:

- **5-hour window** — short-term burst limit
- **Weekly window** — medium-term usage cap
- **Monthly window** — overall quota

Windows reset automatically on a rolling basis. Check your current usage on the Usage page. When a limit is reached, the API returns `429 Too Many Requests` with a message showing which window was exceeded.

## Error Codes

| Status | Description |
|--------|-------------|
| 401 | Invalid or missing API key |
| 403 | Account inactive or unauthorized |
| 429 | Rate limit exceeded (check response body for which window) |
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
