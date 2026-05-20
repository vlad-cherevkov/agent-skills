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

Formats messages for pasting into Slack so bold text and bulleted lists render correctly. Ask Claude Code for "a Slack message about X" and the response is markdown that, when copy-pasted into Slack's composer, becomes real bold and real editable bullet lists — not literal asterisks and dashes.

**How it works:** Claude Code renders the response as rich HTML on your clipboard. Slack's rich composer reads that HTML on paste and converts it to its own formatting.

**Prerequisites:**
- macOS (the rich HTML clipboard path is what makes this work)
- Claude Code
- Slack desktop app

**Slack setting:** Make sure `Preferences → Advanced → "Format messages with markup"` is **OFF** (that's Slack's default — only relevant if you previously enabled it). With that setting on, the paste path behaves differently and the skill won't give you the result you want.

**Try it:** in any Claude Code session, type something like `give me a Slack message about the deploy going out Friday morning`. The whole response will be your message, ready to copy.

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
