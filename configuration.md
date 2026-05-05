# Configuration

*The next generation of Sentient AI entities. The Titans. The future is here.*

---

All Zeus configuration lives in `~/.zeus/config.toml`. This page documents every section and option.

![Provider Freedom Matrix](../provider-freedom-matrix.svg)

---

## Full Config Example

```toml
# ~/.zeus/config.toml

[server]
host = "0.0.0.0"
port = 8080
tls_enabled = false
tls_cert = ""      # Path to TLS certificate (optional)
tls_key = ""       # Path to TLS private key (optional)

[server.auth]
require_auth = true
session_secret = "change-me-to-a-random-string"
session_ttl_hours = 720  # 30 days

[providers]
[providers.anthropic]
api_key = "sk-ant-..."
model = "claude-sonnet-4-6"
max_tokens = 8192

[providers.openai]
api_key = "sk-..."
model = "gpt-5.5"

[providers.ollama]
endpoint = "http://localhost:11434"
model = "llama-4-70b-instruct"

[llm]
default_provider = "anthropic"
fallback_provider = "openai"
temperature = 0.7
max_tokens = 4096

[channels.discord]
enabled = true
token = "your-discord-bot-token"
guild_id = ""      # Optional: restrict to specific guild
channel_ids = []   # Optional: restrict to specific channels

[channels.telegram]
enabled = false
bot_token = ""

[channels.slack]
enabled = false
token = ""
team_id = ""

[memory]
vector_store_path = "~/.zeus/vector.db"
max_session_memory_tokens = 128000
semantic_search_limit = 20

[security.aegis]
enabled = true
rate_limit_per_minute = 60
block_unknown_senders = false
log_all_events = true

[security.tools]
allowed_tools = []    # Empty = all allowed
blocked_tools = []     # Tool names to block
max_tool_calls_per_turn = 50

[security.memory]
sanitize_memory_writes = true
max_memory_write_size_kb = 1024

[paths]
data_dir = "~/.zeus/data"
logs_dir = "~/.zeus/logs"
backups_dir = "~/.zeus/backups"
skills_dir = "~/.zeus/skills"

[logging]
level = "info"       # trace, debug, info, warn, error
format = "json"      # json or pretty
destination = "file" # file, stdout, or both

[economy]
enabled = false
wallet_path = "~/.zeus/wallet.db"
auto_topup = false
spending_limit_usd = 10.00
```

---

## [server]

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `host` | string | `"0.0.0.0"` | Bind address |
| `port` | integer | `8080` | HTTP listen port |
| `tls_enabled` | bool | `false` | Enable HTTPS |
| `tls_cert` | string | `""` | Path to TLS certificate |
| `tls_key` | string | `""` | Path to TLS private key |

---

## [server.auth]

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `require_auth` | bool | `true` | Require login to access the UI |
| `session_secret` | string | required | Secret for signing session cookies |
| `session_ttl_hours` | integer | `720` | Session duration (30 days default) |

---

## [providers]

Define your LLM providers. Zeus supports **19 providers**. At least one is required.

### Provider Options

| Option | Required | Description |
|--------|----------|-------------|
| `api_key` | Yes (most) | API key for the provider |
| `endpoint` | No | Custom endpoint (for self-hosted) |
| `model` | No | Model variant to use (defaults to provider's default) |
| `max_tokens` | No | Max tokens per response |
| `base_url` | No | Base URL for API-compatible providers |

### Supported Providers

| Provider | API Key Required | Self-Hosted | Notes |
|---|---|---|---|
| `anthropic` | Yes | No | Claude 4.5/4.6/4.7 Sonnet/Opus/Haiku |
| `openai` | Yes | No | GPT-5.5 |
| `google` | Yes | No | Gemini 3 Pro/3 Flash |
| `ollama` | No | Yes | Local models (no API key) |
| `openrouter` | Yes | No | Unified access to many models |
| `groq` | Yes | No | Fast inference |
| `mistral` | Yes | No | Mistral Large 3 (`mistral-large-3`) |
| `together` | Yes | No | Together AI (`llama-4-70b`, `mistral-7b`) |
| `fireworks` | Yes | No | Fireworks AI (`mixtral-8x22b`, `llama-4`) |
| `azure` | Yes | No | Azure OpenAI Service |
| `bedrock` | Yes | No | AWS Bedrock (`region` + model in config) |
| `deepseek` | Yes | No | DeepSeek V4 (`deepseek-v4`) |
| `xai` | Yes | No | xAI Grok 4 (`grok-4`) |
| `cerebras` | Yes | No | Cerebras (`llama-4-70b`) |
| `moonshot` | Yes | No | Moonshot Kimi K2.6 (`kimi-k2.6`) |
| `zhipu` | Yes | No | Zhipu GLM 5.1 (`glm-5.1`) |
| `qwen` | Yes | No | Qwen 3.5 (`qwen3.5-72b`) |
| `minimax` | Yes | No | MiniMax M2.7 (`minimax-m2.7`) |

---

## [llm]

Global LLM behavior settings.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `default_provider` | string | required | Primary LLM provider name |
| `fallback_provider` | string | `""` | Failover provider if primary fails |
| `temperature` | float | `0.7` | Sampling temperature (0.0–2.0) |
| `max_tokens` | integer | `4096` | Default max tokens |
| ` Reasoning effort` | string | `"auto"` | `low`, `medium`, `high`, or `auto` |

---

## [channels]

Each channel section (`[channels.discord]`, `[channels.telegram]`, etc.) accepts:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enabled` | bool | `false` | Enable this channel |
| `token` | string | required | Channel-specific token/bot key |

### Channel Requirements

- **Discord:** Bot token from the Discord Developer Portal
- **Telegram:** Bot token from @BotFather
- **Slack:** App token (`xoxb-...`) from api.slack.com/apps
- **Email:** IMAP/SMTP credentials in `[channels.email]`
- **iMessage:** Requires macOS with iTerm + AppleScript enabled
- **WhatsApp:** WhatsApp Business API credentials
- **Signal:** Signal Messenger REST API token
- **Matrix:** Homeserver URL + access token

---

## [memory]

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `vector_store_path` | string | `~/.zeus/vector.db` | SQLite file for vector embeddings |
| `max_session_memory_tokens` | integer | `128000` | Per-session context window |
| `semantic_search_limit` | integer | `20` | Max results from vector search |

---

## [security.aegis]

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enabled` | bool | `true` | Enable Aegis security layer |
| `rate_limit_per_minute` | integer | `60` | Max messages per minute per user |
| `block_unknown_senders` | bool | `false` | Reject messages from unknown channels |
| `log_all_events` | bool | `true` | Write all security events to log |

---

## [paths]

Customize where Zeus stores its data.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `data_dir` | string | `~/.zeus/data` | General data storage |
| `logs_dir` | string | `~/.zeus/logs` | Log files |
| `backups_dir` | string | `~/.zeus/backups` | Config backups |
| `skills_dir` | string | `~/.zeus/skills` | Installed Skills |

---

## [logging]

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `level` | string | `"info"` | Log verbosity |
| `format` | string | `"json"` | `json` or `pretty` |
| `destination` | string | `"file"` | `file`, `stdout`, or `both` |

---

## Environment Variables

Any config value can be overridden with an environment variable:

```bash
export ZEUS_PROVIDERS_ANTHROPIC_API_KEY="sk-ant-..."
export ZEUS_CHANNELS_DISCORD_TOKEN="your-discord-token"
export ZEUS_SERVER_PORT="9090"
```

Format: `ZEUS_` + section uppercase + `_` + key uppercase.

---

## Config Guard

Zeus validates your config on every startup via **Config Guard** (Aegis subsystem):

- ✅ Schema validation — required fields present
- ✅ API key presence check — configured keys aren't empty strings
- ✅ Channel credential expiry — detects expired tokens
- ✅ Corruption detection — backs up config before starting

If Config Guard finds issues, the daemon refuses to start and reports exactly what's wrong. No degraded-mode deployments.

---

**Previous:** [Installation →](installation.md) · **Next:** [Skills →](skills.md)
