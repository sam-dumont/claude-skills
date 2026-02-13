# Sam's Claude Skills

A marketplace collection of custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills.

## Available Skills

| Skill | Description | Tags |
|-------|-------------|------|
| **sams-voice** | Apply Sam Dumont's personal writing voice and style when drafting any written content. Works in English and French. | writing, style, voice, communication |
| **sams-architecture** | Codifies Sam's mature architectural patterns for Python APIs, infrastructure/DevOps, Garmin/embedded systems, and frontend projects. Enforces Service→Repository→Database pattern, 80% test coverage, comprehensive CI/CD, and zero-tolerance security standards. | architecture, python, devops, embedded, security, testing |
| **outcome-engineering** | Reframes tasks as measurable outcomes using o16g principles. Adds outcome specification, execution guardrails, and validation to any workflow. | outcome-engineering, o16g, outcomes, verification |
| **candidate-assessment** | Deep professional assessment combining CV analysis, OSINT research, mentality profiling, and targeted interview question generation. Supports self-assessment and evaluating others with appropriate ethical boundaries. | hiring, interview, osint, cv-review, recruitment |
| **technical-blog-post** | Process skill for turning raw project data (research notes, session logs, drafts, code) into structured technical blog posts. Covers gathering, extraction, structure, and Astro frontmatter. | blog, writing, technical-writing, blog-post, astro, content |

## Installation

Install skills directly from this GitHub repository using Claude Code's marketplace system:

```bash
# Add this marketplace to Claude Code
/plugin marketplace add sam-dumont/claude-skills

# Install specific skills
/plugin install sams-voice@sams-skills
/plugin install sams-architecture@sams-skills
/plugin install outcome-engineering@sams-skills
/plugin install candidate-assessment@sams-skills
/plugin install technical-blog-post@sams-skills
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
    "sams-architecture@sams-skills": true,
    "outcome-engineering@sams-skills": true,
    "candidate-assessment@sams-skills": true,
    "technical-blog-post@sams-skills": true
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

### outcome-engineering

Automatically triggered when:
- Starting significant features or projects
- Making build-vs-buy decisions
- Reframing failed approaches
- Mentioning "outcome engineering", "o16g", or "what's the outcome"

Based on the [o16g manifesto](https://o16g.com/):
- Define measurable outcomes, not implementations
- Specify verification criteria upfront
- Justify the cost before building
- Identify risk gates early
- When uncertain, form hypotheses and test cheaply

### candidate-assessment

Automatically triggered when:
- Evaluating a candidate for hiring
- Reviewing a CV or resume
- Preparing interview questions
- Doing a self-assessment of your own profile
- Researching someone's professional background

Two operating modes:
- **Self-Assessment**: Full profile analysis including pseudonym linking (with identity verification), CV improvement suggestions
- **Evaluating Others**: Ethical boundaries enforced — no pseudonym linking, no CV suggestions, professional assessment only

Delivers:
- Structured profile extraction (CV or name-only input)
- 4-agent parallel deep research (digital footprint, claim verification, company context, reputation signals)
- 8-dimension mentality profile
- Technical depth assessment with evidence levels
- Churn risk matrix
- 10-15 targeted interview questions specific to the candidate

### technical-blog-post

Automatically triggered when:
- Writing a technical blog post from raw project data
- Sam says "write a blog post about [project]" or "turn this into a post"
- Raw source material exists (research notes, session logs, drafts, code)

Process workflow:
- **Phase 1: Gather** — Find all source material (README, research notes, specs, session logs, drafts, code)
- **Phase 2: Extract** — Pull out the six-beat narrative arc (hook, problem, approach, discoveries, validation, result)
- **Phase 3: Write** — Structure with Astro frontmatter, technical deep-dives, and real data
- **Phase 4: Voice Check** — Verify against `sams-voice` standards (no LLM-isms, real numbers, varied structure)
- **Phase 5: Astro Frontmatter** — Correct metadata with back-publishing support

## Repository Structure

```
claude-skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest
├── plugins/
│   ├── sams-voice/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin metadata
│   │   ├── skills/
│   │   │   └── sams-voice/
│   │   │       └── SKILL.md      # Skill definition
│   │   └── templates/            # Supporting materials
│   ├── sams-architecture/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── sams-architecture/
│   │           └── SKILL.md
│   ├── outcome-engineering/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── outcome-engineering/
│   │           └── SKILL.md
│   ├── candidate-assessment/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── candidate-assessment/
│   │           └── SKILL.md
│   └── technical-blog-post/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── technical-blog-post/
│               └── SKILL.md
├── CLAUDE.md                     # Project instructions & maintenance checklist
└── README.md
```

## Adding New Skills

1. Create plugin structure:
```bash
mkdir -p plugins/my-skill/{.claude-plugin,skills/my-skill}
```

2. Create plugin manifest (`plugins/my-skill/.claude-plugin/plugin.json`):
```json
{
  "name": "my-skill",
  "version": "1.0.0",
  "description": "What this skill does",
  "author": { "name": "Your Name" },
  "keywords": ["relevant", "tags"]
}
```

3. Create skill file (`plugins/my-skill/skills/my-skill/SKILL.md`):
```yaml
---
name: my-skill
description: >
  When this skill should trigger
---

# Skill instructions here
```

4. Add to marketplace manifest (`.claude-plugin/marketplace.json`)

5. Update documentation — see `CLAUDE.md` for the full mandatory checklist

6. Test locally:
```bash
/plugin marketplace add ./
/plugin install my-skill@sams-skills
```

## License

MIT

## Author

**Sam Dumont**
- GitHub: [@sam-dumont](https://github.com/sam-dumont)
- Website: [dropbars.be](https://dropbars.be)
