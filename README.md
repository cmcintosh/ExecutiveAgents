# ExecutiveAgents

Multi-agent setup for ${COMPANY_NAME} and ${FAMILY_OFFICE_NAME} using OpenClaw.

## Overview

This repository contains the configuration for 6 specialized AI agents, each with their own role, responsibilities, and security profile.

## Agent Structure

### ${COMPANY_NAME} Agents (Business)

| Agent | Role | Focus | Security |
|-------|------|-------|----------|
| **wembassy-coo-wren** | Chief Operating Officer | Operations, revenue, pipeline | Full access |
| **wembassy-cmo** | Chief Marketing Officer | Lead generation, content, brand | Standard |
| **wembassy-cro** | Chief Revenue Officer | Sales, proposals, closing | Standard |
| **wembassy-cto** | Chief Technology Officer | Development, architecture, AI | Elevated |
| **wembassy-hr** | Human Resources | Team, contractors, hiring | Standard |

### ${FAMILY_OFFICE_NAME} Agents (Private)

| Agent | Role | Focus | Security |
|-------|------|-------|----------|
| **mcintosh-rm** | Relationship Manager | Trust, assets, family office | **Sandboxed** |

## Configuration

### agents.json

The main configuration file defining:
- Agent identities and workspaces
- Sandbox modes (off, shared, all)
- Tool permissions (allow/deny lists)
- Channel bindings

### SOUL.md Files

Each agent has a `SOUL.md` defining:
- Role and responsibilities
- Voice and tone guidelines
- Key metrics and priorities
- Collaboration patterns
- Non-negotiable constraints

## Security Profiles

### Full Access (COO)
- Sandbox: `off`
- Tools: All allowed
- Direct access to business systems

### Standard (CMO, CRO, HR)
- Sandbox: `off`
- Tools: Restricted (no exec/process for CMO/CRO/HR)
- Safe for business operations

### Elevated (CTO)
- Sandbox: `off`
- Tools: All allowed (includes exec, process, apply_patch)
- Needed for development work

### Sandboxed (RM)
- Sandbox: `all` (container per agent)
- Scope: `agent` (isolated)
- Tools: Significantly restricted (no exec, browser, etc.)
- **Maximum security for family office**

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/cmcintosh/ExecutiveAgents
   ```

2. Configure OpenClaw to use `agents.json`:
   ```bash
   openclaw config set agents.configPath ./ExecutiveAgents/agents.json
   ```

3. Start an agent:
   ```bash
   openclaw agent start wembassy-cmo
   ```

## Agent Communication

### Inter-Agent Messaging

Agents can communicate via `sessions_send`:

```javascript
// From CMO to CRO
sessions_send({
  sessionKey: "agent:wembassy-cro",
  message: "New sales letter ready for review"
})
```

### Human Handoff

All agents can escalate to ${CEO_FIRST_NAME} via Discord DM.

## Privacy Considerations

- **${COMPANY_NAME} agents** handle business operations
- **mcintosh-rm agent** handles sensitive family matters
- **No data sharing** between business and family agents
- **RM agent is sandboxed** with restricted tools
- **Secure channels only** for family office communications

## Development

To add a new agent:

1. Create folder in `agents/{agent-id}/`
2. Write `SOUL.md` with role definition
3. Add entry to `agents.json`
4. Configure sandbox and tool permissions
5. Test in isolated environment
6. Document in this README

## Documentation

### Internal Documentation
- [TEAM-HANDBOOK.md](TEAM-HANDBOOK.md) - Information security policies and incident reporting
- [WORKFLOW.md](WORKFLOW.md) - Agent collaboration workflow
- [EMAIL_WORKFLOW.md](EMAIL_WORKFLOW.md) - Email routing and forwarding protocol
- [DISCORD_SETUP.md](DISCORD_SETUP.md) - Discord bot configuration
- [SETUP.md](SETUP.md) - Deployment and server setup guide
- [SKILLS_MIGRATION.md](SKILLS_MIGRATION.md) - Transfer skills to server
- [PROJECTBOARD_SETUP.md](PROJECTBOARD_SETUP.md) - Create agent user accounts in ${PROJECTBOARD_NAME}
- [DEPLOYMENT_CHECKLIST.md](DEPLOYMENT_CHECKLIST.md) - Step-by-step deployment

### OpenClaw Reference
- [OpenClaw Multi-Agent Docs](https://docs.openclaw.ai/tools/multi-agent-sandbox-tools)
- [Sandboxing Guide](https://docs.openclaw.ai/gateway/sandboxing)
- [Tool Policy](https://docs.openclaw.ai/gateway/sandbox-vs-tool-policy-vs-elevated)
- [Skills Documentation](https://docs.openclaw.ai/tools/skills)

## Support

For OpenClaw agent issues:
- `openclaw sandbox explain` - Debug tool restrictions
- `openclaw agent list` - See all configured agents
- `openclaw agent status {agent-id}` - Check agent health

## License

This project is licensed under the [MIT License](LICENSE).
