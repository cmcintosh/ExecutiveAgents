# Email Routing Workflow

## Overview

**Wren (COO) is the exclusive email handler.** All outbound email goes through ${EMAIL_WREN}. Agents request sends via `sessions_send`, and I route inbound emails to the appropriate agent based on content analysis.

## Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│   Agents    │────▶│ Wren (COO)  │────▶│  ${EMAIL_WREN}   │
│  (Request)  │     │  (Router)   │     │   (Outbound)     │
└─────────────┘     └──────┬──────┘     └─────────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  Inbound    │
                    │   Email     │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌─────────┐  ┌─────────┐  ┌─────────┐
        │   CMO   │  │   CRO   │  │   HR    │
        │ (Leads) │  │(Clients)│  │ (Team)  │
        └─────────┘  └─────────┘  └─────────┘
```

## Outbound Email Protocol

### Agents Request, Wren Sends

**Agents NEVER send email directly.** They request via `sessions_send`.

### Request Format

```javascript
// CMO requests email send
sessions_send({
  sessionKey: "agent:wembassy-coo-wren",
  message: `## Email Request

**From:** ${EMAIL_WREN}
**To:** prospect@company.com
**Subject:** Quick question about your website
**Agent:** wembassy-cmo
**Priority:** High
**Approval Needed:** No (routine outreach)

### Body:
Hi [Name],

I noticed your website at company.com while researching ${COMPANY_LOCATION_SHORT} contractors. Your portfolio looks impressive, but your site could be working harder for you.

Would you be open to a 15-minute call to discuss how we might be able to help you close more leads?

Best,
Wren
Wembassy - ${COMPANY_LOCATION_SHORT}'s First AI-Powered Web Agency

### Attachments:
- None

### Context:
- Prospect found via LinkedIn
- Website last updated 2019
- No SSL certificate
- Mobile experience poor`
});
```

### My Response Options

**Option 1: Send immediately**
```javascript
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `✅ Email Sent

**To:** prospect@company.com
**Subject:** Quick question about your website
**Message ID:** <abc123@wembassy.com>
**Sent:** 2026-02-19 12:53 CST

Email delivered successfully. I'll track for replies.`
});
```

**Option 2: Request changes**
```javascript
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `⚠️ Email Needs Revision

The tone is a bit aggressive. Please:
- Soften the opening
- Remove "could be working harder"
- Add specific value prop

Revised draft?`
});
```

**Option 3: Escalate to CEO**
```javascript
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `🚫 Approval Required

This email involves pricing discussion ($5K+). Per policy, ${CEO_FIRST_NAME} must approve before sending.

Forwarding to ${CEO_FIRST_NAME} for review...`
});

// Then I ping ${CEO_FIRST_NAME}
sessions_send({
  sessionKey: "agent:main",
  message: `Approval needed: CMO wants to send proposal quote to client. See forwarded request.`
});
```

---

## Inbound Email Routing

### How I Route Incoming Email

1. **Read** all email at ${EMAIL_WREN}
2. **Analyze** content for intent and sender
3. **Route** to appropriate agent:

| Email Type | Route To | Example |
|------------|----------|---------|
| Sales inquiry | CRO | "Interested in your services..." |
| Partnership | CMO | "Can we collaborate on..." |
| Contractor question | HR | "I have a question about my hours..." |
| Technical support | CTO | "Our site is down..." |
| General inquiry | CRO | "What do you charge?" |
| Family/personal | RM | Private matters |

### Routing Rules

```javascript
// In my routing logic
function routeEmail(email) {
  const subject = email.subject.toLowerCase();
  const body = email.body.toLowerCase();
  const from = email.from;
  
  // Sales-related
  if (subject.includes('quote') || subject.includes('proposal') ||
      body.includes('pricing') || body.includes('cost')) {
    return 'wembassy-cro';
  }
  
  // Marketing/partnership
  if (subject.includes('partnership') || subject.includes('collaboration') ||
      from.includes('marketing') || from.includes('media')) {
    return 'wembassy-cmo';
  }
  
  // Technical
  if (subject.includes('bug') || subject.includes('broken') ||
      body.includes('error') || body.includes('not working')) {
    return 'wembassy-cto';
  }
  
  // Team/contractor
  if (from.includes('${TEAM_FRONTEND_NAME}'.toLowerCase()) || from.includes('${TEAM_VA_NAME}'.toLowerCase()) ||
      from.includes('${TEAM_QA_NAME}'.toLowerCase()) || subject.includes('hours')) {
    return 'wembassy-hr';
  }
  
  // Family office (pattern match)
  if (subject.includes('trust') || subject.includes('property') ||
      from.includes('family')) {
    return 'mcintosh-rm';
  }
  
  // Default to sales
  return 'wembassy-cro';
}
```

### Forwarding Format

```javascript
// I forward to agent with context
sessions_send({
  sessionKey: "agent:wembassy-cro",
  message: `## Inbound Email

**From:** john@contractorco.com
**Subject:** Re: Website redesign quote
**Received:** 2026-02-19 12:53 CST
**Thread:** [View in Gmail]

### Message:
> Hi Wren,
> 
> Thanks for the quote. Can we do a call next week to discuss?
> 
> Best,
> John

### Thread History:
- Feb 18: You sent quote ($8,500)
- Feb 19: This reply

### Suggested Action:
Schedule discovery call via Calendly link: https://calendly.com/wembassy/30m-consultation

### My Recommendation:
Send Calendly link with availability options. Prospect is warm (active reply within 24h).`
});
```

---

## Email Types & Approval Levels

### No Approval Required (I send immediately)
- Routine prospecting emails
- Follow-ups on existing conversations
- Team coordination messages
- Marketing newsletter sends

### Approval Required (I escalate first)
- Proposals/contracts (>$200)
- Pricing changes
- Client complaints
- Legal/compliance matters
- Refund requests

### Never Send (I block)
- Spam
- Harassment
- Obviously phishing
- Unsubscribe requests (process via ${MARKETING_AUTOMATION_PLATFORM})

---

## Agent Quick Reference

### CMO - Mason (Marketing)
**Sends:**
- Prospecting emails
- Marketing campaigns
- Partnership outreach
- Content collaboration

**Receives:**
- Inbound marketing inquiries
- Partnership requests
- Media inquiries

### CRO - Grayson (Sales)
**Sends:**
- Discovery call follow-ups
- Proposals
- Contract negotiations
- Client onboarding

**Receives:**
- New lead inquiries
- Quote requests
- Client questions
- Sales objections

### CTO - Spock (Technical)
**Sends:**
- Scope clarifications
- Technical explanations
- Deployment notifications
- Development updates

**Receives:**
- Technical support requests
- Bug reports
- Integration questions

### HR - Mary (Team)
**Sends:**
- Team communications
- Payment confirmations
- Hiring coordination
- Policy updates

**Receives:**
- Contractor questions
- Team availability
- Payroll inquiries

### RM - Dave (Family Office)
**Sends:**
- Private family communications
- Trust documentation
- Asset management updates

**Receives:**
- Private/personal emails only
- (Strictly DM-based, never team channels)

---

## Templates for Agents

### Email Request Template
```markdown
## Email Request

**To:** recipient@email.com
**Subject:** [Subject line]
**Agent:** [Your agent ID]
**Priority:** High/Medium/Low
**Approval Needed:** Yes/No

### Body:
[Email body text]

### Attachments:
- [List any files]

### Context:
- [Why this email matters]
- [What response is expected]
- [Timeline if urgent]
```

### Reply Needed Template
```markdown
## Reply Needed: [Subject]

**From:** [Original sender]
**Original Date:** [When sent]
**Your Action:** [What response needed]
**Deadline:** [If time-sensitive]

### Original Message:
[Quoted email]

### Suggested Response:
[Draft from Wren, or "Please draft reply"]
```

---

## Logging & Tracking

I maintain an email log for all outbound:

```
2026-02-19 12:53 CST | OUTBOUND | to@company.com | Subject | Agent: CMO | Status: Sent
2026-02-19 13:05 CST | INBOUND  | from@client.com | Re: Quote | Routed to: CRO | Status: Replied
```

This ensures:
- Audit trail
- No duplicate sends
- Response time tracking
- Agent accountability

---

## Emergency Escalation

If I detect:
- Client threatening legal action
- Security breach reported
- Major service outage
- Payment dispute

I immediately escalate to both agent AND ${CEO_FIRST_NAME} (CEO) via DM.

---

## Summary

| Direction | Handler | Method |
|-----------|---------|--------|
| **Outbound** | Wren (COO) | Agents request via sessions_send |
| **Inbound** | Wren (COO) | I read, analyze, route to agents |
| **Urgent** | ${CEO_FIRST_NAME} (CEO) | I escalate per policy |

**Bottom line:** I'm the email router. Agents focus on their expertise. I handle the comms layer.
