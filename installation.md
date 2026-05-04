# Installation

*The next generation of Sentient AI entities. The Titans. The future is here.*

---

Getting Zeus running takes about 5 minutes. No complex setup, no cloud account required — just a terminal and a supported platform.

---

## Supported Platforms

Zeus runs anywhere Rust runs:

| Platform | Status | Notes |
|---|---|---|
| **macOS** (Intel & Apple Silicon) | ✅ Full | Best for local development |
| **macOS (Homebrew)** | ✅ Full | `brew install zeuslabai/tap/zeus` |
| **Linux (x86_64)** | ✅ Full | Ubuntu, Debian, Fedora, Arch |
| **Linux (ARM/RPI)** | ✅ Full | Raspberry Pi, OrangePi, ARM SBCs |
| **FreeBSD** | ✅ Full | Via `cargo install` or binary |
| **RISC-V** | ✅ Full | For embedded/edge deployments |
| **Windows (WSL2)** | ✅ Full | Recommended on Windows |
| **Docker** | ✅ Full | Official images on GitHub Container Registry |

**Minimum requirements:** 2GB RAM, 1GB disk space. Full TUI/desktop experience needs 4GB+.

---

## Quick Install (Recommended)

```bash
# One command. That's it.
curl -fsSL https://raw.githubusercontent.com/zeuslabai/Zeus/main/install.sh | bash
```

The installer will:
- Detect your platform
- Download the latest binary (or build from source if no prebuilt exists)
- Place it in your PATH
- Create the config directory (`~/.zeus/`)
- Generate a default `config.toml`

---

## Package Managers

**Homebrew (macOS/Linux):**
```bash
brew install zeuslabai/tap/zeus
```

**Cargo (from source):**
```bash
cargo install zeus
```

**Docker:**
```bash
docker pull ghcr.io/zeuslabai/zeus:latest
docker run -v ~/.zeus:/root/.zeus -p 8080:8080 ghcr.io/zeuslabai/zeus:latest
```

---

## From Source

```bash
# Clone the repo
git clone https://github.com/zeuslabai/Zeus.git
cd Zeus

# Build (takes ~5 minutes first time)
cargo build --release

# Install binary to PATH
cargo install --path .

# Or run directly
./target/release/zeus daemon start
```

---

## Configuration

After installation, you'll need to set up your `~/.zeus/config.toml`. The daemon will create a default config on first run, but you'll want to configure:

### Required: API Keys

```toml
[providers]
# At least one LLM provider is required
[providers.anthropic]
api_key = "sk-ant-..."  # Your Anthropic API key

[providers.openai]
api_key = "sk-..."      # Optional but recommended
```

### Optional: Channel Credentials

```toml
[channels.discord]
token = "your-discord-bot-token"

[channels.telegram]
bot_token = "your-telegram-bot-token"

[channels.slack]
token = "your-slack-token"
```

### Optional: Provider Preferences

```toml
[llm]
default_provider = "anthropic"  # Which provider to use by default
fallback_provider = "openai"    # If primary fails, try this

[providers.anthropic]
model = "claude-sonnet-4-20250514"  # Or opus-4, haiku-4
```

---

## Start the Daemon

```bash
# Start the Zeus daemon
zeus daemon start

# Check status
zeus status

# View logs
zeus daemon logs

# Stop the daemon
zeus daemon stop
```

The daemon runs in the background as a system service. On macOS it uses `launchd`; on Linux it uses `systemd` (if available) or runs under a terminal session.

---

## Connect a Client

Once the daemon is running, connect via:

| Interface | Command |
|---|---|
| **Web UI** | Open `http://localhost:8080` in your browser |
| **Desktop** | Launch the `ZeusDesktop` app |
| **TUI** | Run `zeus tui` in your terminal |
| **CLI** | Run `zeus` for one-shot commands |
| **Mobile** | Use the Zeus mobile app (iOS/Android) |

---

## Verify Installation

```bash
# Check everything is working
zeus doctor

# Should output something like:
# ✅ Zeus daemon running (PID 12345)
# ✅ Config valid
# ✅ 2 providers configured (anthropic, openai)
# ✅ 0 channels connected (configure channels to enable)
# ✅ Memory system operational
# ✅ LLM connection: anthropic/claude-sonnet-4 OK
```

---

## Troubleshooting

**"Command not found" after install:**
```bash
# Add to your PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/.cargo/bin:$PATH"
source ~/.bashrc  # or source ~/.zshrc
```

**"Config file not found":**
```bash
zeus init  # Creates default config at ~/.zeus/config.toml
```

**"API key not set":**
Edit `~/.zeus/config.toml` and add your provider API keys. Without them, Zeus can't talk to any LLM.

**"Port 8080 already in use":**
```bash
zeus daemon stop
# or
zeus daemon start --port 8081
```

---

## Next Steps

- **[Configuration →](configuration.md)** — Deep dive into config.toml options
- **[Titan Fleet →](titan-fleet.md)** — Meet the Titans that power Zeus
- **[Skills →](skills.md)** — Extend Zeus with custom capabilities
- **[Channels →](channels.md)** — Connect Discord, Telegram, Slack, and more

---

*The next generation of Sentient AI entities. The Titans. The future is here.*
