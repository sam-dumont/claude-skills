---
name: sketch-checker
description: >
  Use when the user asks to check if a band is sketch, fascist, NSBM, or far-right.
  Also trigger for: "is [band] sketch?", "is [band] safe?", "sketch check [band]",
  "check [band] for fascist ties", "is [band] NSBM?", "is [band] fash?",
  "should I listen to [band]?", "is [band] problematic?", "are [band] nazis?",
  "RABM check", "band vetting", "is it sketch", "sketch report",
  "any red flags for [band]", "does [band] have far-right connections",
  "sketch check", "vet this band", "is [band] safe to support".
---

# Sketch Checker: Metal Band Fascist Ties Research

## Overview

Research whether a metal band has ties to far-right, NSBM (National Socialist Black Metal), or fascist movements. Produces a sourced "sketch report" with dual-axis verdicts using the Hellseatic community convention.

This skill uses a **3-phase parallel research methodology**: cast a wide net across community databases, music industry records, and press/interviews simultaneously, then follow up on specific leads, then synthesize into a structured verdict.

**Output:** A markdown report saved to `sketch-reports/{band-name}.md` plus a concise conversation summary.

## Rating System

### Historical Tier

Evaluates the band's entire history from formation to present.

| Tier | Meaning | Criteria |
|------|---------|----------|
| **Safe** | No connections found | No sketch ties in any research phase. Clean across members, labels, splits, and statements. |
| **Mild Sketch** | Tangential or ambiguous connections | One-off split with a sketch band. Appeared on a compilation with sketch acts. Single member's very old side project had sketch ties but was brief. |
| **Moderate Sketch** | Pattern of connections or significant single incident | Multiple splits/collaborations with sketch acts. Played known sketch festivals. Member(s) with sustained sketch side projects. Dog-whistle lyrics that go beyond "edgy." |
| **Major Sketch** | Direct ideological engagement | Band members made public statements supporting far-right ideology. Lyrics with explicit NS themes. Close, sustained ties to NS organizations or labels. Band logo/imagery incorporates explicit NS symbolism (beyond standard BM aesthetic). |
| **Nazi** | Openly NSBM or white supremacist | Self-identifies as NSBM. Explicit NS lyrics across discography. Active in NS organizations. No ambiguity. |

### Current Status

Evaluates the band's **present-day** stance and activity (last ~3 years).

| Tier | Meaning | Criteria |
|------|---------|----------|
| **Safe** | Currently clean | No current sketch ties. May have distanced themselves from past connections. Active statements or actions against far-right. |
| **Mild Sketch** | Minor current concerns | Still on a label with some sketch roster. Hasn't explicitly denounced past connections but isn't actively sketch. |
| **Moderate Sketch** | Active sketch-adjacent behavior | Currently playing sketch festivals. Active collaborations with sketch acts. Ambiguous statements when confronted. |
| **Major Sketch** | Active ideological engagement | Recent far-right statements. Current NS-adjacent projects. Active in sketch scenes. |
| **Nazi** | Currently active NSBM/WS | Self-identifies as NSBM. Active NS organizing. Explicit current propaganda. |

### Rating Guidance

- Weight primary sources (band's own words, credited releases) over secondary sources (forum speculation, anonymous claims)
- **Never round down to be nice.** If evidence points to Moderate Sketch, don't soften it to Mild.
- **Never round up based on genre alone.** Playing black metal is not evidence of anything.
- Satanism, paganism, misanthropy, and general darkness are **NOT** sketch indicators
- If Historical and Current diverge significantly (e.g., Major Sketch / Safe), explain the evolution clearly
- Confidence level should reflect source quality and evidence density

### Confidence Levels

| Level | Meaning |
|-------|---------|
| **High** | Multiple independent sources confirm the assessment. Direct evidence (statements, credited releases). |
| **Medium** | Sources agree but rely partly on secondary evidence. Some gaps in the record. |
| **Low** | Limited sources. Conflicting information. Mostly anonymous/forum-based evidence. |

## Research Architecture: 3-Phase Parallel Swarm

### Phase 1: Core Sources (3 parallel Task agents)

Launch **exactly 3 Task agents in parallel** using the Task tool. Each agent searches independent source categories and returns structured findings.

#### Agent 1: Community Databases & Lists

**Sources to search:**
- **r/rabm** (Reddit: Red and Anarchist Black Metal) — search for band name
- **r/IsItSketch** — search for band name
- **RYM "Bands to Avoid" lists** — search RateYourMusic for the band + "sketch" or "avoid"
- **Hellseatic convention** — web search: `site:hellseatic.de "{band name}"` or `hellseatic "{band name}" sketch`
- **sketchycon.blog** — web search: `site:sketchycon.blog "{band name}"`

**Search queries for each source:**
```
"{band name}" sketch OR nsbm OR fascist OR far-right site:reddit.com/r/rabm
"{band name}" site:reddit.com/r/IsItSketch
"{band name}" sketch OR avoid site:rateyourmusic.com
"{band name}" site:hellseatic.de
"{band name}" site:sketchycon.blog
```

**Reddit access strategy:**
Reddit blocks automated access aggressively. Use this fallback chain:
1. **WebSearch** for `site:reddit.com/r/rabm "{band name}"` — read the snippets in search results
2. If you need full thread content, try **WebFetch** on `old.reddit.com` URLs (more accessible than new Reddit)
3. If WebFetch fails, try Reddit mirror sites: search for `"{band name}" sketch reddit` without site restriction
4. Extract what you can from search snippets — often the key verdict is in the snippet itself

**Agent output template:**
```
## Community Database Findings: {band name}

### r/rabm
- Found: [yes/no]
- Threads: [list URLs and key quotes]
- Community verdict: [summary]

### r/IsItSketch
- Found: [yes/no]
- Threads: [list URLs and key quotes]
- Community verdict: [summary]

### RYM
- Found on avoid lists: [yes/no]
- Details: [summary]

### Hellseatic
- Found: [yes/no]
- Details: [summary]

### sketchycon.blog
- Found: [yes/no]
- Details: [summary]

### Key Leads for Phase 2
- [List any member names, side projects, labels, or connections that need deeper investigation]
```

#### Agent 2: Metal Archives & Music Industry

**Sources to search:**
- **Encyclopaedia Metallum (Metal Archives)** — band page: members (current AND past), discography, labels, side projects for EACH member
- **Bandcamp** — band page, label page, any statements in album descriptions
- **Discogs** — releases, labels, splits/compilations (who are the other bands?)

**Critical: For EACH member listed on Metal Archives, search their side projects.** This is where sketch connections most often hide. A vocalist's ambient side project on a known NS label is a significant finding.

**Known sketch labels to auto-flag:**
Darker Than Black, Hammerbolt, Purity Through Fire (some releases), Heidens Hart, Werwolf (Satanic Warmaster's label), Vinland Winds, Elegy Records, Totenkopf Propaganda, Breath of Pestilence, Signal Rex (some releases), Amor Fati (some releases), Lower Silesian Stronghold, Nebelfee Klangwerke, Militant Zone

**Search queries:**
```
"{band name}" site:metal-archives.com
"{band name}" site:bandcamp.com
"{band name}" discography label site:discogs.com
```

**For each member found on Metal Archives:**
```
"{member name}" metal-archives.com
"{member name}" side project OR other bands
```

**Agent output template:**
```
## Metal Archives & Industry Findings: {band name}

### Band Profile
- Active: [years]
- Country: [country]
- Genre: [listed genre]
- Labels: [list all, flag any known sketch labels]

### Members & Side Projects
For each member:
- **{Member name}** ({instrument}, {years})
  - Side projects: [list each with genre and label]
  - Flagged projects: [any with sketch connections]

### Discography Red Flags
- Splits with: [list split partners, flag any sketch bands]
- Compilations: [list, flag any with sketch acts]
- Label history: [any known sketch labels?]

### Bandcamp/Discogs Notes
- [Any relevant statements, imagery, or connections found]

### Key Leads for Phase 2
- [List specific connections that need deeper investigation]
```

#### Agent 3: Press, Interviews & Public Statements

**Sources to search:**
- **eclipshead.com** — web search: `site:eclipshead.com "{band name}"`
- **Metal press interviews** — web search: `"{band name}" interview` + look for political questions
- **Social media** — search for band's social media presence, look for follows/endorsements
- **Festival lineups** — check if band has played known sketch festivals

**Known sketch festivals to auto-flag:**
Asgardsrei (Ukraine), Hot Shower (Italy, defunct), Steelfest (Finland, some years), Chants de Guerre, Under the Black Sun (some years), Fête du Soleil, Runes & Men (some editions)

**Search queries:**
```
site:eclipshead.com "{band name}"
"{band name}" interview political OR ideology OR nsbm OR fascist
"{band name}" festival lineup
"{band name}" controversy OR problematic OR sketch
```

**Agent output template:**
```
## Press & Public Statements Findings: {band name}

### eclipshead.com
- Found: [yes/no]
- Details: [summary of any coverage]

### Interviews
- [List interviews found with relevant quotes]
- Political statements: [direct quotes if any]

### Social Media
- Platforms found: [list]
- Notable follows/endorsements: [any sketch figures or orgs]
- Statements: [relevant posts]

### Festival History
- Known sketch festivals played: [list with dates]
- Other notable festivals: [context]

### Key Leads for Phase 2
- [List specific claims, connections, or leads that need verification]
```

### Phase 2: Targeted Deep Dive

After Phase 1 agents return, **synthesize their findings** and identify gaps or leads that need follow-up. Launch **1-3 additional Task agents** depending on what was found.

**Only launch Phase 2 agents if Phase 1 produced leads worth chasing.** If Phase 1 returned a clear Safe or clear Nazi verdict with high confidence, Phase 2 may be unnecessary.

#### Deep Dive A: Member Side Projects (if flagged)

If Phase 1 identified specific members with potentially sketch side projects:
- Research each flagged side project in depth
- Check those projects' labels, splits, lyrics, and statements
- Determine if the member's involvement was brief/peripheral or sustained/central

#### Deep Dive B: Label Network (if flagged)

If Phase 1 identified the band on a potentially sketch label:
- Research the label's full roster
- Check the label owner's other projects
- Determine if the label is "sketch by association" (released one NS band among 50) or "sketch by design" (primarily NS/far-right catalog)

#### Deep Dive C: Claim Verification (if needed)

If Phase 1 found conflicting claims or unverified allegations:
- Cross-reference specific claims against primary sources
- Look for the original source of allegations
- Check if the band has publicly responded to allegations

### Phase 3: Synthesis & Report

After all research phases complete, synthesize findings into a structured report.

#### Evidence Weighting

Tier evidence by source reliability:

| Tier | Source Type | Weight |
|------|-----------|--------|
| 1 (Strongest) | Band's own statements, credited releases, official imagery | Highest |
| 2 | Investigative journalism, well-sourced articles (eclipshead, etc.) | High |
| 3 | Community databases, curated lists (r/rabm, Hellseatic, sketchycon) | Medium |
| 4 | Anonymous forum posts, unsourced claims, speculation | Low |

#### Evidence Categories

Distinguish between:

- **Direct evidence**: Band's own words, lyrics, credited projects, official imagery
- **Association evidence**: Splits, shared labels, festival appearances, side project members
- **Aesthetic evidence**: Imagery, song titles, album art that could be sketch or could be standard BM aesthetic

Direct evidence outweighs association evidence, which outweighs aesthetic evidence.

#### Time and Change

- **People can change.** A band member's involvement in a sketch project 15 years ago, followed by 15 years of clean activity and explicit anti-fascist statements, should not get the same weight as current involvement.
- **But track records matter.** A single apology doesn't erase years of NS activity. Look for sustained behavior change, not just PR statements.
- **The Historical Tier captures the past.** The Current Status captures where they are now. This dual-axis system exists specifically to handle evolution.

## Report Format

Save the full report to `sketch-reports/{band-name}.md` (create the directory if it doesn't exist). Use lowercase, hyphenated band name for the filename.

```markdown
# Sketch Report: {Band Name}

**Date:** {YYYY-MM-DD}
**Researcher:** Claude (sketch-checker skill)

## Verdict

| Axis | Rating | Confidence |
|------|--------|------------|
| **Historical** | {tier} | {High/Medium/Low} |
| **Current Status** | {tier} | {High/Medium/Low} |

**Summary:** {2-3 sentence plain-language summary of the verdict and key evidence}

## Band Identity

- **Active:** {years}
- **Country:** {country}
- **Genre:** {genre}
- **Current label:** {label}
- **Current members:** {list}

## Member Analysis

For each member with relevant findings:

### {Member Name} ({instrument})

- **Side projects:** {list}
- **Flagged connections:** {details}
- **Assessment:** {clean / sketch connection with details}

## Lyrics & Imagery

- **Lyrical themes:** {overview}
- **Flagged content:** {specific lyrics, album art, imagery with context}
- **Assessment:** {standard BM aesthetic / dog-whistle concerns / explicit NS themes}

## Labels & Releases

- **Label history:** {list}
- **Flagged labels:** {any known sketch labels with context}
- **Splits/compilations with:** {list, flag sketch acts}

## Festivals & Live Activity

- **Known sketch festivals played:** {list with dates, or "none found"}
- **Other festival context:** {relevant bookings}

## Community Assessment

- **r/rabm verdict:** {summary or "no threads found"}
- **r/IsItSketch verdict:** {summary or "no threads found"}
- **Hellseatic:** {summary or "not listed"}
- **sketchycon.blog:** {summary or "not listed"}
- **Other community sources:** {any additional findings}

## Redemption & Evolution

- **Has the band addressed past connections?** {yes/no, details}
- **Evidence of change:** {specific actions, statements, timeline}
- **Assessment:** {genuine evolution / performative distancing / no change / N/A}

## Sources

{Numbered list of all URLs and sources consulted, with brief descriptions}

1. [Source description](URL)
2. [Source description](URL)
...

## Research Notes

- **Phase 1 findings summary:** {brief overview of what each agent found}
- **Phase 2 investigations:** {what leads were followed and outcomes}
- **Gaps in research:** {what couldn't be verified, what sources were unavailable}
- **Dissenting views:** {any contradictory evidence or alternative interpretations}
```

## Conversation Output

After saving the full report, provide a **concise summary** in the conversation:

```
## {Band Name}: Sketch Check Result

**Historical:** {tier} | **Current:** {tier} | **Confidence:** {level}

{2-3 sentence summary}

**Key findings:**
- {bullet 1}
- {bullet 2}
- {bullet 3}

Full report saved to `sketch-reports/{band-name}.md`
```

## Important Caveats

### What IS Sketch (evidence of far-right ties)

- Lyrics with explicit NS/white supremacist themes (14 words, racial purity, etc.)
- Band members making public far-right statements
- Sustained involvement in NS organizations or movements
- Self-identifying as NSBM
- Splits or close collaborations with known NSBM bands
- Playing festivals organized by and for NS/far-right movements (Asgardsrei, etc.)
- Being on labels with primarily NS/far-right catalogs
- Using explicit NS imagery beyond standard BM aesthetic (actual SS symbols in non-ironic context, swastikas, etc.)
- Band members with sustained side projects in NSBM acts

### What is NOT Sketch (on its own)

- Playing black metal (genre is not ideology)
- Satanic, anti-Christian, or blasphemous lyrics and imagery
- Pagan, heathen, or Norse mythology themes (unless combined with racial ideology)
- Misanthropic lyrics ("I hate all humans equally" is not "I hate specific races")
- Being from Eastern Europe (geography is not ideology)
- "Edgy" or provocative aesthetic (corpse paint, spikes, etc.)
- Having once been on a compilation with a band that later turned out to be sketch
- Using runes or Nordic symbols (unless in explicit NS context)
- Being private or reclusive (not giving interviews is not evidence)
- Playing war-themed music (war metal / bestial BM is a genre, not an ideology)
- Harsh noise or power electronics side projects (genre is not ideology, but check the specifics)

### Edge Cases

**Apolitical band on a sketch label:** This is sketch-adjacent. The label relationship matters: is it a one-off release or their home label? Did they know? Did they leave? Rate the association, not the band's ideology.

**Eastern European context:** Some Eastern European BM scenes have pervasive sketch problems, but individual bands should be evaluated on their own evidence. Being Ukrainian, Polish, or Russian is not evidence. Playing in a local scene with sketch bands is weak evidence. Actively collaborating with sketch bands is evidence.

**Members who left sketch projects:** When did they leave? Did they denounce? What have they done since? A member who left an NS band 10 years ago and has been vocally anti-fascist since is different from one who left last year with no comment.

**"I'm not political" statements:** These are neutral at best. Many sketch bands hide behind "art, not politics." Evaluate the actual evidence, not the PR statement.

**Aesthetic gray zones:** Some bands use imagery that overlaps with far-right aesthetic (Slavic symbols, certain rune configurations, totenkopf imagery) without NS intent. Look at the full context: lyrics, statements, associations, label. One ambiguous symbol is not evidence. A pattern of ambiguous symbols plus sketch associations is evidence.
