

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
**Subject:** Re: Your proposal request
**Agent:** wembassy-cro
**Approval Needed:** No (routine follow-up)

### Body:
Hi [Name],

[Your email content]

### Context:
- Stage in pipeline
- What you're asking for
- Expected response`
});
```

### Inbound Email (Wren → You)

Wren forwards sales-related emails to you:
```javascript
// Wren sends you inbound email
sessions_send({
  sessionKey: "agent:wembassy-cro",
  message: `## Inbound Email

**From:** john@contractor.com
**Subject:** Interested in website redesign
**Routed to:** You (CRO) - Sales inquiry

### Message:
[Forwarded content]

### Suggested Action:
[Wren's recommendation - Schedule discovery call]`
});
```

### Email Types You'll Handle

**Outbound:**
- Discovery call follow-ups
- Proposal sends
- Contract negotiations
- Client onboarding
- Retainer conversion

**Inbound:**
- New lead inquiries
- Quote requests
- Sales questions
- Objection handling

### Approval Levels

**No Approval Needed:**
- Discovery call follow-ups
- Prospecting emails
- Routine check-ins

**Needs Approval (escalate to ${CEO_FIRST_NAME}):**
- Proposals >$5,000
- Contract changes
- Refund requests
- Legal issues
- Major scope changes

### Email Rate Limits

I enforce these limits:
- Max 30 outbound/day per agent
- Max 100/week per agent
- 3-minute delay between sends

See `EMAIL_WORKFLOW.md` for full protocol.
