---
name: agent-org-manager
slug: agent-org-manager
version: 1.0.0
displayName: "Agent Organization Manager"
summary: "Multi-department agent org setup with hierarchy, comms, reporting, learning, and audit systems."
description: "Multi-department agent org setup: hierarchy, comms, reporting, learning, audit. Triggers: 'agent team', 'agent organization', 'agent roster'"
tags:
  - agent
  - organization
  - multi-agent
  - cron
  - management
license: MIT
---

# Agent Organization Manager

Build a multi-department AI agent company with clear hierarchy, communication, and accountability.

## Core Concept

One agent = one department. Each department has a role, a communication channel, and accountability. The CEO agent (main) orchestrates; department agents execute.

## Step 1: Create ROSTER.md

Create `company/shared/ROSTER.md` — the company directory:

```markdown
# Company Roster

| Department | Agent ID | Role | Priority |
|------------|----------|------|----------|
| CEO | main | Strategy, decisions, orchestration | P0 |
| Admin | admin | Operations, scheduling, logistics | P0 |
| Data | datacenter | Infrastructure, data, monitoring | P0 |
| Inspector | inspector | Quality audit, compliance checks | P0 |
| Legal | legal | Compliance, risk review | P1 |
| Finance | finance | Cost tracking, token budgets | P1 |
| Brand | brand | Content, marketing, publishing | P1 |
| Sales | sales | Outreach, conversion, revenue | P1 |
```

## Step 2: Communication Protocol

### Primary: sessions_send (real-time)
```
sessions_send agentId="admin" message="@admin Assign today's tasks"
```

### Async: Shared message files
Write to `company/shared/messages/MSG-{from}-{to}-{timestamp}.md`

### Alerts: Shared alert files
Write to `company/shared/alerts/ALERT-{level}-{timestamp}.md`
Levels: INFO / WARN / FLAG / KILL_SWITCH

### Broadcast: All-department
Write to `company/shared/broadcast/BC-{timestamp}.md`

### Routing Rules
- Unknown recipient → admin
- Compliance → legal
- Violations → inspector
- Data requests → datacenter
- Budget → finance

## Step 3: Daily Report System

Each department reports daily via cron. Use `sessions_send` to main (not `message` tool — it may fail in isolated sessions).

Cron payload pattern:
```json
{
  "kind": "agentTurn",
  "message": "You are [dept]. Daily report: list today's completed work, pending items, next steps. Send sessions_send to main. Do NOT use message tool.",
  "model": "zai/glm-4-flash",
  "lightContext": true
}
```

Stagger report times (e.g., 22:00, 22:03, 22:06...) to avoid rate limits.

## Step 4: Learning System

Each department learns daily in its domain. Stagger morning times (e.g., 07:30-08:30).

```json
{
  "name": "Learning · [Dept]",
  "schedule": {"kind": "cron", "expr": "30 7 * * *", "tz": "Asia/Shanghai"},
  "payload": {
    "kind": "agentTurn",
    "message": "You are [dept]. Search 1-2 trends in [domain]. Write ~300 words to company/departments/[dept]/learning_YYYYMMDD.md.",
    "model": "zai/glm-4-flash",
    "lightContext": true
  }
}
```

## Step 5: Inspection System

Inspector agent audits all departments on schedule (e.g., 10:00, 14:00, 22:00).

Checks:
- Are daily reports filed?
- Are learning notes written?
- Are published outputs on schedule?
- Flag anomalies, escalate consecutive failures

## Step 6: Communication Anti-Patterns

**Never do this:**
- Single point of communication (one channel = one failure = total blackout)
- Direct message tool in isolated sessions (may lack auth config)
- Cross-department file writes (causes data pollution)

**Always do this:**
- Multiple parallel communication paths (sessions_send + shared files + cron)
- Department workspaces stay isolated
- Shared info goes through designated channels only

## File Structure

```
company/
├── shared/
│   ├── ROSTER.md
│   ├── messages/
│   ├── alerts/
│   └── broadcast/
└── departments/
    ├── admin/
    ├── datacenter/
    ├── inspector/
    ├── legal/
    ├── finance/
    ├── brand/
    └── sales/
```

## References

- See [references/communication-patterns.md](references/communication-patterns.md) for detailed inter-agent protocols
- See [references/cron-templates.md](references/cron-templates.md) for ready-to-use cron configurations
