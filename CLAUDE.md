# Claude Skills — Project Instructions

## What This Repo Is

A marketplace of Claude Code skills distributed as plugins. Each skill lives under `plugins/<name>/` with a standard structure.

## Skill Plugin Structure

Every skill follows this exact layout:

```
plugins/{skill-name}/
├── .claude-plugin/
│   └── plugin.json          # Metadata: name, version, description, author, keywords
└── skills/
    └── {skill-name}/
        └── SKILL.md         # Skill definition with YAML frontmatter (name + description triggers)
```

Optional: a `templates/` directory for supporting materials (see sams-voice for an example).

## Current Skills

| Skill | Purpose |
|-------|---------|
| sams-voice | Writing voice and style (English/French) |
| sams-architecture | Architectural patterns (Python/DevOps/Embedded/Frontend) |
| outcome-engineering | Outcome-first workflow framing (o16g) |
| technical-blog-post | Technical blog post writing from raw project data |

## Mandatory Checklist: Adding or Modifying a Skill

When a skill is added, removed, renamed, or has its description changed, ALL of the following files MUST be updated in the SAME commit. No exceptions.

### New Skill

1. **Create** `plugins/{name}/.claude-plugin/plugin.json` — follow the exact schema of existing plugins (name, version "1.0.0", description, author Sam Dumont/sam@dropbars.be, homepage, repository, license MIT, keywords)
2. **Create** `plugins/{name}/skills/{name}/SKILL.md` — YAML frontmatter with `name` and `description` (trigger patterns), then full skill content
3. **Add entry** to `.claude-plugin/marketplace.json` in the `plugins` array — name, source (`./plugins/{name}`), description, version, author, keywords
4. **Add skill** to `.github/workflows/release.yml` — both in the "Available Skills" list AND the installation commands block
5. **Update** `README.md`:
   - Add row to the "Available Skills" table
   - Add install command in the Installation section
   - Add the skill to the "For Teams" settings.json example
   - Add a description section under "Skill Descriptions"
   - Add the skill to the "Repository Structure" tree
6. **Update** `CLAUDE.md` (this file) — add the skill to the "Current Skills" table above

### Removing a Skill

Reverse of the above — remove from all the same files.

### Modifying a Skill's Description or Keywords

Update the description/keywords in ALL places where they appear:
- `plugin.json`
- `marketplace.json`
- `SKILL.md` frontmatter
- `README.md` (table + description section)
- `release.yml` (skill list)

## Versioning

- **Git tags are the version source of truth.** Never manually edit version numbers in plugin.json or marketplace.json.
- Tag format: `v{MAJOR}.{MINOR}.{PATCH}` (e.g., `v1.3.0`)
- The release workflow (`.github/workflows/release.yml`) automatically stamps all manifests with the tag version and commits back to main.
- New skills = minor version bump. Bug fixes to existing skills = patch bump.

## Release Process

1. Commit all changes (skill files + all docs/manifests updated per the checklist above)
2. Push to main
3. Tag: `git tag v{X}.{Y}.{Z}`
4. Push tag: `git push origin v{X}.{Y}.{Z}`
5. The GitHub Action handles the rest (version stamping, GitHub Release creation)

## Conventions

- **Plugin names**: kebab-case, matching directory name exactly
- **Author**: Sam Dumont, sam@dropbars.be
- **License**: MIT (all plugins)
- **Homepage/Repository**: https://github.com/sam-dumont/claude-skills
- **Commit messages**: Descriptive, explain the "why". Include `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>` when Claude assists.
- **SKILL.md frontmatter `description`**: List all natural-language trigger phrases that should activate the skill. Be generous — include variations users might say.

## Anti-Patterns

- Do NOT add a skill without updating ALL documentation files (README, marketplace, release workflow, this file)
- Do NOT manually edit versions — let the release pipeline handle it
- Do NOT commit secrets or private templates (check .gitignore)
- Do NOT create skills with overlapping trigger patterns that would conflict with existing skills
