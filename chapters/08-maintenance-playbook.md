# Chapter 8: Maintenance Playbook

## Daily, Weekly, Monthly Checklists

Maintenance is not optional. It is the difference between a fleet that degrades over weeks and one that stays operational for months. These checklists are ordered by priority within each cadence.

---

## Daily (10 minutes)

- [ ] **Check fleet status.** Review `fleet_status.json` for any agents with non-zero exit codes in the last 24 hours. Investigate any failures before they compound.
- [ ] **Review error logs.** Scan the last 24 hours of agent logs for warnings and errors. Focus on patterns, not individual events. Three warnings from the same agent is a trend.
- [ ] **Verify email health.** Confirm that the email triage agent processed messages and that no drafts are orphaned. Check draft count; if it is growing, something is creating without cleaning up.
- [ ] **Disk usage check.** Confirm usage is below 80%. If above 75%, schedule cleanup before the next day.
- [ ] **Spot-check one agent output.** Pick a random agent. Read its last output. Does it look correct? This catches silent failures that logs miss.

## Weekly (30 minutes, Sunday evening recommended)

- [ ] **Docker cleanup.** Run `docker system prune -af --volumes` and `docker builder prune -af`. Log the space reclaimed.
- [ ] **Security scan.** Run Nmap self-scan. Compare against baseline. Investigate any new open ports.
- [ ] **SSL certificate check.** Verify all certificates have more than 14 days until expiry.
- [ ] **Review failures log.** Read `failures_log.md` end to end. Are any failure categories recurring? Recurring failures indicate a systemic fix is needed, not another patch.
- [ ] **Agent skip log review.** For agents with filters (email triage, content generation), review what they skipped. False negatives hide in the skip log.
- [ ] **Cost check.** Review API usage for the week. Compare against the prior week. Flag any increase over 20%.
- [ ] **Maintenance log entry.** Write a one-paragraph summary of what was found and fixed. Future you will thank present you.

## Monthly (60 minutes, first Monday)

- [ ] **Full agent audit.** For each of the 34 agents: Is it still needed? Is its briefing current? Is its config single-sourced? Has its error rate changed?
- [ ] **Dependency update.** Check for outdated packages across all agent codebases. Update non-breaking dependencies. Test after updating.
- [ ] **Token and credential rotation.** Review all OAuth tokens, API keys, and service account credentials. Rotate any that are older than 90 days.
- [ ] **Cost governance review.** Run the full cost audit (Chapter 9). Compare actual spend against budget. Identify the top 3 cost drivers.
- [ ] **Backup verification.** Confirm that backups of the semantic memory database, fleet config, and agent state files are current and restorable. Test a restore.
- [ ] **Capacity review.** Are any agents hitting rate limits? Is disk usage trending upward month over month? Is the VPS right-sized for the current fleet?
- [ ] **Documentation sweep.** Walk through all agent briefings and shared docs. Remove references to deprecated features. Update any stale instructions.

---

## The Meta-Rule

If you skip maintenance for two consecutive weeks, assume something is already broken and you have not noticed yet. The purpose of maintenance is not to fix things. It is to find the things that are about to break.
