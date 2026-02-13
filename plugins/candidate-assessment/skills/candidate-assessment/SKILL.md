---
name: candidate-assessment
description: >
  Use when evaluating a candidate for hiring, assessing someone's professional profile,
  preparing for an interview (as interviewer), or doing a self-assessment of your own profile.
  Triggers on: "assess candidate", "evaluate this person", "prepare interview questions",
  "review this CV", "review this resume", "who is [name]", "tell me about [name] as a candidate",
  "assess my profile", "how do I look as a candidate", "self-assessment", "candidate research",
  "pre-interview research", "hiring assessment", "background research on [name]".
  Also triggers when the user provides a CV/resume and asks for analysis, or gives a name
  and asks what can be found about them professionally.
---

# Candidate Assessment

Deep professional assessment combining CV analysis, OSINT research, mentality profiling, and targeted interview question generation. Acts as a hiring manager/detective/psychologist building a complete picture before meeting a candidate.

---

## Step 0: Mode Detection

Before starting, determine the operating mode. Ask the user if not obvious from context.

### Question 1: Assessment Target

| Mode | Trigger | Behavior |
|------|---------|----------|
| **Self-Assessment** | "assess me", "my profile", "how do I look", user provides their own CV | Full linking allowed (after identity verification). CV improvement suggestions included. Can attempt to link pseudonymous profiles. |
| **Evaluating Others** | "assess this candidate", "who is [name]", "review this CV", user provides someone else's details | Ethical boundaries enforced. No pseudonym linking. No CV improvement suggestions. Focus on professional assessment only. |

**Note**: Self-Assessment mode has two tiers:
- **Basic** (no verification needed): Public profile analysis, CV review, company context — all public info
- **Enhanced** (identity verification required): Pseudonym linking, breadcrumb following, full digital footprint mapping. Requires passing a challenge-response gate (see below).

### Question 2: Input Type

| Mode | Trigger | Starting Point |
|------|---------|---------------|
| **CV Mode** | User provides a CV, resume, or LinkedIn profile | Extract structured data from document, then research |
| **Name-Only Mode** | User provides just a name (+ optional job title, location, company) | Build profile from scratch using web research |

### Identity Verification Gate (Self-Assessment Enhanced Mode)

Before linking any pseudonymous profile to the candidate in self-assessment mode, you MUST verify identity through challenge-response.

**When to trigger**: Whenever the research phase discovers a profile that MIGHT belong to the user but isn't explicitly linked on their known profiles (e.g., a Reddit account, an anonymous blog, a forum username).

**How it works**:

1. **Announce the discovery** (without revealing the full profile yet):
   > "I found an account on [platform] that appears to be linked to one of your projects. Before I include this in your assessment, I need to verify you're the owner."

2. **Ask 2-3 challenge questions** based on details from the discovered profile:
   - Questions should be things the account owner would know instantly but a researcher would need significant effort to find
   - Mix semi-public details (subreddit posted in) with less obvious ones (approximate karma, recent comment topic)
   - Example challenges:
     - "What subreddit was this post made in?"
     - "Approximately how many upvotes did the post receive?"
     - "Name one other community you've posted in on this platform"
     - "What was the topic of your most recent comment?"
     - "Approximately how old is this account?"

3. **Evaluate the response**:
   - **2-3 correct** → Verified. Proceed with full linking and include in assessment.
   - **0-1 correct** → Not verified. Do NOT link the profile. Continue assessment as if the profile doesn't exist for this user. Do NOT reveal the discovered username or profile details.

4. **If verification fails**: Do not explain what was found. Simply say "I couldn't verify the connection, so I'll proceed without linking that account." This prevents someone from using the skill to discover another person's pseudonymous accounts.

**Critical**: The challenge questions must be answerable from the profile itself (so you can verify the answers), but not so obvious that someone who just Googled the profile could answer them. Prefer questions about patterns of activity over specific content.

---

## Phase 1: Profile Building

### Path A: CV Mode

Extract the following structured data from the provided CV/resume:

```
## Extracted Profile

**Name**:
**Current Role**:
**Location**:
**Years of Experience**:
**Contact/Links**: [website, GitHub, LinkedIn, etc.]

### Career Timeline
| Period | Company | Role | Duration | Key Responsibilities |
|--------|---------|------|----------|---------------------|
| ... | ... | ... | ... | ... |

### Technical Skills (as claimed)
- Languages:
- Frameworks:
- Infrastructure:
- Other:

### Education & Certifications

### Notable Claims
[List any specific achievements, metrics, or notable claims that should be verified]

### First Impressions
- What story does this CV tell?
- What's the narrative arc of this career?
- What stands out immediately (positive or concerning)?
- What's conspicuously absent?
```

**If a website or portfolio URL is provided**: Visit it. Note the tech stack, design quality, content depth, and any hidden details (analytics tools, meta tags, Easter eggs). A personal site reveals intentionality.

### Path B: Name-Only Mode

Build an initial profile using web research:

1. **Start broad**: Search for "Name" + job title + location
2. **Find anchors**: Look for LinkedIn profile, GitHub profile, company bio page
3. **Cross-reference**: Use unique project names, company names, or specializations to confirm identity
4. **Build the same structured profile** as CV Mode, but note confidence level for each data point

Search strategy for Name-Only:
- `"First Last" site:linkedin.com [job title]`
- `"First Last" site:github.com`
- `"First Last" [company name]`
- `"First Last" [specialization] [location]`

**Important**: In Name-Only mode, clearly mark what is confirmed vs. inferred. If multiple people share the name, ask the user to disambiguate.

---

## Phase 2: Deep Research (4 Parallel Agents)

Launch 4 research agents in parallel. Each has a specific focus area.

### Agent A: Online Presence & Digital Footprint

Research the candidate's public digital presence.

**Professional Platforms**:
- GitHub: contribution graph, repo quality, languages, stars, open-source involvement
- Stack Overflow: reputation, top tags, answer quality
- LinkedIn: recommendations, endorsements, activity, post quality
- Personal website/blog: content depth, writing quality, topics, update frequency

**Community Presence**:
- Conference talks, meetup presentations (search YouTube, SlideShare, Speaker Deck)
- Podcast appearances
- Published articles or papers
- Forum participation (Hacker News, relevant subreddits, Discord servers)
- Mailing list archives (for open-source contributors)

**Package/Project Registries**:
- npm, PyPI, crates.io, Maven Central (depending on tech stack)
- Docker Hub
- Garmin Connect IQ Store, App Store, Play Store (if applicable)

**Reddit Search Protocol** (learned from live testing):
- Use PLAIN KEYWORDS, not exact-match quotes — `rkd telemetry extractor site:reddit.com` works, `"rkd-telemetry-extractor" site:reddit.com` often doesn't
- Search for PROJECT NAMES, not person names
- One Reddit post → username → full post/comment history → interests, tone, expertise areas
- Check r/[relevant-subreddits] for domain-specific activity

**Profile Linking Ethics**:

| Mode | Rule |
|------|------|
| **Self-Assessment (Basic)** | Use only profiles that are publicly linked under the candidate's real name. No pseudonym linking at this tier. |
| **Self-Assessment (Enhanced)** | Full linking allowed ONLY AFTER passing the identity verification gate (see Step 0). Follow breadcrumb chains (e.g., GitHub project mentioned on Reddit → Reddit username → post history). If verification fails, do NOT reveal what was found. |
| **Evaluating Others** | ONLY use profiles that are **publicly and explicitly linked by the candidate themselves**. If GitHub profile says "John Smith" and a Reddit user "xXcoder99Xx" posted about the same project but there's no public link between them → **DO NOT CONNECT THEM**. The candidate chose to keep those identities separate. Respect that. NEVER attempt to link. NEVER trigger the verification gate — it does not apply in this mode. |

**Absence of signal is NOT negative signal.** Many excellent engineers have minimal online presence. Not having a blog, GitHub contributions, or Stack Overflow reputation says nothing about competence. Note the absence without judgment.

### Agent B: Claim Verification

Verify specific claims from the CV or profile:

**Company Verification**:
- Do the companies exist? Are they still operating?
- Company size, funding stage, industry
- Company reputation (Glassdoor, press coverage)
- Did the company achieve what the candidate implies? (e.g., "led team at successful startup" — was it actually successful?)

**Role Verification**:
- Job title inflation check — is "CTO at 3-person startup" being presented as equivalent to "CTO at 500-person company"?
- Timeline consistency — do dates make sense? Any unexplained gaps?
- Responsibility claims — are the claimed achievements plausible for the role and company size?

**Technical Claims**:
- If they claim expertise in X, does their GitHub/open-source work reflect it?
- If they claim to have built Y, can you find evidence of Y existing?
- Patent searches (Google Patents, USPTO) for claimed inventions
- Published papers (Google Scholar, arXiv) for claimed research

**Education Verification**:
- Institution exists and offers the claimed program
- Timeline plausibility

**What NOT to verify** (ethical boundaries):
- No salary history
- No political views or affiliations
- No health information
- No family status
- No protected characteristics (age, gender, race, religion, disability)
- No criminal records (not your role — that's for formal background checks with consent)

### Agent C: Company & Market Context

Understand the context around each employer:

- **Company trajectory**: Was it growing, stable, or declining during the candidate's tenure?
- **Industry context**: What was happening in their industry? (e.g., crypto winter, AI boom, COVID pivot)
- **Team size and structure**: How big was the engineering team? What was the candidate's scope?
- **Technology choices**: What stack did the company use? Does it align with the candidate's claimed skills?
- **Exits and outcomes**: Did the company IPO, get acquired, shut down? This contextualizes the candidate's experience.

This context matters because "Senior Engineer at [company]" means very different things depending on whether the company had 5 or 5000 engineers.

### Agent D: Reference & Reputation Signals

Look for indirect reputation signals:

- **Recommendations on LinkedIn**: Who wrote them? What do they highlight?
- **Endorsements patterns**: What skills do colleagues endorse most?
- **Co-authored work**: Papers, projects, or talks with collaborators
- **Open-source contributions**: PRs to other projects, code review quality, community interactions
- **Mentions by others**: Blog posts, articles, or talks that reference the candidate's work
- **Professional network quality**: Who do they interact with publicly?

**Signal Tiers**:

| Tier | Signal Type | Weight |
|------|------------|--------|
| **Tier 1 (Strong)** | Published talks at known conferences, merged PRs to major projects, peer-reviewed papers, patents | High confidence signal |
| **Tier 2 (Moderate)** | Blog posts with technical depth, Stack Overflow reputation, LinkedIn recommendations from credible people, GitHub repos with real users | Medium confidence signal |
| **Tier 3 (Contextual)** | Social media activity, community participation, forum posts, casual mentions | Low confidence — useful for personality/culture fit, not technical ability |
| **Meta-signal** | Absence of any online presence | NOT negative. Many excellent engineers are private. Note it neutrally. |

---

## Phase 3: Synthesis & Output

Compile all research into a structured assessment.

### 3.1 Executive Summary

2-3 paragraphs covering:
- Who is this person, in plain language?
- What's the career narrative? (trajectory, not just timeline)
- What's the overall signal quality? (strong evidence, moderate, or mostly self-reported?)
- Initial hiring thesis: what role would this person excel in? What role would be a poor fit?

### 3.2 Technical Depth Assessment

| Domain | Claimed Level | Evidence Level | Confidence | Notes |
|--------|--------------|----------------|------------|-------|
| [Skill 1] | Expert | Strong GitHub evidence | High | [specifics] |
| [Skill 2] | Advanced | No public evidence | Low | Self-reported only |
| [Skill 3] | Intermediate | Conference talk found | Medium | [specifics] |

**Evidence Levels**:
- **Strong**: Multiple independent sources confirm (code, talks, papers, peer endorsements)
- **Moderate**: Some evidence supports (one project, one reference, consistent narrative)
- **Weak**: Self-reported only, no corroboration
- **Contradictory**: Evidence suggests different level than claimed

### 3.3 Mentality Profile

Assess across 8 dimensions based on all available evidence. Use a spectrum, not a binary.

| Dimension | Assessment | Evidence |
|-----------|-----------|----------|
| **Builder vs. Maintainer** | Where do they fall? Do they start things or finish them? | [What led to this assessment] |
| **Solo vs. Collaborative** | Do they work alone or on teams? How do they interact? | [Evidence] |
| **Depth vs. Breadth** | Specialist or generalist? T-shaped? | [Evidence] |
| **Process vs. Chaos** | Structured methodology or improvisational? | [Evidence] |
| **Ownership vs. Delegation** | Do they take end-to-end ownership or hand off? | [Evidence] |
| **Learning Velocity** | How quickly do they pick up new domains? Evidence of pivots? | [Evidence] |
| **Communication Style** | Technical writer? Casual? Precise? Verbose? | [Evidence from blog posts, PRs, talks] |
| **Risk Tolerance** | Conservative tech choices or bleeding edge? Side projects? | [Evidence] |

**Important**: These are descriptive, not prescriptive. "Builder" is not better than "Maintainer" — it depends on what the role needs. Frame each dimension as fit-for-role, not good/bad.

### 3.4 Strengths & Concerns

**Strengths** (with evidence):
- [Strength 1]: [Evidence supporting this]
- [Strength 2]: [Evidence supporting this]
- [Strength 3]: [Evidence supporting this]

**Concerns** (with evidence and severity):
- [Concern 1]: [Evidence] — Severity: [Low/Medium/High] — [Can be verified in interview? Y/N]
- [Concern 2]: [Evidence] — Severity: [Low/Medium/High] — [Can be verified in interview? Y/N]

**Yellow Flags** (things that aren't concerns but warrant attention):
- [Flag 1]: [Why it caught attention]

### 3.5 Churn Risk Matrix

| Risk Factor | Assessment | Evidence |
|-------------|-----------|----------|
| **Job hopping pattern** | [Average tenure, trend direction] | [Timeline analysis] |
| **Career trajectory** | [Upward, lateral, stagnant, exploratory] | [Role progression] |
| **Side projects** | [Active side projects suggesting potential distraction or entrepreneurial drive] | [GitHub, personal site] |
| **Industry alignment** | [How aligned is their background with your company's domain?] | [Career history] |
| **Seniority fit** | [Overqualified? Underqualified? Right level?] | [Role history vs. target role] |
| **Location/remote** | [Any signals about work arrangement preferences] | [Profile, company history] |

### 3.6 Signal Gaps & Open Questions

What couldn't you find? What remains unverified?

- [Gap 1]: Could not verify [claim]. Recommend asking in interview.
- [Gap 2]: No public evidence of [skill]. May be valid but unverifiable from OSINT.
- [Gap 3]: Timeline gap between [date] and [date]. Could be sabbatical, personal reasons, or unlisted role.

**Important**: Frame gaps neutrally. A gap is not evidence of a problem — it's absence of information.

### 3.7 CV Improvement Recommendations (SELF-ASSESSMENT MODE ONLY)

> **This section ONLY appears when the user is assessing their own profile.** Never include this when evaluating someone else — it would be presumptuous and misplaced.

Constructive feedback on CV/profile presentation:
- What important context is missing? (company size, impact metrics, team sizes)
- Where could claims be more specific or quantified?
- What story is undersold that should be more prominent?
- What structural changes would make the CV clearer?
- What sections are well done and should be kept as-is?

### 3.8 Targeted Interview Questions

Generate 10-15 interview questions specifically tailored to THIS candidate. Not generic questions — questions that probe the gaps, verify the claims, and test the concerns identified above.

**Structure each question as**:

```
**Question**: [The actual question]
**Why ask this**: [What you're trying to learn/verify]
**What good looks like**: [What a strong answer contains]
**Red flag answer**: [What would be concerning]
```

**Categories**:
- 3-4 questions to **verify specific claims** from CV
- 2-3 questions to **probe identified gaps** or concerns
- 2-3 questions to **assess mentality dimensions** relevant to the role
- 2-3 questions to **test technical depth** in claimed areas
- 1-2 questions to **assess culture fit** based on observed communication style

---

## Anti-Patterns

### This is NOT a background check
- Do not try to find personal information (address, family, financial status)
- Do not dig into protected characteristics
- Do not access any non-public information
- Do not present speculation as fact

### This is NOT character assassination
- Do not frame neutral observations as negatives
- Do not use inflammatory language
- "Absence of evidence is not evidence of absence" — always
- Do not penalize candidates for having a private online presence

### This is NOT a one-pass scan
- Do not stop at the first Google result
- Do not rely on a single source
- Cross-reference claims across multiple sources
- When in doubt about a fact, mark it as "unverified" rather than stating it

### Do NOT introduce bias
- Do not favor candidates with strong online presence over those without
- Do not penalize career gaps (they could be anything: caregiving, health, travel, sabbatical)
- Do not assume prestige of company = quality of candidate
- Do not assume a non-traditional path is inferior to a traditional one
- Do not assume GitHub activity = engineering quality (many excellent engineers work on proprietary code)

### Do NOT doxx
- In "Evaluating Others" mode, NEVER attempt to link pseudonymous accounts
- If you discover a likely pseudonymous connection by accident, DO NOT include it in the report
- The candidate's right to maintain separate identities is absolute
- Only use information the candidate has chosen to make publicly accessible under their real identity

---

## OSINT Search Strategy Reference

### Effective Search Patterns

| Goal | Search Pattern | Notes |
|------|---------------|-------|
| Find LinkedIn | `"First Last" site:linkedin.com [job title]` | Most reliable starting point |
| Find GitHub | `"First Last" site:github.com` or search GitHub directly | Check contributions, not just repos |
| Find Reddit activity | `[project name] site:reddit.com` (NO quotes on project name) | Plain keywords work better than exact match |
| Find talks/presentations | `"First Last" [topic] site:youtube.com OR site:slideshare.net` | Also check conference websites directly |
| Find publications | `"First Last" site:scholar.google.com` or `"First Last" arxiv` | For research-oriented candidates |
| Find packages | Search npm/PyPI/crates.io directly for username | Often matches GitHub handle |
| Find patent filings | `"First Last" site:patents.google.com` | For senior/inventive roles |
| Verify company claims | `"[Company Name]" crunchbase OR glassdoor OR linkedin` | Cross-reference size, funding, status |
| Find archived content | `"First Last" site:web.archive.org` | For deleted blogs/sites |

### Search Tips (Learned from Live Testing)
- Plain keywords consistently outperform exact-match quoted strings for Reddit
- Search for unique project names or technical terms, not the person's name
- One confirmed username on any platform opens up their full activity on that platform
- Conference speaker pages often have bios with links to other profiles
- Git commit emails can sometimes be found in public repos (ethically: only if the repo is public)
- Company "About" or "Team" pages often have richer bios than LinkedIn

### Platform-Specific Notes
- **Reddit**: Web search tools have poor Reddit indexing. Try multiple query formulations. If web search fails, note it as a limitation rather than concluding no Reddit presence exists.
- **GitHub**: Stars and followers are vanity metrics. Look at: contribution frequency, code quality in repos, PR review comments, issue discussions.
- **LinkedIn**: Recommendations from senior people carry more weight. Look at WHO recommends them, not just how many recommendations.
- **Stack Overflow**: Look at the questions they ask, not just answers. Someone asking good questions in advanced topics signals genuine learning depth.
