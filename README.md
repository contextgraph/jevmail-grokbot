# Jevmail plugin

Connects your agent to [Jevmail](https://jevmail.xyz) and teaches it to score every email draft before showing it to you.

## What it does

Jevmail scores an email draft against your own sent Gmail: does it sound like you, given how you've written to these people before? Each question returns its own typed answer and probabilities; read the question rather than assuming higher is always better. There is no single combined score.

This plugin bundles:

- **MCP connector** `jevmail` at `https://jevmail.xyz/mcp`
- **Skill** `score-email-drafts`: before presenting any draft or reply, the agent scores it, revises it from the answers (keeping meaning and facts), rescores a few times, and briefly reports what changed.

## Install

1. Install **Jevmail** from the Cursor / Grok Bot plugin marketplace.
2. Sign in when the `jevmail` connector asks (OAuth, no API key).
3. Connect your Gmail at <https://jevmail.xyz> if you haven't already (Gmail read permission is enough for scoring).

## What it can and can't do

- Scores drafts and replies against your sent mail; finds the conversation a reply belongs to.
- Stores each scored draft and its revision chain **in Jevmail**, encrypted. Agents can revise and rescore plain text without Gmail tools.
- Saving to Gmail is a separate `save_draft_to_gmail` action, only when you ask. Jevmail’s scored snapshot remains the source of truth.
- The skill **never sends mail**. You press Send. A passing score is never permission to send. (The server also has an owner-controlled send policy tool; the skill does not use it unless you ask.)
- Does not read attachment contents. Limit: 12 scoring calls per minute.
- Multiple Gmail accounts are supported; Jevmail picks the mailbox from the reply or the draft's From address.

## Privacy

Jevmail reads Gmail metadata, conversations and your sent mail to score drafts. Scoring never creates, edits or reads Gmail drafts. Gmail write permission is needed only for an explicit save or send. The skill sends no mail. See <https://jevmail.xyz> for details. If you see a reconnect error, reconnect Gmail there.

## License

MIT
