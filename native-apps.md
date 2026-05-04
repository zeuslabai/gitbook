# Native Apps — Zeus on Every Screen

Zeus was built for operators who live in their terminals, engineers who prefer GUI interfaces, and power users who demand access from any device, anywhere. Whether you're debugging a misbehaving Titan at your desk, monitoring production health from a mobile device during an incident, or checking your agent swarm status from a browser while traveling, Zeus delivers a native experience optimized for each platform—while sharing a single source of truth.

This document covers Zeus's multi-platform architecture, the capabilities available on each frontend, and how real-time synchronization keeps every interface in perfect alignment.

## Platform Strategy

### One Daemon, Multiple Frontends

The foundation of Zeus's cross-platform experience is the Zeus Daemon—a single Rust-based service that runs on your infrastructure and handles all core logic, state management, and integrations. This daemon exposes a well-defined API surface via WebSocket (for real-time updates) and REST (for programmatic access and web serving). Every Zeus frontend—whether a native desktop application, a terminal interface, a mobile app, or a web browser—is a thin client that connects to this daemon.

This architectural decision means you never run multiple Zeus instances or worry about state drift between platforms. Your Titans run in one place. Your data lives in one place. Every interface you open simply connects to the same source.

### Same Core Capabilities Across All Platforms

No matter which frontend you open, you'll have access to the full range of Zeus capabilities. The Desktop app doesn't expose features that the TUI hides. The mobile app isn't a limited read-only view. Every platform delivers:

- Full Titan lifecycle management: spawn, configure, monitor, and terminate agents
- Real-time event streams from all active Titans
- Complete skill and capability assignment across your agent swarm
- Channel management for Discord, Telegram, Slack, email, and every other integration
- Wallet operations including key management and transaction signing
- Gateway administration including metrics, health checks, and configuration

The only differences between platforms are input paradigms and presentation—not functionality.

### Real-Time Sync via Tailscale API

Zeus leverages Tailscale to solve the hardest problem in distributed systems: secure, low-latency connectivity without complex network configuration. When you connect a new frontend to Zeus, it establishes a WireGuard tunnel through your Tailscale tailnet, enabling instant, encrypted communication with your daemon regardless of where it's running or what network it sits behind.

This means your Desktop app running on a laptop in a coffee shop connects to your Zeus daemon running on a home server through Tailscale's relay infrastructure—no port forwarding, no dynamic DNS, no firewall wrestling. The Tailscale API integration handles authentication, address assignment, and connection management automatically, keeping your Zeus infrastructure invisible to the public internet while remaining accessible to your authorized tailnet.

## Desktop App (macOS, Windows, Linux)

The Zeus Desktop application delivers a full-featured graphical interface for managing your entire agent infrastructure. Built with Tauri for native performance and a small binary footprint, the desktop app provides the most comprehensive view of your Zeus environment.

### 19 Sidebar Sections

The desktop app organizes its interface into 19 sidebar sections, each drilling into a specific aspect of the Titan ecosystem:

1. **Dashboard** — Overview of all Titans, recent events, and system health
2. **Zeus** — Configuration and control of the Zeus orchestrator Titan
3. **Aegis** — Security monitoring, access logs, and threat detection
4. **Mnemosyne** — Memory graph visualization and knowledge base management
5. **Pantheon** — Mission planning, task assignment, and swarm coordination
6. **Hermes** — Channel connections, message routing, and template management
7. **Aria** — Voice pipeline configuration, speech recognition, and audio processing
8. **Skills** — Global skill registry and capability assignment
9. **Wallet** — Key management, addresses, and transaction history
10. **Gateway** — Port configuration, TLS settings, and rate limiting
11. **Tailscale** — Network topology, peer management, and connection status
12. **Metrics** — Prometheus data visualization and performance graphs
13. **Logs** — Aggregated log viewer with filtering and search
14. **Channels** — Discord, Telegram, Slack, email, and all other integrations
15. **Missions** — Active mission queue, execution history, and scheduling
16. **Memory** — Entity browser, relationship graph, and query interface
17. **Alerts** — Alert rules, notification destinations, and incident history
18. **Settings** — App preferences, theme, hotkeys, and startup behavior
19. **Help** — Documentation, support links, and diagnostic tools

### System Tray Integration

When minimized, Zeus runs as a system tray icon (macOS menu bar, Windows system tray, or Linux notification area depending on your platform). Right-click the tray icon to access a compact context menu with quick actions: view daemon health, see active Titan count, check for new alerts, or open the full interface. The tray icon itself reflects system status with color coding—green for healthy, yellow for warnings, red for critical issues—allowing at-a-glance monitoring without launching the full application.

### Native Notifications

Zeus leverages each platform's native notification system to deliver real-time alerts. When a mission fails, when Hermes receives a message on a monitored channel, when Aegis detects a security anomaly, or when any Titan logs a critical error, you receive a platform-native notification with action buttons. Click a notification to jump directly to the relevant section in the Zeus interface. Notifications are fully configurable per-Titan and per-event-type, so you can tune alert thresholds and suppression rules to match your operational tolerance.

### Menu Bar Quick Access (macOS)

On macOS, the Zeus menu bar app provides instant access without occupying your Dock. Click the Zeus icon in the menu bar to reveal a compact overlay showing Titan status, recent activity, and quick command shortcuts. You can trigger common actions—pause a Titan, start a new mission, acknowledge an alert—directly from the menu bar without launching the full desktop window. The menu bar app connects to your daemon over Tailscale, so it works whether you're on the same network or remote.

### Global Hotkey Support

The desktop app registers system-wide keyboard shortcuts that work regardless of which application has focus. Default hotkeys include:

- `Cmd/Ctrl + Shift + Z` — Open/focus Zeus window
- `Cmd/Ctrl + Shift + A` — Quick-add new Titan
- `Cmd/Ctrl + Shift + M` — Open mission launcher
- `Cmd/Ctrl + Shift + L` — Toggle log viewer overlay

All hotkeys are fully customizable in Settings, and you can bind different shortcuts per-profile (e.g., different hotkeys on a laptop versus a desktop).

## Terminal UI (TUI)

For operators who live in the terminal, Zeus ships with a full-featured Text User Interface built with Rust's `ratatui` library. The TUI delivers the same data and capabilities as the desktop app but expressed through a keyboard-driven, terminal-native interface.

### 23 Dedicated Screens

The TUI organizes its navigation into 23 distinct screens covering the complete Zeus ecosystem. Each screen is designed for information density while maintaining readability. Screens include a real-time event stream viewer, an interactive memory graph renderer using Unicode box-drawing characters, a mission timeline with dependency visualization, a log aggregator with ANSI color support, and per-Titan detail views showing live metrics, active skills, and recent messages. Navigation between screens uses vim-style keybindings (`h/j/k/l` or arrow keys, `g` for top, `G` for bottom) for muscle-memory efficiency.

### Full Keyboard Navigation

Every action in the TUI is accessible without a mouse. Launch missions with `m`, configure Titans with `c`, toggle Titan state with `Space`, search across all data with `/`, and switch screens with the number keys or single-letter shortcuts displayed in each screen's header. The TUI supports tmux-style copy mode for selecting and copying log entries, and it preserves a scrollback buffer of 10,000 lines for historical inspection.

### For Operators and Developers

The TUI is purpose-built for two audiences: systems operators who need rapid access to infrastructure status during incident response, and developers who want to observe Titan behavior during development and testing. The interface exposes internal telemetry, message throughput rates, memory graph statistics, and skill execution traces that aren't surfaced in the simplified desktop dashboard—giving technical users the depth they need without leaving their terminal environment.

### Same Data as Desktop, Text-First

The TUI doesn't compromise on data fidelity. Every metric, event, log entry, and configuration option available in the desktop app is queryable in the TUI. Data is presented in text-first formats: ASCII tables for structured data, sparklines for time-series metrics, tree views for nested configurations, and syntax-highlighted JSON for raw API responses. The TUI connects to the same WebSocket stream as the desktop app, ensuring identical real-time updates across both interfaces.

## Mobile (iOS & Android)

The Zeus mobile apps for iOS and Android extend your control plane to your phone, ensuring you can manage your agent infrastructure from anywhere.

### Push Notifications

Mobile apps receive push notifications through platform-native channels (APNs for iOS, FCM for Android). These aren't simple alerts—Zeus mobile notifications carry full context. A mission failure notification includes the Titan name, failure reason, affected dependencies, and a one-tap "View Details" action that opens the relevant screen in the app. Notification groups allow batching: instead of ten separate alerts for ten Titan restarts, you receive one consolidated notification with a summary.

### Voice Input (Siri/Google Assistant Integration)

Zeus mobile integrates with each platform's voice assistant for hands-free operation. You can ask Siri or Google Assistant to start a Zeus mission, pause a specific Titan, check daemon health, or query your memory graph with natural language. Voice commands route through the assistant to the Zeus mobile app, which translates them into API calls to your daemon. Responses are spoken back through the assistant when appropriate (e.g., "Zeus is healthy. 6 Titans active. Mission 'deploy-staging' is running at 73%.")

### Biometric Auth (Face ID, Fingerprint)

The mobile app secures access with biometric authentication. On first launch, you enroll Face ID (iOS) or Fingerprint (Android) as your primary authentication method. When you open the app or execute sensitive operations (wallet transactions, Titan termination, configuration changes), biometric verification is required. The app also supports PIN fallback and device-passcode authentication. All communication with the daemon is encrypted end-to-end via Tailscale, and session tokens are stored in each platform's secure enclave—never in plain storage.

### Offline Command Queue

When your mobile device loses connectivity, the Zeus app doesn't leave you stranded. Commands you issue while offline—starting a mission, pausing a Titan, sending a channel message—are queued locally with timestamps. When connectivity is restored, the app automatically replays the queue to the daemon in order, handling conflicts gracefully (e.g., if you paused a Titan that someone else started while you were offline, you receive a notification explaining the divergence). The offline queue is limited to 100 commands and respects operation dependencies—pausing a Titan queues before terminating its active missions.

## Web App

The Zeus web application runs entirely in the browser with no install required, making it the most accessible frontend for occasional access or shared workstations.

### 45 Pages of Documentation and Controls

The web app provides 45 distinct pages organized into logical groups: Agent Management (Titan spawning, configuration, and lifecycle), Channel Operations (messaging, threading, and channel-specific settings), Memory & Knowledge (graph browsing, entity creation, and query interface), Mission Control (creation, scheduling, and monitoring), Security & Access (Aegis dashboard, audit logs, and permission management), Infrastructure (gateway config, Tailscale peers, and daemon health), and Developer Tools (API explorer, webhook configuration, and payload inspector).

### Real-Time Session Monitoring

The web app maintains a persistent WebSocket connection to your daemon, delivering live updates for all monitored metrics. The session dashboard shows active Titans, message throughput over the last hour, memory utilization trends, and a live event feed. Charts update in real time using streaming data from the daemon's metrics endpoint, and you can pause and zoom into any time window without losing the live feed context.

### No Install Required

The web app is served directly by the Zeus daemon's built-in HTTP server (running on your gateway port, typically 8080). Point your browser to `https://your-zeus-gateway.example.com`, authenticate with your credentials or Tailscale SSO, and you're in. No Electron packaging, no mobile provisioning profiles, no software distribution—just a URL. The app is fully responsive and works on tablets, phones, and desktops through adaptive layout.

### Browser-Based Access Anywhere

Because the web app connects over Tailscale by default, you access your Zeus dashboard through your tailnet's secure tunnel. No public-facing Zeus ports are required for web app access—the daemon initiates an outbound connection to Tailscale's DERP relay servers, and your browser connects through the same relay. This means you can securely manage your infrastructure from any internet-connected device without exposing services to the broader internet.

## Architecture

### All Frontends Connect to Same Zeus Daemon

Every Zeus frontend—whether desktop, TUI, mobile, or web—is a WebSocket client connecting to the same Zeus Daemon process. The daemon is the sole source of truth for all state: Titan configurations, mission state, memory graphs, channel credentials, and operational metrics. Frontends never communicate directly with each other; they subscribe to the daemon's event stream and issue commands through the daemon's command API.

### WebSocket for Real-Time Updates

The daemon exposes a WebSocket endpoint that streams state changes to all connected clients. Events include Titan state transitions, new messages on any channel, mission progress updates, memory graph modifications, metrics snapshots, and alert triggers. Clients receive events within milliseconds of emission, enabling responsive, live interfaces across all platforms. The WebSocket protocol includes client identification, subscription filtering (so mobile clients don't receive irrelevant desktop-specific events), and heartbeat keepalive.

### REST API for Programmatic Access

Beyond real-time streaming, the daemon exposes a comprehensive REST API for programmatic access. Every operation available through any frontend is accessible via HTTP API: create and configure Titans, start and monitor missions, query the memory graph, send messages through connected channels, and manage gateway settings. The REST API supports bearer token authentication, rate limiting, and request validation. API documentation is served at `/docs` on the gateway and includes interactive examples powered by Swagger UI.

### Shared State Across All Clients

Because all frontends connect to one daemon, state is inherently shared. When you configure a new Titan through the desktop app, the TUI reflects the change immediately. When Hermes receives a message on Telegram while you're monitoring through the web app, you see it in real time. When a mission completes while you're on mobile, your desktop notification reflects the updated status. There is no sync protocol to configure, no eventual consistency window to wait through—shared state is a direct consequence of the single-daemon architecture.

---

Whether you prefer the precision of a terminal, the visual clarity of a desktop window, the convenience of a phone in your pocket, or the accessibility of a browser, Zeus meets you where you are. Every interface is a first-class citizen, every screen delivers the same underlying power, and every interaction reflects the current truth of your infrastructure. That's Zeus: the agent platform that adapts to your workflow instead of forcing you into its own.

![Use Cases — Platform Matrix](../use-cases-quadrant.svg)
