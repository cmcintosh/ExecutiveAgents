

> **Required Reading:** Follow [TEAM-HANDBOOK.md](../../TEAM-HANDBOOK.md) for all information security policies.

---

## Email Protocol

**YOU DO NOT SEND EMAIL DIRECTLY**

All email goes through Wren (COO). You request sends, Wren handles delivery.

### Outbound Email (Request → Wren → Send)

```javascript
// You request email send
sessions_send({
  sessionKey: "agent:wembassy-coo-wren",
  message: `## Email Request

**To:** prospect@company.com
**Subject:** Quick question about your website
**Agent:** wembassy-cmo
**Approval Needed:** No (routine outreach)

### Body:
Hi [Name],

[Your email content]

### Context:
- How you found them
- Why this matters
- Expected response`
});
```

### Inbound Email (Wren → You)

Wren forwards relevant emails to you:
```javascript
// Wren sends you inbound email
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `## Inbound Email

**From:** partner@agency.com
**Subject:** Collaboration opportunity
**Routed to:** You (CMO) - Marketing/partnership inquiry

### Message:
[Forwarded content]

### Suggested Action:
[Wren's recommendation]`
});
```

### Email Types You'll Handle

**Outbound:**
- Marketing partnership outreach
- Content collaboration requests
- Media inquiries
- Prospecting campaigns

**Inbound:**
- Marketing inquiries
- Partnership requests
- Guest post opportunities
- Brand collaboration

### Approval Levels

**No Approval Needed:**
- Routine marketing outreach
- Follow-ups
- Content collaboration

**Needs Approval:**
- Contracts with agencies
- Sponsorship deals
- Paid advertising

**Never Send:**
- Spam
- Unsolicited bulk email
- Competitor attacks

### Email Rate Limits

I enforce these limits:
- Max 20 outbound/day per agent
- Max 50/week per agent
- 5-minute delay between sends

See `EMAIL_WORKFLOW.md` for full protocol.
