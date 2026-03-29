# AI Agent Ops Field Guide

**I mass-deployed 34 AI agents for a live business. Here is everything that went wrong.**

Most AI agent content is written by people who deployed one bot last Tuesday. This is a field manual from someone running 34 agents in production across email triage, session recaps, CRM sync, content generation, security monitoring, and client delivery for a real consulting business with real clients paying real money.

Over 44 days, I logged every failure, traced every root cause, and wrote down every fix. The result is this guide.

```
Incidents Logged:    127
Root Cause Categories: 10
Sessions Mined:      383
Agents in Fleet:      34
Uptime Target:       99%
Actual Uptime:       91.3%
```

The numbers are real. The war stories are real. The client names and identifying details are anonymized.

---

## Hall of Shame: Greatest Hits

The incidents that made me question my life choices, roughly in order of severity:

- A bot created **200 duplicate email drafts** in one night. The client inbox looked like a ransom note.
- An agent confidently sent a session recap **to the wrong client**. With the other client's name still in the greeting.
- Token refresh failed silently for 11 days. Eleven. Days. Nobody noticed because the bot just stopped working and nothing alerted.
- An **$84/month mystery charge** surfaced during a cost audit. Still unsure which agent ordered it. The investigation is in Chapter 9.
- One bot hallucinated a follow-up task that didn't exist, then emailed the client about it. The client responded asking what we were talking about.
- Disk hit 100% on the VPS. Every container crashed. Root cause: a logging bot that never learned the word "rotate."
- An agent applied the wrong client's brand voice to a deliverable. Very professional. Very wrong company.

If none of these sound familiar, you probably haven't shipped agents to production yet. Give it a week.

---

## Who This Is For

**This will save you time if you are:**

- Running n8n, Make, LangChain, or custom agent workflows that touch real users
- Managing a fleet of LLM-powered bots and wondering why things keep breaking in new and creative ways
- The person who gets the 3 AM Slack message when an agent goes rogue
- An engineering lead about to pitch "let's automate everything with AI agents" and wanting to know what that actually looks like on month two

**This is probably not for you if:**

- You are looking for a tutorial on building your first chatbot
- You want theoretical best practices from someone who has not deployed anything
- You think "just add retry logic" is a complete error-handling strategy

---

## What You Will Find

Patterns, anti-patterns, fix recipes, and checklists distilled from 383 sessions of operating a 34-agent fleet. No theory. No best practices invented in a lab. Just field notes from an operator who kept a detailed log because he kept getting burned.

---

## Table of Contents

1. [The Fleet](chapters/01-the-fleet.md) - The architecture of 34 agents and how they talk to each other (and sometimes to themselves)
2. [The Boot Sequence](chapters/02-the-boot-sequence.md) - Why every session starts with a mandatory checklist, and what happens when you skip it
3. [Email Commandments](chapters/03-email-commandments.md) - 12 rules written in the blood of 22 email failures. "Never send without human review" is rule one for a reason.
4. [Infrastructure Hygiene](chapters/04-infrastructure-hygiene.md) - Disk, Docker, SSL, DNS. The boring stuff that takes down your entire fleet when you ignore it.
5. [Quality Gates](chapters/05-quality-gates.md) - The Intelligence Loop: how to build a QA system that actually catches hallucinated content before clients see it
6. [Failure Taxonomy](chapters/06-failure-taxonomy.md) - 9 root causes with fix recipes. You will recognize at least three of these from your own fleet.
7. [War Stories](chapters/07-war-stories.md) - 5 anonymized production incidents, from "minor embarrassment" to "called the client to apologize"
8. [Maintenance Playbook](chapters/08-maintenance-playbook.md) - Daily, weekly, and monthly checklists. The stuff that prevents incidents instead of responding to them.
9. [Cost Governance](chapters/09-cost-governance.md) - How to stop your API bill from becoming a mystery novel. Budget frameworks and audit patterns.
10. [Fix Pattern Library](chapters/10-fix-pattern-library.md) - Reusable remediation patterns you can steal for your own fleet

---

## How to Use This Guide

**If you are just starting:** Read Chapter 1 (The Fleet) and Chapter 2 (The Boot Sequence) to understand the architecture patterns. Then read Chapter 6 (Failure Taxonomy) so you know what to watch for.

**If something is broken right now:** Go straight to Chapter 6 (Failure Taxonomy) to identify your root cause, then Chapter 10 (Fix Pattern Library) for the remediation steps.

**If you are auditing your setup:** Chapter 8 (Maintenance Playbook) and Chapter 9 (Cost Governance) will give you the checklists.

---

## About

By Dr. Jeff Bullock, PharmD, and the team at [PRISM AI Consultants](https://prismaiconsultants.com). Built from real production data, anonymized for the community.

If this guide saves you from even one 3 AM incident, it was worth publishing.

Licensed under [MIT](LICENSE). Contributions welcome, see [CONTRIBUTING.md](CONTRIBUTING.md).
