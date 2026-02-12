---
name: outcome-engineering
description: >
  Use when starting significant features, evaluating build-vs-buy decisions,
  reframing failed approaches, or when the user mentions "outcome engineering",
  "o16g", "outcome-first", "what's the outcome", or "frame this as an outcome".
  Also use proactively when a task description focuses on implementation
  ("build X", "add Y") without defining the desired outcome or verification criteria.
---

# Outcome Engineering (o16g)

Based on the [o16g manifesto](https://o16g.com/) by Cory Ondrejka. Software engineering is outcome delivery, not code production.

| Task Thinking | Outcome Thinking |
|---------------|-----------------|
| "Build feature X" | "Enable users to achieve Y, verified by Z" |
| "Fix bug B" | "Restore correct behavior, prevent recurrence" |
| "Refactor module M" | "Reduce change cost from X to Y, measured by Z" |
| "Add technology T" | "Solve problem P with measurable improvement" |

---

## Phase 1: The Outcome Frame

Before starting significant work, produce an **Outcome Frame**. This takes ~3 minutes, not 30. Skip it for trivial tasks (typo fixes, obvious single-line bugs).

### 1. Define the Outcome (P01: Human Intent)

State the measurable change being delivered. Not what you'll build — what will be different when you're done.

**Anti-pattern**: "Build a REST API for user management"
**Outcome-first**: "External services can create, read, update, and delete user records over HTTP, with <200ms p95 latency"

Ask: **"What is measurably different when this is done?"**

### 2. Define Verification (P02: Verified Reality)

How will you PROVE it worked? Not "it looks right" — observable, repeatable evidence.

Verification must be concrete:
- Tests that pass/fail (unit, integration, e2e)
- Observable behavior (API returns X, UI shows Y)
- Metrics that move (latency drops, error rate decreases)
- User action that succeeds (can complete workflow Z)

**Anti-pattern**: "Test it manually and make sure it works"
**Outcome-first**: "Integration test hits /users CRUD endpoints, asserts 2xx responses and correct payloads. Load test confirms p95 <200ms at 100 RPS."

Ask: **"What specific evidence proves this outcome was achieved?"**

### 3. Justify the Cost (P04: Backlog Dead)

Is this worth the compute/effort? Not everything that could be built should be built.

Consider:
- Does this directly serve a user need or business goal?
- What's the cost of NOT doing this?
- Is there a simpler way to achieve 80% of the outcome?
- Are we building or should we buy/integrate?

**Anti-pattern**: "It's in the backlog so we should do it"
**Outcome-first**: "Users currently can't reset passwords without emailing support (3 tickets/day). Self-service reset eliminates this entirely."

Ask: **"Is this worth the effort? What happens if we don't do it?"**

### 4. Identify Risk Gates (P15: Risk Gates)

What unknowns could derail this? Unknown or unmitigated risk blocks progress — surface it now, not after you've built the wrong thing.

Risk categories:
- **Technical**: Can this even work? (performance, compatibility, dependencies)
- **Domain**: Do we understand the problem correctly? (assumptions, edge cases)
- **Integration**: Will this play well with existing systems?
- **Irreversibility**: What can't be undone?

**Anti-pattern**: "We'll figure out the edge cases as we go"
**Outcome-first**: "Risk: We assume the OAuth provider supports PKCE. Gate: Verify PKCE support before building the auth flow."

Ask: **"What could make this fail? What should we verify before committing?"**

### 5. Form a Hypothesis (P07: Build to Learn)

When uncertain, frame work as an experiment. What are you testing? What's the cheapest way to validate it?

**Anti-pattern**: Build the full feature, then discover the approach doesn't work
**Outcome-first**: "Hypothesis: Server-sent events can deliver real-time updates at our scale. Experiment: Spike a minimal SSE endpoint, load test at 1000 concurrent connections. If it fails, evaluate WebSockets before building the full notification system."

Ask: **"What assumption are we testing? What's the cheapest way to validate it?"**

### Output Format

After working through the five questions, produce:

```
## Outcome Frame

**Outcome**: [What is measurably different when done]
**Verification**: [Specific evidence that proves success]
**Cost justification**: [Why this is worth doing now]
**Risk gates**: [What to verify before committing]
**Hypothesis**: [What assumption we're testing, if uncertain] (optional — only when there's genuine uncertainty)
```

If the user provides enough context that answers are obvious, fill in what's clear and only ask about what's genuinely uncertain.

---

## Phase 2: Execution Guardrails

During work, apply these lightweight checks. They don't slow you down — they prevent you from building the wrong thing fast.

### Context First (P06)

**Read before writing. Understand before changing.**

Before modifying any code:
- Read the files you'll change
- Understand existing patterns and conventions
- Check for related code that might be affected
- Look for tests that cover this behavior

When the urge strikes to "just start coding," that's exactly when you need 2 minutes of context-gathering.

### Build to Learn (P07)

**When uncertain, build the cheapest experiment first.**

If you're not sure an approach will work:
- Build a minimal spike/proof-of-concept
- Test the riskiest assumption first
- Get feedback before investing in polish
- Prototype > plan when dealing with unknowns

The goal is to fail fast and cheap, not to fail slow and expensive.

### Show Your Work (P13)

**Document reasoning and rejected paths.**

When making non-obvious decisions:
- State what you chose and why
- Mention alternatives you considered and why you rejected them
- Note assumptions that, if wrong, would change the decision

This isn't bureaucratic overhead — it's future debugging information. When something goes wrong, "why did we do it this way?" has an answer.

### Risk Gates (P15)

**Before irreversible actions, stop and check.**

Before any action that's hard to undo:
- Destructive operations (delete, drop, force-push)
- Public-facing changes (API contracts, published interfaces)
- Data migrations (especially lossy ones)
- External integrations (third-party API calls that trigger actions)

Ask: "If this goes wrong, how do we recover?"

---

## Phase 3: Outcome Validation

When work completes, validate against the Outcome Frame.

### Evidence Check (P02, P16: Verification + Validation)

Go back to the Outcome Frame and check each verification criterion:

- **Tests passing?** Run them. Show the output.
- **Observable behavior confirmed?** Demonstrate it.
- **Metrics moved?** Measure them.
- **User action succeeds?** Walk through it.

Don't claim "done" on vibes. Show evidence.

### Decision Debugging (P08: Debug Decisions)

If the outcome **wasn't** achieved:

Don't just debug the code — **debug the decision**.

1. **Which assumption was wrong?** (Not "which line has a bug" but "what did we believe that turned out to be false?")
2. **When did we know?** (At what point did evidence suggest we were off track?)
3. **What would we do differently?** (Change the approach, not just patch the symptom)

This prevents the pattern of fixing symptoms while the root cause keeps producing new bugs.

### Immune System (P14)

When fixing a bug or failure:

- Fix the immediate issue
- **Then prevent recurrence**: add a test, add validation, improve the error message, update documentation
- Ask: "How do we make sure this category of problem can't happen again?"

A fix without prevention is just a temporary patch.

---

## Anti-Patterns: What o16g is NOT

**Not every task needs an Outcome Frame.** Typo fixes, obvious bugs, simple config changes — just do them. The frame is for significant work where "what does done look like?" isn't immediately obvious.

**Not analysis paralysis.** The Outcome Frame takes 3 minutes, not 30. If you're spending more time framing than building, you're doing it wrong. When in doubt, bias toward action with a lightweight frame.

**Not "build everything from scratch."** Build to Learn means test hypotheses cheaply — use existing tools, libraries, and patterns. The experiment is about validating the approach, not reinventing infrastructure.

**Not ignoring engineering discipline.** Tests, code review, CI/CD, security — these are the verification mechanisms that make outcome engineering work. Without them, "verified reality" is just "hoped-for reality."

**Not a replacement for existing skills.** Outcome engineering is a framing layer. It wraps around brainstorming, architecture, implementation — it adds "why" and "what outcome" without replacing "how."

---

## Quick Reference

| Phase | Key Question | Time |
|-------|-------------|------|
| **Frame** | What is measurably different when done? | ~3 min |
| **Execute** | Am I building toward the outcome or just writing code? | Continuous |
| **Validate** | Can I show evidence the outcome was achieved? | ~2 min |

| Principle | One-Liner |
|-----------|-----------|
| P01 Human Intent | Define the destination before exploring paths |
| P02 Verified Reality | Measurable evidence, not vibes |
| P04 Backlog Dead | Is this worth the compute? |
| P06 Context First | Read before writing |
| P07 Build to Learn | Cheapest experiment to validate |
| P08 Debug Decisions | Wrong outcome? Debug the decision, not just the code |
| P13 Show Work | Document reasoning and rejected paths |
| P14 Immune System | Fix the bug, then prevent the category |
| P15 Risk Gates | Unknowns block progress — surface them early |
| P16 Validation | Continuously verify against reality |
