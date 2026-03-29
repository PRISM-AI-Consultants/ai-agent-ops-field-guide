# Chapter 7: War Stories

## 5 Incidents from Production

These are real. Names and details are anonymized. The lessons are universal.

---

### 1. The 200-Draft Flood

**What happened:** An email drafting agent was triggered by a webhook that fired on every CRM field update. A bulk import of 200 contact records triggered 200 webhook events. The agent dutifully created 200 Gmail drafts in 4 minutes. The operator opened Gmail to find an inbox full of unsent drafts addressed to prospects, each with slight variations because the LLM does not produce identical output twice.

**Root cause:** Guard rails missing (Category 2). The agent had no deduplication check and no rate limit. The webhook had no debounce.

**Fix:**
1. Added deduplication: before creating a draft, check if a draft to the same recipient already exists within 24 hours
2. Added rate limiting: max 5 drafts per 10-minute window, then pause and alert
3. Added webhook debounce: batch events within a 60-second window before triggering the agent

**Lesson:** Any agent that creates external artifacts (emails, messages, CRM records) needs both a dedup gate and a rate limit. The LLM will happily comply with 200 requests without questioning whether it should.

---

### 2. The Content Leak

**What happened:** A content generation agent was building a case study from session transcripts. The agent included a direct quote from a client that contained the name of their patient's medical condition. The draft was caught during human review, 30 seconds before it would have been posted to social media.

**Root cause:** Guard rails missing (Category 2). The agent had instructions to anonymize, but no code-level PII detection. Prompt-level instructions are not reliable filters for sensitive data.

**Fix:**
1. Added a PII detection pass using regex patterns for names, conditions, and identifiers before any content leaves the pipeline
2. Added a hard rule: no direct quotes from transcripts in public content without explicit approval
3. Added the content to the "never auto-publish" list requiring human gate

**Lesson:** If your agents process any data that could contain PII, health information, or financial details, you need code-level detection. Prompt instructions will fail eventually. It is a matter of when, not if.

---

### 3. The Mystery API Bill

**What happened:** Monthly API costs jumped from $12 to $340. No new agents were deployed. No traffic spike. The billing dashboard showed a 28x increase in a single API endpoint.

**Root cause:** Logic too broad (Category 6). A research agent had been updated to process "all new entries" instead of "entries from the last 24 hours." A backlog of 3 months of entries was reprocessed every run. The agent ran every 30 minutes.

**Fix:**
1. Added a processed-items ledger: every item gets a hash recorded after processing
2. Changed the query from "all new" to "created after last_processed_timestamp"
3. Added a cost alert: if estimated cost for a single run exceeds $5, pause and notify

**Lesson:** Every agent that calls a paid API needs a cost ceiling per run. Not per month. Per run. A monthly budget does not help when a single misconfigured run burns through it in an hour.

---

### 4. The Account Lockout Cascade

**What happened:** An OAuth token expired for the primary email account. The email triage agent started failing silently (Category 8). The retry logic hammered the auth endpoint 900 times in 15 minutes. Google temporarily locked the account. Five other agents that depended on that email account also failed. Client emails went unprocessed for 6 hours.

**Root cause:** Silent failure (Category 8) combined with orchestration order (Category 7). No circuit breaker on auth retries. No cross-agent awareness of the auth failure.

**Fix:**
1. Circuit breaker: after 3 consecutive auth failures, stop retrying and alert
2. Token health check in the boot sequence: verify token validity before starting work
3. Shared auth status in fleet_status.json: if one agent detects an auth failure, all agents that use the same account see it immediately

**Lesson:** Auth failures cascade. Every shared credential is a single point of failure for every agent that uses it. Build circuit breakers and make auth status a fleet-level concern, not an agent-level one.

---

### 5. The Rogue Bot

**What happened:** A monitoring agent was configured to post daily security scan results to a Slack channel. A configuration change accidentally pointed it at the general client-facing channel instead of the internal ops channel. For 3 days, clients saw messages like "Port scan complete. 2 unexpected open ports detected on production server."

**Root cause:** Config drift (Category 3). The Slack webhook URL was stored in two places: the agent config and an environment variable. The env var was updated during a migration. The agent config was not.

**Fix:**
1. Single source of truth: removed the hardcoded webhook URL from the agent config, read only from env
2. Channel validation: before posting, verify the channel name matches the expected pattern
3. Added a "dry run" mode that logs what would be posted without actually posting

**Lesson:** Any agent that posts to an external channel needs a channel validation step. Posting internal operational data to an external channel is a security incident. Treat channel routing with the same care as email addressing.
