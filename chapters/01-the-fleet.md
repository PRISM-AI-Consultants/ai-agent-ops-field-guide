# Chapter 1: The Fleet

## Multi-Agent Architecture Patterns

A fleet of 34 agents is not 34 independent scripts. It is a system with communication channels, shared state, dependency chains, and emergent failure modes that no single agent produces on its own.

## Agent Categories

Our fleet breaks into six functional categories:

| Category | Count | Examples |
|----------|-------|----------|
| Email Ops | 6 | Inbox triage, draft generation, send verification, bounce monitoring |
| Session Intelligence | 5 | Transcript processing, recap generation, deliverable extraction |
| CRM Sync | 4 | Pipeline updates, client file maintenance, tier classification |
| Content Generation | 8 | Social posts, video production, QA scoring, thumbnail generation |
| Infrastructure | 5 | Security scanning, disk monitoring, SSL renewal, DNS checks |
| Research / Scheduling | 6 | Conference research, calendar sync, lead enrichment |

## Communication Patterns

### The Briefing Room
Every agent reads a shared context file at boot. This file contains fleet-wide state: who is active, what failed recently, what changed. Think of it as a bulletin board that every agent checks before starting work.

```
shared/
  fleet_status.json      # Which agents ran, when, exit codes
  failures_log.md        # Recent failures with root causes
  bot_maintenance.md     # Fix history and verification checklists
```

### The Server Room Boot Sequence
Agents do not freelance. Every agent follows a mandatory startup routine (see Chapter 2) that loads global context, agent-specific briefing, fleet status, and the failures log. An agent that skips this sequence will duplicate work, contradict recent fixes, or break things that were just repaired.

### Shared State Files
Agents communicate through files, not direct calls. One agent writes a client summary. Another reads it before generating a recap. A third reads the recap before drafting a follow-up email. The dependency chain is implicit, which means ordering matters and race conditions are real.

```
Agent A (Transcript Processor) --> writes client_session.json
Agent B (Recap Generator)      --> reads client_session.json, writes recap_draft.md
Agent C (Email Drafter)         --> reads recap_draft.md, creates Gmail draft
```

### Semantic Memory
A vector database (REST API, ChromaDB backend) stores cross-session context. Agents query it at boot to load recent decisions, unfinished work, and lessons learned. This prevents the "amnesia problem" where an agent repeats a mistake that was already diagnosed and fixed in a prior session.

```python
# Every agent does this at startup
response = requests.post("http://memory-service:8000/api/search", json={
    "query": "recent failures and fixes",
    "n_results": 5
})
context = response.json()
```

## The Key Insight

The architecture is not about making agents smart. It is about making agents aware. An individually brilliant agent that does not know what the rest of the fleet just did is more dangerous than a simple agent that reads the room first.

Fleet-level awareness beats agent-level intelligence every time. Design for coordination, not capability.
