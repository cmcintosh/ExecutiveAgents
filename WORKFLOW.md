# Agent Collaboration: Tool/Skill Development Workflow

## Overview

To maintain clean separation of concerns and ensure technical quality, **tool and skill creation is the exclusive domain of the CTO agent**. Business agents (CMO, CRO, COO) identify needs and request development, but do NOT build tools themselves.

## The Workflow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   CMO/CRO   │────▶│  CTO Agent  │────▶│   Docker    │
│  (Request)  │     │  (Develop)  │     │  (Execute)  │
└─────────────┘     └─────────────┘     └─────────────┘
        │                    │                    │
        │  Skill Request     │  Build Skill       │  Run with
        │  (Markdown)        │  (SKILL.md)        │  Dependencies
        │                    │                    │
        ▼                    ▼                    ▼
   "I need a tool    Creates:            Isolated
    to scrape       ~/.openclaw/       container
    LinkedIn         skills/            with libs
    profiles"
```

## Agent Responsibilities

### **CMO / CRO / COO (Business Agents)**

**What You Do:**
- ✅ Identify a need ("I need to scrape LinkedIn data")
- ✅ Submit a **Skill Request** to CTO via `sessions_send`
- ✅ Provide clear requirements and acceptance criteria
- ✅ Test the skill once delivered
- ✅ Use the skill for business purposes

**What You DON'T Do:**
- ❌ Write SKILL.md files
- ❌ Install libraries or dependencies
- ❌ Modify tool configurations
- ❌ Execute arbitrary code
- ❌ Build Docker containers

---

### **CTO Agent**

**What You Do:**
- ✅ Receive skill requests from business agents
- ✅ Design skills following OpenClaw AgentSkills spec
- ✅ Create SKILL.md with proper YAML frontmatter
- ✅ Build Docker containers for dependencies
- ✅ Install libraries in isolated environments
- ✅ Test skills before delivery
- ✅ Document usage for business agents
- ✅ Maintain and update skills

**What You DON't Do:**
- ❌ Execute skills for business tasks (let business agents do that)
- ❌ Make business decisions about which tools to prioritize

---

## Skill Request Template

Business agents should use this format when requesting tools:

```markdown
## Skill Request: {Tool Name}

**Requesting Agent:** {Your Agent ID} (e.g., wembassy-cmo)
**Priority:** High/Medium/Low
**Target Completion:** {Date}

### Purpose
Brief description of what this tool should do.

### Requirements
- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

### Inputs
- Input 1: Description (e.g., LinkedIn profile URL)
- Input 2: Description (e.g., Search keywords)

### Outputs
- Output 1: Description (e.g., JSON with profile data)
- Output 2: Description (e.g., CSV export)

### Dependencies (if known)
- Library 1: Purpose
- Library 2: Purpose
- Or: "Unknown - please recommend"

### Security Considerations
- Are API keys needed?
- Are there rate limits?
- Any privacy concerns?

### Acceptance Criteria
1. [ ] Criterion 1
2. [ ] Criterion 2
3. [ ] Criterion 3

### Example Usage
```
{ToolName} --input "example" --output format
```
```

## Example Skill: LinkedIn Profile Scraper

### Request from CMO

```markdown
## Skill Request: LinkedIn Profile Scraper

**Requesting Agent:** wembassy-cmo
**Priority:** High
**Target Completion:** 2026-02-26

### Purpose
Extract contact information from LinkedIn profiles for prospecting.

### Requirements
- Accept LinkedIn profile URL as input
- Extract: Name, Title, Company, Location, Email (if available)
- Export to CSV or JSON
- Respect rate limits (not get us banned)

### Inputs
- linkedin_url: Full LinkedIn profile URL
- output_format: "csv" or "json"

### Outputs
- structured_data: Contact information
- status: Success/error message

### Dependencies
- puppeteer/playwright for browser automation
- csv-parser for export

### Security Considerations
- Requires LinkedIn credentials (stored securely)
- Rate limiting to avoid bans
- Only public profile data

### Acceptance Criteria
1. Successfully scrapes 10 test profiles
2. Handles errors gracefully
3. Exports clean CSV format
4. Respects rate limits (5 profiles/minute)

### Example Usage
```
linkedin-scraper --url "https://linkedin.com/in/johndoe" --format csv
```
```

### CTO Development Response

CTO creates:

1. **SKILL.md** at `~/.openclaw/skills/linkedin-scraper/SKILL.md`
2. **Dockerfile** at `~/.openclaw/skills/linkedin-scraper/Dockerfile`
3. **Implementation Script** at `~/.openclaw/skills/linkedin-scraper/linkedin-scraper.js`
4. **Configuration** in `~/.openclaw/skills/linkedin-scraper/config.json`

**SKILL.md Structure:**

```yaml
---
name: linkedin-scraper
description: Extract contact information from LinkedIn profiles
metadata:
  category: prospecting
  author: cto-agent
  version: 1.0.0
  docker:
    image: openclaw/skills/linkedin-scraper:latest
    container_name: linkedin-scraper
    volumes:
      - {baseDir}/data:/data
    env:
      LINKEDIN_EMAIL: ${LINKEDIN_EMAIL}
      LINKEDIN_PASSWORD: ${LINKEDIN_PASSWORD}
---

# LinkedIn Profile Scraper

## Description
Extracts contact information from LinkedIn profiles for sales prospecting.

## Installation
Docker image includes all dependencies.

## Usage
```bash
linkedin-scraper --url <profile_url> --format <csv|json>
```

## Options
- `--url`: LinkedIn profile URL (required)
- `--format`: Output format - csv or json (default: json)
- `--rate-limit`: Seconds between requests (default: 12)

## Example
```bash
linkedin-scraper --url "https://linkedin.com/in/johndoe" --format csv
```

## Security Notes
- Requires LinkedIn credentials stored in environment
- Rate limited to avoid account bans
- Only extracts publicly visible data
```

**Dockerfile Example:**

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install Playwright dependencies
RUN apk add --no-cache \
    chromium \
    nss \
    freetype \
    harfbuzz \
    ca-certificates \
    ttf-freefont

# Install Node dependencies
COPY package.json .
RUN npm install

# Copy skill code
COPY . .

# Set environment
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser

CMD ["node", "linkedin-scraper.js"]
```

---

## Skill Locations

### Development (CTO Only)
```
~/.openclaw/skills/{skill-name}/
├── SKILL.md           (Configuration & documentation)
├── Dockerfile         (Container definition)
├── package.json       (Node dependencies)
├── requirements.txt   (Python dependencies - if needed)
├── src/               (Source code)
└── tests/             (Test cases)
```

### Deployment
- Skills are saved to the shared `~/.openclaw/skills` directory
- All agents can use deployed skills (read-only)
- Only CTO modifies skill code
- Docker containers are ephemeral (created per-use)

---

## Docker Requirements for Skills

Every skill with dependencies MUST include a Dockerfile:

**Minimum Requirements:**
- Base image (e.g., `node:18-alpine`, `python:3.11-slim`)
- All runtime dependencies installed
- Skill code copied into container
- Working directory set
- Entrypoint defined

**Security Requirements:**
- No root execution (use `USER node` or `USER 1000`)
- Minimal base image (Alpine preferred)
- No secrets in image (pass via env vars)
- Read-only filesystem where possible
- Network restrictions (if applicable)

**Example Structure:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
USER node
CMD ["node", "index.js"]
```

---

## Communication Protocol

### Requesting a Skill (CMO → CTO)

```javascript
// CMO sends request
sessions_send({
  sessionKey: "agent:wembassy-cto",
  message: `# Skill Request: LinkedIn Profile Scraper

**Requesting Agent:** wembassy-cmo
**Priority:** High

### Purpose
Extract contact info from LinkedIn for prospecting.

### Requirements
- Scrape profile: name, title, company, location
- Export to CSV
- Rate limit to avoid bans

### Dependencies
- Playwright/Puppeteer for browser automation

Please confirm feasibility and timeline.`
});
```

### CTO Response

```javascript
// CTO responds
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `## Skill Request Approved: LinkedIn Scraper

**Status:** ✅ Approved
**Timeline:** 3 days (by 2026-02-22)
**Complexity:** Medium

### Technical Approach
- Use Playwright with stealth mode
- Docker container with headless Chromium
- Rate limit: 12 seconds between requests
- Credentials via environment variables

### Deliverables
1. SKILL.md with usage docs
2. Docker container
3. Test suite (10 profiles)
4. Documentation for CMO usage

I'll update you when ready for testing.`
});
```

---

## Current Skill Pipeline

| Skill | Status | Requester | Priority | Due |
|-------|--------|-----------|----------|-----|
| LinkedIn Scraper | 🔄 In Progress | CMO | High | Feb 22 |
| Proposal Generator | ⏳ Queued | CRO | Medium | Feb 26 |
| CRM Sync Tool | ⏳ Queued | COO | Low | Mar 1 |

---

## Best Practices

### For Requesters (CMO/CRO/COO):
1. Be specific about requirements
2. Provide example inputs/outputs
3. Define clear acceptance criteria
4. Test thoroughly before production use
5. Report bugs to CTO, don't fix yourself

### For Developer (CTO):
1. Use semantic versioning
2. Write clear documentation
3. Include error handling
4. Add rate limiting where needed
5. Test in isolated environment
6. Review for security before delivery

### Security:
1. No credentials in code
2. Use Docker secrets or env vars
3. Rate limit external API calls
4. Log but don't expose sensitive data
5. Regular dependency updates

---

## Questions?

Refer to:
- [OpenClaw Skills Documentation](https://docs.openclaw.ai/tools/skills)
- [AgentSkills Specification](https://agentskills.io)
- [ClawHub Registry](https://clawhub.com)
- [Docker Security Best Practices](/gateway/security)

---

**Version:** 1.0  
**Last Updated:** 2026-02-19  
**Owner:** CTO Agent
