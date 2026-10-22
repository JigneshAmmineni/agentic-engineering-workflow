---
name: sdlc-process
disable-model-invocation: true
description: >
  The owner's SDLC operating system for solo-dev projects with an AI pair:
  spec-first features with FR numbering, adversarial AI review cycles run to
  zero blockers, owner-approval economics (commits autonomous, every
  push/PR/merge/deploy individually approved), dated behavioral eval docs
  written before the run, an architecture document enforced at merge and
  deploy boundaries, and a two-branch model where prod only moves to
  verified deploys. Has two modes: SETUP (scaffold the whole system into a
  new project, including its CLAUDE.md) and OPERATE (run a session by the
  process). Run only when the owner explicitly invokes /sdlc-process: to set
  the process up in a project, or at the start of a session they want run by
  it. Never auto-invoke.
---

# The SDLC process

This skill captures a working process developed on a real project (product taken from spec through implementation, five-round
adversarial reviews, live behavioral testing, and production deploys). It
exists so any new project starts with the same system instead of
rediscovering it.

Two modes. Decide which applies before doing anything:

- **SETUP** — a new or unstructured project: scaffold the system (§1).
- **OPERATE** — a project already running this process: follow the rules of
  engagement (§2–§6). If the project's CLAUDE.md already encodes them,
  CLAUDE.md wins on any conflict; this skill is the detailed playbook
  behind it.

---

## 1. SETUP mode — scaffolding the system

Create these pieces, tailored to the project, with the owner confirming
each. The point of each piece is noted — keep the point, adapt the form.

**CLAUDE.md** (the project constitution — every session inherits it):
- *What this is*: one paragraph of product identity.
- *Stack table* and *key architectural decisions* with the why.
- *Workflow rules* (the load-bearing section — copy the substance of §2–§3
  below into it, condensed).
- *Hard constraints*: the non-negotiables a reviewer should flag on sight
  (security seams, provenance rules, instrumentation requirements).
- *Docs map*: what each governing doc is and when it must be accurate.

**Governing docs:**
- `ROADMAP.md` — vision, ordered feature list with a small rationale for the
  order, and a decision record: pins, deferrals, and staged decisions get
  dated entries here, including placeholder features ("this needs to
  exist; details researched later"). The roadmap is where live-testing
  findings land as future work instead of being lost.
- `REQUIREMENTS.md` — what the product must do: numbered Functional Requirements (FRs), and System Requirements/Non-Functional Requirements (SRs), separate by product specifications and technical specifications (two coding agents given the same requirements document should produce roughly the same product) and
  constraints, organized in § sections, one section per specced feature.
  This is the document implementation and review are held to. **Each FR is
  atomic — one behavior, individually traceable and testable** (it may
  carry its own technical/implementation spec and even architectural
  design choices where that makes sense, but never bundle separable
  behaviors under one number; when a feature-slice FR needs several
  mechanisms, sub-number them FR-N.1, FR-N.2, …). **The first sentence of
  every FR is strictly the requirement** — the behavior the software must
  have — with implementation detail and design rationale as following
  prose, so someone skimming only first sentences learns exactly what the
  software can do. **It records
  the target state, never the journey**: rework amends FRs in place as if
  they had always said so; a removed feature's requirements are deleted,
  not tombstoned with deprecation entries — the journey lives in the
  roadmap's decision record and in git history.
- `CURRENT-ARCHITECTURE.md` — the accurate map of the current product:
  code structure (module-level map with a one-line role per module, name a handful of load-bearing classes and functions by name, abstractions used for modularity, architectural seams, runtime walkthrough of a session, these are just examples, display the code structure however is best for that project; an engineer must be able to look at it and understand where to go if they want to deep dive specific parts of the system or make specific changes), deployment, every infrastructure piece and service.
  **The hard requirement sits at the boundaries: any commit merging to
  main or deploying to prod carries an architecture document accurate as
  of that commit.** When document and reality disagree, reality is right
  and the document has a bug.
- `docs/evals/` — dated behavioral test plans and results (§5).
- `docs/notes/` — personal brainstorming/learning notes. Background
  reading, never specs, excluded from review scope.

**Branching and gates:**
- `main` is the default branch and integration workbench. Feature branches
  PR into it behind a gate: CI (lint + tests) plus an automated adversarial
  AI review on every PR push.
- `prod` (or equivalent) is a pointer, not a workbench: it only ever moves
  to a verified-deployed commit — the deploy workflow moves it after a
  health check passes, and rollback moves it backward. Promotion is always
  the owner's explicit call.
- Deploys are one button (workflow_dispatch): validate the commit is on
  main, deploy, health-check the live system, only then move the pointer
  and mint a dated `deploy-` tag. A failed run leaves prod untouched.

**The automated reviewer** (a CI workflow invoking a strong model in
review mode on every PR):
- **Read `references/reviewer-setup.md` when setting this up** — it holds
  the full recipe: the Claude Code GitHub App vs. the Action's OAuth-token
  auth (`claude setup-token` + `gh secret set CLAUDE_CODE_OAUTH_TOKEN`),
  the complete workflow YAML with the battle-tested review prompt
  (ADAPT-marked sections for the new project's context and hard
  constraints), the rationale behind each prompt rule, and the observed
  operational gotchas.
- The essentials if not reading the reference: give it the product
  context, the hard constraints, and ordered review priorities; have it
  end with an explicit "Blocking issues: N" line — that line is what
  merge decisions key on.
- Known gotcha, record it in the project: the review action refuses to run
  on a PR that modifies the workflow file invoking it (tamper guard), and
  the skip looks like a 10-second "pass". Changes to that file land on the
  default branch directly, after merge, by owner approval.

**Environment**: everything runs in containers or an equivalently
reproducible environment — no host installs. Tests, lint, and one-off
scripts run inside it.

---

## 2. Owner-approval economics (OPERATE)

The division of authority that makes autonomous work safe:

- **Commits are autonomous** — make them at reasonable checkpoints on your
  own judgment, one concern per commit, with messages that record findings,
  rationale, and review pushbacks (commit messages are part of the
  project's memory).
- **Pushes, PRs, merges, and deploys are the owner's, individually** — a
  batch of pending commits gets one suggestion; each subsequent push needs
  its own approval. Broad instructions ("be agentic", "work end-to-end")
  never override this; only an explicit "push" / "open the PR" does. The
  reason is cost and control: every push to an open PR re-runs the paid
  review, and the unpushed commit is the owner's veto window.
- **Standing conditionals are honored as given**: "merge if no blockers"
  means exactly that — one blocker found means no merge and the fix cycle
  instead, without re-asking.

## 3. The feature lifecycle (OPERATE)

Spec first, always:

1. **Roadmap entry** — the feature exists on the roadmap with its
   rationale and ordering before anything else.
2. **Spec PR** — a new § section in REQUIREMENTS.md with numbered FRs.
   Good FRs from this process: name the normative behavior, mandate the
   tests that enforce it (the spec's test list is the real enforcement —
   prose has unbounded surface), mandate spikes for load-bearing
   assumptions ("implementation MUST begin with a spike proving X"), and
   state accepted tradeoffs explicitly rather than leaving them implied.
3. **Review rounds to zero blockers** — the spec PR goes through the same
   adversarial review cycle as code (§4). Specs routinely take 5–7 rounds;
   each round's findings are real work, not ceremony.
4. **Owner merges the spec**, then implementation starts on a fresh
   branch — opening with any leftover non-blocking findings from the
   previous PR's final round as clean, separately identifiable commits.
5. **Implementation order**: spikes first (they de-risk and often decide
   designs), then build in dependency order, committing in clean slices.
   Run the full gate (lint + full test suite, including any
   environment-specific lanes) before every push.

## 4. The review ritual (OPERATE)

When an automated review lands on a PR:

1. **Findings summary first, as its own complete message** — before any
   edit. For each finding: is it real or churn; will it be fixed or pushed
   back on, with the reason. The summary must stand alone on screen — the
   owner reads it while the fixes happen.
2. **Then fix autonomously** (resume via a short background timer if the
   environment needs one): blockers always; non-blockers on judgment,
   each taken or pushed back explicitly. Pushbacks go in the commit
   message with their reasoning — they are part of the record, and
   reviewers in later rounds respect a well-argued pushback.
3. **Add a regression test for every fix** — review findings are bugs the
   suite missed; the fix without the test is half the work.
4. **Verify empirically before accepting a reviewer's causal claim** —
   reviewers reason from code; when you hold observed evidence (a log
   line, a trace) that contradicts the reasoning, the evidence wins and
   the pushback says so.
5. The unpushed fix commit is the veto window. Suggest the push; the owner
   approves it; the cycle repeats until "Blocking issues: 0".
6. Convergence is normal: expect blocking counts like 3 → 3 → 2 → 1 → 0
   across rounds, with findings getting narrower each time. That's the
   process working, not failing.

## 5. Behavioral evals (OPERATE)

For changes judged by behavior rather than assertions (prompts, agent
character, UX):

- One dated file per evaluation in `docs/evals/` — **the plan is written
  before the run** (a plan written after seeing outcomes is a story, not
  evidence): what changed (verbatim where feasible), the normative rules
  under test mapped to the observed failures they fix, the protocol (same
  scripted scenarios on every arm, multiple runs per scenario because
  single samples mislead), scenario scripts readable during testing, and
  a blank judging sheet.
- Include an **overcorrection check**: for every rule "do less X", a
  scenario proving X still happens when explicitly requested.
- **Results are appended, never rewritten**, iteration by iteration.
  Audit instrumentation (traces, transcripts, logs) alongside human
  judgment — in practice the audit regularly overturns the human read
  (e.g., "the model ignored my instruction" turning out to be an upstream
  step never delivering the instruction to it).
- Pausing or waiving an eval is legitimate **as a recorded owner decision,
  kept distinct from "the bar was met"** — note it in the eval doc and
  wherever the acceptance bar is referenced, with the resume point.

## 6. Everything else that made this work (OPERATE)

- **Instrumentation is part of the feature, not an afterthought**: new
  mechanisms record their own structured logs/traces as they're built —
  "built through its trace, not traced after the fact." The trace store
  is also the eval-audit substrate.
- **Live-testing findings become roadmap entries** the same day — even as
  placeholders — rather than staying in chat history.
- **Docs-vs-reality conflicts found by review get fixed toward reality**,
  and contracts state the *shipped* shape, including deliberate narrowness
  ("X is deliberately in-memory only; persisting it would be a one-line
  move into Y if ever wanted").
- **Scope discipline**: identity/copy changes, ops tweaks, and feature
  work travel in separate commits (or separate PRs when they'd gate each
  other); one-line ops changes that must live on the default branch to
  take effect (workflow files) go there directly, with owner approval,
  stated plainly.
- Worktrees for parallel tracks: spec-writing for feature N+1 proceeds in
  a worktree while feature N is implemented in the main checkout; the
  worktree branch is rebased onto main before its PR merges.

