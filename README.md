# OpenClaw — Personal AI Assistant

![OpenClaw](docs/assets/openclaw.png)

EXFOLIATE! EXFOLIATE!

![CI status](https://raw.githubusercontent.com/openclaw/openclaw/main/.github/badges/ci.svg) ![Discord](https://raw.githubusercontent.com/openclaw/openclaw/main/.github/badges/discord.svg) ![MIT License](https://raw.githubusercontent.com/openclaw/openclaw/main/.github/badges/license.svg)

**OpenClaw** is a _personal AI assistant_ you run on your own devices. It answers you on the channels you already use (WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, WebChat), plus extension channels like BlueBubbles, Matrix, Zalo, and Zalo Personal. It can speak and listen on macOS/iOS/Android, and can render a live Canvas you control. The Gateway is just the control plane — the product is the assistant.
If you want a personal, single-user assistant that feels local, fast, and always-on, this is it.
[Website](https://openclaw.ai) · [Docs](https://docs.openclaw.ai) · [Vision](VISION.md) · [DeepWiki](https://deepwiki.com/openclaw/openclaw) · [Getting Started](https://docs.openclaw.ai/start/getting-started) · [Updating](https://docs.openclaw.ai/install/updating) · [Showcase](https://docs.openclaw.ai/start/showcase) · [FAQ](https://docs.openclaw.ai/help/faq) · [Wizard](https://docs.openclaw.ai/start/wizard) · [Nix](https://github.com/openclaw/nix-openclaw) · [Docker](https://docs.openclaw.ai/install/docker) · [Discord](https://discord.gg/clawd) Preferred setup: run the onboarding wizard (`openclaw onboard`) in your terminal.
The wizard guides you step by step through setting up the gateway, workspace, channels, and skills. The CLI wizard is the recommended path and works on **macOS, Linux, and Windows (via WSL2; strongly recommended)**. Works with npm, pnpm, or bun.
New install? Start here: [Getting started](https://docs.openclaw.ai/start/getting-started) ## Sponsors | OpenAI | Blacksmith | Convex | | ----------------------------------------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------- | | [![OpenAI](docs/assets/sponsors/openai.svg)](https://openai.com/) | [![Blacksmith](docs/assets/sponsors/blacksmith.svg)](https://blacksmith.sh/) | [![Convex](docs/assets/sponsors/convex.svg)](https://www.convex.dev/) | **Subscriptions (OAuth):** - **[OpenAI](https://openai.com/)** (ChatGPT/Codex) Model note: while any model is supported, I strongly recommend **Anthropic Pro/Max (100/200) + Opus 4.6** for long‑context strength and better prompt‑injection resistance.
See [Onboarding](https://docs.openclaw.ai/start/onboarding). ## Models (selection + auth) - Models config + CLI: [Models](https://docs.openclaw.ai/concepts/models) - Auth profile rotation (OAuth vs API keys) + fallbacks: [Model failover](https://docs.openclaw.ai/concepts/model-failover) ## Install (recommended) Runtime: **Node ≥22**.
`bash npm install -g openclaw@latest # or: pnpm add -g openclaw@latest openclaw onboard --install-daemon ` The wizard installs the Gateway daemon (launchd/systemd user service) so it stays running. ## Quick start (TL;DR) Runtime: **Node ≥22**.
Full beginner guide (auth, pairing, channels): [Getting started](https://docs.openclaw.ai/start/getting-started) `bash openclaw onboard --install-daemon openclaw gateway --port 18789 --verbose # Send a message openclaw message send --to +1234567890 --message "Hello from OpenClaw" # Talk to the assistant (optionally deliver back to any connected channel: WhatsApp/Telegram/Slack/Discord/Google Chat/Signal/iMessage/BlueBubbles/Microsoft Teams/Matrix/Zalo/Zalo Personal/WebChat) openclaw agent --message "Ship checklist" --thinking high ` Upgrading? [Updating guide](https://docs.openclaw.ai/install/updating) (and run `openclaw doctor`).

---

## Travel business quickstart (highwayns fork)

This fork is used as a **travel automation workspace**: run OpenClaw locally and install a set of travel-related skills (hotel search/booking, flight search, local service booking, itinerary planning).

> ⚠️ Safety & compliance
>
> - Do not auto-purchase or auto-book without explicit user confirmation.
> - Some skills send **PII** (name/phone/email) to third-party providers to complete bookings.
> - Prefer an approval gate for “create booking / place order” actions.

### 1) Clone + install from source

Runtime: **Node ≥22**. Prefer `pnpm`.

```bash
git clone https://github.com/highwayns/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
pnpm openclaw onboard --install-daemon
```

### 2) Install travel skills into your workspace

Use the ClawHub CLI (installs into your workspace `./skills/` by default):

```bash
npm i -g clawhub
clawhub login
```

Install the travel skill set (adjust to your needs):

```bash
# Itinerary & planning
clawhub install travel
clawhub install travel-planning
clawhub install travel-manager

# Hotels
clawhub install aihotel
clawhub install star-hotel
clawhub install tuniu-hotel
clawhub install booking

# Flights
clawhub install flight-search
clawhub install flight

# Local services (e.g. haircuts, plumbers, etc.)
clawhub install service-booking

# Restaurants/food (region dependent)
clawhub install restaurant-crosscheck-cn
clawhub install swiggy
```

Verify:

```bash
openclaw skills list
openclaw skills check
```

### 3) Provider configuration (keys / binaries)

**tuniu-hotel** (hotel booking via Tuniu MCP) requires an API key:

```bash
export TUNIU_API_KEY="YOUR_TUNIU_API_KEY"
# Optional override:
export TUNIU_MCP_URL="https://openapi.tuniu.cn/mcp/hotel"
```

**flight-search** may require `uvx` on your system (check `openclaw skills info flight-search`).

### 4) Start the gateway

If you installed the daemon, the gateway may already be running. Check/stop/start:

```bash
# Start (will fail if already running on that port)
openclaw gateway --port 18789

# Stop a running gateway
openclaw gateway stop

# Start on the default travel demo port
openclaw gateway --port 18789
```

### 5) Run travel queries (CLI)

Some setups require an explicit session id when using the CLI. Use `--session-id cli` (or your preferred session).

```bash
openclaw agent --session-id cli --message "帮我查北京 2026-02-20 入住一晚的酒店，预算 600 元以内，列 5 个"
```

### 6) Troubleshooting

#### A) "Gateway already running" / "Port already in use"

Stop the existing gateway:

```bash
openclaw gateway stop
# or, if supervised via systemd user service:
systemctl --user stop openclaw-gateway.service
```

#### B) voice-call plugin duplicate id / schema errors (tts Zod schema)

If you see warnings like:

- `duplicate plugin id detected`
- `Invalid element at key "tts": expected a Zod schema`

You likely have **two voice-call plugins** configured. Remove/disable your custom `plugins.entries.voice-call` entry in your config so only the stock plugin remains, then restart the gateway.

Hint: search your config for `plugins.entries.voice-call` and delete/comment it out:

```bash
rg -n "plugins\.entries\.voice-call|voice-call" ~/.openclaw
```

Then restart:

```bash
openclaw gateway stop
openclaw gateway --port 18789
```

#### C) `openclaw agent` says: "Pass --to <E.164>, --session-id, or --agent"

Add a session id (or pick an agent explicitly):

```bash
openclaw agent --session-id cli --message "test"
# or:
openclaw agent --agent travel-manager --session-id cli --message "做一个三日东京行程"
```

## Development channels - **stable**: tagged releases (`vYYYY.M.D` or `vYYYY.M.D-`), npm dist-tag `latest`. - **beta**: prerelease tags (`vYYYY.M.D-beta.N`), npm dist-tag `beta` (macOS app may be missing). - **dev**: moving head of `main`, npm dist-tag `dev` (when published). Switch channels (git + npm): `openclaw update --channel stable|beta|dev`. Details: [Development channels](https://docs.openclaw.ai/install/development-channels). ## From source (development) Prefer `pnpm` for builds from source.

Bun is optional for running TypeScript directly. `bash git clone https://github.com/openclaw/openclaw.git cd openclaw pnpm install pnpm ui:build # auto-installs UI deps on first run pnpm build pnpm openclaw onboard --install-daemon # Dev loop (auto-reload on TS changes) pnpm gateway:watch ` Note: `pnpm openclaw ...` runs TypeScript directly (via `tsx`). `pnpm build` produces `dist/` for running via Node / the packaged `openclaw` binary.

## Security defaults (DM access) OpenClaw connects to real messaging surfaces. Treat inbound DMs as **untrusted input**.

Full security guide: [Security](https://docs.openclaw.ai/gateway/security) Default behavior on Telegram/WhatsApp/Signal/iMessage/Microsoft Teams/Discord/Google Chat/Slack: - **DM pairing** (`dmPolicy="pairing"` / `channels.discord.dmPolicy="pairing"` / `channels.slack.dmPolicy="pairing"`; legacy: `channels.discord.dm.policy`, `channels.slack.dm.policy`): unknown senders receive a short pairing code and the bot does not process their message.

- Approve with: `openclaw pairing approve  `` (then the sender is added to a local allowlist store). - Public inbound DMs require an explicit opt-in: set `dmPolicy="open"`and include`"\*"` in the channel allowlist (`allowFrom`/`channels.discord.allowFrom`/`channels.slack.allowFrom`; legacy: `channels.discord.dm.allowFrom`, `channels.slack.dm.policy`).
Run `openclaw doctor` to surface risky/misconfigured DM policies.

## Highlights - **[Local-first Gateway](https://docs.openclaw.ai/gateway)** — single control plane for sessions, channels, tools, and events. - **[Multi-channel inbox](https://docs.openclaw.ai/channels)** — WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, BlueBubbles (iMessage), iMessage (legacy), Microsoft Teams, Matrix, Zalo, Zalo Personal, WebChat, macOS, iOS/Android.

- **[Multi-agent routing](https://docs.openclaw.ai/gateway/configuration)** — route inbound channels/accounts/peers to isolated agents (workspaces + per-agent sessions). - **[Voice Wake](https://docs.openclaw.ai/nodes/voicewake) + [Talk Mode](https://docs.openclaw.ai/nodes/talk)** — always-on speech for macOS/iOS/Android with ElevenLabs. - **[Live Canvas](https://docs.openclaw.ai/platforms/mac/canvas)** — agent-driven visual workspace with [A2UI](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui).
- **[First-class tools](https://docs.openclaw.ai/tools)** — browser, canvas, nodes, cron, sessions, and Discord/Slack actions. - **[Companion apps](https://docs.openclaw.ai/platforms/macos)** — macOS menu bar app + iOS/Android [nodes](https://docs.openclaw.ai/nodes). - **[Onboarding](https://docs.openclaw.ai/start/wizard) + [skills](https://docs.openclaw.ai/tools/skills)** — wizard-driven setup with bundled/managed/workspace skills with install gating + UI.

## Star History [![Star History Chart](https://api.star-history.com/svg?repos=openclaw/openclaw&type=date&legend=top-left)](https://www.star-history.com/#openclaw/openclaw&type=date&legend=top-left) ## Everything we built so far ### Core platform - [Gateway WS control plane](https://docs.openclaw.ai/gateway) with sessions, presence, config, cron, webhooks, [Control UI](https://docs.openclaw.ai/web), and [Canvas host](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui).

- [CLI surface](https://docs.openclaw.ai/tools/agent-send): gateway, agent, send, [wizard](https://docs.openclaw.ai/start/wizard), and [doctor](https://docs.openclaw.ai/gateway/doctor). - [Pi agent runtime](https://docs.openclaw.ai/concepts/agent) in RPC mode with tool streaming and block streaming. - [Session model](https://docs.openclaw.ai/concepts/session): `main` for direct chats, group isolation, activation modes, queue modes, reply-back.
  Group rules: [Groups](https://docs.openclaw.ai/channels/groups). - [Media pipeline](https://docs.openclaw.ai/nodes/images): images/audio/video, transcription hooks, size caps, temp file lifecycle. Audio details: [Audio](https://docs.openclaw.ai/nodes/audio).

### Channels - [Channels](https://docs.openclaw.ai/channels): [WhatsApp](https://docs.openclaw.ai/channels/whatsapp) (Baileys), [Telegram](https://docs.openclaw.ai/channels/telegram) (grammY), [Slack](https://docs.openclaw.ai/channels/slack) (Bolt), [Discord](https://docs.openclaw.ai/channels/discord) (discord.js), [Google Chat](https://docs.openclaw.ai/channels/googlechat) (Chat API), [Signal](https://docs.openclaw.ai/channels/signal) (signal-cli), [BlueBubbles](https://docs.openclaw.ai/channels/bluebubbles) (iMessage, recommended), [iMessage](https://docs.openclaw.ai/channels/imessage) (legacy imsg), [Microsoft Teams](https://docs.openclaw.ai/channels/msteams) (extension), [Matrix](https://docs.openclaw.ai/channels/matrix) (extension), [Zalo](https://docs.openclaw.ai/channels/zalo) (extension), [Zalo Personal](https://docs.openclaw.ai/channels/zalouser) (extension), [WebChat](https://docs.openclaw.ai/web/webchat).

- [Group routing](https://docs.openclaw.ai/channels/group-messages): mention gating, reply tags, per-channel chunking and routing. Channel rules: [Channels](https://docs.openclaw.ai/channels).

### Apps + nodes - [macOS app](https://docs.openclaw.ai/platforms/macos): menu bar control plane, [Voice Wake](https://docs.openclaw.ai/nodes/voicewake)/PTT, [Talk Mode](https://docs.openclaw.ai/nodes/talk) overlay, [WebChat](https://docs.openclaw.ai/web/webchat), debug tools, [remote gateway](https://docs.openclaw.ai/gateway/remote) control.

- [iOS node](https://docs.openclaw.ai/platforms/ios): [Canvas](https://docs.openclaw.ai/platforms/mac/canvas), [Voice Wake](https://docs.openclaw.ai/nodes/voicewake), [Talk Mode](https://docs.openclaw.ai/nodes/talk), camera, screen recording, Bonjour pairing. - [Android node](https://docs.openclaw.ai/platforms/android): [Canvas](https://docs.openclaw.ai/platforms/mac/canvas), [Talk Mode](https://docs.openclaw.ai/nodes/talk), camera, screen recording, optional SMS.
- [macOS node mode](https://docs.openclaw.ai/nodes): system.run/notify + canvas/camera exposure. ### Tools + automation - [Browser control](https://docs.openclaw.ai/tools/browser): dedicated openclaw Chrome/Chromium, snapshots, actions, uploads, profiles. - [Canvas](https://docs.openclaw.ai/platforms/mac/canvas): [A2UI](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui) push/reset, eval, snapshot.
- [Nodes](https://docs.openclaw.ai/nodes): camera snap/clip, screen record, [location.get](https://docs.openclaw.ai/nodes/location-command), notifications. - [Cron + wakeups](https://docs.openclaw.ai/automation/cron-jobs); [webhooks](https://docs.openclaw.ai/automation/webhook); [Gmail Pub/Sub](https://docs.openclaw.ai/automation/gmail-pubsub). - [Skills platform](https://docs.openclaw.ai/tools/skills): bundled, managed, and workspace skills with install gating + UI.

### Runtime + safety - [Channel routing](https://docs.openclaw.ai/channels/channel-routing), [retry policy](https://docs.openclaw.ai/concepts/retry), and [streaming/chunking](https://docs.openclaw.ai/concepts/streaming). - [Presence](https://docs.openclaw.ai/concepts/presence), [typing indicators](https://docs.openclaw.ai/concepts/typing-indicators), and [usage tracking](https://docs.openclaw.ai/concepts/usage-tracking).

- [Models](https://docs.openclaw.ai/concepts/models), [model failover](https://docs.openclaw.ai/concepts/model-failover), and [session pruning](https://docs.openclaw.ai/concepts/session-pruning).
- [Security](https://docs.openclaw.ai/gateway/security) and [troubleshooting](https://docs.openclaw.ai/channels/troubleshooting). ### Ops + packaging - [Control UI](https://docs.openclaw.ai/web) + [WebChat](https://docs.openclaw.ai/web/webchat) served directly from the Gateway.
- [Tailscale Serve/Funnel](https://docs.openclaw.ai/gateway/tailscale) or [SSH tunnels](https://docs.openclaw.ai/gateway/remote) with token/password auth. - [Nix mode](https://docs.openclaw.ai/install/nix) for declarative config; [Docker](https://docs.openclaw.ai/install/docker)-based installs. - [Doctor](https://docs.openclaw.ai/gateway/doctor) migrations, [logging](https://docs.openclaw.ai/logging).
