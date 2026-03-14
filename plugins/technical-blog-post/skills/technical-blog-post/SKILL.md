---
name: technical-blog-post
description: Use when writing a technical blog post from raw project data like research notes, session logs, Reddit/LinkedIn drafts, code, hex dumps, or screenshots. Also use when Sam says "write a blog post about [project]" or "turn this into a post".
---

# Technical Blog Post from Raw Data

## Overview

A process skill for turning raw project material into a structured technical blog post. Complements `sams-voice` (which handles tone and language): this skill handles structure and workflow.

**REQUIRED:** Always invoke `sams-voice` skill alongside this one. Voice = how it sounds. This skill = how it's organized.

## When to Use

- Sam asks to write a blog post about a project
- Raw source material exists (research notes, session logs, drafts, code)
- The post is technical: "I built/decoded/reverse-engineered/automated X"

## When NOT to Use

- Opinion pieces or commentary (no raw data to process)
- Short announcements (no structure needed)
- Non-technical content

## Process

### Phase 1: Gather

Find and read all source material in the project repo. Look for:

| Source Type | Where to Find | What It Contains |
|-------------|---------------|------------------|
| README | Root | Project summary, usage, results |
| Research notes | `RESEARCH_NOTES.md`, `docs/` | Chronological narrative of discoveries |
| Format specs | `*_SPEC.md`, `*_FORMAT.md` | Technical details, struct layouts |
| Session logs | `SESSION_LOG.md` | Claude Code session summaries |
| Drafts | `.nocommit/`, `docs/drafts/` | Reddit, LinkedIn, or blog drafts |
| Code | `src/`, `*.py`, `*.go` | Implementation details |
| Tests | `tests/`, `*_test.*` | Validation approach, edge cases |
| Binary data | `*.hex`, hex dumps in notes | Raw data examples |

**Key:** Don't just read: extract the narrative arc. What was the problem? What failed? What worked? What was surprising?

### Phase 2: Extract the Story

Pull out the six-beat narrative from the source material:

1. **The hook**: What's the one-sentence version? (e.g., "Five years of telemetry trapped in a proprietary format")
2. **The problem**: Why did this matter? What was at stake? What failed before?
3. **The approach**: How was it tackled? What was the methodology? What tools?
4. **The discoveries**: What were the key technical findings? Order by "aha moment", not chronology.
5. **The validation**: How was each discovery confirmed? Real numbers, cross-checks.
6. **The result**: What was delivered? Links, numbers, concrete output.

### Phase 3: Write Using This Structure

```markdown
---
title: "[Action verb]-ing [specific thing] [qualifier]"
pubDate: YYYY-MM-DD          # Use the actual project date, not today
description: "One sentence. Specific. Includes key technologies and outcome."
tags: ["relevant", "specific", "tags"]
draft: false
---

[HOOK: 1-2 sentences. Jump straight in. No "In today's..." opener.]

## [The problem / context section]

[2-3 paragraphs: what existed, what didn't work, why it mattered.
Include real specifics: dates, model names, circuit names, file sizes.]

## [The approach / methodology]

[How it was tackled. The human-AI split if relevant.
The loop: hypothesis > test > validate > next.]

## [Technical deep-dive]

[The meat. Structured by component/layer, not chronology.
Include:]
- ASCII struct diagrams
- Hex dump snippets with annotations
- Code blocks (keep short: 5-15 lines each)
- Real numbers from real data
- Cross-validation results

[Break into subsections with ### headers for each component.]

## [Validation]

[How each discovery was confirmed with real-world data.
Specific numbers: "<1% error", "exactly 9.81 m/s^2", etc.]

## [Result + what it means]

[Concrete deliverables. Links to repo, videos, docs.
Open questions if any remain.
What worked well, what needed human judgment.]
```

### Phase 4: Voice Check

Before finalizing, run these checks (from `sams-voice`):

- [ ] No LLM-isms (delve, leverage, navigate, landscape, etc.)
- [ ] No em dashes: use colons instead
- [ ] No performative writing ("So here's the story", "Buckle up")
- [ ] Opens by jumping straight into the topic
- [ ] Affirmative statements, not question-led setups
- [ ] Real numbers everywhere (not "a few hours" but "4 hours")
- [ ] Varied paragraph lengths (some one sentence, some five)
- [ ] Parenthetical asides for clarification and humor
- [ ] Strong opinions backed with evidence

### Phase 5: Astro Frontmatter

For Sam's blog specifically (`site/src/content/blog/`):

```yaml
---
title: string              # Lowercase after first word unless proper noun
pubDate: YYYY-MM-DD        # The actual project completion date, not publication date
description: string        # 1 sentence, specific, includes key tech
tags: string[]             # Lowercase, specific (not "technology" but "python")
draft: boolean             # false when ready
image: string              # Optional, path to hero image
heroImage: string          # Optional, path to AI-generated hero (e.g. "/heroes/my-post.jpg")
heroImagePrompt: string    # Optional, the prompt used to generate heroImage (shown on page)
---
```

**Back-publishing:** When writing about past projects, set `pubDate` to the project date. The blog should read as a timeline of work, not a dump of posts all dated today.

### Phase 6: OG Hero Image Prompt

Generate an image generation prompt (for Gemini/Imagen) to create a hero image for the post's Open Graph social preview. The image will be used as a background in a 1200×630 OG card with a dark overlay and text on top.

**Prompt template:**

> A dark moody digital illustration of [SCENE SPECIFIC TO THE POST'S CORE CONCEPT]. [KEY VISUAL ELEMENT 1] and [KEY VISUAL ELEMENT 2]. Color palette: deep navy (#101218), electric blue (#468CDC) accents, [ONE ADDITIONAL ACCENT COLOR RELEVANT TO THE TOPIC]. Technical illustration style, slightly stylized, 16:9 landscape, dark background suitable for text overlay.

**Rules for good prompts:**

| Rule | Why |
|------|-----|
| Anchor to the post's central metaphor | A post about binary reverse-engineering → hex dumps flowing over a race car, not a generic "code on screen" |
| Always specify dark background (#101218) | The OG template uses dark theme colors; a bright image won't blend |
| Include the accent blue (#468CDC) | Maintains brand consistency across all cards |
| Add one topic-specific accent color | Warm amber for legacy systems, orange for racing/speed, green for git |
| Say "suitable for text overlay" | Prevents the generator from filling the entire frame with detail |
| Keep it 16:9 landscape | Matches the 1200×630 OG dimensions |
| "Technical illustration, slightly stylized" | Avoids photorealism (uncanny) and pure cartoon (unprofessional) |
| Reference concrete objects from the post | Not "technology concept" but "weathered terminal connected to modern API" |

**Output format:** Include the prompt in a fenced block at the end of the blog post draft, tagged for easy extraction:

```
<!-- og-hero-prompt
[The full image generation prompt here]
-->
```

The author generates the image externally (Gemini, DALL-E, etc.), saves it as `site/public/heroes/{slug}.jpg`, and adds both fields to the frontmatter:

```yaml
heroImage: /heroes/my-post-slug.jpg
heroImagePrompt: "The full prompt used to generate this image"
```

The blog template displays the hero image above the article content with a collapsible "AI-generated image — show prompt" caption that reveals the `heroImagePrompt` text. This is intentional transparency about AI-generated visuals.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Chronological dump of session notes | Structure by component/discovery, not timeline |
| Too much Claude praise | Focus on what was built, not the tool |
| Vague "it was hard" | Specific: "tried Unix epoch, dates were 40 years off" |
| Missing validation | Every claim needs a cross-check with real data |
| Wall of code | Max 15 lines per block. Annotate, don't dump. |
| Burying the result | Concrete deliverable in first 3 paragraphs |
