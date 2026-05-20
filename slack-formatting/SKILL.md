---
name: slack-formatting
description: Use this skill whenever the user asks for a Slack message, Slack DM, Slack post, channel update, Slack reply, or anything they intend to paste into Slack OR send via a Slack MCP tool (e.g. slack_send_message, slack_send_message_draft). Triggers on any mention of "slack", "DM", "channel post", "thread reply", "send to #channel", or when the user says "draft a message" / "format this for Slack" / "give me a Slack message" / "send this to Slack". Applies to BOTH the copy-paste workflow (user pastes into Slack) and the MCP workflow (you call the Slack MCP tool to send the message yourself).
---

# Slack message formatting

## Two delivery paths — same format

This skill covers two ways a Slack message can get sent:

1. **Copy-paste path** — you write the message as your response. The user copies your response and pastes into Slack's composer. Claude Code's rendered HTML on the clipboard becomes Slack rich text on paste.

2. **MCP path** — you call a Slack MCP tool (`slack_send_message`, `slack_send_message_draft`) and pass the message as the `message` argument. The MCP tool wrapper converts standard markdown to Slack's Block Kit `rich_text` blocks server-side, giving real bold and real bulleted lists.

**The formatting syntax is the same for both paths: standard markdown** (`**bold**`, `- bullets`, etc.). What differs is the workflow around producing it — covered below.

## Choosing the path

- If the user says "give me a Slack message", "draft a Slack message", "format this for Slack", "I want to send this to Slack" → **copy-paste path** (default).
- If the user says "send a message to #channel about X", "DM @person to say Y", or anything that names a destination they want YOU to deliver to → **MCP path**.
- If ambiguous, default to copy-paste. The user can ask you to actually send it.

## Copy-paste path: STRICT OUTPUT RULE

On the copy-paste path, your entire response must be the Slack message and nothing else. The user copies your full response with one click and pastes into Slack.

That means: NO preamble ("Here's your message:", "I'll format that for you:"), NO trailing commentary ("Let me know if you want changes", "Hope this works"), NO test instructions, NO explanation of what you did, NO suggestions, NO follow-up questions, NO code blocks wrapping the message. If you write anything other than the message body, the user has to clean it out by hand — which defeats the entire skill.

The only allowed exception is if the user's request is genuinely ambiguous (you literally cannot tell what they want said). Even then, prefer making reasonable assumptions and producing the message — they'll iterate in the next turn.

## MCP path: workflow

When the user asks you to actually send via the Slack MCP tool:

1. If the channel/user isn't clear, use `slack_search_channels` or `slack_search_users` to resolve it.
2. Compose the message text using the same format rules below. Pass it as the `message` argument.
3. Default to `slack_send_message_draft` if the user hasn't reviewed the message yet — it creates a draft in Slack so they can read it before sending. Only call `slack_send_message` directly if the user has explicitly asked for it to be sent immediately or has already approved the draft content.
4. After the tool returns, respond to the user briefly with what happened (e.g., "Draft created in #channel — link: …"). The strict output rule does NOT apply on this path; the user needs to know what you did.

## Why standard markdown

**Copy-paste path:** Claude Code renders markdown in your responses to rich HTML. When the user clicks the copy button, Claude Code puts that rich HTML on the macOS clipboard. When they paste into Slack, Slack's rich composer reads the HTML and converts `<strong>` → bold, `<ul><li>` → real editable bullets, `<code>` → inline code.

**MCP path:** The Slack MCP tool's own documentation says: *"Message uses standard markdown (`**bold**`, `_italic_`, `code`, `~~strikethrough~~`, blockquotes, lists, links, code blocks, tables, headers)."* The wrapper converts to Block Kit `rich_text` blocks under the hood. So `**bold**` and `- item` produce real bold and real bullets in the sent message.

Both paths share the same syntax, which is why we use **standard markdown** (`**bold**`, `- bullets`), NOT Slack's `*single-asterisk*` mrkdwn:
- `**bold**` → renders to `<strong>` → bold in Slack ✅
- `*bold*` → renders to `<em>` → ITALIC in Slack ❌ (banned by user)

The user has **"Format messages with markup" OFF** in Slack Preferences → Advanced (only relevant for the copy-paste path; doesn't affect the MCP path). Do not advise them to enable it; that mode interferes with their normal typing in Slack.

## Format scope

Only three formats. Nothing else:

- `**bold**` — double asterisks for emphasis
- `- item` — dash + space at line start for bullets; 4-space indent for one level of nesting
- `` `code` `` — backticks for technical tokens (phone numbers, emails, identifiers, template variables, URLs)

Out of scope: italics (banned), underline, strikethrough, numbered lists, `#`-style headings, block quotes, markdown links.

## Rules

**Bold**
- `**like this**` ✅
- `*like this*` ❌ — renders as italic (banned)
- `***like this***` ❌ — renders as bold-italic (italic banned)

Use bold for the TL;DR line and for each section heading line.

**Bullets**
- `- item` ✅
- `* item` ❌ — same character as markdown emphasis, risk of confusion
- `• item` ❌ — user prefers dashes
- Nested bullets: 4 spaces of indent before the `-`, one level of nesting.

Example:
```
- Top-level point
    - Nested sub-point
    - Another sub-point
- Next top-level point
```

**Inline code** — backticks around technical tokens only (numbers like `+971 58 505 7074`, emails like `mo.aziz@joinsooner.com`, domains like `joinsooner.com`, template variables like `{{1}}`). Don't use code as a general emphasis style — bold covers that.

**Italics — NEVER.** No `*single asterisk*`, no `_underscore_`. The user dislikes italics and has said so multiple times. If you want emphasis, use `**bold**`. Even for foreign terms, titles of works, technical jargon — just use bold or leave plain.

**Numbered lists — out of scope.** If sequence really matters, write it as prose ("first…then…finally") or as bullets with bold prefixes like `**Step 1:**`.

**Headings — use bold lines, not `#`.** A bold line on its own functions as the heading. The `#` syntax renders as `<h1>`/`<h2>` HTML which Slack handles inconsistently; stick with `**heading**` on its own line.

## Spacing

Slack collapses plain blank lines when pasting from the HTML clipboard, so a blank line in the markdown source disappears in Slack and sections render tight against each other. To force a visible empty line between sections, use `&nbsp;` (the HTML entity for a non-breaking space) on its own line. It survives the paste as a real empty paragraph and produces the visual breathing room.

Rules (apply to BOTH paths — `&nbsp;` is harmless on the MCP path; the wrapper either preserves it as a non-breaking-space paragraph or strips it, never breaks the message):
- Place `&nbsp;` on its own line (surrounded by blank lines in the markdown source) ABOVE each bold heading line. This is the gap the user wants between sections.
- Do NOT place `&nbsp;` below a heading — content (bullets or paragraph) starts immediately on the next line after the heading.
- Inside a bullet list, NO blank lines between bullets.
- Standard markdown blank lines between paragraphs are fine but Slack collapses them visually on copy-paste — only `&nbsp;` produces a visible gap there.

Correct:
```
Previous content.

&nbsp;

**TL;DR**
- Bullet one
- Bullet two
- Bullet three

&nbsp;

**Next section**
- More bullets
```

Wrong — no `&nbsp;`, Slack will paste with the heading flush against the prior content:
```
Previous content.

**TL;DR**
- Bullet one
- Bullet two
```

Also wrong — `&nbsp;` below the heading adds unwanted vertical space inside the section:
```
&nbsp;

**TL;DR**

&nbsp;

- Bullet one
```

## Output style

- Concise. Slack messages are skimmed in a busy channel, not read. Short lines.
- Lead with TL;DR / outcome / action item before supporting detail.
- Prefer bullets over prose paragraphs once you have more than two facts.
- @-mentions stay as `@name` literal — the user re-types them in Slack to trigger actual mentions.

## Reminder

Your entire response is the Slack message. Nothing else. The user is going to ⌘C → ⌘V — what they paste must be ready to send.
