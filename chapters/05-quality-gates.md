# Chapter 5: Quality Gates

## The Intelligence Loop

Every piece of content our fleet produces passes through a four-stage quality loop: Generate, Grade, Fix, Verify. We call it the Intelligence Loop. Without it, agents ship mediocre output and no one catches it until a client does.

## The Pattern

```python
def intelligence_loop(task: dict, max_iterations: int = 3) -> dict:
    """
    Generate -> Grade -> Fix -> Verify
    Default score: 5/10. Threshold: 8/10.
    """
    output = generate(task)

    for attempt in range(max_iterations):
        score, feedback = grade(output, task["criteria"])

        if score >= 8:
            return {"output": output, "score": score, "attempts": attempt + 1}

        output = fix(output, feedback)

    # If we exhaust iterations, flag for human review
    return {"output": output, "score": score, "needs_review": True}
```

## Grading Rules

The grading agent must be strict. Our early mistake was a grader that gave 8/10 to everything. The fix was explicit calibration:

- **Default score is 5/10.** Not 7. Not 8. Five. The output must earn every point above baseline.
- **Misspellings or factual errors = automatic reject.** No partial credit.
- **Tone drift from the brief = reject.** If the task says "direct and concise" and the output is flowery, it fails.
- **Missing required elements = reject.** If a recap must include action items and they are absent, the score cannot exceed 4.

```python
def grade(output: str, criteria: dict) -> tuple[int, str]:
    score = 5  # Start at baseline, not at 10

    # Hard failures (automatic reject)
    if has_spelling_errors(output):
        return 0, "Spelling errors detected. Reject."
    if missing_required_fields(output, criteria["required_fields"]):
        return 3, f"Missing: {get_missing_fields(output, criteria)}"

    # Scoring adjustments
    if matches_tone(output, criteria["tone"]):
        score += 2
    if meets_length_target(output, criteria["length"]):
        score += 1
    if includes_specific_evidence(output):
        score += 2

    feedback = generate_feedback(output, criteria, score)
    return score, feedback
```

## Cooldown Timers

Agents that send notifications (Slack, email, SMS) must have cooldown periods. Without them, a monitoring agent that detects a flapping service will send 47 alerts in an hour.

```python
COOLDOWN_MINUTES = 30

def should_notify(event_key: str) -> bool:
    last_sent = get_last_notification_time(event_key)
    if last_sent and (now() - last_sent).minutes < COOLDOWN_MINUTES:
        return False
    record_notification(event_key, now())
    return True
```

## Deduplication

Before creating any output (email draft, social post, CRM update), check if an equivalent output already exists. The check is cheap. The duplicate is expensive.

```python
def is_duplicate(new_content: str, recent_outputs: list, threshold: float = 0.85) -> bool:
    for existing in recent_outputs:
        if similarity(new_content, existing) > threshold:
            return True
    return False
```

## The Human Gate

Some outputs should never ship without human review, regardless of score. For us, that includes: any email to a client, any public-facing content, any CRM field update that changes deal stage, and any financial data. The Intelligence Loop catches mechanical errors. The human gate catches judgment errors.
