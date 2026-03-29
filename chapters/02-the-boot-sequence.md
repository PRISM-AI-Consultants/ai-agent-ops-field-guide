# Chapter 2: The Boot Sequence

## The Mandatory Startup Routine

Every agent in the fleet executes the same boot sequence before doing any work. No exceptions. An agent that skips steps will eventually cause an incident. We learned this after agent number 12 sent a follow-up email to a client who had already been contacted 20 minutes earlier by agent number 7.

## The Sequence

```python
def boot_agent(agent_name: str) -> AgentContext:
    """
    Mandatory boot sequence. Every agent. Every run. No shortcuts.
    """

    # Step 1: Load global context
    # The master document that every agent shares.
    # Contains business rules, pricing, communication style, hard constraints.
    global_ctx = load_file("shared/global_context.md")

    # Step 2: Load agent-specific briefing
    # What THIS agent does, its scope, its rules, its known failure modes.
    agent_brief = load_file(f"agents/{agent_name}/briefing.md")

    # Step 3: Check fleet status
    # Who ran recently? What were exit codes? Any agents in error state?
    fleet_status = load_json("shared/fleet_status.json")
    if fleet_status.get("blocked_agents", []):
        log.warning(f"Blocked agents detected: {fleet_status['blocked_agents']}")

    # Step 4: Read the failures log
    # What broke recently? What was the root cause? What was the fix?
    # This prevents repeating mistakes that were already diagnosed.
    failures = load_file("shared/failures_log.md")

    # Step 5: Load maintenance history
    # What was fixed, when, and how. Verification checklists.
    maintenance = load_file("shared/bot_maintenance.md")

    # Step 6: Query semantic memory
    # Cross-session context from the vector database.
    # Recent decisions, unfinished work, lessons learned.
    memory_results = query_memory(
        query=f"{agent_name} recent context and decisions",
        n_results=5
    )

    # Step 7: Load metrics (if applicable)
    # Agent-specific performance data: success rates, error counts, timing.
    metrics = load_metrics(agent_name) if has_metrics(agent_name) else None

    return AgentContext(
        global_ctx=global_ctx,
        agent_brief=agent_brief,
        fleet_status=fleet_status,
        failures=failures,
        maintenance=maintenance,
        memory=memory_results,
        metrics=metrics
    )
```

## Why Each Step Matters

| Step | What Happens If You Skip It |
|------|-----------------------------|
| Global context | Agent violates business rules. Sends wrong pricing. Uses wrong tone. |
| Agent briefing | Agent operates outside its scope. Does work that belongs to another agent. |
| Fleet status | Agent duplicates work another agent just completed. |
| Failures log | Agent repeats a mistake that was fixed yesterday. |
| Maintenance history | Agent undoes a fix that was applied in the last maintenance window. |
| Semantic memory | Agent has amnesia. No awareness of cross-session decisions. |
| Metrics | Agent cannot self-assess. No awareness of its own error rate. |

## Implementation Notes

The boot sequence should be a shared library, not copy-pasted into each agent. When we had it duplicated across 34 agents, step 4 (failures log) got dropped from 6 agents during a refactor. Those 6 agents repeated the same email formatting bug for two weeks before anyone noticed.

**One function. Imported everywhere. Tested once.**

The boot sequence adds 2-4 seconds of latency per agent run. That is a small price for fleet-level awareness. The alternative is spending hours debugging incidents caused by agents that did not read the room.
