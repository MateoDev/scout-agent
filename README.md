# Scout Agent

Scout is a shopping cart generation agent built on [OpenClaw](https://openclaw.ai) and hosted on [XO](https://xo.builders). Its mandate: make Amazon Share-a-Cart link creation easy — real API calls, real cart IDs, no fuss.

Give Scout a list of Amazon products and it returns a shareable cart link your customers can click to load everything into their Amazon cart instantly.

---

## How It Works

Scout calls the [Share-a-Cart](https://share-a-cart.com) REST API with product data and returns a real cart ID and shareable link. It is triggered via:

- **Slack DM or channel mention** — tag Scout and ask for a cart
- **MCP** — call Scout's cart-creation tool programmatically from another agent or workflow

No fake IDs. No simulations. Real API calls only.

---

## Deployment Overview

This agent was originally deployed as follows:

1. **XO Instance** — deployed on [beta.xo.builders](https://beta.xo.builders) as `scout-agent`
2. **Slack** — connected via manual Slack app setup + manifest JSON (Socket Mode)
3. **Model** — powered by an LLM API key added at deploy time (see [Supported Models](#supported-models))
4. **Mandate** — given via chat on first run by the owner account; other users interact with it as a Slack app

To self-host, follow the steps below.

---

## Self-Hosting Setup

### 1. Deploy an OpenClaw Agent on XO

Go to [beta.xo.builders](https://beta.xo.builders) and create a new agent instance. Name it `scout-agent` or similar.

Alternatively, run OpenClaw locally:
```bash
npm install -g openclaw
openclaw gateway
```

### 2. Add Your Workspace Files

Clone this repo into your OpenClaw workspace:

```bash
git clone https://github.com/MateoDev/scout-agent ~/.openclaw/workspace
```

Or copy the files manually into your agent's workspace directory.

### 3. Connect Slack

Follow the official guide to create your Slack app and get your tokens:
👉 **[Connect Slack to OpenClaw](https://docs.xo.builders/agents/openclaw/channels/connect-slack#create-the-slack-app)**

Steps:
1. Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From a manifest**
2. Paste the manifest from the guide (enables Socket Mode + all required bot scopes)
3. **Install to Workspace** → copy **Bot Token** (`xoxb-...`)
4. **Basic Information → App-Level Tokens** → generate with `connections:write` → copy **App Token** (`xapp-...`)

### 4. Choose a Model

Scout works with any LLM provider supported by OpenClaw. Anthropic Claude is recommended for best instruction-following.

| Provider | Models | Get API Key |
|---|---|---|
| **Anthropic** ⭐ Recommended | `claude-sonnet-4-5`, `claude-haiku-3-5` | [console.anthropic.com](https://console.anthropic.com) |
| **OpenAI** | `gpt-4o`, `gpt-4o-mini` | [platform.openai.com](https://platform.openai.com) |
| **Google** | `gemini-1.5-pro`, `gemini-1.5-flash` | [aistudio.google.com](https://aistudio.google.com) |
| **Mistral** | `mistral-large`, `mistral-small` | [console.mistral.ai](https://console.mistral.ai) |

### 5. Configure OpenClaw

Create `~/.openclaw/openclaw.json` — **never commit this file**, it contains your credentials:

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

> Using OpenAI? Replace `ANTHROPIC_API_KEY` with `OPENAI_API_KEY` and set `"primary": "openai/gpt-4o"`.

### 6. Give Scout Its Mandate

On first run, DM Scout in Slack from your admin account and tell it:

> "You are Scout — your job is to build Share-a-Cart links using the Share-a-Cart API. Read SCOUT_PERSISTENT.md and follow it strictly."

This initializes Scout's context. Other users can then interact with it as a Slack app without needing to configure anything.

---

## Workspace Files

| File | Purpose |
|---|---|
| `SOUL.md` | Scout's mandate — cart creation only, real API only |
| `AGENTS.md` | Session startup rules, memory structure, group chat behavior |
| `IDENTITY.md` | Agent name, vibe, emoji |
| `SCOUT_PERSISTENT.md` | Share-a-Cart API rules, working curl command, output format |
| `HEARTBEAT.md` | Periodic background task config (optional) |
| `TOOLS.md` | Your environment-specific notes |

---

## Share-a-Cart API Reference

Full API docs: [SAC API Reference](https://share-a-cart.com/sac-api-reference)

**Production base URL:** `https://share-a-cart.com`
**Staging base URL:** `https://cart-a-share.com`

### Create a Cart

`POST /api/make/saveWithMetadata` — no auth required

```bash
curl -X POST "https://share-a-cart.com/api/make/saveWithMetadata" \
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

Response contains a `cartid`. Share link: `https://share-a-cart.com/c/{cartid}`

### Other Useful Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/get/:cartid` | Get cart by ID |
| `POST` | `/api/merge` | Merge 2–5 carts (same store) |
| `GET` | `/api/make/vendorlist` | List supported vendors |
| `GET` | `/api/make/:vendor/:keywords` | Search items by keyword |

---

## Response Format

Scout always replies in this format (Slack mrkdwn):

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
- [XO](https://xo.builders) — Agent hosting
- [Share-a-Cart](https://share-a-cart.com) — Amazon cart sharing API
