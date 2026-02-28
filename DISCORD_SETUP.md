# Agent Discord Configuration

This document describes the Discord bot configuration for each agent.

## Bot Tokens

Each agent has its own Discord bot token, configured via environment variables:

| Agent ID | Bot Name | Env Variable | Default Channel |
|----------|----------|--------------|-----------------|
| `wembassy-coo-wren` | Wren (COO) | `DISCORD_TOKEN_WREN` | DMs, ${DISCORD_CHANNEL_HUDDLE} |
| `wembassy-cmo` | Mason (CMO) | `DISCORD_TOKEN_CMO_MASON` | ${DISCORD_CHANNEL_MARKETING} |
| `wembassy-cro` | Grayson (CRO) | `DISCORD_TOKEN_CRO_GRAYSON` | ${DISCORD_CHANNEL_SALES} |
| `wembassy-cto` | Spock (CTO) | `DISCORD_TOKEN_CTO_SPOCK` | ${DISCORD_CHANNEL_HUDDLE}, ${DISCORD_CHANNEL_DEVOPS} |
| `wembassy-hr` | Mary (HR) | `DISCORD_TOKEN_HR_MARY` | ${DISCORD_CHANNEL_CONTRACTORS} |
| `mcintosh-rm` | Dave (RM) | `DISCORD_TOKEN_RM_DAVE` | DM only |

## Configuration

Tokens are referenced in `agents.json` as environment variables:

```json
{
  "channels": {
    "discord": {
      "token": "${DISCORD_TOKEN_CMO_MASON}",
      "defaultChannels": ["${DISCORD_CHANNEL_MARKETING}"]
    }
  }
}
```

## Server Deployment

1. Copy `.env.example` to `.env` on the server
2. Fill in actual bot tokens
3. Source the file: `export $(cat .env | xargs)`
4. Or add to OpenClaw's config.json

## Bot Permissions Needed

Each bot needs these Discord permissions:
- Send Messages
- Read Messages
- Read Message History
- Mention @everyone, @here, and All Roles
- Attach Files
- Embed Links
- Add Reactions

## Security

- Never commit `.env` with real tokens
- Tokens are in `.gitignore`
- Rotate tokens periodically
- RM agent is sandboxed and DM-only
