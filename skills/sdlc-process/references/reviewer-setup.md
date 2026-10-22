# The automated PR reviewer — full setup recipe

How the adversarial review gate is built, with the workflow and prompt
template refined on a real project. Read this during SETUP mode
when creating the reviewer workflow for a new project.

## What it is

The reviewer is [`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action)
— the official GitHub Action that runs Claude Code headlessly inside a
GitHub Actions job. Triggered on every PR push, it reads the PR, the
repo's governing docs, and the diff, then posts inline comments plus one
sticky summary comment ending in a machine-keyable verdict line
("Blocking issues: N"). Merge decisions key on that line; the owner's
standing instruction "merge if no blockers" means exactly N = 0.

Two related pieces, don't confuse them:
- **The Claude Code GitHub App** (installed on the repo/account) powers
  interactive `@claude` mentions and gives the action its GitHub-side
  identity for posting comments.
- **The Action's Anthropic-side auth** is a separate credential: a Claude
  Code OAuth token stored as a repo secret (below). The App being
  installed does not by itself authenticate the model calls.

## Setup steps for a new repo

1. Install the Claude Code GitHub App on the repository (github.com/apps
   — "Claude", or via `claude /install-github-app` from the repo).
2. Mint the OAuth token locally and store it as a repo secret:
   ```bash
   claude setup-token          # prints a Claude Code OAuth token
   gh secret set CLAUDE_CODE_OAUTH_TOKEN   # paste the token
   ```
3. Add the workflow below as `.github/workflows/claude-code-review.yml`,
   replacing the product-context paragraph and the hard-constraints list
   with the new project's (sourced from its CLAUDE.md).
4. Open a trivial PR to verify a real review posts.

## The workflow template

Adapt the `prompt:` sections marked `ADAPT`; keep the mechanics. The
`<placeholders>` show the shape of what goes there.

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  claude-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
      actions: read # lets the reviewer inspect CI results on the PR

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 1

      - name: Run Claude Code Review
        id: claude-review
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          use_sticky_comment: true # re-reviews update one comment instead of stacking
          claude_args: |
            --allowedTools "mcp__github_inline_comment__create_inline_comment,Bash(gh pr view:*),Bash(gh pr diff:*),Bash(gh run view:*),Bash(gh pr comment:*)"
            --max-turns 150
            --model claude-opus-5
            --effort xhigh
          prompt: |
            Review PR #${{ github.event.pull_request.number }} in ${{ github.repository }}.
            Use `gh pr view` / `gh pr diff` to read the PR description and diff.

            # ADAPT: one paragraph of product context for the new project
            You are reviewing a PR for <product name>, <one-sentence
            description of what it does> (<core architecture in one line,
            e.g. the main pipeline or request path>).

            ## Before reviewing
            1. Read CLAUDE.md (hard constraints), ROADMAP.md (feature order and
               vision), and the FR numbers referenced in the PR description in
               REQUIREMENTS.md. If the PR description references a design doc,
               read the relevant sections.
            2. If the PR description states no intent or references no FRs, say
               so — that alone is a finding.

            ## Review priorities, in order
            These priorities guide emphasis and ordering — they are NOT an
            exhaustive checklist. Report any genuine issue you find, of any
            severity, in any category, whether or not it is mentioned anywhere
            in this prompt.

            1. Intent — Does the diff implement the referenced FRs, all of
               them? Unrelated miscellaneous edits riding along are expected
               and fine — the author routinely batches small fixes with main
               changes. Do NOT flag scope creep or drive-by changes; comment
               on an unrelated change only if it is itself defective,
               dangerous, or violates a hard constraint.
            2. Correctness — Construct concrete failing scenarios: edge cases
               (empty/duplicate/concurrent), error paths (an external call
               fails midway, a write fails after a partial state change),
               async mistakes (blocking calls in async code, un-awaited
               coroutines), resource and session lifecycle (reconnects,
               abandoned sessions, cleanup).
            3. Security & user isolation — the highest-stakes area:
               # ADAPT: restate the new project's auth seam and isolation
               # invariants; these bullets are the common shape
               - Every query touching user-owned data must be scoped to the
                 authenticated user (or tenant).
               - The user's identity must ALWAYS come from the server-side
                 auth layer (<auth dependency name>), never from a request
                 body, query param, or client-supplied header. Flag any data
                 access function that gives the user id a default value.
               - Every new route or endpoint must declare authentication.
                 List any that don't.
               - No secrets, tokens, or .env contents in the diff.
            4. Hard constraints (violations are always blocking):
               # ADAPT: copy the new project's CLAUDE.md hard-constraints list
               - <constraint, e.g. third-party SDKs are constructed only in
                 <module>; flag direct usage elsewhere>
               - <constraint, e.g. sensitive user content is stored apart
                 from metadata>
               - <constraint, e.g. new components emit structured logs with
                 <correlation ids>>
            5. Performance (<critical path>) — advisory, NOT strict:
               # ADAPT: name the latency- or cost-sensitive path and its budget
               flag any addition of synchronous work (network, DB, heavy
               compute) to <critical path>, with a rough cost estimate, so
               the author can decide. Additions are not blocking by default;
               treat one as blocking only if it plainly endangers
               <budget, e.g. the NFR it maps to>.
            6. Design — Does the change respect existing seams (# ADAPT:
               <the project's main seams>)? Judge by "how expensive
               is the next change." Flag BOTH under-design (logic smeared
               across layers, duplication of an existing helper) AND
               over-design (abstractions with one caller, speculative
               config/flexibility, error handling for impossible cases). This
               project prefers the simplest code that solves the stated
               problem.
            7. Tests — New behavior needs tests in this PR, mapped to the FR
               acceptance criteria. Tests must assert behavior (they'd fail if
               the feature broke), including at least the primary error path.
               For auth and user-scoping changes, require a negative test
               (user A cannot access user B's data). A diff that weakens or
               deletes an existing test to get CI green is always a blocking
               finding.
            8. Readability & craft — Names, function size, and comments that
               explain why. Flag nits and best-practice preferences too,
               prefixed "nit:" — the author wants to see and learn
               senior-level coding practices, not only blocking problems.

            ## Output rules
            - Post line-specific findings as inline comments using
              mcp__github_inline_comment__create_inline_comment with
              confirmed: true — comments left unconfirmed are silently
              discarded when the job ends, never posted.
            - Post the overall summary and the final verdict as ONE PR
              comment, updating your previous summary on re-reviews instead
              of stacking new ones:
              `gh pr comment <pr> --edit-last --create-if-none --body ...`.
              Text that only reaches the workflow log is invisible to the
              author.
            - Order findings by severity: blocking issues first, then
              "nit:"-prefixed suggestions. Aim for roughly 10 findings,
              leading with what matters most — a soft cap: exceed it freely
              when more findings are pressing or high-priority.
            - For every finding: file:line, what's wrong, a concrete failing
              scenario or consequence, and a suggested fix.
            - Do not comment on formatting/style a linter would catch
              (<linter> runs in CI).
            - Do not pad with praise or restate what the diff does. If the PR
              is clean, say so in two sentences and stop.
            - End with a one-line verdict: "Blocking issues: N" or "No
              blocking issues."
```

## Why the prompt is shaped this way (learned, not guessed)

- **"Read the governing docs first"** makes the reviewer judge intent
  against the spec rather than vibes — in practice it verifies FR claims
  line by line and catches doc-vs-code contradictions.
- **"Don't flag scope creep"** exists because the owner deliberately
  batches small fixes into feature PRs; without it, reviews drown in
  scope noise.
- **Ordered priorities with "not an exhaustive checklist"** gets severity
  ordering without suppressing out-of-category findings.
- **"nit:" prefix + learning framing** produces the senior-practice
  suggestions the owner wants without inflating blocking counts.
- **The single verdict line** makes "merge if no blockers" mechanically
  checkable.
- **Inline comments need `confirmed: true`** or they are silently lost
  at job end — this was learned the annoying way.
- **Strong model + xhigh effort** is a deliberate cost choice: reviews
  take 10–15 minutes and real money, which is exactly why every push is
  owner-approved.

## Operational gotchas (all observed in production)

- **Tamper guard / silent skip**: the action refuses to run on a PR that
  modifies the workflow file that invokes it. The symptom is a ~10-second
  "pass" with the log line "Exiting due to workflow validation skip" and
  no review comment. Any edit to this workflow file must land on the
  default branch directly (owner-approved), never ride a PR.
- **Dead token signature**: an expired/revoked OAuth token fails the job
  in ~14 seconds with `is_error: true` (sometimes reported as subtype
  success). Fix: regenerate with `claude setup-token`, re-store with
  `gh secret set CLAUDE_CODE_OAUTH_TOKEN`.
- **A 10–15 minute run is normal**; anything completing in seconds did
  not actually review — check the log before trusting a fast green.
- **Pushing to a PR mid-review** starts a second run without cancelling
  the first; cancel the superseded run to avoid paying for both.
- **Wait for the run to fully complete** before reading the verdict —
  the sticky comment updates late in the job.
