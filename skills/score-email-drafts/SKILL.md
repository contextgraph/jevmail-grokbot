---
name: score-email-drafts
description: Score every email draft or reply with Jevmail before showing it to the user. Use whenever you are about to write, present, or send an email draft or reply on the user's behalf.
---

# Score email drafts with Jevmail

Jevmail (MCP server `jevmail`) scores a draft against the owner's own sent Gmail. Each question returns a probability (higher = closer to how the owner writes). There is no combined score.

## Before presenting any draft

1. **Replying:** find the Gmail message you are answering with `find_conversations` (pass the recipients in `to`, and `subject` if known). Use the returned `messageId`.
2. **Score:** call `create_and_review_draft` with a fresh UUID `requestId`, `draft` (`to`, `subject`, `text`, optional `cc`), and either `messageId` (reply) or `newEmail: true`. Never guess a conversation.
   - With several mailboxes connected, set `draft.from` if known; otherwise Jevmail picks one and returns it as `mailbox`.
   - This saves ONE new Gmail draft; it never sends. On a retry, reuse the same `requestId` so no duplicate draft is created.
3. **Read the answers:** every question has its own probability. Find the weak ones and follow `workflow.next_action`.
4. **Revise and rescore:** edit that same Gmail draft (with your Gmail tools), then call `review_draft` with its `gmailDraftId` and `previousCallId` set to the last `callId`. Preserve the meaning and all facts. Stop after a few rounds, or when `workflow` asks for owner review.
5. **Report briefly** to the user: which draft was scored, the weakest and strongest questions, what you changed, and where the draft is (Gmail draft, mailbox).

## Rules

- Never treat a passing score as permission to send. The user presses Send. Do not call `send_reviewed_draft` or `choose_review_policy` unless the user explicitly asks.
- Do not invent facts or change the meaning just to raise a score. Never loop around a stop for owner review.
- Questions are read-only for you: `list_draft_questions` shows them; the owner edits them at https://jevmail.xyz.
- Limit: 12 scoring calls per minute. On `rate_limited`, wait `retryAfterSeconds`.
- If Jevmail returns reconnect / unauthorized / missing draft permission, tell the user to reconnect Gmail at https://jevmail.xyz and stop.
- If no Jevmail tools are available, say so and show the draft marked "unscored".
