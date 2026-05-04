# Scout Agent

Scout is a shopping cart generation agent built on [OpenClaw](https://openclaw.ai). Its mandate: make Share-a-Cart link creation easy — real API calls, real cart IDs, no fuss.

Give Scout a list of Amazon products and it returns a shareable cart link your customers can click to load everything into their Amazon cart instantly.

---

## How It Works

Scout calls the [Share-a-Cart](https://share-a-cart.com) API (`crtsh.net`) with product data and returns a real cart ID and link. It can be triggered via:

- **Slack DM or channel mention** — tag Scout and ask for a cart
- **MCP** — call Scout's cart-creation tool programmatically

No fake IDs. No simulations. Real API calls only.

---

## Prerequisites

- An [OpenClaw](https://openclaw.ai) agent deployed (XO or self-hosted)
- A Slack workspace (admin access)
- An LLM API key (see [Supported Models](#supported-models) below)

---

## Setup

### 1. Connect Slack

Follow the official XO guide to create your Slack app and get your tokens:
👉 **[Connect Slack to OpenClaw](https://docs.xo.builders/agents/openclaw/channels/connect-slack#create-the-slack-app)**

Short version:
1. Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From a manifest**
2. Paste the manifest from the guide above
3. Install to your workspace → copy the **Bot Token** (`xoxb-...`)
4. Go to **Basic Information** → **App-Level Tokens** → generate with `connections:write` scope → copy **App Token** (`xapp-...`)

---

### 2. Choose a Model

Scout works with any LLM provider supported by OpenClaw. Add your chosen provider's API key to your `openclaw.json`.

#### Supported Models

| Provider | Models | Notes |
|---|---|---|
| **Anthropic** ⭐ | `claude-sonnet-4-5`, `claude-haiku-3-5` | Recommended — best instruction-following |
| **OpenAI** | `gpt-4o`, `gpt-4o-mini` | Strong alternative |
| **Google** | `gemini-1.5-pro`, `gemini-1.5-flash` | Good for high-volume/low-cost |
| **Mistral** | `mistral-large`, `mistral-small` | Lightweight option |

> **Recommended:** Anthropic Claude Sonnet — best at following the strict output format Scout requires.

Get your API key:
- Anthropic: [console.anthropic.com](https://console.anthropic.com)
- OpenAI: [platform.openai.com](https://platform.openai.com)
- Google: [aistudio.google.com](https://aistudio.google.com)

---

### 3. Configure OpenClaw

Create `~/.openclaw/openclaw.json` with your credentials (never commit this file):

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "your-anthropic-key-here"
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-5"
      }
    }
  },
  "channels": {
    "slack": {
      "enabled": true,
      "mode": "socket",
      "botToken": "xoxb-your-bot-token",
      "appToken": "xapp-your-app-token",
      "dmPolicy": "open",
      "groupPolicy": "open",
      "allowFrom": ["*"],
      "ackReaction": "mag"
    }
  },
  "plugins": {
    "entries": {
      "anthropic": { "enabled": true },
      "slack": { "enabled": true, "config": {} }
    }
  }
}
```

> **Using OpenAI instead?** Replace the `env` key with `OPENAI_API_KEY` and set `"primary": "openai/gpt-4o"`.

---

### 4. Install Workspace Files

Clone this repo into your OpenClaw workspace:

```bash
git clone https://github.com/MateoDev/scout-agent ~/.openclaw/workspace
```

Or copy the files manually into your existing workspace directory.

---

### 5. Start the Gateway

```bash
openclaw gateway
```

Scout is now live. DM your Slack bot or tag it in a channel.

---

## Workspace Files

| File | Purpose |
|---|---|
| `SOUL.md` | Scout's mandate — cart creation only, real API only |
| `AGENTS.md` | Session startup rules, memory structure, group chat behavior |
| `IDENTITY.md` | Agent name, vibe, emoji |
| `SCOUT_PERSISTENT.md` | Share-a-Cart API rules, working curl command, output format |
| `HEARTBEAT.md` | Periodic background task config (optional) |
| `TOOLS.md` | Your environment-specific notes (cameras, SSH, etc.) |

---

## Share-a-Cart API Reference

**Endpoint:** `POST https://crtsh.net/api/make/saveWithMetadata`

```bash
curl -X POST "https://crtsh.net/api/make/saveWithMetadata" \
  -H "Content-Type: application/json" \
  -H "User-Agent: Scout-Agent/1.0" \
  -d '{
    "cart": [
      {
        "asin": "B08N5WRWNW",
        "title": "Product Name",
        "price": 29.99,
        "quantity": 1
      }
    ],
    "vendor": "amazon",
    "title": "My Cart"
  }'
```

Response contains a cart ID. Share link format: `https://crtsh.net/c/{cartId}`

---

## Response Format

Scout always replies in this format:

```
📋 *Cart Name Created*

*Cart ID: XXXXX*

• Item 1 - $price
• Item 2 - $price

*Total: $sum*

*Next:* Go to share-a-cart.com → Enter XXXXX → Compare prices → Purchase
```

---

## Built On

- [OpenClaw](https://openclaw.ai) — AI agent platform
- [Share-a-Cart](https://share-a-cart.com) — Amazon cart sharing
- [XO](https://xo.builders) — Agent hosting
