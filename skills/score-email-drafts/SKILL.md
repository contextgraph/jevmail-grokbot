---
name: score-email-drafts
description: Score every email draft or reply with Jevmail before showing it to the user. Use whenever you are about to write, present, or send an email draft or reply on the user's behalf.
---

# Score email drafts with Jevmail

Jevmail (MCP server `jevmail`) scores a draft against the owner's own sent Gmail. Each question returns its native Noul, Choice or Score answer. Read the question and acceptable-outcome probability; higher is not always better, and rubric positions are not percentages. There is no combined score.

## Before presenting any draft

1. **Replying:** find the Gmail message you are answering with `find_conversations` (pass the recipients in `to`, and `subject` if known). Use the returned `messageId`.
2. **Score:** call `score_draft` with `draft` (`to`, `subject`, `text`, optional `cc`), and either `messageId` (reply) or `newEmail: true`. Never guess a conversation.
   - With several mailboxes connected, set `draft.from` if known; otherwise Jevmail picks one and returns it as `mailbox`.
   - This stores the encrypted scored draft in Jevmail, the source of truth. Scoring never creates, edits or reads a Gmail draft and never sends. Only Gmail read access is needed for sent-mail and conversation evidence.
3. **Read the answers:** every question has its own probability. Find the weak ones and follow `workflow.next_action`.
4. **Revise and rescore:** revise the text and call `score_draft` with the revised `draft` and `previousCallId` set to the last `callId`. The mailbox and reply target are inherited; no Gmail tools are needed. Retrieve any stored version and its answers with `get_scored_draft` and its `callId`. Preserve the meaning and all facts. Stop after a few rounds, or when `workflow` asks for owner review.
5. **Save to Gmail only if the user explicitly asks:** call `save_draft_to_gmail` with the chosen `callId`. It exports that exact snapshot, never sends, and reuses the same call ID on retries to prevent duplicate Gmail drafts. A Gmail edit does not update its scores: paste revisions into `score_draft`, or use legacy `review_draft` for Gmail attachment review. `create_and_review_draft` remains a deprecated explicit-write operation for compatibility; never use it as the default scoring path.
6. **Report briefly** to the user: which draft was scored, the weakest and strongest questions, what you changed, and where the draft is (stored in Jevmail, callId, mailbox; Gmail only if exported).

## Rules

- Never treat a passing score as permission to send. The user presses Send. Do not call `send_reviewed_draft` or `choose_review_policy` unless the user explicitly asks.
- Do not invent facts or change the meaning just to raise a score. Never loop around a stop for owner review.
- Questions are read-only for you: `list_draft_questions` shows them; the owner edits them at https://jevmail.xyz.
- Limit: 12 scoring calls per minute. On `rate_limited`, wait `retryAfterSeconds`.
- If Jevmail returns reconnect / unauthorized, tell the user to reconnect Gmail at https://jevmail.xyz and stop. Gmail compose permission is needed only for explicit save or send; scoring works with read access.
- If no Jevmail tools are available, say so and show the draft marked "unscored".
