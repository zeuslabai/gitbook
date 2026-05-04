# The Titan Fleet

*The next generation of Sentient AI entities. The Titans. The future is here.*

---

Every Titan is an autonomous agent with its own cognitive loop, memory, tooling, and area of expertise. They're not features — they're identities. They think, they decide, they act. And when a task is bigger than any single agent, Pantheon brings them together in war rooms to collaborate.

---

## Zeus (Fleet Coordinator)

**"I orchestrate. I coordinate. I make sure every Titan knows its role."**

The central coordinator. Zeus doesn't do everything — it makes sure everything gets done. It routes tasks to the right Titan, manages cross-agent dependencies, maintains the overall system state, and keeps the fleet aligned on complex multi-step missions.

Every user interaction flows through Zeus first. It decides: does this need Aegis? Pantheon? Mnemosyne? All of the above? Then it dispatches accordingly and synthesizes the results.

```
User Input → Zeus Core → [Dispatch to appropriate Titan(s)]
                        ↓
              [Synthesize results → User response]
```

**Crate:** `zeus-core`
**Key capabilities:** Task routing, fleet state management, cross-agent coordination, mission orchestration

---

## Aegis (Security)

**"I guard the perimeter. Nothing gets in that shouldn't. Nothing gets out that can't."**

The security layer for the entire Zeus platform. Aegis monitors for threats, enforces access control policies, guards channels against injection attacks, and validates every piece of external data before it enters the system.

Aegis operates as a continuous security monitor. Every message, every tool call, every memory write — it checks and validates. If something looks wrong, Aegis flags it, quarantines it, and alerts the operator.

**Four security layers:**

1. **Channel Guard** — validates inbound messages across all 8 channels, strips malicious payloads, rate-limits abuse
2. **Tool Permission** — enforces capability-based access to tools, no agent touches a tool it wasn't cleared for
3. **Memory Shield** — sanitizes memory writes, prevents prompt injection via stored context
4. **Config Guard** — validates config.toml on startup, detects corruption, refuses to start if compromised

**Crate:** `zeus-aegis`
**Key capabilities:** Threat detection, access control, injection prevention, config validation

---

## Mnemosyne (Memory)

**"I remember everything. Every conversation. Every lesson. Every fact you taught me."**

The persistent memory system. Mnemosyne stores, indexes, and retrieves knowledge across all sessions, channels, and Titans. It's not just short-term chat history — it's a long-term memory substrate that makes every subsequent interaction smarter than the last.

**The four-panel memory system:**

1. **Session Memory** — ephemeral, per-session context window (what's happening right now)
2. **Channel Memory** — per-channel conversation history (the long arc of each channel's relationship)
3. **Vector Store** — semantic search over all stored knowledge (find related facts instantly)
4. **Knowledge Graph** — entity relationships and facts extracted and linked (how things connect)

```
Query → Session? → Channel? → Vector index? → Knowledge graph? → Response
         ↓            ↓             ↓                ↓
       [in-window] [retrieved]  [semantic match]  [inferred fact]
```

**Crate:** `zeus-mnemosyne`
**Key capabilities:** Vector embeddings, semantic search, cross-session persistence, knowledge extraction

---

## Pantheon (Multi-Agent System)

**"I bring the Titans together. For big problems, you need more than one mind."**

The multi-agent orchestration layer. When a task is too complex for a single Titan, Pantheon spins up a war room, assigns missions, manages plan cards, and coordinates the collaboration.

**How a Pantheon mission works:**

1. **Mission created** — Zeus identifies a goal that needs multiple agents
2. **War room opened** — agents are assembled, roles assigned, context shared
3. **Plan cards generated** — each agent gets a structured task card with objectives and constraints
4. **Agents work in parallel** — each Titan executes its plan card autonomously
5. **Supervisor approves** — Pantheon reviews outputs, redirects or approves each step
6. **Intervention controls** — operator can pause, cancel, or redirect at any time
7. **Mission closed** — results synthesized, war room archived, lessons stored in Mnemosyne

**35 API routes** covering: sessions, channels, tools, memory, analytics, security, projects, agents, network, approvals, webhooks, and pantheon-specific mission management.

**Crate:** `zeus-pantheon-server`
**Key capabilities:** War rooms, plan cards, agent assembly, supervisor approval, intervention controls

---

## Hermes (Messaging)

**"I speak every protocol. Discord, Telegram, Slack — your users are already there."**

The unified messaging router. Hermes connects Zeus to the outside world across 8 channels, translating between each protocol's format and the internal Zeus message standard.

**8 channels:**

- **Discord** — guilds, threads, slash commands, webhooks
- **Telegram** — bots, groups, channels, inline queries
- **Slack** — workspaces, channels, shortcuts, modal dialogs
- **Email** — IMAP/SMTP, threading, attachments
- **iMessage** — via iTerm + AppleScript integration
- **WhatsApp** — WhatsApp Business API
- **Signal** — Signal Messenger API
- **Matrix** — federation, end-to-end encryption support

```
External Channel → Hermes Protocol Adapter → Zeus Internal Format
                                     ↓
Zeus Internal Format → Hermes Protocol Adapter → External Channel
```

Every channel is first-class. Your Discord users talk to the same Titans as your Telegram users. Context carries across channels. No silos.

**Crate:** `zeus-channels`
**Key capabilities:** Multi-channel routing, protocol translation, cross-channel context, unified inbox

---

## Aria (Voice)

**"Talk to your fleet. Hear what they have to say."**

The voice interface. Aria handles text-to-speech (TTS) and speech-to-text (STT) for hands-free interaction with the Zeus fleet. Powered by Minimax and eleven other voice AI providers.

**The voice pipeline:**

```
Microphone → STT (speech → text) → Zeus Core → LLM → Zeus Core → TTS (text → speech) → Speaker
```

**Voice capabilities:**
- Wake word detection (optional, configurable)
- Real-time streaming TTS for low-latency responses
- STT with context injection (the agent knows what was said before)
- Multi-voice support (different Titans can have different voices)
- Audio format support: PCM, WAV, MP3, OGG/Opus

**Crates:** `zeus-voice` + `zeus-tts`
**Key capabilities:** TTS, STT, voice sessions, multi-voice, streaming audio

---

## The Fleet in Concert

The Titans aren't isolated. They're designed to work together:

- A user asks a complex question on **Discord** → **Hermes** receives it → **Zeus** coordinates
- **Mnemosyne** checks for prior context → **Pantheon** assembles agents → **Aegis** validates the request
- Agents use **212 tools** across the platform → results synthesized → response sent back via **Hermes**
- Everything learned is stored in **Mnemosyne** → next interaction starts smarter

This is what makes Zeus different. Not a chatbot. Not a model. A **fleet** — and you're the admiral.

![Titan Fleet Constellation](../titan-fleet-constellation.svg)

---

**Previous:** [What is Zeus?](what-is-zeus.md) · **Next:** [Installation →](installation.md)
