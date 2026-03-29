# Chapter 3: The Email Commandments

## 12 Rules Learned from 22 Email Failures

Email is the most dangerous capability you can give an AI agent. It is also the most useful. These 12 rules were each born from a real production incident. The number in parentheses is how many incidents that rule would have prevented.

### 1. Verify the address before every send (3 incidents)
Never trust a cached or CRM-sourced email address without verification. People change emails. CRM data decays. One bounced email to a prospect mid-deal costs more than the 200ms it takes to verify.

### 2. Check the sent folder before drafting (4 incidents)
Before generating any outbound message, search sent mail for recent messages to the same recipient. Duplicate emails are worse than no email. The recipient does not know you have 34 agents. They just see spam.

### 3. Show the draft in chat before creating it in Gmail (2 incidents)
The operator must see and approve the draft text before it touches the email system. Do not create a Gmail draft and then show it. Show it first. Create it after approval.

### 4. Never auto-send (3 incidents)
No agent sends email without human approval. Period. The path is: agent drafts, human reviews, human approves, agent sends. Any shortcut here will eventually produce an email that damages a client relationship.

### 5. Paginate all email searches (2 incidents)
Gmail API returns a maximum of 20 results per page. If you do not paginate, you will miss emails. If you miss emails, you will claim a follow-up was never sent when it was. The client will correct you, and trust erodes.

```python
def search_all_messages(query: str) -> list:
    results = []
    page_token = None
    while True:
        response = gmail.users().messages().list(
            userId="me", q=query, pageToken=page_token
        ).execute()
        results.extend(response.get("messages", []))
        page_token = response.get("nextPageToken")
        if not page_token:
            break
    return results
```

### 6. Map tokens to accounts (2 incidents)
When managing multiple email accounts, maintain an explicit token-to-account mapping. Never guess which OAuth token corresponds to which mailbox. One wrong token sends a client email from your personal account.

### 7. Read the full thread before drafting a reply (1 incident)
An agent that replies based on the subject line alone will contradict something said three messages deep in the thread.

### 8. Never use bot-generated drafts as-is (2 incidents)
Automated drafts are starting points, not finished products. Every bot draft must be rewritten by the operator or a senior agent before it reaches a client.

### 9. Check all mailboxes (1 incident)
If your operation uses multiple email accounts (personal, business, role-based), search all of them before claiming a message was never received.

### 10. Verify send success after sending (1 incident)
After sending, confirm the message appears in the sent folder. Network failures, token expiration, and rate limits can all cause silent send failures.

### 11. Delete stale drafts after sending (1 incident)
Bot-generated drafts that were not used must be cleaned up. A mailbox with 200 orphaned drafts is an operational hazard. Someone will accidentally send one.

### 12. Never apologize for things that did not happen (0 incidents, but close calls)
Before drafting an apology or follow-up that references a missed commitment, verify the commitment was actually missed. Check sent mail, check transcripts, check CRM. False apologies destroy credibility faster than real mistakes.
