# Server Deployment Checklist

## Pre-Deployment Setup

### 1. ${PROJECTBOARD_NAME} User Creation

**Login:** ${PROJECTBOARD_URL} as admin

**Create these users:**
- [ ] `wren-coo` - ${EMAIL_WREN} (Project Admin)
- [ ] `mason-cmo` - ${EMAIL_MASON} (Member)
- [ ] `grayson-cro` - ${EMAIL_GRAYSON} (Member)
- [ ] `spock-cto` - ${EMAIL_SPOCK} (Developer)
- [ ] `mary-hr` - ${EMAIL_MARY} (Member)
- [ ] `dave-rm` - ${EMAIL_DAVE} (Member)

**Generate API Keys:**
- [ ] Login as each user → My Account → Access Tokens → Generate
- [ ] Copy each API key
- [ ] Add to `.env` file as `PROJECTBOARD_API_KEY_{AGENT}`

### 2. Clone Repository on Server

```bash
# SSH to server
ssh user@your-server

# Clone repo
git clone https://github.com/cmcintosh/ExecutiveAgents
cd ExecutiveAgents

# Copy environment template
cp .env.example .env

# Edit with actual values
nano .env
```

### 3. Install OpenClaw (if not already)

```bash
# Install OpenClaw
npm install -g openclaw

# Or download OpenClaw.app for macOS
```

### 4. Configure OpenClaw Gateway

```bash
# Edit config
nano ~/.openclaw/config.json
```

**Add:**
```json
{
  "agents": {
    "configPath": "~/ExecutiveAgents/agents.json"
  },
  "skills": {
    "load": {
      "extraDirs": ["~/.openclaw/shared-skills"]
    }
  }
}
```

### 5. Configure Environment

**Create `.env` file:**
```bash
# From repo
cp ExecutiveAgents/.env ~/.openclaw/.env

# Or create new
nano ~/.openclaw/.env
```

**Fill in:**
- [ ] All Discord tokens
- [ ] All ${PROJECTBOARD_NAME} API keys
- [ ] ${MARKETING_AUTOMATION_PLATFORM} credentials
- [ ] Google OAuth tokens (if using gog)
- [ ] Chatterbox TTS URL

### 6. Migrate Skills

```bash
# Option 3: Hybrid approach (recommended)

# Create shared folder
mkdir -p ~/.openclaw/shared-skills

# Copy shared skills
cp -r ~/ExecutiveAgents/skills/marketing-automation ~/.openclaw/shared-skills/
cp -r ~/ExecutiveAgents/skills/projectboard ~/.openclaw/shared-skills/
cp -r ~/ExecutiveAgents/skills/gog ~/.openclaw/shared-skills/

# Per-agent skills (if any)
mkdir -p ~/.openclaw/agents/wembassy-cmo/skills
mkdir -p ~/.openclaw/agents/wembassy-cro/skills
# etc.
```

### 7. Test Skills

```bash
# Test ${PROJECTBOARD_NAME}
openclaw agent run wembassy-cmo "List my ${PROJECTBOARD_NAME} tasks"

# Test ${MARKETING_AUTOMATION_PLATFORM}
openclaw agent run wembassy-cmo "List recent contacts in ${MARKETING_AUTOMATION_PLATFORM}"

# Verify skills loaded
openclaw skills list
```

### 8. Start Agents

```bash
# Start all agents
openclaw gateway restart

# Or start individually
openclaw agent start wembassy-cmo
openclaw agent start wembassy-cro
openclaw agent start wembassy-cto
openclaw agent start wembassy-hr
openclaw agent start mcintosh-rm

# Wren (COO) runs as main
openclaw agent start wembassy-coo-wren
```

### 9. Verify Connectivity

**Discord:**
- [ ] Each bot appears online in Discord
- [ ] Test message to each channel

**${PROJECTBOARD_NAME}:**
- [ ] Each agent can create tasks
- [ ] Task shows correct user attribution
- [ ] Time tracking works

**${MARKETING_AUTOMATION_PLATFORM}:**
- [ ] Can list contacts
- [ ] Can create contacts

**Skills:**
- [ ] All shared skills accessible
- [ ] Skills load without errors

### 10. Test Inter-Agent Communication

```bash
# Test CMO → CRO
openclaw sessions send agent:wembassy-cro "Test from CMO"

# Test CRO → CTO
openclaw sessions send agent:wembassy-cto "Test from CRO"

# Test escalation to Wren
openclaw sessions send agent:wembassy-coo-wren "Test from CMO"
```

## Post-Deployment

### Monitor Logs
```bash
# View agent logs
openclaw agent logs wembassy-cmo --tail 100
openclaw agent logs wembassy-cro --tail 100

# View gateway logs
openclaw gateway logs --tail 50
```

### Verify Task Routing
1. Create test task in ${PROJECTBOARD_NAME}
2. Assign to mason-cmo
3. Verify CMO agent receives notification
4. Verify task attribution

### First Week Monitoring
- [ ] Check agent status daily
- [ ] Monitor Discord for errors
- [ ] Verify ${PROJECTBOARD_NAME} time tracking
- [ ] Test skill execution

## Rollback Plan

If issues:
1. Stop agents: `openclaw agent stop --all`
2. Restore previous config: `~/.openclaw/config.json`
3. Restart: `openclaw gateway restart`
4. Debug in logs

## Troubleshooting Quick Reference

| Issue | Check |
|-------|-------|
| "Agent not found" | agents.json path in config |
| "Discord connection failed" | Discord token validity |
| "Skill not found" | skills.load.extraDirs path |
| "API authentication failed" | .env credentials |
| "${PROJECTBOARD_NAME} 401" | API key, user permissions |
| "${MARKETING_AUTOMATION_PLATFORM} connection error" | ${MARKETING_AUTOMATION_PLATFORM} URL, credentials |

---

## Summary: What Gets Transferred

| Component | Location | Migration Method |
|-----------|----------|------------------|
| **Skills** | `~/.openclaw/shared-skills/` | rsync/scp |
| **Agent configs** | `~/ExecutiveAgents/agents.json` | git clone |
| **Environment vars** | `~/.openclaw/.env` | manual setup |
| **${PROJECTBOARD_NAME}** | Web UI | create users, generate keys |
| **Discord bots** | Token manager | already shared |

---

**Ready to deploy?** Start with ${PROJECTBOARD_NAME} user creation, then proceed through checklist.
