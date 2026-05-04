# Aegis Security — Zero-Trust Defense in Depth

Security is not an afterthought in Zeus — it is woven into the architecture at every layer. Aegis is the name given to Zeus's comprehensive security subsystem, a multi-layered defense strategy built on the principle that trust must be earned, not assumed. In this chapter, we examine the zero-trust philosophy that underpins everything, and walk through each of the four independent security layers that protect your Zeus deployment.

## 1. The Zero-Trust Model

The security industry has spent decades learning that perimeter-based security — firewalls, VPNs, "trust this network" — fails catastrophically when the perimeter dissolves. Modern deployments span multiple clouds, include remote workers, and handle sensitive data that must be protected everywhere it goes. Zeus was designed from day one with this reality in mind.

**Never trust, always verify** is not a slogan at Zeus — it is enforced policy. Every incoming request, whether from a user in a chat channel, an API call from your monitoring system, or an inter-Titan message on the internal bus, is treated as potentially hostile until it has been fully authenticated, authorized, and validated.

This means:

- A request from a "trusted" internal service is subjected to the same rigorous checks as one from a public channel
- Authentication tokens expire and must be refreshed — no indefinite session grants
- Permissions are granted sparingly and revoked immediately when no longer needed
- Every action is logged, and anomalous behavior triggers automatic response

**Least-privilege access by default** is the corollary. When a new Titan is configured, it starts with zero permissions. Every tool, every API route, every memory namespace must be explicitly granted. This limits the blast radius of any compromise — a hacked Titan can only do what it was already allowed to do, which is hopefully very little.

**Defense in depth** means that no single security mechanism is trusted alone. Aegis implements four independent security layers, each capable of stopping threats that slip past the others. A vulnerability in the authentication layer does not automatically mean a breach of the execution environment. A compromised tool does not automatically mean access to sensitive memory. Each layer gates the next, creating multiple opportunities to detect and stop an attack.

## 2. Layer 1 — Identity & Authentication

The outermost security layer establishes and verifies identity for every actor in the system — users, Titans, API clients, and external services.

### Bearer Token Authentication

Every API route in Zeus requires authentication. The mechanism is **Bearer token authentication**: a cryptographically signed token, issued by Zeus Core, must be included in every request as `Authorization: Bearer <token>`. These tokens are:

- **Short-lived**: Default lifetime of 1 hour, configurable down to minutes for high-security environments
- **Signed with Ed25519**: Tokens are cryptographically signed, not merely encrypted. A forged token is mathematically impossible without the private key.
- **Scoped**: Each token carries explicit permissions — read-only Titan status, read-write memory access, admin panel access — matching the principle of least privilege
- **Revocable**: Any token can be invalidated immediately via the API or config, invalidating it before its natural expiration
- **Audited**: Every use of a token is logged with the requesting IP, timestamp, and action

Tokens are issued through a dedicated auth endpoint that accepts credentials (username/password, API key, OAuth callback) and returns a signed token pair — an access token and a refresh token. Refresh tokens have longer lifetimes and are stored hashed in the local database, equivalent to how password databases store password hashes rather than plain text.

### Per-Channel Credential Management

Each communication channel that Zeus connects to — Discord, Telegram, Slack, Email, iMessage, WhatsApp, Signal, Matrix — requires its own credentials (bot tokens, API keys, OAuth secrets). Aegis treats these as first-class security principals, not mere configuration values.

Channel credentials are stored in **encrypted vaults** — AES-256-GCM encrypted blobs that are only decrypted in memory at the moment they are needed, and only for the specific channel that owns them. The encryption key is derived from a master secret that you provide at installation and never store in plaintext.

Critically, channel credentials are **isolated from each other**. A compromise of the Telegram bot token does not give an attacker access to the Discord bot token. Each credential namespace is independently encrypted and independently auditable.

When a channel credential rotates (a bot token is refreshed, an OAuth secret is renewed), Aegis re-encrypts the new credential and invalidates the old one atomically, ensuring no window of vulnerability during credential updates.

### Ed25519 Key Pairs for Wallet and Economy Operations

For operations involving wallet integration, economy systems, or any financial-adjacent functionality, Zeus uses **Ed25519 key pairs** rather than simple Bearer tokens. Ed25519 is a modern elliptic curve signature scheme offering:

- **Fast verification**: Signatures are 64 bytes, verifying in microseconds
- **Small keys**: Public keys are 32 bytes — practical for embedding in configurations
- **Side-channel resistance**: Designed to resist timing attacks and power analysis
- **Provable security**: Based on well-understood discrete logarithm assumptions

Each wallet operation (crediting a user, debiting a balance, transferring funds) requires a valid Ed25519 signature from the appropriate key pair. This means that even if an attacker obtains a Bearer token, they cannot perform wallet operations without also possessing the corresponding Ed25519 private key — which lives in an HSM, a secrets manager, or a hardware token, not on the Zeus server's filesystem.

## 3. Layer 2 — Channel Security

The second security layer governs how Zeus interacts with the outside world through its eight supported communication channels. Each channel has its own attack surface, threat model, and security controls.

### All Eight Channels Under Aegis Guard

Zeus connects to: **Discord**, **Telegram**, **Slack**, **Email** (SMTP/IMAP), **iMessage** (via私有 APNs integration), **WhatsApp**, **Signal**, and **Matrix**. Each channel is a first-class citizen under Aegis, meaning every message that flows through any channel is:

- **Sanitized**: HTML injection, markdown abuse, Unicode spoofing, and other common attack vectors are stripped at the channel boundary before the message enters Zeus Core
- **Validated**: Message schemas are enforced. Unexpected field types, oversized payloads, and malformed structures are rejected at the gate
- **Rate-limited**: Per-sender rate limits prevent spam, flooding, and abuse. A Discord user cannot overwhelm the system by sending 1,000 messages per second
- **Logged**: Every inbound and outbound message is recorded in the audit log with full metadata

### Channel-Specific Input Sanitization

Different channels have different attack vectors. Aegis applies channel-specific sanitization rules:

- **Discord**: Message content is stripped of custom emoji (to prevent embedding malicious payloads), webhook URLs are blocked, and mention spam is rate-limited
- **Telegram**: Inline keyboards and callback queries are validated against active session state to prevent CSRF-like attacks; file uploads are scanned for MIME type mismatches
- **Slack**: OAuth tokens are scoped to the minimum permissions needed; workspace tokens are never exposed to Titans
- **Email**: SPF, DKIM, and DMARC validation are performed on inbound emails; outbound emails are signed and DKIM-aligned; HTML emails are rendered in a sandboxed viewer
- **Signal / WhatsApp**: Phone numbers are validated against E.164 format; message metadata beyond sender/recipient is not stored
- **iMessage / Matrix**: End-to-end encryption status is verified before processing; fallback to unencrypted delivery requires explicit opt-in

These sanitization rules are maintained as a living ruleset, updated as new attack vectors are discovered. You receive updates through the normal Zeus update mechanism.

### No Elevated Permissions Required

A crucial design decision: **Zeus never requests elevated permissions from channels**. When you add Discord to your Zeus deployment, you do not need to grant the bot admin permissions, manage server roles, or give it access to private channels. Zeus requests only the minimum permissions it needs to function:

- Read message history (for context)
- Send messages (to respond)
- Manage threads (for organized conversations)

This follows the least-privilege principle at the platform integration level. If Zeus is compromised, it cannot abuse Discord admin powers because it never had them.

## 4. Layer 3 — Execution Sandbox

The third security layer governs what happens when Zeus needs to actually do something — run a tool, execute a skill, process untrusted code. This is where the most dangerous attack surface lies, and Aegis responds with hardware-enforced isolation.

### zeus-sandbox: WASM Runtime

**zeus-sandbox** is Zeus's WebAssembly-based execution environment for all external and untrusted code. Every tool invocation, every skill script, every plugin that runs on behalf of a user executes inside a zeus-sandbox instance.

WASM was chosen as the sandboxing technology for several compelling reasons:

- **Hardware-level isolation**: WASM runs in a memory-constrained, address-space-isolated environment that browsers have battle-tested at internet scale
- **Formal verification**: The WASM specification has clear memory and execution models that are amenable to formal verification
- **Language agnosticism**: Any language that compiles to WASM (Rust, C, C++, Go, TypeScript via AssemblyScript) can be used for tools and plugins
- **Performance**: Near-native execution speed with no interpreter overhead
- **Determinism**: WASM execution is deterministic by default, making audit logs reproducible

When a tool is invoked, zeus-sandbox loads the compiled WASM module, allocates a bounded memory region, and sets up a capability table — the list of system resources the module is allowed to access.

### Capability-Based Security

zeus-sandbox implements **capability-based security**: by default, a sandboxed module has zero access to any system resource. To do anything useful, it must be explicitly granted capabilities:

```
capability fs.read   = "/home/zeus/skills/my-skill/data/"
capability net.http  = "api.example.com:443"
capability time.now  = true
capability crypto.md5 = true
```

These capabilities are declared in the tool's metadata (in its SKILL.md or plugin manifest) and approved by the Titan's operator before the tool is registered. At runtime, every attempt to access a resource is checked against the capability table — a tool that tries to read a file outside its granted directory is terminated immediately.

This model means that even if an attacker successfully injects malicious code into a tool, the damage is limited to what the tool was already allowed to do. A compromised skill cannot exfiltrate your memory data if it was never granted filesystem or network access.

### Tool Execution Gated by Aegis Permissions

Beyond the sandbox, **Aegis permissions** gate every tool invocation at the Zeus Core level. Before a tool call is dispatched to zeus-sandbox, Aegis verifies:

- The requesting Titan has permission to invoke this tool
- The requesting user has permission to invoke this tool through this Titan
- The tool is within its rate limits and resource budget
- The tool's input conforms to its schema

This double-gating — Aegis at the request level, capabilities at the execution level — creates defense in depth. A bypass of one layer is stopped by the other.

### Memory Isolation Between Sandboxes

Each tool invocation runs in a **fresh sandbox instance**. There is no shared memory between sandboxes. A previous tool's data is completely inaccessible to a subsequent tool. Cross-tool data sharing is only possible through explicit Mnemosyne memory operations, which are themselves gated by Aegis permissions.

For Titans that need to run multiple tool calls with shared state, zeus-sandbox supports **session sandboxes** — a persistent WASM instance that retains state between calls but remains isolated from all other Titans and tool sessions. Session sandboxes are scoped to a single Titan and automatically destroyed when the Titan terminates.

## 5. Layer 4 — Audit & Response

The innermost security layer is the detection and response system. Even with perfect authentication, perfect channel security, and perfect sandboxing, incidents happen. Aegis's audit and response layer ensures you know about them immediately and can act decisively.

### Comprehensive API Call Logging

Every API call — every single one — is logged to the **Zeus Audit Log**. The log entry captures:

- **Timestamp**: Microsecond-precision UTC timestamp
- **Actor**: The identity that made the request (user ID, Titan ID, API token scope)
- **Action**: The endpoint, method, and normalized action description
- **Request metadata**: IP address, user agent, channel source
- **Result**: Success, failure, or exception, with full error details
- **Correlations**: Linked request ID, Titan session ID, and mission ID (if part of a Pantheon task)

Audit logs are written to an append-only, tamper-evident store. Each log entry is hashed and chained to the previous entry (a blockchain-style chain), making it impossible to retroactively alter or delete historical entries without detection.

Logs can be exported to external SIEM systems (Splunk, Elastic, Grafana Loki) via the `/v1/audit/stream` endpoint for long-term retention and advanced analysis.

### Prometheus Metrics for Anomaly Detection

Zeus exposes a comprehensive **Prometheus metrics endpoint** (`/metrics`) that tracks security-relevant metrics in real time:

- Failed authentication attempts per endpoint and per source IP
- Token refresh rates and anomaly spikes
- API call latency distributions (anomalously slow calls may indicate a compromised Titan)
- Sandboxed tool execution times and memory usage
- Memory query patterns (unusual query frequency may indicate scraping)
- Channel message volumes and rate-limit hits

These metrics feed directly into your existing Prometheus-based monitoring stack. Alerting rules can be configured to fire when thresholds are breached — for example, more than 10 failed auth attempts in 60 seconds from the same IP triggers an immediate alert.

### Automatic Session Revocation

When Aegis detects a threat, it does not wait for human intervention — it acts. **Automatic session revocation** is a configurable response system that takes immediate action when security thresholds are crossed:

- **Failed auth threshold**: After N failed authentication attempts from the same source, Aegis blocks that source for a cooldown period and revokes all active tokens for the affected account
- **Anomalous behavior detection**: If a Titan starts making requests that deviate significantly from its baseline (querying memory at unusual hours, accessing tools it has never used), Aegis pauses the Titan and alerts the operator
- **Rate limit violations**: Sources consistently exceeding rate limits are automatically blacklisted at the API gateway level
- **Channel compromise indicators**: An unusually high volume of outbound messages from a channel (potential spam or phishing) triggers automatic channel suspension

These responses are configurable — you decide the thresholds, the actions, and the notification channels. Aegis provides sensible defaults out of the box, but you own the policy.

### Encrypted Local Storage

All sensitive data at rest — channel credentials, API tokens, Titan secrets, user PII in memory stores — is encrypted using **AES-256-GCM** before being written to disk. Encryption keys are derived from a master key that you provide at installation via environment variable or secrets manager integration (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager).

The encryption is transparent to the rest of the system — Titans and skills read and write data as if it were plaintext, with the encryption/decryption handled by the storage layer. This means you do not need to modify your skills or Titan prompts to use encrypted storage — it is always on.

For the most sensitive operations, you can additionally enable **encrypted memory namespaces** in Mnemosyne — specific memory regions that require an additional decryption key, ensuring that even a compromised database file does not expose your most sensitive information.

---

Together, these four layers — Identity, Channel, Execution, and Audit — form a comprehensive security posture that protects your Zeus deployment at every level. Security is not a feature you add later; it is the foundation on which everything else is built.
