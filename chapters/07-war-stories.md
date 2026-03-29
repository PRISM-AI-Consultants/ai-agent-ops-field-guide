# Chapter 7: War Stories

## 5 Incidents from Production

These are real. Names and details are anonymized. The mechanisms, root causes, and fixes are exactly what happened. Nothing is invented.

---

### 1. The 200-Draft Flood

**What happened:** A session recap agent ran on a 30-minute cycle. Every cycle, it loaded its state file to check which calendar events it had already processed. One morning, the operator opened Gmail to find 87 new drafts. 30 for one client, 40 for another. Over the next few days, the count passed 200.

**Root cause:** During an infrastructure migration from a local machine to a VPS, a symlink was created pointing the state file to a path that only existed on the VPS. On the local machine, the symlink resolved to nothing. The agent loaded an empty state every cycle, saw every calendar event as new, created drafts for all of them, then crashed before saving state. Next cycle, same thing.

**Fix:**
1. Disabled the local agent entirely. Only the VPS copy runs now.
2. Replaced the broken symlink with a real file. Added `--copy-links` to the sync script so VPS symlinks get dereferenced during sync.
3. Added a dedup check: before creating a draft, verify no draft to that recipient exists for that event.
4. Added a grade gate: output scoring below 7/10 gets rejected automatically.
5. Added a check for both sent emails AND existing drafts before creating new ones.

**Lesson:** If an agent's "memory" of what it already did lives in a file that can go missing, the agent will redo everything from scratch. Symlinks across environments are a time bomb. And any agent that creates external artifacts needs a dedup gate.

---

### 2. The 30-Day Silent Failure

**What happened:** A fleet briefing system generated daily status reports for the operator. It ran every day via cron. The logs showed "successful execution" every day. One month later, the operator noticed the reports had been blank since February 7th.

**Root cause:** A code change introduced a KeyError on a field name. The function caught the exception, logged a generic success message, and returned an empty result. No alerting. No error in the logs. The system was "working" by every observable measure except the one that mattered: it produced nothing useful.

**Fix:**
1. Added output validation: if the generated report is empty or below a minimum length, that is an error, not a success.
2. Added alerting on empty output, not just on crashes.
3. Established a rule: every try/except block must either re-raise, alert, or log to a monitored channel. Zero silent catches for state operations.

**Lesson:** A bot that runs without crashing is not the same as a bot that works. Silent failures are worse than loud crashes. If your agent catches exceptions and continues, you need output validation to prove it actually produced something.

---

### 3. The Docker IP Cascade

**What happened:** Six automation scripts connected to databases running in Docker containers. One day, all six failed simultaneously. Connection refused. The databases were running fine.

**Root cause:** The Docker containers had been rebuilt two days earlier. When Docker recreates containers, it can assign new internal IP addresses. The database container moved from 172.18.0.30 to 172.18.0.40. All six scripts had the IP hardcoded. The credentials had also changed because the docker-compose environment variables used different defaults than what the scripts expected.

**Fix:**
1. Updated all six scripts with the correct IPs and credentials.
2. Documented the container-to-IP mapping so the next rebuild has a reference.
3. Added a note: consider Docker network aliases so scripts can use container names instead of IPs.

**Lesson:** Hardcoded infrastructure values are fine until the infrastructure changes. Docker container IPs will change on every rebuild. If you cannot use DNS/service discovery, at minimum document the mapping and have a checklist for post-rebuild verification.

---

### 4. The Nginx Content Leak

**What happened:** A collaborator received a set of links that were supposed to point to student course videos. Instead, the links served the operator's internal business dashboard. The collaborator saw client names, revenue figures, and operational data that was never meant to be shared.

**Root cause:** The Nginx server had a default server block that fell back to serving the root application when a requested path did not match any configured route. The video links pointed to paths that did not exist on that server. Instead of returning a 404, Nginx served the default application, which happened to be the internal dashboard.

**Fix:**
1. Updated the Nginx configuration to return a proper 404 for unmatched routes instead of falling through to the default application.
2. Audited all other Nginx server blocks for similar fallback behavior.
3. Moved the internal dashboard to a separate subdomain with authentication.

**Lesson:** Default server blocks in reverse proxies are a security risk. If your proxy serves a fallback application on unmatched routes, any broken link becomes a potential data exposure. Every public-facing proxy should return 404 on unknown paths, not serve internal tools.

---

### 5. The Incomplete Migration

**What happened:** The fleet was migrated from one LLM API provider to another to reduce costs. After migration, the operator assumed everything was switched over. Three weeks later, a cost audit revealed unexpected charges from the old provider. Five agents were still calling the old API.

**Root cause:** The migration updated the main pipeline files but missed utility modules, helper scripts, and files that were loaded conditionally. The agents that were missed were not in the primary import chain. They imported the old SDK directly in files that nobody thought to check.

**Fix:**
1. Ran `grep -r` across the entire codebase for the old SDK import. Found five files.
2. Migrated all five to the new provider.
3. Added cost tracking so every agent reports its API usage.
4. Established a rule: after any migration, grep the entire codebase for the old pattern. Do not trust that you found everything by following imports.

**Lesson:** Migrations are never complete until you grep. The files you miss are the ones not in the obvious dependency chain. And if you do not have per-agent cost tracking, you will not know something is wrong until the bill arrives.
