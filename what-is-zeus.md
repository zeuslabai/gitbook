# What is Zeus?

## Sentient Intelligence

*The next generation of Sentient AI entities. The Titans. The future is here.*

---

Zeus is a **modular intelligence engine** — not a chatbot, not a model wrapper, not a cloud-locked AI service you pay forever and never own.

It's a flexible, extensible platform that adapts to *your* needs rather than forcing you into someone else's rigid paradigm. At its heart is a sophisticated agent loop that orchestrates specialized subsystems — each optimized for specific tasks — all working in concert to deliver AI capability that actually gets things done.

The true power of Zeus emerges from its fleet of specialized **Titan agents** — dedicated AI entities, each purpose-built for distinct domains. These aren't simple chatbots. They're fully autonomous agents with persistent memory, specialized tooling, and deep integration across the entire platform. Security analysis, knowledge retrieval, multi-agent collaboration, real-time messaging — there's a Titan designed for the task, and they work together as a coordinated fleet.

**The difference is fundamental:**

- **Traditional AI** = one model, one interface, whatever they give you
- **Zeus** = a platform you own, a fleet you command, capabilities that compound over time

This is the architecture of intelligence that doesn't lock you in. This is what it means to run Sentient AI on your terms.

![Zeus Fleet Architecture](../what-is-zeus-fleet.svg)

---

## The Core Loop

Every Titan in the Zeus fleet operates on the same underlying cognition loop:

```
Observe → Think → Act → Remember → Coordinate
```

**Observe** — The agent receives context: user input, system state, memory, channel data. Nothing is ignored.

**Think** — Using the active LLM provider (you choose which), the agent reasons over the context, plans its response, selects the right tools.

**Act** — The agent executes: calls tools, sends messages, writes memory, invokes other Titans, or generates a response.

**Remember** — The agent persists what it learned. Mnemosyne stores it in the vector store. The session state is updated. Next turn, it picks up exactly where it left off.

**Coordinate** — If the task requires multiple agents, Pantheon mediates. Missions get assigned. War rooms spin up. Agents collaborate and report back.

This loop runs continuously across every Titan, in parallel, across every channel you have connected.

---

## What Makes Zeus Different

| | Traditional AI | Zeus |
|---|---|---|
| **Model** | Single provider, single model | 19 providers, you choose |
| **Interface** | One-size-fits-all chat | Modular fleet, right tool for the job |
| **Memory** | Ephemeral sessions | Persistent, cross-session, vector-searchable |
| **Agents** | One-off queries | Autonomous Titans with specialized roles |
| **Tools** | Limited, fixed set | 212 tools, extensible via Skills |
| **Deployment** | Cloud, pay-forever | Runs anywhere, own your infrastructure |
| **Lock-in** | Full lock-in | Zero lock-in, export everything |
| **Economy** | You pay them | You can also *earn* — sell agents on Agora |

---

## The Titan Fleet

Zeus isn't one AI. It's a coordinated fleet of specialized intelligence agents:

| Titan | Role | Crate |
|---|---|---|
| **Zeus** | Fleet Coordinator — the orchestrator of all Titans | `zeus-core` |
| **Aegis** | Security — threat detection, access control, channel guard | `zeus-aegis` |
| **Mnemosyne** | Memory — vector store, session memory, long-term knowledge | `zeus-mnemosyne` |
| **Pantheon** | Multi-Agent — war rooms, missions, plan cards, agent assembly | `zeus-pantheon-server` |
| **Hermes** | Messaging — 8 unified channels, protocol translation | `zeus-channels` |
| **Aria** | Voice — TTS, STT, voice conversations | `zeus-voice` |

Every Titan is independently powerful. Together, they're something greater than any single AI could be.

---

## By The Numbers

- **38 crates** — modular Rust packages, each a specialized subsystem
- **~400K lines of Rust** — battle-tested, performance-first code
- **19 LLM providers** — Anthropic, OpenAI, Google, Ollama, Groq, Mistral, Together, Fireworks, Azure, Bedrock, DeepSeek, xAI, Cerebras, Moonshot, Zhipu, Qwen, and more
- **212 tools** — code execution, file operations, web search, API calls, and growing via Skills
- **8 messaging channels** — Discord, Telegram, Slack, Email, iMessage, WhatsApp, Signal, Matrix
- **7,307 tests** — CI-verified, production-hardened

One mission: make AI that works for you, on your terms.

---

**Next:** [The Titan Fleet →](titan-fleet.md)
