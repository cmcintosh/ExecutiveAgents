# Multi-Agent Setup Guide

## Discord Bot Configuration

Each agent has its own Discord bot token for independent channel communication.

### Tokens Reference

| Agent | Bot Name | Env Variable |
|-------|----------|--------------|
| Wren (COO) | ecobot | `DISCORD_TOKEN_WREN` |
| Mason (CMO) | CMO-Mason | `DISCORD_TOKEN_CMO_MASON` |
| Grayson (CRO) | CRO-Grayson | `DISCORD_TOKEN_CRO_GRAYSON` |
| Spock (CTO) | CTO-Spock | `DISCORD_TOKEN_CTO_SPOCK` |
| Mary (HR) | HR-Mary | `DISCORD_TOKEN_HR_MARY` |
| Dave (RM) | RM-Dave | `DISCORD_TOKEN_RM_DAVE` |

### Setting Environment Variables

**Option A: .env file (Recommended for server)**
```bash
# Copy the template
cp .env.example .env

# Edit with your tokens
nano .env

# Source the file
export $(cat .env | xargs)
```

**Option B: Direct export**
```bash
export DISCORD_TOKEN_CMO_MASON="your_token_here"
export DISCORD_TOKEN_CRO_GRAYSON="your_token_here"
export DISCORD_TOKEN_CTO_SPOCK="your_token_here"
export DISCORD_TOKEN_HR_MARY="your_token_here"
export DISCORD_TOKEN_RM_DAVE="your_token_here"
```

**Option C: OpenClaw config.json**
```json
{
  "agents": {
    "configPath": "~/ExecutiveAgents/agents.json",
    "env": {
      "DISCORD_TOKEN_CMO_MASON": "your_token_here",
      "DISCORD_TOKEN_CRO_GRAYSON": "your_token_here"
    }
  }
}
```

### Channel Mappings

| Agent | Primary Channel | Secondary Channels |
|-------|-----------------|-------------------|
| CMO (Mason) | ${DISCORD_CHANNEL_MARKETING} | ${DISCORD_CHANNEL_SOCIAL_MEDIA} |
| CRO (Grayson) | ${DISCORD_CHANNEL_SALES} | ${DISCORD_CHANNEL_PROPOSALS} |
| CTO (Spock) | ${DISCORD_CHANNEL_HUDDLE} | ${DISCORD_CHANNEL_DEVOPS}, ${DISCORD_CHANNEL_AUTOMATION} |
| HR (Mary) | ${DISCORD_CHANNEL_CONTRACTORS} | ${DISCORD_CHANNEL_GENERAL} |
| RM (Dave) | DM only | Family office private |

**Note:** RM agent (Dave) is restricted to DMs only due to sandbox security settings.

### Security

- `.env` file is in `.gitignore` - never commit tokens
- Each bot has its own rate limits (better than shared token)
- Token rotation: Update in .env and restart agents

---

## Deploying Agents

### Option 1: Via Gateway Config

Add to `~/.openclaw/config.json`:

```json
{
  "agents": {
    "configPath": "~/ExecutiveAgents/agents.json"
  }
}
```

Restart OpenClaw:
```bash
openclaw gateway restart
```

### Option 2: Direct Command

```bash
openclaw agent deploy --config ./agents.json --agent wembassy-cmo
```

## Monitoring Agents

Check all agent status:
```bash
openclaw agent list
```

Check specific agent:
```bash
openclaw agent status wembassy-coo-wren
```

View agent logs:
```bash
openclaw agent logs wembassy-cto --tail 100
```

## Updating Agents

After modifying SOUL.md or agents.json:

```bash
# Reload configuration
openclaw agent reload wembassy-cmo

# Or restart specific agent
openclaw agent restart wembassy-cmo
```

## Troubleshooting

### "Agent not found"
- Verify agent ID in agents.json
- Check config path is correct
- Run `openclaw agent list` to see loaded agents

### "Tool denied"
- Check tool permissions in agents.json
- Review agent's `tools.allow` and `tools.deny`
- Use `openclaw sandbox explain` for details

### "Sandbox error"
- RM agent runs in Docker - ensure Docker is running
- Check disk space for container images
- Review `/tmp/mcintosh-rm-sandbox` permissions

## Security Best Practices

1. **Regular audits** - Review agent permissions quarterly
2. **Principle of least privilege** - Agents only get tools they need
3. **Sandbox sensitive agents** - RM agent always sandboxed
4. **Monitor communications** - Log inter-agent messaging
5. **Rotate credentials** - Update auth keys periodically
6. **Backup configs** - Keep agents.json in version control

## Scaling

To add more agents:

1. Create new folder in `agents/`
2. Write SOUL.md with clear role definition
3. Add to agents.json with appropriate permissions
4. Test in development environment
5. Deploy to production
6. Monitor for 48 hours

## Integration Testing

Test agent collaboration:

```bash
# Start multiple agents
openclaw agent start wembassy-cmo &
openclaw agent start wembassy-cro &

# Send test message between agents
openclaw sessions send agent:wembassy-cro "Test from CMO"
```

## Performance

Monitor agent resource usage:
- CPU: Agents use minimal CPU
- Memory: ~100MB per agent instance
- Disk: Sandbox containers add ~500MB each

Optimize by:
- Running non-sandboxed agents on shared infrastructure
- Scheduling sandboxed agents only when needed
- Cleaning up unused containers regularly
