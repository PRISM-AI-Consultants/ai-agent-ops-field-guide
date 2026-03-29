# Chapter 9: Cost Governance

## API Budget Management

Running 34 agents with LLM calls, email APIs, CRM APIs, image generation, and vector database queries adds up. Without governance, costs drift upward until someone notices a bill that is 10x the expected amount (see War Story #3).

## The Cost Stack

Map every external API call in your fleet:

| Service | Usage Pattern | Cost Model | Monthly Budget |
|---------|--------------|------------|----------------|
| LLM (generation) | Every agent run | Per-token | Set ceiling per agent |
| LLM (grading/QA) | Intelligence Loop | Per-token | Included in agent ceiling |
| Email API | Triage + send | Free tier, rate limited | Monitor rate limits |
| Image generation | Content pipeline | Per-image or free tier | Hard cap per day |
| Vector DB | Boot sequence queries | Self-hosted or per-query | Infrastructure cost |
| CRM API | Sync agents | Free tier or per-call | Monitor call volume |

## The Two-Tier Model

Split your fleet into cost tiers:

**Tier 1: Use the cheapest model that works.** For automated agents that run on schedules (triage, monitoring, CRM sync), use the fastest and cheapest available model. These agents run hundreds of times per week. Cost per run matters more than peak capability.

**Tier 2: Use the best model for human-facing work.** For interactive sessions, strategy work, and complex generation, use the most capable model. These runs are infrequent and high-value. Cost per run is irrelevant compared to output quality.

```python
def get_model_for_task(task_type: str) -> str:
    AUTOMATED_TASKS = ["triage", "monitoring", "sync", "cleanup", "scoring"]
    if task_type in AUTOMATED_TASKS:
        return "gemini-flash"  # Fast, cheap, good enough
    return "claude-opus"       # Best quality for human-facing work
```

## Per-Run Cost Ceilings

Every agent that calls a paid API should have a per-run cost estimate and ceiling:

```python
MAX_COST_PER_RUN = 5.00  # dollars

def check_cost_ceiling(estimated_items: int, cost_per_item: float) -> bool:
    estimated_cost = estimated_items * cost_per_item
    if estimated_cost > MAX_COST_PER_RUN:
        alert(f"Estimated cost ${estimated_cost:.2f} exceeds ceiling ${MAX_COST_PER_RUN}")
        return False
    return True
```

## The Human vs. AI Cost Comparison

Before adding any paid tool or API, run this comparison:

```
Human cost for this task:   $X/hr * Y hours = $Z
AI/tool cost for this task: $A/run * B runs/month = $C
Break-even:                 If C < Z, automate. If not, keep it manual.
```

This sounds obvious. In practice, teams automate tasks where the human cost is $2/month and the API cost is $15/month because automation feels like progress. It is not progress if it costs more.

## Subscription Audit

Monthly, review every active subscription and API key:

1. **Is it still used?** Check logs for the last 30 days. If no calls, cancel or pause.
2. **Is it right-sized?** Many APIs have tiered pricing. If you are on the $50/month plan but using $8 worth of calls, downgrade.
3. **Is there a free alternative?** Self-hosted models, free-tier APIs, and open-source tools can replace paid services for non-critical paths.

## Budget Alerts

Set alerts at three levels:
- **50% of monthly budget:** Informational. On track.
- **75% of monthly budget:** Warning. Review what is driving usage.
- **90% of monthly budget:** Action required. Pause non-critical agents until reviewed.

The alert at 75% is the one that matters. By 90%, you are already over budget for the month. The 75% alert gives you a week to adjust.
