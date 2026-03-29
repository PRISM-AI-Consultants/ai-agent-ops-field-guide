# Chapter 6: Failure Taxonomy

## 9 Root Cause Categories

After classifying 127 incidents across 44 days of production, every failure mapped to one of these 9 categories. Knowing the category accelerates diagnosis from hours to minutes.

---

### 1. Deployment Drift
**Symptoms:** Agent works in staging, fails in production. Config values differ between environments. "It worked yesterday."
**Root cause:** Code was updated but not redeployed. Environment variables changed on one server but not another. Docker image tag points to a stale build.
**Fix recipe:** Deploy + Verify pattern (Chapter 10). After every code change, redeploy AND confirm the running version matches the committed version.

### 2. Guard Rails Missing
**Symptoms:** Agent produces output that violates business rules. Sends emails it should not. Updates records it should not touch.
**Root cause:** The constraint was in documentation but not in code. Someone wrote "never auto-send" in a briefing doc but the agent's code path had no approval gate.
**Fix recipe:** Every business rule needs a code-level check, not just a prompt instruction. LLMs do not reliably follow negative instructions.

### 3. Config Drift
**Symptoms:** Agent uses wrong API endpoint, wrong model, wrong temperature, wrong token. Intermittent failures that depend on which config file was loaded.
**Root cause:** Config exists in multiple places (env vars, config files, hardcoded defaults). They disagree.
**Fix recipe:** Single source of truth for config. One file. Everything reads from it. No defaults that silently override.

### 4. Incomplete Migration
**Symptoms:** Half the system uses the old pattern, half uses the new one. Some agents upgraded, others not. Data format mismatches.
**Root cause:** Migration was started but not finished. The old path was not removed. Both paths are active.
**Fix recipe:** Post-migration grep (Chapter 10). After any migration, search the entire codebase for references to the old pattern. Every hit is a bug.

### 5. Feature Not Wired
**Symptoms:** Feature was built but never connected to the trigger that invokes it. Code exists, tests pass, but it never runs in production.
**Root cause:** The feature was developed in isolation. The integration step was deferred and forgotten.
**Fix recipe:** End-to-end test that starts from the trigger, not from the function. If the cron job does not call the function, unit tests on the function are meaningless.

### 6. Logic Too Broad
**Symptoms:** Agent processes items it should skip. Sends notifications for non-events. Flags false positives.
**Root cause:** Filter conditions are too permissive. The query matches more than intended. The threshold is too low.
**Fix recipe:** Tighten filters. Add explicit exclusion lists. Log what the agent skips, not just what it processes. Review the skip log weekly.

### 7. Orchestration Order
**Symptoms:** Agent B runs before Agent A finishes. Reads stale data. Produces output based on yesterday's state.
**Root cause:** Agents are scheduled independently without dependency awareness. Agent B's cron fires before Agent A's output is written.
**Fix recipe:** Explicit dependency chain. Agent B checks for Agent A's output timestamp before proceeding. If stale, wait or skip.

### 8. Silent Failure
**Symptoms:** Agent appears healthy. No errors in logs. But output is wrong or missing. Discovered days later by a human.
**Root cause:** Exception was caught and swallowed. API returned an empty result instead of an error. Agent treated "no data" as "nothing to do" instead of "something is wrong."
**Fix recipe:** Distinguish between "no work to do" and "failed to find work." Log both paths. Alert on unexpected empty results.

### 9. Process Failure
**Symptoms:** The code is correct but the human process around it failed. Approved a draft that should have been rejected. Skipped a verification step. Deployed without testing.
**Root cause:** Human error, fatigue, or missing checklists.
**Fix recipe:** Checklists, not memory. The maintenance playbook (Chapter 8) exists because humans forget. Automate the checklist enforcement where possible.
