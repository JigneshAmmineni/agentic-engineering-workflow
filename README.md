# Agentic Engineering Workflow

This is my personal suite of Claude Code tools, packaged as one plugin so I can install my whole workflow on any machine. It covers the loop I follow on my own projects, from writing a spec and stress-testing the design to running parallel work in git worktrees, getting every PR reviewed by an automated reviewer, and checking security before launch. It's public in case it's useful to anyone else.

Some of the skills are mine and some are copied from other authors under their licenses. The table below says which is which.

## Install

In Claude Code, add this repo as a marketplace and install the plugin:

```
/plugin marketplace add JigneshAmmineni/agentic-engineering-workflow
/plugin install agentic-engineering-workflow@agentic-engineering-workflow
```

Skills are namespaced by the plugin, so they show up as `agentic-engineering-workflow:<skill>`. To get updates later, run `/plugin marketplace update agentic-engineering-workflow`.

## My skills

| Skill | What it does |
|---|---|
| `sdlc-process` | A spec-first development process for solo projects with an AI pair. Sets up numbered requirements, a roadmap, an architecture doc, a two-branch deploy model, and a GitHub Actions reviewer that reviews every PR until there are zero blocking issues. The reviewer setup, including the workflow file and review prompt, is in `references/reviewer-setup.md`. It never runs on its own. Run `/sdlc-process` to set it up in a project, or at the start of a session you want to follow it. |
| `critical-review` | Reviews an idea, plan or architecture from three angles at once (a critic, a researcher and an advocate) using parallel subagents, then combines them into one set of recommendations. |
| `security-controls-review` | Threat-models a whole app or a single feature and proposes security controls, with every threat tied to real code or design and every claim cited to a primary source. |
| `teach-me` | Turns a messy list of terms and concepts into one structured lesson built up from first principles. |
| `multiple-worktrees` | Creates one git worktree per task under `.trees/` so several agents can work in parallel. Run it as `/multiple-worktrees write tests, add feature`. |
| `merge-worktrees` | Merges the named worktrees from `.trees/` back into the current branch. |

## Skills by other authors

These are included unchanged so the whole setup installs as one unit. Each folder keeps its original license file.

| Skill | Author | License |
|---|---|---|
| `grilling`, `grill-me`, `grill-with-docs`, `domain-modeling`, `codebase-design`, `improve-codebase-architecture`, `handoff`, `teach` | [Matt Pocock](https://github.com/mattpocock/skills) | MIT |
| `skill-creator`, `frontend-design`, `web-artifacts-builder` | [Anthropic](https://github.com/anthropics/skills) | Apache 2.0 |

## How I use it

I start a new project with `sdlc-process`, which writes the project's CLAUDE.md and governing docs and sets up the PR reviewer. Before building a feature I use `grill-with-docs` or `critical-review` to find the holes in the plan, and `domain-modeling` keeps the project's vocabulary consistent. When a feature splits into independent pieces, `multiple-worktrees` gives each agent its own branch, and `merge-worktrees` brings them back together. Before a launch I run `security-controls-review`.

## License

My skills and the rest of this repo are under the [MIT License](LICENSE). Skills by other authors keep their own licenses, found in each skill's folder.
