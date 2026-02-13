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

### Skills & Expertise (as claimed)
Adapt categories to the candidate's domain:
- **Technical roles**: Languages, Frameworks, Infrastructure, Tools
- **Business roles**: Methodologies, Markets, Industries, Tools (CRM, ERP, etc.)
- **Creative roles**: Media, Tools, Styles, Formats
- **Licensed professions**: Licenses, Certifications, Jurisdictions, Specializations
- **Academic roles**: Research areas, Methods, Grants, Teaching subjects

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
- `"First Last" site:getprog.ai` — for tech candidates, getprog.ai mirrors full LinkedIn profiles (experience, education, skills, about). WebFetch the result URL for complete data.
- `"First Last" site:linkedin.com [job title OR institution OR location]` — Google/Bing snippets show headline, role, location even without LinkedIn access. **When multiple results appear, evaluate ALL of them** — the correct profile may not be the first result. Cross-reference each result's headline, location, and institution against known data before dismissing any.
- `"First Last" site:linktr.ee OR site:beacons.ai OR site:about.me` — link aggregator profiles often link to the correct LinkedIn profile and other platforms
- `"First Last" site:github.com`
- `"First Last" [company name]`
- `"First Last" [specialization] [location]`

**Important**: In Name-Only mode, clearly mark what is confirmed vs. inferred. If multiple people share the name, ask the user to disambiguate.

---

## Phase 1.5: Identifier Collection

Before launching research agents, compile ALL discovered identifiers into a seed list. This list is passed to ALL 4 agents as input.

**Collect from Phase 1 (CV, LinkedIn, GitHub, personal website, any provided links)**:

- **Full name** + variations (First Last, F. Last, username-style: firstlast, first-last, first.last)
- **All usernames/handles** found on any platform (GitHub handle, LinkedIn slug, etc.)
- **Email addresses** visible in profiles, git commits, or website contact pages
- **All project/repo names** — every single one, including forks
- **Company names** with periods of employment
- **Domain names** (personal sites, company sites)
- **Unique technical terms** — niche technologies, specific product names, framework names the candidate is associated with

This seed list is the foundation for cross-platform research. Every identifier gets searched on every relevant platform.

**Domain Classification**: Based on Phase 1 data, classify the candidate into one or more professional domains. This determines which search protocols, registries, and verification databases agents should prioritize. A candidate can span multiple domains (e.g., "Software Engineering + Entrepreneurship" or "Legal + Government/Policy"). Refer to the Domain-Adaptive Search table in the OSINT Reference section for the full list of domains and their associated search strategies.

---

## Phase 1.75: Pre-Research Identity Collection (Self-Assessment Enhanced Only)

**Skip this phase entirely** in Evaluating Others mode and Self-Assessment Basic mode. Go directly to Phase 2.

Before launching research agents, ask the user to voluntarily declare their online accounts. This prevents handles from leaking through Claude Code's permission prompts during research — if the user declares a handle, agents can freely fetch it without the permission prompt revealing something the user didn't already know.

**Procedure**:

1. **Show already-discovered accounts**: List all accounts/handles found from explicit sources during Phase 1 (CV links, LinkedIn, GitHub bio cross-links, personal website). Example:
   > "From your CV and profiles, I've already found these accounts:
   > - GitHub: @yourhandle
   > - LinkedIn: /in/yourslug
   > - Personal site: yoursite.com
   >
   > These will be included in the research."

2. **Open-ended question**: Ask:
   > "Do you have accounts on any other platforms you'd like me to include in the assessment? This helps me research more thoroughly. Just list the platform and username — for example, 'Reddit: @myhandle' or 'Stack Overflow: user12345'."

   **Do NOT provide a platform checklist** — listing specific platforms (Reddit, Stack Overflow, Steam, etc.) is leading and could reveal what you intend to search. Keep it open-ended.

3. **Skip option**: Offer:
   > "You can also skip this — I'll still find what I can through public research, though some accounts may need identity verification before I can include them."

4. **Build the pre-authorized list**: Combine:
   - All accounts from Phase 1 (explicit links on CV/profiles)
   - All accounts the user declares here
   - These become **pre-authorized** — agents can freely fetch them, and the handles appearing in permission prompts is acceptable since the user already knows about them

**Pass to agents**: Along with the seed list from Phase 1.5, pass agents:
- The **pre-authorized accounts list** (platform + handle pairs)
- A **deferred-fetch flag** signaling that the Deferred Fetching Protocol is active

**Edge cases**:
- If the user skips this phase: all discovered handles go through the Deferred Fetching Protocol → more verification questions in Phase 2.5, but same quality result
- If the user provides a handle that doesn't exist: agent notes "pre-authorized but not found" — not suspicious, people forget old usernames
- If the user provides a handle that belongs to someone else: the assessment will include that account's data (they authorized it), but cross-referencing in Phase 2.75 may flag inconsistencies

---

## Phase 2: Deep Research (4 Parallel Agents)

Launch 4 research agents in parallel. Each has a specific focus area. Pass the full seed list from Phase 1.5 to every agent.

**CRITICAL — NO INTERMEDIATE REPORTS, NO POST-REPORT COMMENTARY**:

The user receives exactly ONE output from this entire skill: the final synthesized report from the synthesis agent in Phase 3. Nothing before it, nothing after it (except the identity verification questions in Phase 2.5 if applicable).

Specifically, you must NOT:
- Narrate agent completions ("Agent A is done", "Agent C completed", "all 4 agents finished")
- Produce partial assessments as agents return
- Write recap summaries after the report is delivered
- Add "one more thing" or "small correction" addendums when late agents finish
- Show research statistics tables (search counts, tool calls, duration)
- Provide "actionable takeaways" or commentary after the report

**How to handle agent completions**: Do NOT run agents in background. Launch all 4 agents and wait for all to return. Then proceed to Phase 2.5 (verification gates) and Phase 2.75 (synthesis agent). Once the synthesis agent returns the final report, relay it to the user and STOP. The assessment is complete. Say nothing more unless the user asks a follow-up question.

**Thoroughness calibration**: A thorough assessment should perform 40-60+ web searches per agent. If an agent completes with fewer than 20 searches, it likely didn't go deep enough. Each project, username, and company name should be searched across multiple platforms. Each claimed skill should be verified through at least 2-3 different search angles. Don't be satisfied with the first result — dig into second and third pages of results, try alternative query formulations, and follow every promising lead.

**Critical instruction for ALL agents**: You MUST end your output with two structured sections:

**Section 1 — Discovered Identifiers**:
- All usernames/handles found (platform: username)
- All project names found with URLs
- All email addresses found
- Any other unique identifiers

**Section 2 — Agent Digest** (MANDATORY — this is what the synthesis agent reads):

Produce a structured summary of ALL your findings. This digest must be self-contained — a reader who sees ONLY this digest should understand everything you found. Include:
- Every factual finding with its source URL
- Every evidence rating (Strong/Moderate/Weak/None) with justification
- Every concern, gap, or notable observation
- Specific quotes or data points that matter (don't just say "active GitHub" — say "mass contributions in 2023, mass Go code, mass Terraform providers, 2.1k stars on project X")
- Target length: 2500-4000 words. If your findings fill 5000 words, include all 5000. Err on the side of MORE detail, not less. The synthesis agent cannot go back and read your full output — if it's not in the digest, it's lost.

**Mandatory Inclusion Checklist** — If you discovered ANY of the following, they MUST appear in your digest with full detail. Omitting these is the #1 cause of thin reports:

1. **Every website/domain** (URL + full analysis of content, purpose, tech stack, design — not just "site exists")
2. **Every Reddit post/comment pattern** (subreddits active in, topics discussed, upvote patterns, personality signals revealed — not just "Reddit account found")
3. **Company financial data** (revenue, profit, equity, source registry — not just "company exists")
4. **GitHub stars and follows analysis** (notable repos starred, notable people followed, what interest patterns emerge — not just star count)
5. **Conference talk titles IN FULL** (creative or unusual titles are personality signals — never summarize as "gave 7 talks")
6. **Personal interests and hobbies from any source** (Reddit, Strava, forum posts, social media — these are character signals)
7. **Career timeline with dates and sources** (every role, every transition, with the source for each data point)
8. **Business entity details** (registration date, address, legal form, financials, registered activities — for every entity discovered)
9. **Third-party mentions** (blog posts, case studies, manager attributions, customer stories that mention the candidate by name)
10. **Platform-specific profile details** (even minor platforms — Last.fm, Steam, Garmin Connect, niche forums — if found, describe what they reveal)

**Section 3 — Deferred Handles** (Self-Assessment Enhanced mode only — omit in other modes):

List every handle discovered during research that was NOT fetched due to the Deferred Fetching Protocol. For each:
- **Platform**: e.g., Reddit, Stack Overflow, Steam, DeviantArt
- **Handle**: the discovered username
- **Discovery method**: how you found it (e.g., "same handle as GitHub username", "mentioned in HN comment", "project name search returned post by this user")
- **Link strength**: Strong implicit / Weak implicit / Circumstantial (per the Link Classification table)
- **Snippet context**: any information gleaned from search snippets WITHOUT fetching the profile URL (e.g., "WebSearch snippet shows posts in r/golang and r/running")
- **Suggested challenge questions**: 2-3 questions the main agent can use in Phase 2.5 to verify identity, based on whatever snippet context is available

These will be cross-referenced by a dedicated synthesis agent after all 4 agents complete. The synthesis agent operates with a fresh context window and reads ONLY the digests, so completeness is critical.

**Depth over breadth**: Do not skim 20 platforms. Go DEEP on the platforms where you find signal. One Reddit username leading to 50 comments is worth more than checking 10 platforms and finding nothing. Follow every thread.

#### Deferred Fetching Protocol (Self-Assessment Enhanced Only)

In Self-Assessment Enhanced mode, agents receive three additional inputs from Phase 1.75:
1. **Pre-authorized accounts**: handles the user declared + explicit links from Phase 1. Agents may freely fetch these.
2. **Seed identifiers**: unchanged from Phase 1.5.
3. **Deferred-fetch flag**: signals that this protocol is active.

For handles discovered DURING research that are NOT pre-authorized and NOT from explicit links (i.e., the candidate didn't publicly link them from a known profile):

**Agents CAN:**
- Run WebSearch queries mentioning the handle (search queries don't expose profile URLs in permission prompts)
- Read search result snippets returned by WebSearch
- Search archive APIs by keyword (e.g., `?q=[project-name]`) — NOT by author parameter

**Agents MUST NOT:**
- Fetch any URL containing the discovered handle in path or query parameters via WebFetch or curl
- This includes: `old.reddit.com/user/{handle}`, `arctic-shift.../api/posts/search?author={handle}`, `stackoverflow.com/users/{id}/{slug}`, `hub.docker.com/v2/repositories/{handle}/`, etc.
- The reason: Claude Code's permission system shows the full URL before executing the fetch, which leaks the handle to the user before identity verification

**Agents MUST instead:**
- Record the handle in their **Section 3 — Deferred Handles** output (see agent output format above)
- Include: platform, handle, discovery method, link strength, snippet context, and suggested challenge questions
- Continue researching using search-only methods to gather what context they can from snippets

**Exemption**: If the handle was found through an **explicit link** on a verified profile (e.g., the candidate's GitHub bio says "Reddit: @myhandle"), it is NOT subject to deferral. Fetch freely.

**In all other modes** (Evaluating Others, Self-Assessment Basic): This protocol does not apply. Evaluating Others never fetches pseudonymous profiles anyway. Self-Assessment Basic does not do pseudonym linking.

### Agent A: Online Presence & Digital Footprint

Research the candidate's public digital presence using the seed list from Phase 1.5. This agent must be EXHAUSTIVE — the goal is to find everything a thorough human researcher would find.

**Professional Platforms**:
- GitHub: contribution graph, repo quality, languages, stars, open-source involvement, org memberships
- Stack Overflow: reputation, top tags, answer quality, questions asked
- LinkedIn: recommendations, endorsements, activity, post quality
- Personal website/blog: content depth, writing quality, topics, update frequency

#### Website Discovery & Analysis Protocol

When a personal website, management company site, or side-project domain is discovered during research:

1. **ALWAYS attempt to fetch it** (use the full fallback tier: WebFetch → curl with browser headers → WebSearch for cached versions). Do not skip a domain just because the first fetch method fails.
2. **If the site is live**: Describe its content, purpose, design choices, tech stack signals (view-source for frameworks, analytics, meta tags), and any business entity connections. Note hidden details: custom 404 pages, robots.txt contents, Easter eggs, print CSS, source code comments, bot traps.
3. **If the site is down**: Search archive.org (`web.archive.org/web/*/[domain]`) for cached versions. Describe what the site was, when it was last active, and what its existence reveals.
4. **Search for ALL domains associated with discovered business entities**: If you find a management company (e.g., Pilcrow BV), search for every domain registered to that entity or associated with it.
5. **Include FULL analysis in your digest** — do not compress domain findings to "site exists" or "operates a website." The synthesis agent cannot visit the site — your digest IS the site analysis.

A personal domain someone registered, designed, and maintains is one of the richest personality signals available. Treat it accordingly.

**Community Presence**:
- Conference talks, meetup presentations (search YouTube, SlideShare, Speaker Deck)
- Podcast appearances
- Published articles or papers
- Forum participation (Hacker News, relevant subreddits, Discord servers)
- Mailing list archives (for open-source contributors)

#### Link Aggregator Discovery Protocol

Many professionals — especially creators, freelancers, early-career researchers, and anyone managing a multi-platform presence — use link aggregator services as their primary professional hub. These aggregate portfolio items, publications, social links, and project pages in a single, publicly accessible page — often richer than any individual platform profile.

**Search for link aggregators EARLY in the research process** — a single Linktree or equivalent can reveal profiles on 5+ platforms that would otherwise require separate searches:

1. **Search by name**: `"First Last" site:linktr.ee OR site:beacons.ai OR site:carrd.co OR site:about.me OR site:bio.link`
2. **Search by username variants**: Try `linktr.ee/[firstlast]`, `linktr.ee/[first.last]`, `linktr.ee/[first_last]` directly via curl
3. **Check LinkedIn bio/about sections**: LinkedIn profiles frequently link to Linktree in their contact info or about section
4. **Check institutional profiles**: University directory pages and ResearchGate profiles sometimes link to aggregators

**When you find a link aggregator**:
- **Fetch via curl** — Linktree returns clean HTML with JSON-LD `ProfilePage` structured data, all link URLs, and profile metadata (name, status, institution, creation date, last modified date)
- Extract ALL links — these are the candidate's self-curated professional presence and may include: publication PDFs, conference presentations, academic profiles, social accounts, institutional pages
- Use discovered links as new seed identifiers for further research
- Link aggregator profiles are **explicit links** (the candidate created them) — no verification gate needed

**Accessibility notes**:
| Platform | Curl Access | Data Quality |
|----------|------------|-------------|
| **Linktree** (`linktr.ee`) | Full access, clean HTML + JSON-LD | Rich: all links, titles, profile metadata, timestamps |
| **Beacons** (`beacons.ai`) | Usually accessible | Moderate: links and profile info |
| **Carrd** (`carrd.co`) | Usually accessible | Variable: depends on user's page design |
| **About.me** | Usually accessible | Moderate: bio, links, photo |
| **bio.link** | Usually accessible | Moderate: links and profile info |

#### Project-by-Project Cross-Platform Sweep

For EVERY project/repo discovered in Phase 1.5 or during research (not just the person's name), systematically search:

1. **Reddit** (see detailed protocol below)
2. **Hacker News**: `[project name] site:news.ycombinator.com`, also search for "Show HN" posts
3. **Stack Overflow**: project name, error messages, API questions — `[project-name] site:stackoverflow.com`
4. **Dev.to, Medium, Hashnode**: `[project name] site:dev.to`, `[project name] site:medium.com`
5. **YouTube**: demos, talks, tutorials — `[project name] site:youtube.com`
6. **Twitter/X**: `[project name] site:twitter.com` or `site:x.com`
7. **Domain-specific forums**: Match project domain to relevant communities:
   - Running/fitness apps → r/running, r/AdvancedRunning, r/Garmin, r/Stryd, Garmin forums
   - DevOps/Cloud tools → r/devops, r/aws, r/kubernetes, r/terraform, HashiCorp Discuss
   - Programming tools → r/golang, r/python, r/programming, relevant language forums
   - The candidate's hobby/interest subreddits based on profile signals

#### Reddit Deep Search Protocol

Reddit's search is notoriously poor. Web search indexing of Reddit is also unreliable. You MUST try ALL of these query patterns for each project/identifier:

1. **Plain project name**: `[project-name] site:reddit.com`
2. **Project name without hyphens**: `[project name words separated] site:reddit.com`
3. **Project URL**: `github.com/[username]/[project-name] site:reddit.com`
4. **Project name + domain keyword**: `[project-name] [domain-keyword] site:reddit.com`
5. **Project name without site filter**: `[project-name] reddit` (sometimes works better)
6. **Username + reddit**: `[github-username] site:reddit.com`
7. **Domain-specific subreddit search**: `[project concept] site:reddit.com/r/[relevant-subreddit]`

**When you find ONE Reddit post by the candidate**:
1. Extract the Reddit username
2. **Check the Deferred Fetching Protocol** (Self-Assessment Enhanced only): If this handle is NOT pre-authorized and NOT from an explicit link, do NOT fetch `site:reddit.com/user/[username]` or any URL containing the handle. Instead, record it in the Deferred Handles output section with suggested challenge questions, and use search-only methods (WebSearch queries, keyword searches on archive APIs without the `author=` parameter) to gather context from snippets.
3. If the handle IS pre-authorized or from an explicit link: Search for that username's full post and comment history: `author:[username] site:reddit.com` or `site:reddit.com/user/[username]`
4. Scan their history for: other projects mentioned, expertise signals, tone/personality, interests, community involvement
5. Check if they moderate any subreddits
6. Look for AMAs, detailed technical comments, or help given to others

**Do NOT give up after 1-2 failed queries.** Try at least 5 different formulations before concluding no Reddit presence exists. Log which queries you tried.

#### Reddit Archive Fallback (when web search fails)

Reddit actively blocks AI tools — both `site:reddit.com` web searches and direct WebFetch of Reddit URLs frequently return empty results or 403 errors. When standard Reddit searches fail, use these archive APIs via WebFetch which provide full post/comment history:

**Primary: Arctic Shift API**
- Search posts by author: `https://arctic-shift.photon-reddit.com/api/posts/search?author=[username]&limit=100`
- Search comments by author: `https://arctic-shift.photon-reddit.com/api/comments/search?author=[username]&limit=100`
- Search posts by keyword: `https://arctic-shift.photon-reddit.com/api/posts/search?q=[search-term]&limit=100`
- Search posts in a subreddit: `https://arctic-shift.photon-reddit.com/api/posts/search?subreddit=[subreddit]&q=[search-term]&limit=100`

**Secondary fallback: PullPush API** (if Arctic Shift is down)
- Search posts by author: `https://api.pullpush.io/reddit/search/submission/?author=[username]&size=100`
- Search comments by author: `https://api.pullpush.io/reddit/search/comment/?author=[username]&size=100`

**Usage protocol**:
1. Try standard web search queries first (the 7 patterns above)
2. If web search returns no results or errors, immediately pivot to Arctic Shift API
3. If you found a plausible Reddit username from another platform (e.g., same handle on GitHub, Last.fm, DeviantArt, Steam), search that username via Arctic Shift even if web search found nothing
4. Use the keyword search to find posts mentioning the candidate's projects or unique identifiers
5. **ALWAYS try the archive APIs for every known username/handle** — do not rely solely on web search for Reddit
6. **Deferred Fetching Protocol** (Self-Assessment Enhanced only): If the handle is NOT pre-authorized and NOT from an explicit link, do NOT use `author=[username]` in API calls (e.g., `arctic-shift.../api/posts/search?author=[username]`) — this is a direct profile query that may leak the handle via permission prompts. Instead, use **keyword searches** (`?q=[project-name]&limit=100`) to find relevant posts without specifying the author. Record the handle in the Deferred Handles output section.

**Identity verification still applies**: Finding a Reddit account with a matching username does NOT confirm identity. Same username ≠ same person. In Self-Assessment Enhanced mode, still run the identity verification gate before linking. In Evaluating Others mode, only use if the candidate explicitly links their Reddit account from a known profile.

#### Stack Overflow Deep Search

1. Search by display name: `user:[name] site:stackoverflow.com`
2. Search by project name: `[project-name] site:stackoverflow.com`
3. Search by GitHub username — SO profiles often link to GitHub
4. **Check the Deferred Fetching Protocol** (Self-Assessment Enhanced only): If you discover a SO user ID/profile URL for a handle that is NOT pre-authorized and NOT from an explicit link, do NOT fetch `stackoverflow.com/users/{id}/{slug}`. Record it in Deferred Handles with suggested challenge questions (e.g., "What is your approximate reputation?", "Name one of your top tags"). Use WebSearch snippets only.
5. If the handle IS pre-authorized or from an explicit link: Check for questions ASKED (reveals learning areas) and answers GIVEN (reveals expertise)
6. Check tags the user is active in
7. Look for self-answered questions (often document solutions to hard problems they encountered)

#### Credential Verification Protocol (licensed professions)

For candidates in regulated professions (legal, medical, real estate, engineering, accounting, trades), always verify through the authoritative public databases:
- State bar associations (every state has public lawyer lookup)
- State medical boards + NPI Registry (npiregistry.cms.hhs.gov)
- FINRA BrokerCheck (brokercheck.finra.org) for finance
- CPA state board verification (every state)
- CFA Institute verification (cfainstitute.org)
- State real estate commissions
- PE state board lookup

#### Publication/Byline Protocol (writing, journalism, academic)

- Google News: `"First Last" inurl:author` or `"By First Last" site:[publication]`
- Google Scholar: citation counts, h-index, co-author network
- Muck Rack: journalist profile aggregator
- Amazon: authored books
- SSRN: working papers (legal, finance, social science)

#### Business Records Protocol (executive, founder, finance)

- SEC EDGAR: search for name in proxy filings (DEF 14A), 10-K, 8-K
- Crunchbase / PitchBook: funding rounds, company data
- State Secretary of State: business entity search (incorporation records)
- USPTO: patent and trademark filings
- ProPublica Nonprofit Explorer: 990 filings listing officers and compensation

#### Creative Portfolio Protocol (design, arts, video, music)

- Search the domain-appropriate portfolio platforms from the Domain-Adaptive table
- Look for awards (Cannes Lions, Webby, Effie, Oscars, Emmy, Grammy, Pulitzer — all have public winner/nominee databases)
- IMDb for film/TV credits, Discogs for music credits
- Festival selection databases (FilmFreeway)

#### Low-OSINT Domain Protocol (sales, HR, consulting, supply chain, PM)

When the domain classification indicates a Tier 3 (low OSINT) domain:
- Don't waste 40+ searches on platforms that won't have artifacts — shift effort to credential verification and LinkedIn deep analysis
- Focus on: certifiable credentials (PMP, SHRM, Salesforce certs, Six Sigma), LinkedIn recommendations (read the actual text for specificity and credibility), conference speaking history, published thought leadership
- Explicitly note in the output: "This domain has low public artifact visibility. Assessment is weighted toward credential verification, peer endorsements, and company context rather than independent artifact discovery."

#### Evidence Source Refinement Pass

Before searching registries, determine which ones are relevant based on the candidate's profile. Don't blindly search every registry — identify the RIGHT registries for THIS person.

**Step 1: Identify the candidate's domain bucket(s)** from Phase 1 data and domain classification. A candidate may span multiple buckets.

**Technical:**
- Software Engineering → code registries (npm via `registry.npmjs.org/{package}` API, PyPI, crates.io, Docker Hub via `hub.docker.com/v2/repositories/{namespace}/{repo}/` API, etc.)
- Data Science / ML → model hubs (Hugging Face), competition platforms (Kaggle), data registries
- DevOps / Platform → infra registries (Terraform Registry via `registry.terraform.io/v1/providers/{namespace}/{name}` API, Ansible Galaxy, Helm/artifacthub.io, Docker Hub via `hub.docker.com/v2/repositories/` API)
- Security / InfoSec → vulnerability databases, CTF platforms (CTFtime, HackerOne, Bugcrowd)
- Mobile Development → app stores (App Store, Play Store, Garmin Connect IQ)
- Design / UX → portfolio platforms (Dribbble, Behance, Figma Community)
- Hardware / Embedded → electronics communities, maker platforms

**Business & Finance:**
- Sales / Business Development → LinkedIn recommendations, President's Club mentions, Salesforce Trailblazer profiles, conference talks (SaaStr, Dreamforce)
- Marketing / Growth → published campaigns (Cannes Lions, Effie, Webby, Shorty Awards), Google/HubSpot/Meta certifications (Credly), Substack/Medium articles
- Finance / Accounting → FINRA BrokerCheck (public), CPA state board lookup (public), CFA Institute verification (public), SEC EDGAR, Seeking Alpha
- Consulting / Strategy → firm alumni pages, published thought leadership, PMP (PMI registry), books (Amazon)
- Entrepreneurship / Founder → Crunchbase, AngelList/Wellfound, state SOS business registrations, USPTO patents/trademarks, Product Hunt launches
- Executive / C-Suite → SEC EDGAR proxy filings (DEF 14A), board memberships, ProPublica Nonprofit Explorer (990s), Bloomberg Executive Profile

**Creative & Media:**
- Journalism / Writing → Muck Rack, Google News byline search, Amazon (books), Substack, publication archives
- Visual Arts / Illustration / Photography → ArtStation, DeviantArt, Behance, 500px, Getty/Shutterstock contributor profiles
- Video / Film → IMDb, Vimeo, FilmFreeway, festival archives
- Music / Audio → Discogs, ASCAP/BMI repertoire search, Spotify/Bandcamp/SoundCloud

**Licensed Professions:**
- Legal → state bar directories (public lookup), PACER/CourtListener, Google Scholar (legal opinions), Justia, Avvo
- Healthcare → NPI Registry (public), state medical board lookup, ABMS certification verify (certificationmatters.org), PubMed, ClinicalTrials.gov
- Architecture → state licensing boards, NCARB verification, ArchDaily, Dezeen
- Engineering (PE) → state PE board lookup, USPTO patents, NCEES registry
- Real Estate → state real estate commission lookup, Zillow/Redfin agent profiles, county deed records

**Operations & Organizational:**
- HR / People Ops → SHRM/HRCI certification verification, conference speaker archives
- Non-Profit → ProPublica Nonprofit Explorer (IRS 990 filings — lists officers, directors, compensation), GuideStar/Candid, state charity registrations
- Government / Policy → government websites, think tank publications, LegiStorm (legislative staff), GovTrack
- Academic / Research → publication databases (Google Scholar, ORCID, DBLP, Semantic Scholar, arXiv)
- Project Management → PMP (PMI), CSM (ScrumAlliance), SAFe (Scaled Agile)

**OSINT Searchability Tiers** — calibrate depth of artifact search accordingly:
- **Tier 1 (HIGH)**: Software eng, data science, legal, healthcare, finance credentials, entrepreneurship, journalism, real estate, non-profit leadership, public company executives → Rich public data, go deep
- **Tier 2 (MEDIUM)**: DevOps, security, design, academia, visual arts, video/film, architecture, government → Some public signals but gaps; mix artifact search with credential verification
- **Tier 3 (LOW)**: Sales, marketing, consulting, HR, supply chain, project management, customer success → Most work is proprietary; focus on credential verification, LinkedIn recommendation analysis, and conference speaking history. Explicitly acknowledge the limitation in the output.

**Step 2: For EACH identified domain**, search the relevant registries for the candidate's username, real name, and known project names.

**Deferred Fetching applies here** (Self-Assessment Enhanced only): When searching registries that include a discovered handle in the URL (e.g., `hub.docker.com/v2/repositories/{handle}/`, `registry.terraform.io/v1/providers/{handle}/{name}`), check whether the handle is pre-authorized. If NOT, use WebSearch to find evidence of published artifacts (e.g., `"{handle}" site:hub.docker.com`) rather than fetching the API URL directly. Record the handle in Deferred Handles if you find signal.

**Step 3: Check for cross-domain registries** that apply broadly:
- LinkedIn (universal)
- GitHub/GitLab (nearly universal for technical roles)
- Google Scholar (academic, research, healthcare, legal scholarship)
- Crunchbase (entrepreneurship, executive, finance)
- USPTO (patents across all domains)

Publishing to any registry or platform is a strong signal — it requires packaging, documentation, and maintenance discipline beyond just writing code. A published Terraform provider signals deep Go + IaC knowledge. A Kaggle competition ranking signals applied ML skills. A Google Scholar profile with citations signals research impact. A verified state bar listing confirms legal practice. A FINRA BrokerCheck entry confirms financial licensing.

#### Link Classification & Ethics

| Link Type | Definition | Example | Evaluating Others | Self-Assessment Basic | Self-Assessment Enhanced |
|-----------|-----------|---------|-------------------|----------------------|------------------------|
| **Explicit** | Candidate publicly links Profile A to Profile B | GitHub bio says "Also on Twitter @handle" | ✅ USE | ✅ USE | ✅ USE |
| **Implicit (Strong)** | Same unique project appears on both platforms with matching technical detail | GitHub repo `foo-tool` + Reddit post "I built foo-tool" with implementation details only the author would know | ❌ IGNORE | ❌ IGNORE | ✅ USE after verification gate |
| **Implicit (Weak)** | Same common name, overlapping interests, but no unique identifier | LinkedIn "John Smith, Python dev" + SO user "john_s" answering Python questions | ❌ IGNORE | ❌ IGNORE | ⚠️ Flag for verification but low confidence |
| **Circumstantial** | Timing correlation, geographic match, but no content link | Reddit account created same month as a job change | ❌ IGNORE | ❌ IGNORE | ❌ IGNORE (not enough signal) |

**For Evaluating Others**: Only explicit links. Period. If you can't trace a public, candidate-created link between two profiles, they are separate identities. This is not optional. Note: the handle-leak problem via permission prompts is N/A for this mode — you never fetch pseudonymous profile URLs in the first place.

**For Self-Assessment Enhanced**: Find ALL implicit links. For each one, run the identity verification gate BEFORE including it. Batch verifications when possible — "I found potential accounts on Reddit and Stack Overflow. Let me verify both."

**Explicit links found during research are exempt from deferral**: If an agent discovers that a known, verified profile (e.g., GitHub bio, personal website) explicitly links to another account (e.g., "Find me on Reddit: @myhandle"), that handle is treated as an explicit link per the table above. Agents may fetch it freely — no deferred fetching needed, no verification gate required. The key test: did the candidate themselves publicly create the link from a verified identity?

**Absence of signal is NOT negative signal.** Many excellent professionals have minimal online presence. Not having a blog, GitHub contributions, or public portfolio says nothing about competence. Note the absence without judgment.

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

**Professional Claims — Artifact Verification**:
- If they claim expertise in X, does their public work (repos, publications, portfolios, registry entries) reflect it?
- If they claim to have built Y, can you find evidence of Y existing?
- Patent searches (Google Patents, USPTO) for claimed inventions
- Published papers (Google Scholar, arXiv) for claimed research
- License/credential verification through authoritative public databases

For each claimed skill, search for published artifacts that prove it. The artifact type depends on the domain:

**Engineering/DevOps examples**:
- Claimed Go expertise → Go repos, Terraform providers (written in Go), CLI tools
- Claimed Python expertise → PyPI packages, GitHub repos with Python code
- Claimed K8s expertise → Helm charts, operators, CRDs in their repos
- Claimed Terraform expertise → Terraform Registry modules/providers
- Claimed AWS expertise → CDK constructs, CloudFormation templates, published solutions

**Research/Academic examples**:
- Claimed expertise in a field → published papers, citation counts, h-index
- Claimed novel methodology → pre-prints, conference presentations, invited talks
- Claimed research impact → Google Scholar profile, citation network, grants

**Design/Product examples**:
- Claimed UX expertise → portfolio case studies, shipped products, Dribbble/Behance work
- Claimed product leadership → product launches, blog posts about decisions, speaking engagements

**Business/Leadership examples**:
- Claimed revenue growth → verify company trajectory via Crunchbase funding, press coverage, Glassdoor headcount trends
- Claimed board membership → verify via SEC proxy filings (public companies) or IRS 990 filings (non-profits)
- Claimed company founding → verify via state SOS business entity search, Crunchbase, AngelList
- Claimed patent → verify via Google Patents or USPTO TESS

**Licensed profession examples**:
- Claimed bar admission → verify via state bar public directory (every state has one)
- Claimed medical license → verify via state medical board + NPI Registry
- Claimed CPA/CFA → verify via state board / CFA Institute directory
- Claimed PE license → verify via state board / NCEES

**Creative examples**:
- Claimed publication byline → verify via publication archive, Google News
- Claimed film credit → verify via IMDb
- Claimed music credit → verify via Discogs, ASCAP/BMI repertoire database
- Claimed award → verify via the award organization's public archive

**General principle**: For each claimed skill, look for ARTIFACTS, not just mentions:
- Published work in that domain's natural registry/platform
- Contributions (PRs, issues, discussions, reviews, comments) to projects in that ecosystem
- Conference talks, blog posts, or teaching materials demonstrating depth
- Third-party references or citations of their work

Also check:
- The candidate's GitHub org memberships — some contributions are under org accounts
- Git commit history in their repos for co-authors and contribution patterns
- Forked repos with significant modifications (indicates learning or contribution intent)
- For non-technical candidates: company "About"/"Team" pages, conference speaker archives, professional association member directories, and any publications or awards databases relevant to their domain

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
- **Team size and structure**: How big was the candidate's team/department? What was their scope?
- **Tools and methods**: What tools, technologies, or methodologies did the company use? Does it align with the candidate's claimed skills?
- **Exits and outcomes**: Did the company IPO, get acquired, shut down? This contextualizes the candidate's experience.

This context matters because "Senior Engineer at [company]" means very different things depending on whether the company had 5 or 5000 engineers.

#### Side Business & Management Company Deep Dive Protocol

When management companies, freelance entities, or side businesses are discovered (common in Belgium/Netherlands: BV/BVBA structures; in other countries: LLC, Ltd, sole proprietorships):

1. **Search the national company registry**:
   - Belgium: KBO/BCE (Kruispuntbank van Ondernemingen) — search via `kbopub.economie.fgov.be`
   - Netherlands: KvK (Kamer van Koophandel) — search via `kvk.nl`
   - UK: Companies House — search via `find-and-update.company-information.service.gov.uk`
   - US: State Secretary of State databases
   - Other countries: search for "[country] company registry" + company name

2. **Pull financial data**: Revenue, profit/loss, equity, total assets, employee count. For Belgian companies, annual accounts are public via the National Bank (NBB). For UK companies, filings are on Companies House. Include the source.

3. **Identify ALL registered business activities**: Company registries list NACE codes or SIC codes describing business activities. A tech consultant with "pig farming" as a registered activity is a critical personality signal.

4. **Search for other domains, trademarks, or subsidiaries** associated with the entity.

5. **Analyze what the side business reveals**: Farms, creative ventures, non-tech businesses — these are NOT footnotes. They reveal values, interests, risk tolerance, entrepreneurial tendencies, and life priorities. A data engineer who runs a pig farm through the same BV as their consulting work is telling you something about who they are. Include this analysis in your digest.

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
| **Meta-signal** | Absence of any online presence | NOT negative. Many excellent professionals are private. Note it neutrally. |

---

## Phase 2.5: Identity Verification Gates (Main Agent — Self-Assessment Enhanced Only)

**Prerequisite**: ALL 4 research agents from Phase 2 must have returned their results.

**Skip this phase entirely** in Evaluating Others mode and Self-Assessment Basic mode. Go directly to Phase 2.75.

In Self-Assessment Enhanced mode, the MAIN agent must handle identity verification BEFORE launching the synthesis agent. This prevents pseudonymous account details from leaking into any output before verification.

**Procedure**:

0. **Read Deferred Handles** from all 4 agents. Collect the full list of handles that agents discovered but did NOT fetch due to the Deferred Fetching Protocol. Each entry includes: platform, handle, how discovered, link strength, snippet context, and suggested challenge questions. Cross-reference against the pre-authorized accounts list from Phase 1.75 — any handle the user already declared is already authorized and does not need verification.

1. **Read ONLY the Discovered Identifiers sections** from all 4 agents (these are short lists — they fit in context). Do NOT read the full agent outputs or digests yet.

2. **Identify potential pseudonymous links**: Compare identifiers across agents, including Deferred Handles. Look for:
   - Same username appearing on multiple platforms without explicit cross-linking
   - Same project name on platforms associated with different identities
   - Usernames matching known handles on new, unexpected platforms

3. **Run verification gates** for ALL potential links BEFORE proceeding:
   - Batch verifications when possible ("I found potential accounts on Reddit, DeviantArt, and Steam. Let me verify each.")
   - Follow the Identity Verification Gate protocol from Step 0 exactly: ask 2-3 challenge questions, do NOT reveal the full profile until verified
   - **CRITICAL**: Do NOT list the discovered accounts, do NOT mention the platforms, do NOT show the correlation results. Only reveal enough to ask the verification question (e.g., "I found a potential account on [platform] that appears linked to one of your projects.")

4. **Record results**: For each potential link, record: Verified ✅ or Rejected ❌. Rejected links are PERMANENTLY excluded — they must never appear in any subsequent output.

5. **Post-Verification Fetch** (Deferred Handles only): For each deferred handle that PASSED verification:
   - **NOW** fetch the profile URL (e.g., `old.reddit.com/user/{handle}`, `stackoverflow.com/users/{id}/{slug}`, archive APIs with `author={handle}`). The user has confirmed ownership, so the handle appearing in a permission prompt is acceptable.
   - Use the same platform-specific access methods documented in the OSINT Reference section (WebFetch → API → browser-emulating curl → WebSearch fallback).
   - Append the retrieved data to the relevant agent's digest before passing to the synthesis agent.
   - For each deferred handle that FAILED verification: the profile URL is **never fetched**. The handle never appears in any permission prompt. The user never learns what was discovered. Remove it from all outputs.

6. **Pass verification results AND fetched data to the synthesis agent** in Phase 2.75. This must happen AFTER step 5 — the synthesis agent needs the post-verification profile data to produce a complete report.

---

## Phase 2.75 + Phase 3: Synthesis Agent (Dedicated Sub-Agent)

**WHY a dedicated agent**: The 4 research agents produce massive outputs. The main agent's context is already partially consumed by Phases 0–2.5. Attempting to read all 4 full outputs in the main context leads to truncation and lost data. Instead, delegate ALL synthesis work to a fresh agent with a clean context window.

**How to launch**: Once Phase 2.5 is complete (or skipped), launch ONE synthesis agent (using the Task tool) with the following:

1. **Input**: Pass the synthesis agent ALL of the following in its prompt:
   - The candidate's name, target role, and assessment mode (Self-Assessment / Evaluating Others)
   - The structured profile from Phase 1
   - The seed list from Phase 1.5
   - The **pre-authorized accounts list** from Phase 1.75 (if Self-Assessment Enhanced)
   - The full Agent Digest sections from all 4 research agents (copy-paste the digest text — do NOT ask the synthesis agent to read files)
   - The full Discovered Identifiers sections from all 4 research agents
   - The full Deferred Handles sections from all 4 research agents (if any)
   - Identity verification results from Phase 2.5 (which links are verified ✅, which are rejected ❌, which deferred handles were fetched post-verification). Instruct the synthesis agent: "Do NOT include any information from rejected links in the report. Treat rejected accounts as if they do not exist."

2. **Instructions for the synthesis agent**: The synthesis agent must perform Phase 2.75 (Cross-Platform Correlation) AND Phase 3 (Final Report) as described below. Its output IS the final report shown to the user.

3. **Report length requirements**: The final report MUST be at least 5000 words for candidates with moderate-to-rich data. If the combined agent digests exceed 8000 words, the report should be at least 6000 words. A 2000-word report from 10000 words of research means 80% of the findings were dropped — that is unacceptable.

4. **"No Data Left Behind" completeness check**: Before finalizing the report, the synthesis agent must re-read each agent digest and verify that every factual finding appears somewhere in the report. The following data types are the most commonly dropped — check these specifically:
   - Websites/domains mentioned but not analyzed (e.g., "akkervarken.be was found" without describing what the site is, what it contains, what it reveals)
   - Reddit activity noted but personality signals omitted (e.g., "Reddit activity found" without listing subreddits, topics, and what the posting patterns reveal about the person)
   - Business financials listed in one table cell without discussing implications (e.g., revenue and profit figures that are never interpreted)
   - Conference talks summarized as a count without listing creative titles (e.g., "7 conference talks" without noting that "The Data Engineering Shepherd" connects farming and engineering)
   - GitHub stars noted without analyzing interest patterns (e.g., "follows 40 people" without noting which tools/communities they follow)
   - Hobbies and personal interests discovered but treated as footnotes instead of personality signals

5. **Website analysis mandate**: The synthesis agent MUST produce a dedicated analysis paragraph for every domain discovered by any research agent. "akkervarken.be was found" is not analysis. "akkervarken.be is a regenerative pig farm operated by Pilcrow BV, the same entity that houses his tech consulting work — connecting his agricultural roots to his professional identity through a single legal structure" IS analysis. If an agent's digest mentions a domain, it gets a paragraph in Section 3.1.5.

6. **Inline source citation mandate**: Every factual claim in every section of the report must include its source inline as `(source: [URL or description])`. This applies to all sections — executive summary, website analysis, skills assessment, mentality profile, reading between the lines, strengths, concerns, solo/team, churn risk, and signal gaps. The verification table (3.2.5) already has per-row sources; all other sections must match this standard.

**CRITICAL — NO INTERMEDIATE OUTPUT, NO POST-REPORT OUTPUT**: The main agent must NOT produce any text before or after the synthesis agent's report, except:
- Identity verification questions (Phase 2.5, if applicable)
- A brief "Starting assessment..." message at the very beginning

Once the synthesis agent returns, **write the full report to a markdown file** (e.g., `{candidate-name}-assessment.md` in the current working directory) for easy review. Then relay its complete output to the user and STOP IMMEDIATELY. Mention the file path so the user can open it.

Do not:
- Summarize the report ("Here's a quick summary of what 4 agents found...")
- Add commentary ("The biggest finding is...")
- Show agent statistics ("4 agents, 390+ tool calls, 200+ web searches...")
- Provide post-report insights or takeaways
- Respond to late-arriving agent completion notifications
- Add corrections or clarifications from other agents

The report is the deliverable. Everything else is noise.

---

### Phase 2.75: Cross-Platform Correlation (performed by synthesis agent)

The synthesis agent runs a FINAL correlation pass using the Discovered Identifiers from all 4 research agents:

0. **Generic cross-platform sweep first**: Before targeted platform searches, run a broad sweep for every discovered username:
   - Search: `"[username]" -site:github.com -site:linkedin.com -site:reddit.com -site:stackoverflow.com -site:twitter.com -site:medium.com`
   - Adjust the `-site:` exclusions to include ALL platforms already searched during Phase 2
   - This catches unexpected platforms (DeviantArt, Steam, gaming profiles, Last.fm, Bandcamp, niche forums, personal wikis, Mastodon instances) that targeted searches miss
   - Run this for EVERY unique username discovered, not just the primary one
   - Any new platform discovered here should then get a targeted deep search

1. **Username propagation**: For every username found on any platform, search for that SAME username on all OTHER platforms:
   - GitHub username → search Reddit, SO, HN, Twitter, Docker Hub, PyPI, npm, etc.
   - Reddit username → search GitHub, SO, Twitter, etc.
   - This catches cases where someone uses the same handle everywhere

2. **Project propagation**: For every project found anywhere, search for it on ALL platforms:
   - GitHub repo → search Reddit, HN, SO, blog posts, YouTube demos, forum discussions
   - PyPI package → search GitHub (source), SO (usage questions), Reddit (announcements)
   - Published app → search app reviews, forum discussions, Reddit threads

3. **Include only verified links**: In Self-Assessment Enhanced mode, only include accounts that passed the verification gate in Phase 2.5. Never mention, reference, or use data from rejected accounts.

4. **Deferred fetching for new discoveries** (Self-Assessment Enhanced only): If cross-platform correlation reveals a NEW handle not in the pre-authorized list and not already verified in Phase 2.5:
   - Use **search-only methods** (WebSearch) — do NOT fetch the profile URL directly
   - Do NOT reveal the handle in any output
   - Note in "Signal Gaps & Open Questions" (Section 3.6): "Additional account potentially discovered on [platform]. Not included as identity was not verified."
   - This ensures no new handle leaks through permission prompts during synthesis

5. **Evidence strengthening**: Use cross-platform findings to upgrade evidence levels:
   - A skill rated "Weak" based on GitHub alone might become "Moderate" if a SO answer or Reddit discussion demonstrates depth
   - A skill rated "Moderate" might become "Strong" if a published registry artifact confirms it

**For Evaluating Others mode**: Only use explicit links (candidate publicly links Profile A to Profile B). No implicit linking.

---

### Phase 3: Final Report (performed by synthesis agent)

The synthesis agent compiles all research into one single, definitive, comprehensive assessment. This is the ONLY output the user receives — there are no drafts, previews, partial reports, or addendums.

**Source Citation Rule**: Every factual claim in the report must include an inline source citation in parentheses. This is not optional. A report without sources is not a report — it's an opinion. Format: `(source: [URL or description])`. Apply this to ALL sections, not just the verification table. Example: "He evaluated eight different catalogs (source: https://datahub.com/customer-stories/dpg-media/)." Not just "VERIFIED" — the actual source.

**Writing Style Directive**: Default to free-flowing narrative paragraphs. Use tables ONLY for structured reference data that benefits from scannable columns (verification status, skills evidence matrix). For everything else — personality insights, strengths, concerns, churn analysis, website analysis, team assessment — write in prose paragraphs.

Tables are for DATA. Paragraphs are for INSIGHT.

Specifically:
- Section 3.1 Executive Summary: paragraphs (already is)
- Section 3.1.5 Website Analysis: paragraphs — describe each site as a narrative, not bullet points
- Section 3.2 Professional Depth: TABLE is appropriate here (structured skills data)
- Section 3.2.5 Verification Table: TABLE is appropriate (structured verification data)
- Section 3.3 Mentality Profile: TABLE for the 8-dimension overview, then PARAGRAPHS for elaboration
- Section 3.3.5 Reading Between the Lines: PARAGRAPHS — numbered narrative insights
- Section 3.4 Strengths & Concerns: PARAGRAPHS — each strength/concern is a full paragraph with evidence, not a bullet point
- Section 3.4.5 Solo vs Team: PARAGRAPHS with embedded ratings
- Section 3.5 Churn Risk: TABLE for the matrix overview, then PARAGRAPHS for "What Would Make Them Leave/Stay"
- Section 3.6 Signal Gaps: PARAGRAPHS per gap, not a table
- Section 3.7.5 Hidden Detail: PARAGRAPH — a boxed narrative insight
- Section 3.8 Final Verdict: PARAGRAPHS
- Section 3.9 Interview Questions: structured format (Question/Why/Good/Red flag) is fine

### 3.1 Executive Summary

2-3 paragraphs covering:
- Who is this person, in plain language?
- What's the career narrative? (trajectory, not just timeline)
- What's the overall signal quality? (strong evidence, moderate, or mostly self-reported?)
- Initial hiring thesis: what role would this person excel in? What role would be a poor fit?

### 3.1.5 Website & Portfolio Analysis

Produce a dedicated analysis paragraph for EVERY domain discovered during research — personal sites, management company sites, side-project domains, portfolio sites. This section is mandatory if any domains were found.

For each domain:
- **What it is**: The site's purpose, who it serves, what it contains
- **Tech stack signals**: Frameworks, hosting, CMS, analytics tools, meta tags — what do the technology choices reveal about the person?
- **Design language**: Visual style, quality, attention to detail, accessibility, mobile responsiveness
- **Business entity connections**: Is the domain registered to a company? Which one? What does the ownership structure reveal?
- **Hidden details**: Easter eggs, bot traps, custom 404 pages, print CSS, robots.txt contents, interesting meta tags, source code comments — anything that reveals intentionality or personality
- **What it tells you about the person**: Synthesize the above into a personality/professional insight

If a site was down at time of research: note the URL, describe what was found via archive.org or search engine cached snippets, and what the site's existence (even if defunct) tells you.

If NO domains were discovered, explicitly state: "No personal websites or portfolio domains were found during research."

**Write this as narrative prose** — one or more paragraphs per domain. Do NOT compress domain analysis to a single bullet point like "operates a farm website." A domain someone registered, built, and maintains is a window into their priorities.

### 3.2 Professional Depth Assessment

For each claimed area of expertise, provide a comprehensive evidence map:

| Domain | Claimed Level | Evidence Level | Confidence | Key Evidence |
|--------|--------------|----------------|------------|-------------|
| [Skill 1] | Expert | Very Strong | High | [List SPECIFIC artifacts: repo URLs, package links, SO answers, registry entries, PR links] |
| [Skill 2] | Advanced | Weak | Low | Self-reported only. Searched relevant registries and platforms — no artifacts found. |
| [Skill 3] | Intermediate | Moderate | Medium | [Specific artifact links and descriptions] |
| [Skill 4] | Licensed | Verified | High | State bar #12345 — active status confirmed via public directory |

**Evidence Levels**:
- **Very Strong**: 3+ independent sources confirm (code + published artifact + third-party endorsement + community activity)
- **Strong**: 2+ independent sources confirm (code + peer endorsement, or artifact + community discussion)
- **Moderate**: Some evidence supports (one project, one reference, consistent narrative)
- **Weak**: Self-reported only, no corroboration found despite searching
- **Contradictory**: Evidence suggests different level than claimed

**For each domain rated Moderate or below**: Explicitly list what you searched for and didn't find. "Searched PyPI, Terraform Registry, Docker Hub, and GitHub repos — no Go artifacts found" is more useful than "No public evidence."

**Artifact inventory**: For every area of expertise, list the discovered artifacts (adapt to the candidate's field):
- **Code/Technical**: Repos, published packages, registry entries, SO activity
- **Publications**: Papers, articles, bylines, books, patents
- **Credentials**: Verified licenses, certifications, board memberships
- **Portfolio**: Case studies, shipped products, published designs, produced media
- **Community**: Conference talks, forum posts, recommendations, endorsements
- **Recognition**: Awards, rankings, citations, grants

### 3.2.5 Claim Verification Summary

Present a verification table covering every significant factual claim from the candidate's CV, LinkedIn, or discovered profiles. Every row must cite the specific source used for verification — not just "VERIFIED" but the actual URL, registry name, or database.

| Claim | Verification Status | Evidence Source |
|-------|-------------------|----------------|
| [Specific claim from CV/profile] | VERIFIED | [Exact source: URL, registry name, article title] |
| [Another claim] | PARTIALLY VERIFIED | [What was confirmed + what wasn't + source] |
| [Another claim] | NOT CONFIRMED | [What was searched, why it couldn't be confirmed] |
| [Another claim] | UNVERIFIABLE | [Why: proprietary work, NDA, too old, no public records] |
| [Another claim] | CONTRADICTED | [What the evidence actually shows + source] |

**Verification statuses**:
- **VERIFIED**: Confirmed by at least one independent source with direct evidence
- **PARTIALLY VERIFIED**: Some aspects confirmed, others not (explain which parts)
- **NOT CONFIRMED**: Searched but found no corroborating evidence (not the same as contradicted)
- **UNVERIFIABLE**: Cannot be checked via OSINT (proprietary projects, NDA-covered work, verbal agreements)
- **CONTRADICTED**: Evidence suggests the claim is inaccurate or misleading (explain the discrepancy)

Include at minimum: job titles, employment dates, education claims, specific achievement claims, technology expertise claims, and any quantitative metrics cited on the CV. Aim for 15+ rows for candidates with detailed CVs.

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

### 3.3.5 Reading Between the Lines

This is the narrative layer that transforms data into character insight. Produce 6-10 numbered personality insights derived from the FULL body of evidence — hobbies, side projects, Reddit activity, talk titles, tool choices, career transitions, financial structure, writing style, community involvement.

Each insight follows this structure:
1. **[Observation Label]**: [Interpretation] — [Specific evidence]

Examples of the depth expected:
- "**Builder, not a talker**: He ships tools for personal use and publishes them without marketing. The GitHub activity shows mass creation with minimal self-promotion — no blog posts announcing releases, no Twitter threads. The work speaks; he doesn't." (source: GitHub profile)
- "**Autonomy-driven**: Every career move goes toward more independence — from employee to freelancer to management company owner. The side businesses aren't hobbies; they're escape valves from corporate structure." (source: career timeline + KBO registry)
- "**Endurance-sports identity as character signal**: Marathon running, cycling, long-distance events — these aren't casual hobbies. They select for delayed gratification, pain tolerance, and structured training. The Garmin data obsession signals a metrics-driven personality." (source: Strava/Garmin profile + Reddit posts)

Draw from ALL evidence sources: Reddit post history reveals interests and communication style. Conference talk titles reveal creativity and how they frame ideas. Tool choices (suckless, Arch Linux, vim) reveal philosophy. Side businesses reveal values. Financial structures reveal planning orientation. Hobbies reveal character.

**Write as flowing paragraphs**, not bullet points. Each insight should be a mini-narrative that connects evidence to character.

### 3.4 Strengths & Concerns

**Minimum targets**: Identify at least 8 strengths and at least 6 concerns. A report with 3 strengths and 2 concerns is a sketch, not an assessment. Dig deeper — every career has more texture than that.

**Strengths** (minimum 8, with evidence — write each as a full paragraph, not a single sentence):

Each strength should be a dedicated paragraph that includes: what the strength is, specific evidence from research (with source citations), and why it matters for a hiring decision. Don't just list "good communicator" — explain HOW you know, WHAT the communication looks like, and WHY it's relevant.

**Concerns** (minimum 6, with evidence and severity — write each as a full paragraph):

Each concern should include: what the concern is, specific evidence, severity rating (Low/Medium/High), whether it can be verified in an interview (Y/N), and if Y — what the interviewer should specifically look for or ask. A concern is not an accusation; it's a data point that warrants further investigation.

**Yellow Flags** (things that aren't concerns but warrant attention):

Yellow flags are patterns or signals that aren't negative on their own but could become relevant depending on role fit, team dynamics, or company culture. Write each as a short paragraph explaining what caught your attention and why.

### 3.4.5 Solo vs. Team Player Assessment

Rate both dimensions independently with evidence:

**Solo Effectiveness: [X/10]**

Write 1-2 paragraphs on how this person operates independently. Evidence to consider: solo side projects, solo business ventures, self-directed learning patterns, ability to ship without a team, self-managed freelance work, independent open-source maintenance.

**Team Effectiveness: [X/10]**

Write 1-2 paragraphs on how this person operates in collaborative settings. Evidence to consider: open-source collaboration (PR reviews, issue discussions, co-authored commits), team sports or group hobbies, conference co-presentations, multi-author publications, management experience, mentoring signals.

**Best-Fit Team Structure**: Describe the ideal team setup for this person — team size, reporting structure, level of autonomy, collaboration frequency.

**Communication & Soft Skills**: Assess written communication (blog posts, documentation, PR descriptions, forum posts), verbal communication (conference talks, podcast appearances), client/stakeholder management signals, and interpersonal style (tone in online interactions, conflict resolution patterns).

### 3.5 Churn Risk Matrix

| Risk Factor | Assessment | Evidence |
|-------------|-----------|----------|
| **Job hopping pattern** | [Average tenure, trend direction] | [Timeline analysis] |
| **Career trajectory** | [Upward, lateral, stagnant, exploratory] | [Role progression] |
| **Side projects** | [Active side projects suggesting potential distraction or entrepreneurial drive] | [GitHub, personal site] |
| **Industry alignment** | [How aligned is their background with your company's domain?] | [Career history] |
| **Seniority fit** | [Overqualified? Underqualified? Right level?] | [Role history vs. target role] |
| **Location/remote** | [Any signals about work arrangement preferences] | [Profile, company history] |

**What Would Make Them Leave** (write as narrative paragraphs, not just a table):

For each identified churn trigger, write a paragraph covering: what the trigger is, how likely it is to occur (High/Medium/Low probability), what evidence supports this assessment, and what a company could do to mitigate it. Consider: boredom from unchallenging work, lack of autonomy, bureaucratic friction, insufficient compensation growth, better offers in their niche, entrepreneurial pull from side projects, geographic preferences, technology stack misalignment.

**What Would Make Them Stay** (write as narrative paragraphs):

For each retention lever, write a paragraph covering: what the lever is, what evidence suggests it works for this person, and how a company could actively deploy it. Consider: technical challenge and learning opportunities, ownership and autonomy, team quality, flexible work arrangements, equity/upside participation, conference speaking support, open-source contribution time, side project tolerance.

### 3.6 Signal Gaps & Open Questions

What couldn't you find? What remains unverified? Write each gap as a dedicated paragraph, not a single bullet point.

For each gap, include:
- **What the gap is**: Describe what couldn't be found or verified
- **What was searched**: List the specific platforms, registries, and queries attempted. "No evidence found" is useless without "searched X, Y, and Z."
- **What the absence might mean**: Interpret charitably — proprietary work, NDA, private person, platform not relevant to their domain. Absence of signal is NOT negative signal.
- **Which interview question addresses it**: Cross-reference to a specific question in Section 3.9 that would help close this gap

Example gaps to consider: unverifiable role claims, timeline gaps, skills with no public artifacts, missing online presence on expected platforms, uncorroborated achievement metrics, education details that can't be confirmed, unexplained career transitions.

**Important**: Frame gaps neutrally. A gap is not evidence of a problem — it's absence of information.

### 3.7 CV Improvement Recommendations (SELF-ASSESSMENT MODE ONLY)

> **This section ONLY appears when the user is assessing their own profile.** Never include this when evaluating someone else — it would be presumptuous and misplaced.

Constructive feedback on CV/profile presentation:
- What important context is missing? (company size, impact metrics, team sizes)
- Where could claims be more specific or quantified?
- What story is undersold that should be more prominent?
- What structural changes would make the CV clearer?
- What sections are well done and should be kept as-is?

### 3.7.5 The Hidden Detail That Tells You Everything

Identify the single most revealing data point about this candidate — the one finding that, by itself, tells you more about who they are than any resume line item could.

Present it as a boxed insight:

> **The Detail**: [What was discovered — be specific]
>
> **Why It Matters**: [What this reveals about their character, values, or working style]
>
> **What It Predicts**: [How this insight translates to workplace behavior — what you can expect from this person]

This should be a genuinely surprising or non-obvious finding. Not "they have 10 years of experience" — that's a resume fact. More like: "their personal farm website is registered under the same business entity as their tech consulting company" or "they built an AI bot trap into their personal site's robots.txt" or "their conference talk titles all follow a pattern of combining farming metaphors with data engineering concepts." The hidden detail is where the detective work pays off.

### 3.8 Final Verdict

Clear, actionable hiring recommendation. No hedging behind "it depends" — commit to a position.

**Recommendation**: [Hire / Conditional Hire / Pass]

If Conditional Hire, state the specific conditions that must be met (e.g., "Hire if the role offers X" or "Hire contingent on verifying Y in interview").

**Best-Fit Roles** (2-3 specific roles with justification):
- [Role 1]: [Why they'd excel — specific evidence]
- [Role 2]: [Why they'd excel — specific evidence]

**Roles to Avoid** (2-3 with justification):
- [Role 1]: [Why it would be a poor fit — specific evidence]
- [Role 2]: [Why it would be a poor fit — specific evidence]

**Bottom Line**: One paragraph that captures the essence of this candidate. What is the core question a hiring manager should ask themselves? Not "are they qualified?" — that's what the rest of the report covers. The bottom line should be the strategic question: "The question isn't whether they're good enough — it's whether your environment is [X] enough to [Y]."

### 3.9 Targeted Interview Questions

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
- 2-3 questions to **test professional depth** in claimed areas of expertise
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
- Do not assume online activity = professional quality (many excellent professionals work on proprietary or confidential matters)

### Do NOT doxx
- In "Evaluating Others" mode, NEVER attempt to link pseudonymous accounts
- If you discover a likely pseudonymous connection by accident, DO NOT include it in the report
- The candidate's right to maintain separate identities is absolute
- Only use information the candidate has chosen to make publicly accessible under their real identity

---

## OSINT Search Strategy Reference

### Platform Access & Fallback Protocol

Many platforms block AI tool fetches (403 errors, JavaScript-only rendering, tool deny-lists). Use this **tiered fallback strategy** for every URL:

1. **WebFetch first** (fast, AI-processed, easiest to use)
2. **Platform-specific API** if WebFetch fails (see table below)
3. **Browser-emulating curl** if both fail (see below — works on LinkedIn, Reddit, Stack Overflow, and most platforms that block AI tools but serve HTML to browsers)
4. **WebSearch** as last resort (Google/Bing snippets)

Do NOT retry a blocked URL or conclude the platform has no data until you've tried all tiers.

#### Universal Fallback: Browser-Emulating Curl

When WebFetch and platform APIs fail, use curl via the Bash tool with browser headers. This bypasses most bot detection by mimicking a real Chrome browser request. **Confirmed working on: LinkedIn, Reddit (old.reddit.com), Stack Overflow, and most platforms that serve server-rendered HTML.**

```bash
curl -s -L \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  -H 'Sec-Fetch-Mode: navigate' \
  -H 'Sec-Fetch-Site: none' \
  -H 'Sec-Fetch-Dest: document' \
  'URL_HERE'
```

**Parsing curl output** (raw HTML — extract what matters):
- **Meta tags**: `grep -i 'og:title\|og:description\|description' output.html` — headline, about section, current role
- **JSON-LD structured data**: `grep -o '{"@context":"http://schema.org"[^}]*}' output.html` — rich Person/Organization data (jobs, education, dates)
- **Specific HTML sections**: `grep -i 'experience\|education\|skill' output.html` — profile sections

**LinkedIn-specific curl notes**:
- Returns 406KB+ of data including: JSON-LD Person schema (name, jobTitle, worksFor with dates, alumniOf, location, follower count), og:description (headline + about + experience + education summary), recent LinkedIn posts with content and like counts, full HTML experience/education sections
- Profile must be publicly visible (some profiles with strict privacy settings return 404)
- Use the LinkedIn vanity URL: `linkedin.com/in/{slug}`
- **Disambiguation when multiple profiles share the same name**: When WebSearch returns multiple LinkedIn results for a common name, do NOT assume the first result is the right one. Curl EACH candidate profile and check `worksFor`, `alumniOf`, `jobTitle`, and location in the JSON-LD schema against known data. Non-vanity URL slugs (e.g., `jane-doe-9a7bbb270`) are common for less-active profiles — do not skip them.
- **Check LinkedIn bio for link aggregators**: LinkedIn profiles frequently contain Linktree or similar links in the about section or contact info. These are goldmines — a single Linktree can reveal portfolio items, project pages, and profiles on 5+ platforms.

**Reddit-specific curl notes**:
- Use `old.reddit.com/user/{username}` — old Reddit serves full server-rendered HTML
- New reddit (www.reddit.com) is a JS SPA and won't work with curl
- Returns full post/comment history in HTML

**Stack Overflow-specific curl notes**:
- Returns complete profile HTML: reputation, badges, top tags, member duration, about section
- Follow redirects (`-L` flag) as SO uses 301 redirects

#### Platforms with Working API Alternatives

| Platform | Block Type | Working Alternative URL | Notes |
|----------|-----------|------------------------|-------|
| **LinkedIn** | Login wall + JS SPA | **curl with browser headers** (see above) on `linkedin.com/in/{slug}` | Returns JSON-LD Person schema + og:description + full HTML profile. Also: getprog.ai for tech profiles (WebSearch `site:getprog.ai` → WebFetch), theorg.com for org charts. |
| **Twitter/X** | Full block (JS wall + Nitter dead + syndication empty) | No working direct access. | All Nitter instances and syndication endpoint are dead. Use WebSearch only: `"username" site:twitter.com OR site:x.com`. |
| **Stack Overflow** | Tool deny-list (WebFetch blocked) | **curl with browser headers** (see above) on `stackoverflow.com/users/{id}/{slug}` | Returns full profile HTML: reputation, badges, top tags, about section. First find URL via WebSearch. |
| **Reddit** | Bot blocking (403/empty) | **curl with browser headers** on `old.reddit.com/user/{username}` | Returns full server-rendered HTML. Also: Arctic Shift API `arctic-shift.photon-reddit.com/api/posts/search?author={username}&limit=100` for archived data. |
| **npm** | 403 on website | `registry.npmjs.org/{package}` or `registry.npmjs.org/-/v1/search?text={query}&size=10` | JSON API returns full package metadata |
| **Docker Hub** | JS SPA | `hub.docker.com/v2/repositories/{namespace}/{repo}/` | JSON API with pull counts, stars, tags |
| **Terraform Registry** | JS wall | `registry.terraform.io/v1/providers/{namespace}/{name}` | JSON API with download counts, versions |
| **ORCID** | JS SPA | `pub.orcid.org/v3.0/{orcid-id}` | JSON API with publications, employment, keywords |
| **FINRA BrokerCheck** | JS SPA | `api.brokercheck.finra.org/search/individual?query={name}&nrows=10` | JSON API with CRD numbers, firm affiliations |
| **Google Scholar** | JS SPA | **curl with browser headers** on `scholar.google.com/citations?user={id}` | Returns full HTML: citation count, h-index, i10-index, research areas, paper list. Also: Semantic Scholar API `api.semanticscholar.org/graph/v1/author/search?query={name}&fields=name,hIndex,citationCount,paperCount` |
| **IMDb** | JS SPA | **curl with browser headers** on `imdb.com/name/{id}/` | Returns full JSON-LD Person schema (name, jobTitle, birthDate, bio) + og:description with filmography. Also: `dbpedia.org/data/{Person_Name}.json` or `themoviedb.org` |
| **Kaggle** | JS SPA | **curl with browser headers** on `kaggle.com/{username}` | Returns og:title + og:description with name, bio, titles, affiliations. |
| **SEC EDGAR** | 403 on sec.gov | **curl with browser headers** on `efts.sec.gov/LATEST/search-index?q={query}&forms={form_type}` | Returns JSON with filing data: company names, CIK, dates, filing types. Works for proxy statements (DEF 14A), 10-K, etc. |
| **Google Patents** | WebFetch fails | **curl with browser headers** on `patents.google.com/patent/{patent_id}` | Returns full patent HTML with title, abstract, claims. |
| **NPI Registry** | — | `npiregistry.cms.hhs.gov/api/?number={npi}&version=2.1` | JSON API for healthcare provider lookup. Works with both WebFetch and curl. |
| **Discogs** | 403 on website | `api.discogs.com/artists/{id}` or `/database/search?q={query}&type=artist` | JSON API with artist profiles, discography, band members. |
| **Linktree** | No blocking | **curl with browser headers** on `linktr.ee/{username}` | Returns clean HTML + JSON-LD ProfilePage with all link URLs, titles, profile metadata (name, status, timestamps). Also works with WebFetch. Best single source for candidates who aggregate their professional presence in one place. |

#### Platforms with No Direct Access (Web Search Only)

| Platform | Block Type | Recommended Fallback |
|----------|-----------|---------------------|
| **Medium** | 403 (even curl) | WebSearch; check author's personal blog for cross-posts |
| **Glassdoor** | 403 (even curl) | WebSearch `"company" reviews site:glassdoor.com` |
| **Crunchbase** | 403 (even curl) | WebSearch for funding data; Wikipedia company pages |
| **Product Hunt** | 403 (even curl) | WebSearch `site:producthunt.com "product name"` |
| **Wellfound/AngelList** | 403 (even curl) | WebSearch `site:wellfound.com "company"` |
| **ResearchGate** | 403 (even curl) | WebSearch `"First Last" site:researchgate.net` — Google snippets return profile metadata (title, institution, department, publications). Also try: `"First Last" ResearchGate [specialty]`. Profile URLs follow pattern `/profile/First_Last` or `/profile/First_Last2` (numbered suffix for common names). Use Google Scholar or Semantic Scholar API for deeper publication data. |
| **Muck Rack** | 403 (even curl) | WebSearch `"First Last" site:muckrack.com`; try Google News byline search instead |
| **Behance** | 404 (even curl) | WebSearch `site:behance.net "username"`; try Dribbble or ArtStation instead |
| **USPTO TSDR** | 403 | Use Google Patents with curl (works!) as full alternative |

### Effective Search Patterns

| Goal | Search Pattern | Notes |
|------|---------------|-------|
| Find LinkedIn (full profile) | `"First Last" site:getprog.ai` then WebFetch the result URL | Returns complete experience, education, skills, about section. Tech profiles only (~60M). |
| Find LinkedIn (any profile) | `"First Last" site:linkedin.com [job title]` | Google snippets show headline, current role, location. Bing often shows more. |
| Find GitHub | `"First Last" site:github.com` or search GitHub directly | Check contributions, not just repos |
| Find Reddit activity | `[project name] site:reddit.com` (NO quotes on project name) | Plain keywords work better than exact match |
| Reddit via archive API | `arctic-shift.photon-reddit.com/api/posts/search?author=[username]&limit=100` | Bypasses Reddit bot blocking; also search comments and keywords. Use for EVERY known handle. |
| Find Stack Overflow | `[username] site:stackoverflow.com` AND `[project-name] site:stackoverflow.com` | Search BOTH by name and by project |
| Find Terraform artifacts | `[username] site:registry.terraform.io` AND search GitHub for `terraform-provider-*` repos | Strong Go + IaC signal |
| Find Docker images | `[username] site:hub.docker.com` | Check image descriptions for links |
| Find HN activity | `[project name] site:news.ycombinator.com` | Also search for "Show HN" posts |
| Find forum posts | `[project name] [technology] forum` | Try technology-specific forums |
| Cross-platform username | `"[username]" -site:[known-platform]` | Exclude known platforms to find new ones |
| Find talks/presentations | `"First Last" [topic] site:youtube.com OR site:slideshare.net` | Also check conference websites directly |
| Find publications | `"First Last" site:scholar.google.com` or `"First Last" arxiv` | For research-oriented candidates |
| Find Google Scholar profile | WebSearch `"First Last" site:scholar.google.com/citations` then curl the profile URL | Discovers author profile with citation metrics; also try `scholar.google.com/citations?view_op=search_authors&mauthors=First+Last` |
| Find ResearchGate profile | WebSearch `"First Last" site:researchgate.net [institution]` | Returns profile metadata in Google snippets even though direct access is blocked |
| Find link aggregator | `"First Last" site:linktr.ee OR site:beacons.ai OR site:about.me` | Especially useful for creators, freelancers, and early-career professionals; aggregates all other profile links |
| Find academic profile | `"First Last" site:researchgate.net OR site:orcid.org` | Check citation counts, h-index |
| Find packages | Search npm/PyPI/crates.io directly for username | Often matches GitHub handle |
| Find patent filings | `"First Last" site:patents.google.com` | For senior/inventive roles |
| Verify company claims | `"[Company Name]" crunchbase OR glassdoor OR linkedin` | Cross-reference size, funding, status |
| Find archived content | `"First Last" site:web.archive.org` | For deleted blogs/sites |
| Find design work | `"First Last" site:dribbble.com OR site:behance.net` | For design-oriented candidates |
| Fetch Twitter/X profile | `syndication.twitter.com/srv/timeline-profile/screen-name/{username}` | Twitter's own SSR endpoint; fallback to `nitter.tiekoetter.com/{username}` |
| Search npm packages | `registry.npmjs.org/-/v1/search?text={query}&size=10` | JSON API; also `registry.npmjs.org/{package}` for specific package |
| Fetch Docker Hub repo | `hub.docker.com/v2/repositories/{namespace}/{repo}/` | JSON API with pull counts, stars, tags |
| Fetch Terraform provider | `registry.terraform.io/v1/providers/{namespace}/{name}` | JSON API with download counts, versions |
| Fetch ORCID profile | `pub.orcid.org/v3.0/{orcid-id}` | JSON API with publications, employment, keywords |
| Search FINRA brokers | `api.brokercheck.finra.org/search/individual?query={name}&nrows=10` | JSON API with CRD numbers, firm affiliations |
| Search academic authors | `api.semanticscholar.org/graph/v1/author/search?query={name}` | JSON; add `&fields=name,hIndex,citationCount,paperCount` for metrics |
| Fetch film/TV credits | `dbpedia.org/data/{Person_Name}.json` | Structured data; also try `themoviedb.org/person/{id}` or `rottentomatoes.com/celebrity/{slug}` |

### Search Tips (Learned from Live Testing)
- Plain keywords consistently outperform exact-match quoted strings for Reddit
- Search for unique project names or technical terms, not the person's name
- One confirmed username on any platform opens up their full activity on that platform
- Conference speaker pages often have bios with links to other profiles
- Git commit emails can sometimes be found in public repos (ethically: only if the repo is public)
- Company "About" or "Team" pages often have richer bios than LinkedIn
- Try at least 5 different query formulations per platform before concluding no presence exists
- When one query finds a username, immediately pivot to searching that username across all other platforms
- Search for project names WITH and WITHOUT hyphens/separators — `my-cool-project` vs `my cool project`
- Check for link aggregator profiles (Linktree, Beacons, Carrd) EARLY — especially for creators, freelancers, and early-career professionals. A single aggregator page can reveal 5+ platform profiles and direct links to portfolio items, publications, or projects that individual platform searches would miss
- When multiple people share a name on LinkedIn, do NOT default to the first result — curl each candidate profile and check the JSON-LD worksFor/alumniOf fields against known institutional affiliations

### Platform-Specific Notes

**Reddit**:
- Reddit actively blocks AI crawlers — both `site:reddit.com` searches and direct URL fetches often fail with 403 errors or empty results.
- Try web search first: plain project name, project name without hyphens, project URL, project + domain keyword, username + reddit, domain-specific subreddit searches.
- **When web search fails, use archive APIs immediately** — do not conclude "no Reddit presence." Use Arctic Shift API (`arctic-shift.photon-reddit.com/api/posts/search?author=[username]&limit=100`) or PullPush (`api.pullpush.io`) as fallback. See the Reddit Archive Fallback section in Agent A for full URL patterns.
- When you find a post (via any method), extract the username and search their full history through the archive API.
- **Verification reminder**: A matching username on Reddit (even identical to their GitHub/Last.fm handle) does NOT confirm identity. Always apply the Link Classification rules — identity verification gate for Self-Assessment Enhanced, explicit links only for Evaluating Others.

**GitHub**:
- Stars and followers are vanity metrics. Look at: contribution frequency, code quality in repos, PR review comments, issue discussions.
- Check org memberships — contributions may be under org accounts.
- Look at the language breakdown across repos for a true skill map.
- Check for Terraform providers (`terraform-provider-*` repos) — these are written in Go and signal deep IaC knowledge.

**Stack Overflow**:
- **WebFetch is blocked** (tool deny-list), but **curl with browser headers works perfectly** — returns full profile HTML with reputation, badges, top tags, member duration, about section. Use `curl -s -L -H 'User-Agent: Mozilla/5.0 ...' 'https://stackoverflow.com/users/{id}/{slug}'`. See the Browser-Emulating Curl section for the full command.
- If you don't know the SO user ID, first WebSearch `"{name}" site:stackoverflow.com` to find the profile URL, then curl it.
- Look at the questions they ask, not just answers. Someone asking good questions in advanced topics signals genuine learning depth.
- Check tags the user is active in — this reveals their actual expertise areas vs. claimed ones.
- Self-answered questions often document solutions to hard problems.

**LinkedIn**:
- **Direct WebFetch is blocked** (login wall). Use this tiered approach:
  1. **curl with browser headers** (BEST): `curl -s -L -H 'User-Agent: Mozilla/5.0 ...' 'https://www.linkedin.com/in/{slug}'` — returns full JSON-LD Person schema (name, jobTitle, worksFor with dates, alumniOf, location, follower count), og:description (headline + about + experience summary), recent posts with content, and full HTML profile sections. See the Browser-Emulating Curl section for the full command.
  2. **getprog.ai** (tech profiles only): WebSearch `"First Last" site:getprog.ai` → WebFetch the result URL. Returns complete profile for ~60M software engineer profiles.
  3. **WebSearch** (any profile): `"First Last" site:linkedin.com [job title]` — Google/Bing snippets show headline, current role, location, and sometimes about section excerpts.
- For company org charts and team data, try WebFetch on `theorg.com/org/{company-name}`
- Recommendations from senior people carry more weight. Look at WHO recommends them, not just how many recommendations.
- Check activity feed for posts and comments — reveals communication style and thought leadership.

**Terraform Registry**:
- The website is JS-only. Use the API: `registry.terraform.io/v1/providers/{namespace}/{name}` returns JSON with download counts and versions.
- Search for the candidate's GitHub username/org.
- Published providers require deep Go AND Terraform knowledge — this is a very strong signal.
- Published modules indicate IaC maturity.

**Academic/Research Platforms** (Google Scholar, ResearchGate, ORCID, Semantic Scholar):
- **Google Scholar**: Two access patterns: (1) If you know the user ID: `curl ... 'scholar.google.com/citations?user={id}'` returns full HTML. (2) **To DISCOVER a profile**: WebSearch `"First Last" site:scholar.google.com/citations` to find the author profile URL, OR use `scholar.google.com/citations?view_op=search_authors&mauthors=First+Last` via curl (returns author search results in HTML). Also search for their publications directly: `"First Last" scholar.google.com` — Google indexes individual paper listings even when the author hasn't created a formal profile.
- **ResearchGate is blocked** (403 even with curl), but **WebSearch works well**: `"First Last" site:researchgate.net` returns Google snippets with profile metadata including title, institution, department, and recent publications. Profile URLs use the pattern `/profile/First_Last` or `/profile/First_Last2` (numbered suffixes for common names). **Do NOT conclude "no ResearchGate profile" without trying this WebSearch pattern** — it frequently succeeds even when direct access fails.
- Check citation counts and h-index for research impact.
- Look for co-authors — reveals collaboration networks.
- Check if papers are in high-impact venues (top-tier conferences, high-IF journals).
- Pre-prints on arXiv/bioRxiv signal active research engagement.

**IMDb** (film/TV/media candidates):
- **Works with curl** — `curl -s -L -H 'User-Agent: Mozilla/5.0 ...' 'https://www.imdb.com/name/{id}/'` returns JSON-LD Person schema with name, jobTitle, birthDate, and full bio + filmography in og:description.

**SEC EDGAR** (executive/finance/board member candidates):
- The main sec.gov site blocks all automated access (403), but the **full-text search API works with curl**: `curl -s -L -H 'User-Agent: Mozilla/5.0 ...' 'https://efts.sec.gov/LATEST/search-index?q=%22person+name%22&forms=DEF+14A'` returns JSON with filing data. Use `forms=DEF+14A` for proxy statements (board members, executive compensation), `forms=10-K` for annual reports.

**Twitter/X**:
- **Completely walled off as of 2025.** All twitter.com/x.com pages require JS. The syndication endpoint (`syndication.twitter.com`) now returns empty data. All Nitter instances (xcancel.com, nitter.tiekoetter.com) and Nitter proxies (twiiit.com) are dead or 403.
- **Only working approach**: WebSearch with `"username" site:twitter.com OR site:x.com`. Google still indexes tweets and profiles. Also try `"username" twitter` without site filter.
- For thread-heavy users, try `threadreaderapp.com/user/{username}` for unrolled threads (may still work as it's a separate service).
- Do NOT waste time retrying Nitter instances or syndication URLs — they are confirmed dead. If web search finds nothing, note it as a platform limitation and move on.

**npm / Docker Hub**:
- npm website returns 403. Use the JSON API: `registry.npmjs.org/{package}` for package details, `registry.npmjs.org/-/v1/search?text={query}&size=10` for searching by author/keyword.
- Docker Hub is a JS SPA. Use the API: `hub.docker.com/v2/repositories/{namespace}/{repo}/` returns JSON with pull counts, stars, and tags.

**Crunchbase / Glassdoor**:
- Both return 403 with no API workaround (paid API keys required). Use WebSearch as fallback.
- For Crunchbase data: WebSearch for funding rounds, check Wikipedia company pages, and AngelList/Wellfound profiles.
- For Glassdoor: WebSearch `"company" reviews site:glassdoor.com` — Google snippets often include rating and review counts.

**SEC EDGAR**:
- The main sec.gov site returns 403, but the **full-text search API works with curl**: `efts.sec.gov/LATEST/search-index?q=%22person+name%22&forms=DEF+14A` returns JSON. Also WebSearch `site:sec.gov "company" 10-K` for Google-indexed filings.

**IMDb**:
- **Works with curl** — returns JSON-LD Person schema + full bio. Also try as fallbacks:
  - `dbpedia.org/data/{Person_Name}.json` — structured data from Wikipedia/DBpedia
  - `themoviedb.org/person/{id}-{name}` — full credits and bio (web-accessible)
  - `rottentomatoes.com/celebrity/{slug}` — filmography with audience/critic scores

### Domain-Adaptive Search

The platforms and registries to search depend on the candidate's professional domain. Adapt your search strategy:

| Domain | Primary Artifact Sources | Key Registries/Databases | Community Platforms | OSINT Quality |
|--------|------------------------|-------------------------|--------------------|----|
| **Software Engineering** | GitHub, GitLab | npm, PyPI, crates.io, Docker Hub, Terraform Registry | SO, HN, Reddit, Dev.to | HIGH |
| **Data Science / ML** | GitHub, Kaggle, Hugging Face | PyPI, conda-forge, HF Hub | Kaggle, Papers with Code, r/MachineLearning | HIGH |
| **DevOps / Platform** | GitHub | Docker Hub, Terraform Registry, artifacthub.io, Ansible Galaxy | r/devops, HashiCorp Discuss, CNCF Slack | HIGH |
| **Security / InfoSec** | GitHub, CTF profiles | — | HackerOne, Bugcrowd, CTFtime, r/netsec | MEDIUM |
| **Mobile Development** | GitHub | App Store, Play Store, Garmin Connect IQ | r/iOSProgramming, r/androiddev | HIGH |
| **Design / UX** | Dribbble, Behance, Figma Community | — | Designer News, r/design, ADPList | MEDIUM |
| **Academic / Research** | Google Scholar, ORCID, DBLP | arXiv, bioRxiv, Semantic Scholar | ResearchGate, academic Twitter | HIGH |
| **Journalism / Writing** | Muck Rack, Google News, Amazon | — | Twitter/X, r/journalism, SPJ | HIGH |
| **Visual Arts / Photography** | ArtStation, Behance, 500px | Getty/Shutterstock contributor | r/Art, r/photography | MEDIUM |
| **Video / Film** | IMDb, Vimeo, YouTube | FilmFreeway | r/Filmmakers, No Film School, Stage 32 | MEDIUM |
| **Music / Audio** | Discogs, Spotify, Bandcamp | ASCAP/BMI repertoire | r/WeAreTheMusicMakers, Gearspace | MEDIUM |
| **Sales / Business Dev** | LinkedIn | Salesforce Trailblazer (trailblazer.me) | r/sales, Pavilion, Sales Hacker | LOW |
| **Marketing / Growth** | LinkedIn, Substack, Medium | Google/HubSpot/Meta certs (Credly) | r/marketing, GrowthHackers | LOW |
| **Finance / Accounting** | SEC EDGAR, Seeking Alpha | FINRA BrokerCheck, CPA state boards, CFA verify | Wall Street Oasis, r/FinancialCareers | HIGH (credentials) |
| **Legal** | PACER/CourtListener, Google Scholar | State bar directories, Martindale-Hubbell | Above the Law, JD Supra, r/Lawyers | HIGH |
| **Healthcare** | PubMed, ClinicalTrials.gov | NPI Registry, state medical boards, ABMS verify | Doximity, r/medicine, KevinMD | HIGH |
| **Entrepreneurship / Founder** | Crunchbase, AngelList, Product Hunt | State SOS, USPTO, SEC EDGAR | Indie Hackers, HN, r/startups | HIGH |
| **Executive / C-Suite** | SEC EDGAR (proxy), Bloomberg | ProPublica Nonprofit Explorer (990s) | — | HIGH (public co) / LOW (private) |
| **Architecture** | ArchDaily, Dezeen | State licensing boards, NCARB | Archinect, r/architecture | MEDIUM |
| **Engineering (PE)** | USPTO patents | State PE boards, NCEES | eng-tips.com, r/engineering | HIGH (credentials) |
| **Real Estate** | Zillow/Redfin agent profiles | State RE commissions, county deed records | BiggerPockets, r/RealEstate | HIGH |
| **Non-Profit** | ProPublica Nonprofit Explorer | GuideStar/Candid, state charity registrations | SSIR, Chronicle of Philanthropy | MEDIUM-HIGH |
| **Government / Policy** | Government sites, think tanks | LegiStorm, GovTrack | r/PublicPolicy, GovLoop | MEDIUM |
| **HR / People Ops** | LinkedIn | SHRM/HRCI cert verification | r/humanresources, SHRM Connect | LOW |
| **Consulting** | LinkedIn, firm bio pages | PMP (PMI registry), CMC directory | r/consulting, Fishbowl | LOW |
| **Project Management** | LinkedIn | PMP (PMI), CSM (ScrumAlliance), SAFe (Scaled Agile) | r/projectmanagement | LOW |

Don't limit yourself to this table — it's a starting point. If the candidate works in a niche domain, search for domain-specific communities and registries.
