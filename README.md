# Sam's Claude Skills

A marketplace collection of custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills.

## Available Skills

| Skill | Description | Tags |
|-------|-------------|------|
| **sams-voice** | Apply Sam Dumont's personal writing voice and style when drafting any written content. Works in English and French. | writing, style, voice, communication |
| **sams-architecture** | Codifies Sam's mature architectural patterns for Python APIs, infrastructure/DevOps, Garmin/embedded systems, and frontend projects. Enforces Service→Repository→Database pattern, 80% test coverage, comprehensive CI/CD, and zero-tolerance security standards. | architecture, python, devops, embedded, security, testing |
| **outcome-engineering** | Reframes tasks as measurable outcomes using o16g principles. Adds outcome specification, execution guardrails, and validation to any workflow. | outcome-engineering, o16g, outcomes, verification |
| **technical-blog-post** | Process skill for turning raw project data (research notes, session logs, drafts, code) into structured technical blog posts. Covers gathering, extraction, structure, and Astro frontmatter. | blog, writing, technical-writing, blog-post, astro, content |
| **sketch-checker** | Research whether a metal band has ties to far-right, NSBM, or fascist movements. Uses parallel research agents with tiered verdicts for both historical and current status. | metal, nsbm, black-metal, sketch, antifascist, rabm, music, research |
| **code-quality** | Comprehensive Python code quality skill. Sets up and runs ruff, mypy, xenon, vulture, file-length gates, and pre-commit hooks. Full Makefile-based workflow using uv. | code-quality, python, linting, ruff, mypy, formatting, complexity |
| **advanced-code-quality** | Advanced Python quality gates beyond basic linting. Covers cognitive complexity (complexipy), code duplication (jscpd), architectural enforcement (import-linter), dependency hygiene (deptry), test quality gates (diff-cover, mutmut), docstring coverage (interrogate), and AI code detection (sloppylint). | advanced-code-quality, complexipy, jscpd, import-linter, deptry, diff-cover, mutation-testing, interrogate, sloppylint |
| **code-security** | Python code security combining static analysis (Bandit, pip-audit) with LLM-powered dynamic analysis for injection, path traversal, auth bypasses, and logic flaws. | security, python, bandit, pip-audit, owasp, sast, vulnerability |

## Installation

Install skills directly from this GitHub repository using Claude Code's marketplace system:

```bash
# Add this marketplace to Claude Code
/plugin marketplace add sam-dumont/claude-skills

# Install specific skills
/plugin install sams-voice@sams-skills
/plugin install sams-architecture@sams-skills
/plugin install outcome-engineering@sams-skills
/plugin install technical-blog-post@sams-skills
/plugin install sketch-checker@sams-skills
/plugin install code-quality@sams-skills
/plugin install advanced-code-quality@sams-skills
/plugin install code-security@sams-skills
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
    "technical-blog-post@sams-skills": true,
    "sketch-checker@sams-skills": true,
    "code-quality@sams-skills": true,
    "advanced-code-quality@sams-skills": true,
    "code-security@sams-skills": true
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

### sketch-checker

Automatically triggered when:
- Asking "is [band] sketch?" or "is [band] safe?"
- Vetting a metal band for fascist, NSBM, or far-right connections
- Requesting a sketch report or RABM check
- Asking about red flags for a specific band

Research methodology:
- **Phase 1: Core Sources** — 3 parallel agents search community databases (r/rabm, r/IsItSketch, Hellseatic, sketchycon.blog), Metal Archives/Bandcamp/Discogs (members, side projects, labels), and press/interviews (eclipshead.com, festivals, social media)
- **Phase 2: Targeted Deep Dive** — 1-3 follow-up agents chase leads from Phase 1 (member side projects, label networks, claim verification)
- **Phase 3: Synthesis & Report** — Dual-axis verdict (Historical Tier + Current Status) using 5 tiers: Safe / Mild Sketch / Moderate Sketch / Major Sketch / Nazi

Reports saved to `sketch-reports/{band-name}.md` with full sourcing.

### code-quality

Automatically triggered when:
- Setting up Python linting, formatting, or type checking
- Configuring ruff, mypy, or pre-commit hooks
- Running code quality checks or quality gates
- Auditing code for complexity, dead code, or file length
- Asking "check my code quality" or "set up linting"

Tool stack:
- **ruff**: Linting + formatting (replaces flake8, isort, black) — 10-100x faster
- **mypy**: Static type checking with strict defaults
- **xenon**: Cyclomatic complexity gating (grade C)
- **vulture**: Dead code detection (90% confidence)
- **pre-commit**: Git hook automation for ruff + mypy
- **Makefile**: All checks as make targets using `uv run` / `uvx`

Provides complete `pyproject.toml` configuration, Makefile targets, and `.pre-commit-config.yaml`.

### advanced-code-quality

Automatically triggered when:
- Going beyond basic linting to enforce advanced quality gates
- Setting up cognitive complexity checks, duplication detection, or architectural enforcement
- Configuring complexipy, jscpd, import-linter, deptry, interrogate, or sloppylint
- Adding mutation testing, diff coverage, or dependency hygiene checks
- Asking "bulletproof Python", "tighten quality gates", or "catch AI-generated code issues"

Complements the code-quality skill (ruff, mypy, xenon, vulture) with 20+ additional tools:
- **Cognitive complexity**: complexipy (threshold ≤10), wily (trend tracking), radon MI
- **Code duplication**: jscpd (≤5%), pylint R0801
- **Architectural enforcement**: import-linter (layer/independence/forbidden contracts), pytestarch
- **Dependency hygiene**: deptry (DEP001–DEP005), pipdeptree
- **Test quality**: diff-cover (≥95% on changed lines), mutmut (≥75% mutation score), pytest-benchmark
- **Documentation**: interrogate (≥95% docstring coverage), griffe (API breaking changes)
- **AI code detection**: sloppylint (hallucinated imports), tightened thresholds for AI-heavy codebases

Includes complete pre-commit config (staged by speed), CI pipeline with parallel jobs and quality gate, and pyproject.toml configuration for all tools.

### code-security

Automatically triggered when:
- Running security scans or auditing code for vulnerabilities
- Checking dependencies for CVEs
- Reviewing code for OWASP top 10 issues
- Asking "is this code secure?" or "find vulnerabilities"
- Setting up a security pipeline

Two-layer approach:
- **Layer 1 — Static Analysis**: Bandit (SAST), pip-audit (dependency CVEs), detect-secrets (credentials), ruff S-rules
- **Layer 2 — LLM Dynamic Analysis**: Claude actively reasons about auth bypasses, path traversal, injection vectors, race conditions, IDOR, insecure deserialization, and logic flaws that no static tool can detect

Includes Makefile targets, CI integration examples, vulnerability pattern reference, and a security hardening checklist.

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
│   ├── technical-blog-post/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── technical-blog-post/
│   │           └── SKILL.md
│   ├── sketch-checker/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── sketch-checker/
│   │           └── SKILL.md
│   ├── code-quality/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── code-quality/
│   │           └── SKILL.md
│   ├── advanced-code-quality/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       └── advanced-code-quality/
│   │           └── SKILL.md
│   └── code-security/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── code-security/
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
