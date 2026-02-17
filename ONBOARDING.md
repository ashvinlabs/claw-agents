# Onboarding Guide

Step-by-step guide for first-time setup of your Ashvin OpenClaw gateway.

## Prerequisites

- Docker and Docker Compose installed
- `.env` file created with `OPENCLAW_GATEWAY_TOKEN` set (see [README](README.md))
- At least one model API key ready (Anthropic, OpenAI, Gemini, or OpenRouter)

## Run Onboarding

```bash
docker compose run --rm ashvin onboard --no-install-daemon
```

> The `--no-install-daemon` flag skips systemd daemon installation (not needed in Docker).

## What to Expect

The onboarding wizard walks you through these steps interactively:

### 1. Security Warning

You'll see a security disclaimer. Read it, then confirm to continue.

- OpenClaw can run shell commands, read files, and take actions
- Use strong models for any bot with tools enabled
- Run `openclaw security audit --deep` regularly

### 2. Onboarding Mode

Choose between:

| Mode | When to use |
|------|-------------|
| **QuickStart** | First time, just get it running (recommended) |
| **Manual** | Fine-tune port, network bind, Tailscale, auth details |

**For Docker, choose QuickStart** — the compose file already handles port/bind/auth config.

### 3. Auth & Model Provider

You'll be asked how you want to authenticate with AI providers. Options include:

| Choice | What it means |
|--------|---------------|
| **Anthropic API Key** | Paste your `sk-ant-...` key for Claude models |
| **OpenAI API Key** | Paste your `sk-...` key for GPT models |
| **OpenRouter** | Paste your `sk-or-...` key for multi-provider access |
| **Google Gemini** | Paste your Gemini API key |
| **Custom API** | Any OpenAI-compatible endpoint |

After entering your key, you'll pick a **default model** (e.g., Claude Sonnet, GPT-4o, etc.).

### 4. Gateway Configuration

QuickStart defaults (already configured in our compose):

```
Gateway port:     18789
Gateway bind:     LAN
Gateway auth:     Token
Tailscale:        Off
```

**If prompted for these, accept the defaults** — the compose file overrides them anyway.

### 5. Channels (Optional)

Set up messaging channels for your agents:

| Channel | Setup |
|---------|-------|
| **WhatsApp** | Scan QR code from terminal |
| **Telegram** | Enter bot token from [@BotFather](https://t.me/BotFather) |
| **Discord** | Enter bot token from Discord Developer Portal |
| **Slack** | Enter bot + app tokens |
| **Signal** | Install Signal CLI (prompted) |

You can skip this and add channels later:

```bash
docker compose run --rm ashvin channels login          # WhatsApp
docker compose run --rm ashvin channels add --channel telegram --token <TOKEN>
docker compose run --rm ashvin channels add --channel discord --token <TOKEN>
```

### 6. Skills (Optional)

The wizard offers to install community skills from [ClawHub](https://github.com/openclaw/clawhub). You can skip and add them later through the dashboard.

### 7. Done!

The wizard writes everything to `./ashvin/openclaw.json`. Now start the gateway:

```bash
docker compose up -d
```

## After Onboarding

```bash
# Check it's running
docker compose logs -f ashvin

# Open the dashboard
open http://localhost:18789

# Health check
docker compose run --rm ashvin health --token "$OPENCLAW_GATEWAY_TOKEN"

# Re-run onboarding to change settings
docker compose run --rm ashvin onboard --no-install-daemon

# Edit config directly
nano ./ashvin/openclaw.json
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `OPENCLAW_GATEWAY_TOKEN` error | Make sure `.env` exists and has the token set |
| Onboard hangs at channel QR | Your terminal needs TTY — make sure `stdin_open: true` and `tty: true` are set |
| Can't reach dashboard | Check `docker compose ps` — gateway should be `healthy` |
| Port conflict on 18789 | Change `OPENCLAW_GATEWAY_PORT` in `.env` |
