# Ashvin — Multi-Agent OpenClaw Gateway

A production-ready Docker setup for running [OpenClaw](https://github.com/openclaw/openclaw) as a multi-agent office. One container, one command.

## Quick Start

```bash
# 1. Clone this repo
git clone git@github.com:ashvinlabs/claw-agents.git && cd claw-agents

# 2. Configure your environment
cp .env.example .env
nano .env  # Set OPENCLAW_GATEWAY_TOKEN + at least one API key

# 3. First-time onboard
docker compose run --rm ashvin onboard --no-install-daemon

# 4. Start the gateway
docker compose up -d

# 5. Open the dashboard
open http://localhost:18789
```

## Usage

```bash
# Gateway management
docker compose up -d              # Start
docker compose down               # Stop
docker compose logs -f ashvin     # Tail logs
docker compose restart ashvin     # Restart

# CLI commands (one-shot, reuses same container image)
docker compose run --rm ashvin onboard
docker compose run --rm ashvin channels login          # WhatsApp QR
docker compose run --rm ashvin channels add --channel telegram --token <TOKEN>
docker compose run --rm ashvin channels add --channel discord --token <TOKEN>
docker compose run --rm ashvin health --token "$OPENCLAW_GATEWAY_TOKEN"
docker compose run --rm ashvin config
```

## Architecture

| Component | Details |
|-----------|---------|
| **Image** | `ghcr.io/openclaw/openclaw:latest` |
| **Ports** | `18789` (gateway/dashboard), `18790` (bridge) |
| **Data** | `./ashvin/` — all config, memory, skills, devices, workspace |
| **Memory limit** | 4GB cap / 512MB reserved |
| **Healthcheck** | Every 30s, 3 retries, auto-restart on failure |
| **Log rotation** | 50MB × 5 files |
| **Restart** | `unless-stopped` — survives crashes and host reboots |

### Volumes

| Mount | Purpose |
|-------|---------|
| `./ashvin` → `/home/node/.openclaw` | All OpenClaw state (config, memory, skills, devices, workspace) |
| `/mnt/g` → `/mnt/gdrive` | Windows G: drive access for agents |
| `docker.sock` → `docker.sock` | Container management — agents can spin up/manage other containers |

## Configuration

Copy `.env.example` to `.env` and set your values:

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENCLAW_GATEWAY_TOKEN` | ✅ | Auth token. Generate with `openssl rand -hex 32` |
| `ANTHROPIC_API_KEY` | At least one | Claude models |
| `OPENAI_API_KEY` | At least one | GPT models |
| `GEMINI_API_KEY` | At least one | Gemini models |
| `OPENROUTER_API_KEY` | At least one | Multi-provider router |
| `TELEGRAM_BOT_TOKEN` | Optional | Telegram channel |
| `DISCORD_BOT_TOKEN` | Optional | Discord channel |
| `SLACK_BOT_TOKEN` | Optional | Slack channel |

See [.env.example](.env.example) for the full list.

## Managing Agents

Configure agents in `./ashvin/openclaw.json`. Each agent gets its own personality, skills, model assignments, and channel bindings. The gateway manages all agent lifecycles automatically.

## Container Management

The Docker socket is mounted, giving agents the ability to manage sibling containers — spin up services, run one-off tasks, or orchestrate multi-container workflows.

## Network Connectivity

The gateway binds to LAN by default, making it accessible to other devices and OpenClaw instances on your network:

- **Gateway:** `http://<your-ip>:18789`
- **Bridge:** `http://<your-ip>:18790`

Use the same `OPENCLAW_GATEWAY_TOKEN` across instances to federate them.

## License

This is a deployment configuration for [OpenClaw](https://github.com/openclaw/openclaw) (MIT licensed).
