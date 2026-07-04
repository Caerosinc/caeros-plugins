---
name: gmail-triage
description: Triage a Gmail inbox — surface what needs action, summarize the rest, and prepare draft replies for anything urgent.
---

# Gmail Triage

Use this skill when the user asks to "triage", "catch up on", or "go through" their inbox.

1. Fetch recent unread mail: `apps_execute_tool` with app=gmail, tool=GMAIL_FETCH_EMAILS, arguments `{"query":"is:unread newer_than:2d"}` (widen the window if empty).
2. Bucket the results:
   - **Needs reply** — direct questions, requests with deadlines, anything from the user's manager/close collaborators.
   - **Needs action, no reply** — invoices, approvals, calendar conflicts.
   - **FYI / can archive** — newsletters, notifications, receipts.
3. Present the buckets in that order, one line per message: sender — subject — the one thing that matters.
4. For each "needs reply" item, offer to draft a response. Create drafts with GMAIL_CREATE_EMAIL_DRAFT; never send without explicit confirmation.
5. If the user wants inbox cleanup, propose the label/mark-as-read changes first and apply them only after approval.
