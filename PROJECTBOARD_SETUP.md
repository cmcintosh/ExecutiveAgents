# ${PROJECTBOARD_NAME} Agent User Setup

## Overview

Create dedicated ${PROJECTBOARD_NAME} user accounts for each agent to enable proper task attribution, workload tracking, and time logging.

## User Accounts to Create

**Email Strategy:** Using plus-addressing (`+` aliases) on the COO email for cost-free attribution. All emails route to the COO inbox.

| Agent | ${PROJECTBOARD_NAME} Username | Email | Role | Permissions |
|-------|---------------------|-------|------|-------------|
| **Wren (COO)** | `wren-coo` | ${EMAIL_WREN} | Project Admin | Full access, can assign to others |
| **Mason (CMO)** | `mason-cmo` | ${EMAIL_ALIAS_CMO} | Member | Create tasks, self-assign, log time |
| **Grayson (CRO)** | `grayson-cro` | ${EMAIL_ALIAS_CRO} | Member | Create tasks, self-assign, log time |
| **Spock (CTO)** | `spock-cto` | ${EMAIL_ALIAS_CTO} | Developer | Technical tasks, code reviews, deploy |
| **Mary (HR)** | `mary-hr` | ${EMAIL_ALIAS_HR} | Member | Team tasks, contractor coordination |
| **Dave (RM)** | `dave-rm` | ${EMAIL_ALIAS_RM} | Member | Private family office tasks only |

**Why `+` email trick:**
- All emails go to ${EMAIL_WREN} inbox
- $0 extra Google Workspace cost
- Gmail ignores everything after `+`
- Still allows proper ${PROJECTBOARD_NAME} attribution
- Easy filtering in Gmail by alias address

## Setup Instructions

### Step 1: Create Users in ${PROJECTBOARD_NAME}

1. Login to ${PROJECTBOARD_URL} as admin
2. Go to **Administration → Users**
3. Create each user:

```
User: wren-coo
Email: ${EMAIL_WREN}
First Name: Wren
Last Name: COO
Password: [Generate strong password]
Authentication: Password
```

Repeat for:
- mason-cmo, grayson-cro, spock-cto, mary-hr, dave-rm

### Step 2: Set Roles

Assign roles per project:

| Project | Users | Role |
|---------|-------|------|
| Wembassy Operations | wren-coo | Project Admin |
| | mason-cmo, grayson-cro, mary-hr | Member |
| | spock-cto | Developer |
| ${FAMILY_OFFICE_NAME} | wren-coo | Project Admin |
| | dave-rm | Member (private) |

### Step 3: Generate API Keys

1. Login as each user
2. Go to **My Account → Access Tokens**
3. Generate new API key
4. Copy and store securely

**Note:** Each agent needs its own API key for proper attribution.

## Agent Configuration

### Update agents.json

```json
{
  "id": "wembassy-cmo",
  "env": {
    "PROJECTBOARD_USERNAME": "mason-cmo",
    "PROJECTBOARD_API_KEY": "${PROJECTBOARD_API_KEY_CMO}"
  }
},
{
  "id": "wembassy-cro", 
  "env": {
    "PROJECTBOARD_USERNAME": "grayson-cro",
    "PROJECTBOARD_API_KEY": "${PROJECTBOARD_API_KEY_CRO}"
  }
},
{
  "id": "wembassy-cto",
  "env": {
    "PROJECTBOARD_USERNAME": "spock-cto",
    "PROJECTBOARD_API_KEY": "${PROJECTBOARD_API_KEY_CTO}"
  }
},
{
  "id": "wembassy-hr",
  "env": {
    "PROJECTBOARD_USERNAME": "mary-hr",
    "PROJECTBOARD_API_KEY": "${PROJECTBOARD_API_KEY_HR}"
  }
},
{
  "id": "mcintosh-rm",
  "env": {
    "PROJECTBOARD_USERNAME": "dave-rm",
    "PROJECTBOARD_API_KEY": "${PROJECTBOARD_API_KEY_RM}"
  }
}
```

### Update .env with API Keys

```bash
# ${PROJECTBOARD_NAME} API Keys per agent
PROJECTBOARD_API_KEY_WREN=your_key_here
PROJECTBOARD_API_KEY_CMO=your_key_here
PROJECTBOARD_API_KEY_CRO=your_key_here
PROJECTBOARD_API_KEY_CTO=your_key_here
PROJECTBOARD_API_KEY_HR=your_key_here
PROJECTBOARD_API_KEY_RM=your_key_here
```

## Task Assignment Workflow

### Current (Without Agent Users)
```
1. Wren creates task in ${PROJECTBOARD_NAME}
2. Assigns to self or human
3. Wren delegates via Discord
4. Who actually did it? Unclear
```

### New (With Agent Users)
```
1. Wren creates task
2. Assigns to mason-cmo (CMO agent)
3. Mason (CMO) works task, updates status
4. "Assigned to: mason-cmo" "Updated by: mason-cmo"
5. Time logged under mason-cmo
```

## Benefits

### 1. Clear Attribution
- See which agent did what work
- No confusion about task ownership
- Audit trail per agent

### 2. Workload Balancing
- View each agent's task queue
- Balance work across agents
- Identify bottlenecks

### 3. Time Tracking
```
Project: Wembassy Operations
Week: Feb 17-23, 2026

Agent          | Hours | Tasks
---------------|-------|------
mason-cmo      | 12.5  | 8
grayson-cro    | 8.0   | 5
spock-cto      | 15.0  | 3
wren-coo       | 6.0   | 12
```

### 4. Permission Granularity
- CMO can't see CTO's sensitive tasks
- RM (Dave) only sees family office project
- Proper access control

### 5. Activity Tracking
```
Task #2043: Family Office Automation Post
- Created: Wren (wren-coo)
- Assigned: CMO (mason-cmo)
- In Progress: Mason (2026-02-19 08:00)
- Time Logged: 2.5 hours by mason-cmo
- Completed: Mason (2026-02-19 10:30)
```

## Inter-Agent Task Assignment

### Wren → CMO
```javascript
// Wren creates task for CMO
sessions_send({
  sessionKey: "agent:wembassy-cmo",
  message: `## New Task Assigned

**Project:** Wembassy Operations  
**Task ID:** #2050  
**Subject:** Review sales letter copy  
**Assigned to:** mason-cmo (You)

### Description:
Review and approve the ${COMPANY_LOCATION_SHORT} Contractor sales letter before deployment.

### Acceptance Criteria:
- [ ] Review sales letter
- [ ] Approve or request revisions
- [ ] Confirm ready for LinkedIn ad test

### Deadline: 2026-02-20 EOD

View in ${PROJECTBOARD_NAME}: ${PROJECTBOARD_URL}/work_packages/2050`
});
```

### CMO Updates Task
```javascript
// CMO updates task when complete
sessions_send({
  sessionKey: "agent:wembassy-coo-wren",
  message: `## Task Complete

**Task:** #2050 - Review sales letter copy  
**Status:** Completed  
**Time:** 1.5 hours  
**Agent:** mason-cmo

### Summary:
Reviewed and approved sales letter. Minor tweaks to call-to-action. Ready for deployment.`
});

// CMO also updates ${PROJECTBOARD_NAME} via skill
```

## Reporting

### Weekly Agent Report
```
Wembassy Agent Activity - Week of 2026-02-17

┌──────────────┬────────┬───────────┬──────────┐
│ Agent        │ Tasks  │ Time (hrs)│ Complete │
├──────────────┼────────┼───────────┼──────────┤
│ mason-cmo    │ 12     │ 15.5      │ 10       │
│ grayson-cro  │ 8      │ 12.0      │ 7        │
│ spock-cto    │ 5      │ 18.0      │ 4        │
│ mary-hr      │ 6      │ 8.0       │ 6        │
│ wren-coo     │ 25     │ 10.0      │ 24       │
│ dave-rm      │ 3      │ 4.0       │ 3        │
└──────────────┴────────┴───────────┴──────────┘
```

## Integration with Existing Workflow

### Task Creation
```javascript
// Wren creates task via skill (as wren-coo)
// Currently uses admin account, now uses wren-coo credentials
```

### Task Updates
```javascript
// CMO updates task (as mason-cmo)
// API call authenticated as mason-cmo
// Activity shows "Updated by: mason-cmo"
```

## Implementation Steps

1. **Create users in ${PROJECTBOARD_NAME}** (as admin)
2. **Set roles per project**
3. **Generate API keys for each**
4. **Update .env with keys**
5. **Update agents.json with env vars**
6. **Test task creation/updates**
7. **Document for team**

## Security Considerations

- Store API keys in .env, not in code
- Rotate keys quarterly
- Give minimal permissions needed
- RM user only accesses family office project
- Review agent activity monthly

## Troubleshooting

### "Authentication failed"
- Check API key is correct
- Verify user exists in ${PROJECTBOARD_NAME}
- Check user has access to project

### "Permission denied"
- Verify role assignment in project
- Check API key hasn't expired
- Review user permissions

### Task shows wrong user
- Check API key matches correct agent
- Verify .env variable names
- Look for credentials mix-up

---

**Recommendation:** Create agent users before server deployment. It's the foundation for proper multi-agent task tracking.
