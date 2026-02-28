# Skill Development Guidelines for CTO

## AgentSkills Specification

All skills MUST follow the OpenClaw AgentSkills specification. Each skill is a directory containing:

```
skills/{skill-name}/
├── SKILL.md           # Required: Configuration + documentation
├── Dockerfile         # Required if external dependencies
├── package.json       # Node.js dependencies
├── requirements.txt   # Python dependencies (if applicable)
├── src/               # Source code
└── tests/             # Test files
```

## SKILL.md Format

```yaml
---
name: skill-name
description: Brief description of what the skill does
metadata:
  category: prospecting | development | automation | analysis
  author: cto-agent
  version: 1.0.0
  requires:
    bins: ["node", "docker"]
  docker:
    image: openclaw/skills/{skill-name}:{version}
    container_name: {skill-name}
    env:
      API_KEY: ${API_KEY}
    volumes:
      - {baseDir}/data:/data
---

# Skill Name

## Description
Full description of the skill's purpose.

## Installation
How to install/setup (usually just docker build).

## Usage
```bash
{skill-name} [options] [arguments]
```

## Options
- `--option`: Description

## Examples
Detailed usage examples.

## Security Notes
Security considerations and best practices.

## Error Handling
Expected errors and how to handle them.

## Changelog
### 1.0.0
- Initial release
```

## Docker Requirements

### Dockerfile Template

```dockerfile
# Node.js based skill
FROM node:18-alpine

# Install system dependencies
RUN apk add --no-cache \
    chromium \
    nss \
    freetype \
    harfbuzz \
    ca-certificates \
    ttf-freefont

# Set working directory
WORKDIR /app

# Install dependencies first (for caching)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Create non-root user
USER node

# Set environment
ENV NODE_ENV=production
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser

# Run skill
CMD ["node", "index.js"]
```

### Security Checklist

- [ ] Use specific base image version (not `latest`)
- [ ] Run as non-root user
- [ ] Install only required dependencies
- [ ] No secrets in Dockerfile or image
- [ ] Read-only filesystem where possible
- [ ] Minimal exposed ports
- [ ] Health check endpoint

## Skill Categories

| Category | Use Case | Examples |
|----------|----------|----------|
| `prospecting` | Lead generation | linkedin-scraper, email-finder |
| `development` | Coding assistance | code-review, security-audit |
| `automation` | Workflow automation | marketing-automation-sync, freshbooks-export |
| `analysis` | Data processing | competitor-analysis, seo-audit |

## Development Workflow

### 1. Receive Request
- Business agent submits skill request via sessions_send
- Review requirements for feasibility
- Confirm complexity and timeline

### 2. Design Skill
- Define inputs/outputs
- Select technology stack
- Design Docker container
- Plan error handling

### 3. Develop
- Create skill directory
- Write SKILL.md
- Implement main functionality
- Add Dockerfile
- Write tests

### 4. Test
- Run unit tests
- Test in isolated container
- Validate against requirements
- Check security

### 5. Deliver
- Notify requesting agent
- Provide usage documentation
- Schedule training (if needed)
- Add to skill registry

### 6. Maintain
- Monitor for issues
- Update dependencies
- Add features as requested
- Deprecate if obsolete

## Testing Standards

```javascript
// Test structure
const { skill } = require('../src/index');

describe('Skill Tests', () => {
  test('processes valid input', async () => {
    const result = await skill.run({
      input: 'valid-data',
      format: 'json'
    });
    expect(result.success).toBe(true);
    expect(result.data).toBeDefined();
  });
  
  test('handles invalid input', async () => {
    const result = await skill.run({
      input: 'invalid-data'
    });
    expect(result.success).toBe(false);
    expect(result.error).toBeDefined();
  });
  
  test('respects rate limits', async () => {
    const start = Date.now();
    await skill.run({ url: 'test-url-1' });
    await skill.run({ url: 'test-url-2' });
    const elapsed = Date.now() - start;
    expect(elapsed).toBeGreaterThan(12000); // 12 second min
  });
});
```

## Deployment

### Local Development
```bash
cd ~/.openclaw/skills/{skill-name}
docker build -t openclaw/skills/{skill-name}:dev .
docker run -it --rm openclaw/skills/{skill-name}:dev
```

### Production Build
```bash
# Build with proper tagging
docker build -t openclaw/skills/{skill-name}:1.0.0 .

# Test in isolated container
docker run --rm \
  -e API_KEY=$API_KEY \
  -v $(pwd)/data:/data \
  openclaw/skills/{skill-name}:1.0.0
```

### Versioning
- Follow semantic versioning (MAJOR.MINOR.PATCH)
- Tag Docker images with version
- Update SKILL.md version
- Maintain changelog

## Security Guidelines

### Handling Secrets
```javascript
// ✅ GOOD: Read from environment
const apiKey = process.env.API_KEY;

// ❌ BAD: Hardcoded secret
const apiKey = "sk-1234567890abcdef";
```

### Input Validation
```javascript
// ✅ GOOD: Validate and sanitize
function validateInput(url) {
  if (!url || typeof url !== 'string') {
    throw new Error('URL is required');
  }
  if (!url.match(/^https?:\/\/linkedin\.com\/in\/[\w-]+$/)) {
    throw new Error('Invalid LinkedIn URL format');
  }
  return url;
}

// ❌ BAD: Trust user input
fetch(userInputUrl); // Dangerous!
```

### Rate Limiting
```javascript
// ✅ GOOD: Implement delays
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

async function scrapeWithRateLimit(urls) {
  const results = [];
  for (const url of urls) {
    await delay(12000); // 12 seconds between requests
    results.push(await scrape(url));
  }
  return results;
}
```

## Documentation Standards

Every skill must include:

1. **Clear description** - What it does and why
2. **Installation steps** - How to set up
3. **Usage examples** - Command patterns
4. **Options reference** - All flags/parameters
5. **Security notes** - Potential risks
6. **Error codes** - What failures mean
7. **Changelog** - Version history

## Skill Registry

CTO maintains a registry of all skills:

| Skill | Version | Category | Status | Last Updated |
|-------|---------|----------|--------|--------------|
| linkedin-scraper | 1.0.0 | prospecting | active | 2026-02-19 |
| proposal-generator | - | - | queued | - |
| marketing-automation-sync | - | - | planned | - |

## Inter-Agent Communication

### Receiving Requests

```javascript
// CTO listens for skill requests
sessions_list({
  kinds: ['group', 'discord'],
  activeMinutes: 60
});

// Respond with timeline and approach
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `Skill Request Received: LinkedIn Scraper

**Feasibility:** ✅ Approved
**Complexity:** Medium
**Timeline:** 3 days
**Approach:** Playwright + Docker

I'll notify you when ready for testing.`
});
```

### Delivering Skills

```javascript
// Notify requester skill is ready
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `✅ Skill Delivered: LinkedIn Scraper v1.0.0

**Location:** ~/.openclaw/skills/linkedin-scraper/
**Docker Image:** openclaw/skills/linkedin-scraper:1.0.0
**Documentation:** SKILL.md

### Quick Start
\`\`\`
linkedin-scraper --url "https://linkedin.com/in/profile" --format csv
\`\`\`

### Test Results
- ✅ 10/10 profiles scraped successfully
- ✅ Rate limiting working (12s delays)
- ✅ CSV export validated
- ✅ Errors handled gracefully

Ready for your testing!`
});
```

## Maintenance

### Regular Tasks

**Weekly:**
- Review skill usage logs
- Check for errors or failures
- Update outdated dependencies

**Monthly:**
- Security audit of skill code
- Performance review
- Dependency updates
- Documentation refresh

**Quarterly:**
- Full skill review
- Deprecate unused skills
- Archive old versions
- Plan new skill roadmap

## Resources

- [AgentSkills Specification](https://agentskills.io)
- [OpenClaw Skills Docs](https://docs.openclaw.ai/tools/skills)
- [ClawHub Registry](https://clawhub.com)
- [Docker Security](https://docs.docker.com/develop/security-best-practices/)

---

**Remember:** Business agents focus on using skills. You focus on building them securely and reliably.
