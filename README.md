# Sam's Claude Skills

A marketplace collection of custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills.

## Available Skills

| Skill | Description | Tags |
|-------|-------------|------|
| **sams-voice** | Apply Sam Dumont's personal writing voice and style when drafting any written content. Works in English and French. | writing, style, voice, communication |
| **sams-architecture** | Codifies Sam's mature architectural patterns for Python APIs, infrastructure/DevOps, Garmin/embedded systems, and frontend projects. Enforces Service→Repository→Database pattern, 80% test coverage, comprehensive CI/CD, and zero-tolerance security standards. | architecture, python, devops, embedded, security, testing |

## Installation

Install skills directly from this GitHub repository using Claude Code's marketplace system:

```bash
# Add this marketplace to Claude Code
/plugin marketplace add sam-dumont/claude-skills

# Install specific skills
/plugin install sams-voice@sams-skills
/plugin install sams-architecture@sams-skills
```

**That's it!** Skills are now available and auto-trigger when Claude detects matching prompts.

### Update Skills

```bash
# Update the marketplace to get latest versions
/plugin marketplace update sams-skills
```

### Remove Skills

```bash
# Remove a specific skill
/plugin uninstall sams-voice@sams-skills

# Remove the entire marketplace
/plugin marketplace remove sams-skills
```

## For Teams: Auto-Install Skills

Add to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "sams-skills": {
      "source": {
        "source": "github",
        "repo": "sam-dumont/claude-skills"
      }
    }
  },
  "enabledPlugins": {
    "sams-voice@sams-skills": true,
    "sams-architecture@sams-skills": true
  }
}
```

Now all team members automatically get these skills when they open the project!

## Skill Descriptions

### sams-voice

Automatically triggered when drafting written content like:
- Blog posts and articles
- Email communications
- Slack/Discord messages
- Documentation
- Social media posts
- Comments and reviews

Applies Sam's voice characteristics:
- Direct, conversational French-Canadian English
- Technical precision without jargon
- Authentic personality (humor, strong opinions)
- Code-first examples
- Bilingual support (English/French)

### sams-architecture

Automatically triggered when:
- Planning new projects
- Designing system architecture
- Conducting architecture reviews
- Making technology choices
- Setting up CI/CD pipelines
- Auditing existing codebases

Enforces patterns:
- Python APIs: FastAPI + Service→Repository→Database
- Testing: 80% minimum coverage
- Security: Zero-tolerance (auth, validation, scanning)
- CI/CD: Comprehensive automation
- Embedded: Memory optimization for Garmin/MonkeyC
- Infrastructure: Terraform + Kubernetes
- Documentation: CLAUDE.md + README + ADRs

## Repository Structure

\`\`\`
claude-skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest
├── plugins/
│   ├── sams-voice/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin metadata
│   │   └── skills/
│   │       └── sams-voice/
│   │           └── SKILL.md      # Skill definition
│   └── sams-architecture/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── sams-architecture/
│               └── SKILL.md
└── README.md
\`\`\`

## Adding New Skills

1. Create plugin structure:
\`\`\`bash
mkdir -p plugins/my-skill/{.claude-plugin,skills/my-skill}
\`\`\`

2. Create plugin manifest (\`plugins/my-skill/.claude-plugin/plugin.json\`):
\`\`\`json
{
  "name": "my-skill",
  "version": "1.0.0",
  "description": "What this skill does",
  "author": { "name": "Your Name" },
  "keywords": ["relevant", "tags"]
}
\`\`\`

3. Create skill file (\`plugins/my-skill/skills/my-skill/SKILL.md\`):
\`\`\`yaml
---
name: my-skill
description: >
  When this skill should trigger
---

# Skill instructions here
\`\`\`

4. Add to marketplace manifest (\`.claude-plugin/marketplace.json\`)

5. Test locally:
\`\`\`bash
/plugin marketplace add ./
/plugin install my-skill@sams-skills
\`\`\`

## License

MIT

## Author

**Sam Dumont**
- GitHub: [@sam-dumont](https://github.com/sam-dumont)
- Website: [dropbars.be](https://dropbars.be)
