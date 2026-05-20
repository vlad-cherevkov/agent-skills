# Agent Skills

A small collection of skills for Claude Code (and other agent-style AI coding assistants). Drop any of these into your local skills directory and Claude Code will load it automatically.

## Install a skill

Skills live in `~/.claude/skills/<skill-name>/`. To install all of them:

```bash
git clone https://github.com/vlad-cherevkov/agent-skills.git
mkdir -p ~/.claude/skills
cp -r agent-skills/*/ ~/.claude/skills/
```

Or just one:

```bash
mkdir -p ~/.claude/skills/slack-formatting
curl -L https://raw.githubusercontent.com/vlad-cherevkov/agent-skills/main/slack-formatting/SKILL.md \
  -o ~/.claude/skills/slack-formatting/SKILL.md
```

The skill loads automatically the next time you start Claude Code — no restart of anything else needed.

## Skills

### slack-formatting

Formats Slack messages so bold and bulleted lists actually render correctly. Works for both delivery paths:

- **Copy-paste:** ask Claude Code "give me a Slack message about X" and the entire response is the message, ready to ⌘C → ⌘V into Slack's composer. Bold, bullets, nested bullets, and inline code all render properly.
- **MCP send:** if you have a Slack MCP connector installed (the official Anthropic one or similar) and ask "send a message to #channel about X", Claude formats the message the same way and delivers it via the MCP tool. The tool wrapper converts the markdown to Slack's Block Kit `rich_text` blocks server-side, so you get real native bold and real native lists in the sent message.

Same standard-markdown syntax for both paths (`**bold**`, `- bullets`, `` `code` ``).

**How the copy-paste path works:** Claude Code renders the response as rich HTML on your clipboard. Slack's rich composer reads that HTML on paste.

**Prerequisites:**
- macOS (the rich HTML clipboard path relies on it)
- Claude Code
- Slack desktop app

**Slack setting:** Make sure `Preferences → Advanced → "Format messages with markup"` is **OFF** (that's Slack's default — only relevant if you previously enabled it). With that setting on, the copy-paste path behaves differently and the skill won't give you the result you want. (This setting doesn't affect the MCP path.)

**Try it (copy-paste):** in any Claude Code session, type `give me a Slack message about the deploy going out Friday morning`. The whole response will be your message, ready to copy.

**Try it (MCP):** if you have a Slack MCP connector configured, type `send a message to #general about the deploy going out Friday`. Claude will format the message and call the Slack MCP tool to send (or draft) it for you.

## Contributing

If you write a skill you think other people would find useful, open a PR. One directory per skill, each with a `SKILL.md` at the root using Claude Code's standard frontmatter:

```markdown
---
name: skill-name
description: When this skill should trigger. Be specific about the trigger phrases and contexts.
---

# Skill content
```

## License

MIT.
