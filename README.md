# Scout Agent

Scout is a Share-a-Cart agent built on [OpenClaw](https://openclaw.ai). Its one job: build real Amazon shared cart links via the [Share-a-Cart](https://share-a-cart.com) API.

## What Scout Does

- Receives cart requests (ASINs, product names, quantities)
- Calls the real Share-a-Cart API (`https://crtsh.net/api/make/saveWithMetadata`)
- Returns a real cart ID and shareable link (`https://crtsh.net/c/CART_ID`)

No fake IDs. No simulations. Real API calls only.

## Repo Structure

```
scout-agent/
├── SOUL.md               # Scout's mandate and core behaviors
├── AGENTS.md             # Session startup rules and memory structure
├── IDENTITY.md           # Agent identity (name, vibe, emoji)
├── SCOUT_PERSISTENT.md   # Share-a-Cart API rules, curl command, format
├── HEARTBEAT.md          # Periodic task config (optional)
├── TOOLS.md              # Environment-specific notes
└── README.md             # This file
```

## Setup

### Prerequisites

- [OpenClaw](https://openclaw.ai) installed
- Anthropic API key (or other supported LLM provider)
- Slack app with bot token + app token (for Socket Mode)

### Installation

1. Clone this repo into your OpenClaw workspace:
   ```bash
   git clone https://github.com/MateoDev/scout-agent ~/.openclaw/workspace
   ```

2. Create your `~/.openclaw/openclaw.json` with your credentials:
   ```json
   {
     "env": {
       "ANTHROPIC_API_KEY": "your-key-here"
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

3. Start the gateway:
   ```bash
   openclaw gateway
   ```

## Response Format

Scout always responds in this format (Slack mrkdwn):

```
📋 *[Cart Name] Created*

*Cart ID: XXXXX*

• [Item 1] - $[price]
• [Item 2] - $[price]

*Total: $[sum]*

*Next:* Go to <http://share-a-cart.com|share-a-cart.com> → Enter *XXXXX* → Compare prices → Purchase
```

## API Reference

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

Response contains a cart ID — share link: `https://crtsh.net/c/{cartId}`

## Built By

Built on [OpenClaw](https://openclaw.ai) — the AI agent platform.
