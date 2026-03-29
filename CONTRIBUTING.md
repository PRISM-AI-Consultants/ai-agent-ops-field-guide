# Contributing to the AI Agent Ops Field Guide

We want your war stories, fix patterns, and hard-won lessons. This guide gets better with every operator who contributes.

## What We Accept

### War Stories (Chapter 7)
Real incidents from real production systems. Format:

```
### [Descriptive Title]
**What happened:** One paragraph.
**Root cause:** One sentence.
**Fix:** Numbered steps.
**Lesson:** One sentence takeaway.
```

Rules:
- Must be a real incident, not a hypothetical
- Anonymize all client names, company names, email addresses, and API keys
- Include the root cause category from Chapter 6 if applicable
- Keep it under 200 words

### Fix Patterns (Chapter 10)
Reusable remediation recipes. Format:

```
### [Pattern Name]
**When to use:** One sentence.
**Steps:**
1. Step one
2. Step two
**Example:** Brief code or config snippet.
```

### Failure Categories (Chapter 6)
If you have identified a root cause category not covered in the existing 9, propose it with at least 3 real examples.

### Checklist Items (Chapter 8)
Additional daily/weekly/monthly maintenance items with rationale.

## How to Submit

1. Fork this repository
2. Create a branch named `contribution/your-description`
3. Add your content to the appropriate chapter file
4. Open a pull request with a one-sentence summary of what you are contributing

## What We Do Not Accept

- Marketing content or product placement
- Hypothetical scenarios without real-world basis
- Content that identifies specific clients, companies, or individuals
- AI-generated filler (we will know)

## Style Guide

- No emojis
- Operator-to-operator tone. Write like you are explaining to a peer, not presenting at a conference.
- Concrete over abstract. Numbers over adjectives.
- Code examples in Python unless the pattern is infrastructure-specific

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Questions

Open an issue. We respond within 48 hours.
