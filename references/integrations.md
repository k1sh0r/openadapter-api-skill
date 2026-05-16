# OpenAdapter Integrations Reference

Detailed setup instructions for each supported client. All use the same API key (`sk-cv-...`) from [Dashboard → API Keys](https://dashboard.openadapter.in/keys).

## Quick Reference

| Client | Surface | Config Method |
|--------|---------|---------------|
| Claude Code | Anthropic Messages | Env vars or one-liner |
| Cursor | OpenAI-compatible | Settings → Override Base URL |
| Cline | OpenAI-compatible | VS Code panel → OpenAI Compatible |
| Aider | OpenAI-compatible | Env vars |
| Zed | OpenAI-compatible | `settings.json` |
| Windsurf | OpenAI-compatible | Settings → Base URL |
| OpenCode | OpenAI-compatible | One-liner (writes `opencode.json`) |
| Pi | OpenAI-compatible | One-liner (writes `~/.pi/agent/models.json`) |
| OpenClaw | OpenAI-compatible | One-liner (writes `~/.openclaw/openclaw.json`) |
| Roo Code | OpenAI-compatible | VS Code panel → OpenAI Compatible |
| Kilo Code | OpenAI-compatible | VS Code panel → OpenAI Compatible |
| Void | OpenAI-compatible | Settings → OpenAI-Compatible |
| PearAI | OpenAI-compatible | `config.yaml` (Continue.dev format) |
| Continue.dev | OpenAI-compatible | `~/.continue/config.yaml` |

---

## Claude Code

Anthropic's official coding CLI. Uses the Anthropic Messages API surface (`/v1/messages`). The gateway maps Claude Code's role names (sonnet/haiku/opus) to real upstream models.

### One-liner install (recommended)

Writes `~/.claude/settings.json` with your API key and role → model mapping.

**macOS / Linux:**
```bash
curl -sL "https://api.openadapter.in/api/setup/claude-code?key=YOUR_API_KEY" | bash
```

**Windows (PowerShell):**
```powershell
irm "https://api.openadapter.in/api/setup/claude-code-ps?key=YOUR_API_KEY" | iex
```

Override default models by appending `&sonnet=<alias>&haiku=<alias>&opus=<alias>` to the URL.

### Manual setup

```bash
export ANTHROPIC_BASE_URL=https://api.openadapter.in
export ANTHROPIC_AUTH_TOKEN=sk-cv-...
```

Then run `claude`.

### Role → model mapping

| Claude Code role | Default gateway alias |
|-----------------|----------------------|
| `sonnet` | `Kimi-K2.5` |
| `haiku` | `MiniMax-M2.7` |
| `opus` | `GLM-5` |

Change these in `~/.claude/settings.json` or via installer URL params. List all available aliases: `GET /v1/models` or Dashboard → Models.

### Notes

- The translating proxy preserves Anthropic-specific features (tool use, vision) when the upstream model supports them.
- Quota is enforced by request count, not tokens.
- If you previously used Anthropic directly, just swap the two env vars — no code changes.

---

## Cursor

AI-powered code editor. Routes through OpenAdapter via custom-key feature.

### Setup

1. Open **Settings** (`Cmd+,` on macOS, `Ctrl+,` elsewhere) → **Models** → **OpenAI API Key**.
2. Paste your OpenAdapter API key.
3. Click **Override OpenAI Base URL** and set:
   ```
   https://api.openadapter.in/v1
   ```
4. **Verify**.
5. Pick any model from the model picker.

### Notes

- All Cursor features that use the OpenAI API (chat, autocomplete, agent, Inline Edit) route through your gateway.
- Cursor's own internal models (cursor-small etc.) still ship from Cursor — only the OpenAI-routed paths come from your plan.
- Disable Cursor's free tier in **Settings → Models** to force everything through OpenAdapter.

---

## Cline

Autonomous VS Code coding agent. Configured in its VS Code panel — no config file editing.

### Setup

1. Install the **Cline** extension from the VS Code marketplace.
2. Click the Cline icon in the activity bar → gear icon (top-right of the Cline panel).
3. Set **API Provider** to `OpenAI Compatible`.
4. Fill in:
   ```
   Base URL:  https://api.openadapter.in/v1
   API Key:   sk-cv-...
   Model ID:  GLM-4.7   (or any alias your plan unlocks)
   ```
5. Save — Cline reloads automatically.

### Notes

- Cline runs in agent mode, so each task may consume many requests. Keep an eye on your plan's 5h/week/month windows in the Dashboard.
- Reasoning models (e.g. `o3`, `Kimi-K2.6`) generally produce better agentic plans but cost more requests.

---

## Aider

Terminal-based AI pair programmer. Two env vars and you're done.

### Setup

```bash
export OPENAI_API_KEY=sk-cv-...
export OPENAI_API_BASE=https://api.openadapter.in/v1

aider --model openai/GLM-4.7
```

Note the `openai/` prefix on the model — Aider uses it to indicate the OpenAI-compatible backend.

### Quick install

If you don't have Aider yet:

**macOS / Linux:**
```bash
curl -sL "https://api.openadapter.in/api/setup/aider?key=YOUR_API_KEY" | bash
```

Installs Aider (via `pip install aider-chat`) and appends the env vars to `~/.zshrc` or `~/.bashrc`.

### Notes

- Aider can multi-file edit and run tests in a loop. Each LLM call = one request against your quota.
- Use `--no-auto-commits` if you don't want Aider committing on every change.

---

## Zed

High-performance editor with built-in AI. Accepts custom OpenAI-compatible providers via `settings.json`.

### Setup

1. Open Zed settings (`Cmd+,` on macOS, `Ctrl+,` elsewhere) or edit `~/.config/zed/settings.json`.
2. Add (or merge):
   ```json
   {
     "language_models": {
       "openai_compatible": {
         "OpenAdapter": {
           "api_url": "https://api.openadapter.in/v1",
           "available_models": [
             {
               "name": "GLM-4.7",
               "display_name": "GLM-4.7",
               "max_tokens": 200000,
               "max_output_tokens": 16384
             },
             {
               "name": "glm-4.7",
               "display_name": "glm-4.7",
               "max_tokens": 128000,
               "max_output_tokens": 16384
             }
           ]
         }
       }
     }
   }
   ```
3. Set the API key as an env var (Zed maps provider name → uppercase env var):
   ```bash
   export OPENADAPTER_API_KEY="sk-cv-..."
   ```
   Windows PowerShell: `$env:OPENADAPTER_API_KEY="sk-cv-..."`
4. Restart Zed. Pick your model from the model picker in the Agent Panel.

### Notes

- Zed derives the env-var name from the provider name (`OpenAdapter` → `OPENADAPTER_API_KEY`). Spaces are stripped.
- `max_tokens` here is the model's **context window**, not output.

---

## Windsurf

AI-first code editor by Codeium. Route through OpenAdapter via custom OpenAI-compatible provider.

### Setup

1. Open Windsurf settings (`Cmd+,` / `Ctrl+,`) → **AI Provider** configuration.
2. Enter:
   ```
   Base URL:  https://api.openadapter.in/v1
   API Key:   sk-cv-...
   ```
3. Pick your preferred model and save.

Windsurf's Cascade and chat panes both route through your gateway.

---

## OpenCode

Open-source AI coding agent for the terminal. Run the one-liner from inside your project root — it installs OpenCode (if needed), writes `opencode.json` with your API key and the available models.

### One-liner install

**macOS / Linux:**
```bash
curl -sL "https://api.openadapter.in/api/setup/opencode?key=YOUR_API_KEY" | bash
```

**Windows (PowerShell):**
```powershell
irm "https://api.openadapter.in/api/setup/opencode-ps?key=YOUR_API_KEY" | iex
```

Run from your project root — `opencode.json` is written in the current directory.

---

## Pi

Extensible AI coding CLI with thinking levels and tool calling.

### One-liner install

**macOS / Linux:**
```bash
curl -sL "https://api.openadapter.in/api/setup/pi?key=YOUR_API_KEY" | bash
```

**Windows (PowerShell):**
```powershell
irm "https://api.openadapter.in/api/setup/pi-ps?key=YOUR_API_KEY" | iex
```

Configures `~/.pi/agent/models.json` with OpenAdapter as a custom provider — all gateway models with reasoning + vision become available.

### Switching models

Inside Pi, run `/model` to reload the model list. No restart required.

---

## OpenClaw

Open-source AI agent that bridges messaging apps to AI models, with built-in coding tools.

### One-liner install

**macOS / Linux:**
```bash
curl -sL "https://api.openadapter.in/api/setup/openclaw?key=YOUR_API_KEY" | bash
```

**Windows (PowerShell):**
```powershell
irm "https://api.openadapter.in/api/setup/openclaw-ps?key=YOUR_API_KEY" | iex
```

Configures `~/.openclaw/openclaw.json` with OpenAdapter as a provider. Default model: `glm-5`.

OpenClaw watches its config file, so edits apply automatically without restart.

---

## Roo Code

VS Code autonomous coding agent (Cline fork). Connect via OpenAI-Compatible provider.

### Setup

1. Install **Roo Code** from the VS Code marketplace.
2. Open the Roo Code panel → gear icon → API Configuration.
3. Set **API Provider** to `OpenAI Compatible`.
4. Fill in:
   ```
   Base URL:  https://api.openadapter.in/v1
   API Key:   sk-cv-...
   Model ID:  GLM-4.7
   ```

Same shape as Cline since Roo Code is a fork — see [Cline](#cline) for usage notes.

---

## Kilo Code

VS Code coding agent. Connect via OpenAI-Compatible provider.

### Setup

1. Install **Kilo Code** from the VS Code marketplace.
2. Open Kilo Code settings → API Provider → `OpenAI Compatible`.
3. Configure:
   ```
   Base URL:  https://api.openadapter.in/v1
   API Key:   sk-cv-...
   Model ID:  GLM-4.7
   ```

---

## Void

Open-source AI code editor (Cursor alternative). OpenAI-compatible endpoint support built in.

### Setup

1. Download Void from [voideditor.com](https://voideditor.com) and open it.
2. Click the gear icon (top-right) → **OpenAI-Compatible** section.
3. Enter:
   ```
   Base URL:  https://api.openadapter.in/v1
   API Key:   sk-cv-...
   ```
4. Click **Add Model**, type the model alias (e.g. `GLM-4.7`), enable it.

You can add multiple models from the same provider entry.

---

## PearAI

Open-source AI code editor (VS Code fork with Continue.dev built in). Uses the same config format as Continue.dev.

### Setup

1. Download PearAI from [trypear.ai](https://trypear.ai) and open it.
2. Open the command palette (`Cmd+Shift+P` / `Ctrl+Shift+P`) → **Open config.json**.
3. Add models matching the Continue.dev format:
   ```yaml
   name: OpenAdapter
   version: 1.0.0
   schema: v1

   models:
     - name: GLM-4.7
       provider: openai
       model: GLM-4.7
       apiBase: https://api.openadapter.in/v1
       apiKey: YOUR_API_KEY
       roles:
         - chat
         - edit
         - apply
   ```
4. Save — PearAI picks up changes automatically.

See [Continue.dev](#continuedev) for the full config reference.

---

## Continue.dev

VS Code / JetBrains extension. Configured via a single YAML file.

### Setup

1. Install the **Continue** extension from the VS Code marketplace (or JetBrains plugin marketplace).
2. Open `~/.continue/config.yaml` (Windows: `%USERPROFILE%\.continue\config.yaml`). Create it if it doesn't exist.
3. Paste the config below, replacing `YOUR_API_KEY`:
   ```yaml
   name: OpenAdapter
   version: 1.0.0
   schema: v1

   models:
     - name: GLM-4.7
       provider: openai
       model: GLM-4.7
       apiBase: https://api.openadapter.in/v1
       apiKey: YOUR_API_KEY
       roles:
         - chat
         - edit
         - apply

     - name: glm-4.7
       provider: openai
       model: glm-4.7
       apiBase: https://api.openadapter.in/v1
       apiKey: YOUR_API_KEY
       roles:
         - chat
         - edit
         - apply

     - name: Qwen3 Embedding Small
       provider: openai
       model: qwen3-embedding-small
       apiBase: https://api.openadapter.in/v1
       apiKey: YOUR_API_KEY
       roles:
         - embed
   ```
4. Save — Continue picks up changes automatically.

### Notes

- The `embed` role enables codebase indexing for in-repo @-mentions.
- Add more chat-model entries as needed; the alias must match what your gateway plan exposes.
- See Dashboard → Models for the full list.
