# Skills Migration Guide

## Current Skills Inventory

| Skill | Location | API/Capabilities | Dependencies | Priority |
|-------|----------|------------------|--------------|----------|
| **marketing-automation** | `skills/marketing-automation/` | CRM, email campaigns, segments | MARKETING_AUTOMATION_USERNAME, MARKETING_AUTOMATION_PASSWORD, MARKETING_AUTOMATION_URL | HIGH |
| **projectboard** | `skills/projectboard/` | Task management, work packages | PROJECTBOARD_API_KEY, PROJECTBOARD_URL | HIGH |
| **gog** | `skills/gog/` | Google Workspace, Gmail, Drive | Google OAuth tokens | MEDIUM |
| **chatterbox-tts** | `skills/chatterbox-tts/` | Text-to-speech | Local TTS server (localhost:8004) | LOW |

## Migration Approach

**Recommended: Option 3 - Hybrid**

Shared core skills for all agents + per-agent skills for custom tools.

## Migration Options

### Option 1: Shared Skills Directory Only

**Best for:** All agents need same skills, disk space not an issue

**Setup:**
```bash
# On server
mkdir -p ~/.openclaw/shared-skills
cp -r /path/to/ExecutiveAgents/skills/* ~/.openclaw/shared-skills/

# Update agents.json
{
  "agents": {
    "configPath": "~/ExecutiveAgents/agents.json",
    "skills": {
      "load": {
        "extraDirs": ["~/.openclaw/shared-skills"]
      }
    }
  }
}
```

**Pros:**
- One copy of each skill
- Updates apply to all agents
- Shared credentials via .env

**Cons:**
- Skill conflicts if agents need different versions
- Shared state

---

### Option 2: Per-Agent Skills (Isolated)

**Best for:** Agents need different skill versions

**Setup:**
```bash
# Create skill folders per agent
for agent in wembassy-coo-wren wembassy-cmo wembassy-cro wembassy-cto wembassy-hr mcintosh-rm; do
  mkdir -p ~/.openclaw/agents/$agent/skills
  cp -r ~/ExecutiveAgents/skills/* ~/.openclaw/agents/$agent/skills/
done
```

**Update agents.json:**
```json
{
  "id": "wembassy-cmo",
  "workspace": "~/.openclaw/agents/wembassy-cmo/workspace",
  "skills": {
    "load": {
      "dirs": ["~/.openclaw/agents/wembassy-cmo/skills"]
    }
  }
}
```

**Pros:**
- Complete isolation
- Agents can have different versions
- Independent updates

**Cons:**
- Uses more disk space
- Updates require touching multiple copies

---

### Option 3: Workspace + Shared Hybrid (Recommended for Wembassy)

**Best for:** Balance of sharing and isolation

**Setup:**
```
~/.openclaw/
├── shared-skills/          # Common skills (marketing-automation, projectboard)
│   ├── marketing-automation/
│   ├── projectboard/
│   └── gog/
├── agents/
│   ├── wembassy-cmo/
│   │   └── skills/         # CMO-specific skills
│   │       └── linkedin-scraper/
│   ├── wembassy-cro/
│   │   └── skills/         # CRO-specific skills
│   │       └── proposal-generator/
│   └── wembassy-cto/        # Uses only shared
```

**Update agents.json:**
```json
{
  "agents": {
    "skills": {
      "load": {
        "extraDirs": ["~/.openclaw/shared-skills"]
      }
    },
    "list": [{
      "id": "wembassy-cmo",
      "skills": {
        "load": {
          "dirs": ["~/.openclaw/agents/wembassy-cmo/skills"]
        }
      }
    }]
  }
}
```

## Environment Variables for Skills

Create `.env` on server with:

```bash
# ${PROJECTBOARD_NAME}
PROJECTBOARD_URL=${PROJECTBOARD_URL}
PROJECTBOARD_API_KEY=your_api_key_here

# ${MARKETING_AUTOMATION_PLATFORM}
MARKETING_AUTOMATION_URL=${MARKETING_AUTOMATION_URL}
MARKETING_AUTOMATION_USERNAME=${MARKETING_AUTOMATION_USERNAME}
MARKETING_AUTOMATION_PASSWORD=your_password_here

# Google (for gog skill)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GOOGLE_REFRESH_TOKEN=...

# Chatterbox TTS
CHATTERBOX_URL=http://localhost:8004
```

## Migration Checklist

### Pre-Migration
- [ ] Backup existing skills
- [ ] Document current API credentials
- [ ] Test skills on current system
- [ ] Note any deprecated scripts

### During Migration
- [ ] Copy skills to new location
- [ ] Update environment variables
- [ ] Test each skill individually
- [ ] Update agent configurations
- [ ] Restart gateway

### Post-Migration
- [ ] Test agent skill usage
- [ ] Verify API connectivity
- [ ] Check log files for errors
- [ ] Document any issues

## Skill-Specific Notes

### ${MARKETING_AUTOMATION_PLATFORM} Skill
- Requires HTTP Basic Auth (username/password)
- No OAuth, just Basic Auth
- API endpoints at `/api/{endpoint}`

### ${PROJECTBOARD_NAME} Skill
- Uses API key (not OAuth)
- Key set in header: `X-API-KEY: {key}`
- Some endpoints may need authentication review

### Chatterbox TTS
- Requires local FastAPI server running
- Server must be accessible to agents
- Consider Dockerizing this service

## Troubleshooting

### "Skill not found"
- Check path in agents.json
- Verify file permissions
- Restart OpenClaw gateway

### "API authentication failed"
- Verify .env variables loaded
- Check credentials haven't expired
- Test API manually with curl

### "Permission denied"
- Skill scripts need execute permissions
- Check ownership of skill directories
- Verify user running OpenClaw has access

## Quick Test Commands

After migration, test each agent:

```bash
# Test ${MARKETING_AUTOMATION_PLATFORM} skill
openclaw agent run wembassy-cmo "Run the ${MARKETING_AUTOMATION_PLATFORM} list-contacts script"

# Test ${PROJECTBOARD_NAME} skill
openclaw agent run wembassy-cmo "List my ${PROJECTBOARD_NAME} tasks"

# Test gog skill
openclaw agent run wembassy-cmo "Send a test email via gog"
```

## Security Considerations

- Store credentials in `.env` (not in skills)
- Use environment variables, not hardcoded
- Restrict .env file permissions: `chmod 600 .env`
- Rotate API keys after migration
- Review skill permissions quarterly

---

**Recommendation:** Use Option 3 (Hybrid) - Shared core skills (marketing-automation, projectboard, gog) with agent-specific skills for custom tools.
