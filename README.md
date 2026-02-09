# claude-skills

A collection of custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills.

## Available Skills

| Skill | Description |
|-------|-------------|
| [sams-voice.md](./sams-voice.md) | Apply Sam Dumont's personal writing voice and style when drafting any written content. Works in English and French. |
| [sams-architecture.md](./sams-architecture.md) | Codifies Sam's mature architectural patterns for Python APIs, infrastructure/DevOps, Garmin/embedded systems, and frontend projects. Automatically triggers during project planning, architecture reviews, and codebase audits. Enforces Service→Repository→Database pattern, 80% test coverage, comprehensive CI/CD, and zero-tolerance security standards for personal projects. Adapts recommendations for client work. |

## Installation

Claude Code skills are Markdown files with YAML frontmatter that live in your `~/.claude/skills/` directory.

### Install a single skill

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill file
cp sams-voice.md ~/.claude/skills/
```

### Install all skills from this repo

```bash
mkdir -p ~/.claude/skills
cp *.md ~/.claude/skills/
```

> **Note:** This copies `README.md` too, which is harmless - Claude Code only triggers skills that have valid YAML frontmatter with a `name` and `description`.

### Verify installation

```bash
ls ~/.claude/skills/
```

Once installed, the skill activates automatically when Claude Code detects a matching prompt (e.g. "write a blog post in my voice", "draft a Slack message for me").

## Writing Your Own Skills

A skill is a Markdown file with YAML frontmatter:

```yaml
---
name: my-skill
description: >
  When to trigger this skill. Claude uses this text to decide
  if the skill is relevant to the current prompt.
---

# Skill Title

Instructions for Claude go here...
```

Drop the file into `~/.claude/skills/` and it's ready to use.
