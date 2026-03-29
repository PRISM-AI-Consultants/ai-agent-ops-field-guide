# Chapter 10: Fix Pattern Library

## Reusable Remediation Patterns

These patterns appear repeatedly across the 127 incidents in our dataset. Each one is a named, repeatable procedure.

---

### 1. Deploy + Verify

**When to use:** After any code change, config change, or migration step.

**Steps:**
1. Deploy the change (push, rebuild, restart)
2. Verify the running version matches the deployed version (check build hash, version endpoint, or log output)
3. Run one end-to-end test that exercises the changed code path
4. Confirm output is correct by inspecting the actual artifact (email, file, database record)

**Example:**
```bash
# Deploy
docker compose up -d --build agent-recap

# Verify version
docker exec agent-recap cat /app/VERSION
# Should match: git rev-parse --short HEAD

# End-to-end test
curl -X POST http://localhost:8080/test/recap -d '{"session_id": "test-001"}'
# Inspect output
cat /data/recaps/test-001.md
```

**Why it matters:** 23 of 127 incidents would have been caught by a verify step that was skipped.

---

### 2. Grade Gate

**When to use:** Before any agent output reaches a client, external system, or public channel.

**Steps:**
1. Generate output
2. Score against criteria (default baseline: 5/10)
3. If score < threshold (8/10), regenerate with feedback
4. After max iterations, flag for human review

**Example:**
```python
output = generate(task)
for _ in range(3):
    score, feedback = grade(output, criteria)
    if score >= 8:
        break
    output = fix(output, feedback)
else:
    flag_for_human_review(output, score, feedback)
```

---

### 3. Cooldown Timer

**When to use:** Any agent that sends notifications, alerts, or messages.

**Steps:**
1. Before sending, check last send time for this event type
2. If within cooldown window, suppress and log
3. If outside window, send and record timestamp

**Example:**
```python
COOLDOWN = timedelta(minutes=30)

def notify_if_allowed(event_key: str, message: str):
    last = cache.get(f"notify:{event_key}")
    if last and datetime.now() - last < COOLDOWN:
        log.info(f"Suppressed notification for {event_key} (cooldown)")
        return
    send_notification(message)
    cache.set(f"notify:{event_key}", datetime.now())
```

---

### 4. Cross-Reference Before Presenting

**When to use:** Before telling the operator that something is missing, overdue, or failed.

**Steps:**
1. Check the primary source (CRM, database, file system)
2. Check the secondary source (sent email, calendar, memory service)
3. Check the tertiary source (transcripts, logs, prior session records)
4. Only present the finding if all sources agree

**Example:** Before claiming "recap was never sent for Client X," check: (a) sent email folder, (b) recap output directory, (c) memory service for session records. False negatives erode operator trust faster than real ones.

---

### 5. Post-Migration Grep

**When to use:** After any migration, refactor, or rename.

**Steps:**
1. Identify the old pattern (function name, endpoint, file path, variable)
2. Grep the entire codebase for the old pattern
3. Every hit is either a missed migration step or needs an explicit exception comment
4. Zero hits = migration complete

**Example:**
```bash
# After migrating from /api/v1/ to /api/v2/
grep -r "api/v1" --include="*.py" --include="*.yaml" --include="*.json" .
# Every result is a bug until proven otherwise
```

---

### 6. Circuit Breaker

**When to use:** Any agent that retries on failure (API calls, auth, external services).

**Steps:**
1. Track consecutive failures
2. After N failures (we use 3), open the circuit: stop retrying, alert operator
3. After a cooldown period, attempt one probe request
4. If probe succeeds, close circuit and resume. If not, keep circuit open.

**Example:**
```python
class CircuitBreaker:
    def __init__(self, max_failures=3, cooldown=300):
        self.failures = 0
        self.max = max_failures
        self.cooldown = cooldown
        self.last_failure = None
        self.open = False

    def call(self, func, *args):
        if self.open:
            if time.time() - self.last_failure < self.cooldown:
                raise CircuitOpenError("Circuit is open, waiting for cooldown")
            # Probe
            self.open = False

        try:
            result = func(*args)
            self.failures = 0
            return result
        except Exception as e:
            self.failures += 1
            self.last_failure = time.time()
            if self.failures >= self.max:
                self.open = True
                alert(f"Circuit opened after {self.max} failures")
            raise
```

---

### 7. Dedup Gate

**When to use:** Before creating any external artifact (email, draft, post, record).

**Steps:**
1. Define the dedup key (recipient + subject, content hash, record ID)
2. Check if an artifact with the same key was created within the dedup window (24 hours default)
3. If duplicate found, skip and log
4. If no duplicate, proceed and record the key

---

### 8. Token Health Check

**When to use:** In the boot sequence, before any API calls.

**Steps:**
1. Attempt a lightweight API call (profile fetch, token info endpoint)
2. If success, proceed
3. If 401/403, attempt token refresh
4. If refresh fails, alert and abort

This prevents an entire agent run from failing at step 47 of 50 because the token expired. Fail fast at step 0.
