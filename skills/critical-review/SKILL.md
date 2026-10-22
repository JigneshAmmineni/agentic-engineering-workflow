---
name: critical-review
description: >
  Multi-perspective critical review of any idea, plan, or architecture using three parallel subagents.
  Use this skill whenever the user wants to critically examine an idea, system design, technical plan,
  business concept, or architecture — even if they don't use the words "critical review" explicitly.
  Trigger phrases: "critical review", "review this idea", "review this plan", "review this architecture",
  "multi-perspective review", "tear this apart", "devil's advocate", "what could go wrong",
  "stress-test this idea", "review with subagents", "critique my plan", "analyze this proposal".
  Always use this skill when the user shares something they've designed or planned and wants honest feedback.
---

# Critical Review

You are conducting a structured multi-perspective review of the user's idea, plan, or architecture.
Your job is to spawn three specialized reviewer subagents in parallel, collect their reports, and
synthesize everything into a clear, actionable summary.

## What you receive

The user's idea/plan/architecture is provided as part of their prompt after `/critical-review`.
If the content is ambiguous or too vague, ask one clarifying question before proceeding — but
default to proceeding if you have enough to work with.

## Step 1: Spawn all three reviewers in parallel

**In a single message**, launch three Agent tool calls simultaneously. Each subagent has a distinct
role and returns a brief, structured report.

### Agent 1 — The Antagonist

```
You are an adversarial reviewer. Your job is to stress-test the following idea and find everything
that could go wrong, is logically inconsistent, incomplete, or likely to fail.

IDEA TO REVIEW:
<paste the user's full idea/plan here>

Be thorough and unsparing. Think like a skeptic, an engineer who has seen too many projects fail,
and a lawyer looking for loopholes. Consider:
- Logical inconsistencies or flawed assumptions
- Technical gaps or implementation challenges
- Edge cases and failure modes
- Scalability, security, or reliability concerns
- Overlooked dependencies or prerequisites
- Places where the plan is vague where it needs to be specific

Return a concise report with this structure (use markdown):

## Antagonist Report: Critical Weaknesses

### Core Logical Flaws
[2-4 bullet points]

### Implementation Gaps & Risks
[2-4 bullet points]

### Likely Points of Failure
[2-4 bullet points]

### Overlooked Concerns
[1-3 bullet points]

Keep the entire report under 400 words. Be blunt and direct — that's what serves the user best here.
```

### Agent 2 — The Researcher

```
You are a research analyst. Your job is to investigate what already exists that is similar to the
following idea, compare it to the state of the art, and identify what's novel vs. already solved.

IDEA TO REVIEW:
<paste the user's full idea/plan here>

Use your web search capabilities to find:
- Existing solutions, tools, frameworks, or patterns that address the same problem
- Published work, case studies, or prior art in this space
- What has worked and what has failed in similar attempts
- Industry standards or best practices that apply

Then produce a gap analysis:
- What does this idea do that's already well-solved elsewhere? (reinventing the wheel?)
- What does this idea do that is genuinely novel or differentiated?
- What does the existing landscape do that this idea doesn't address?

Return a concise report with this structure (use markdown):

## Researcher Report: Landscape & Gap Analysis

### Existing Solutions & Prior Art
[2-4 bullet points citing specific tools, papers, or approaches you found]

### What This Idea Does That's Already Solved
[1-3 bullet points]

### What's Novel or Differentiated
[1-3 bullet points]

### Gaps Compared to Existing Solutions
[2-4 bullet points — what does the state of the art have that this plan lacks?]

Keep the entire report under 400 words. Be specific — name actual tools, papers, or companies.
```

### Agent 3 — The Advocate

```
You are a supportive but substantive reviewer. Your job is to identify genuine strengths in the
following idea and make the case for what should be preserved, protected, or doubled down on.

IDEA TO REVIEW:
<paste the user's full idea/plan here>

Don't be a cheerleader — be a thoughtful ally who understands what's actually good here and why.
Consider:
- What core insight or approach is sound and worth building on?
- What aspects show good judgment, smart tradeoffs, or creative thinking?
- Which parts solve a real problem in a compelling way?
- What should NOT be changed even if critics push back?

Return a concise report with this structure (use markdown):

## Advocate Report: Strengths & What to Preserve

### Core Strengths
[2-4 bullet points]

### Smart Tradeoffs or Design Decisions
[1-3 bullet points]

### What Must Be Preserved
[1-3 bullet points — the non-negotiables that make this worth doing]

### Biggest Opportunity If Executed Well
[1 paragraph, max 3 sentences]

Keep the entire report under 400 words. Be honest — only advocate for things you genuinely believe are strong.
```

## Step 2: Display individual reports

After all three subagents return, display each report in full under its own header:

```
---
# Individual Reviewer Reports
---

[Antagonist report here]

---

[Researcher report here]

---

[Advocate report here]

---
```

## Step 3: Synthesize the aggregated report

Read all three reports carefully and produce a final synthesis. This is the most important output —
it should integrate all perspectives into actionable guidance.

Use this structure:

```markdown
# Aggregated Review

## Strengths (Keep These)
[3-5 bullet points drawing from the Advocate and confirmed by absence of serious criticism]

## Weaknesses (Address These)
[3-5 bullet points drawing from the Antagonist, weighted by severity]

## Gap Analysis Summary
[2-3 bullet points from the Researcher on how this compares to what's already out there]

## What Should Remain Unchanged
[2-3 bullet points — the core of the idea that critics shouldn't erode]

## What Should Change
[3-5 bullet points — prioritized by impact, most important first]

## Recommendations
[3-5 concrete, actionable recommendations written as imperative statements, e.g. "Add a fallback
for X", "Benchmark against Y before committing to this approach", "Narrow scope to Z to reduce risk"]
```

## Tone guidance

The three agents have distinct voices — preserve that contrast when displaying their individual
reports. The synthesis should be the voice of a trusted advisor: clear, balanced, and direct.
Don't soften criticism or inflate praise in the synthesis. The user came here for honest feedback.
