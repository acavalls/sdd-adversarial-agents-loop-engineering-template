# Adopting the methodology in a new project

This repository ships the methodology only. A project adopts it by copying
the shared artifacts and customizing four injection points. Estimated
effort: under an hour, most of it writing `config.yaml`.

## 1. Prerequisites

- Install [opencode](https://opencode.ai) and the
  [openspec CLI](https://github.com/Fission-AI/OpenSpec).
- In the new project repo, run `openspec init` if you want the CLI to
  regenerate the vendored skills instead of copying them (the vendored
  copies in `.opencode/skills/openspec-*` are standard CLI skills and can
  be refreshed that way).

## 2. Copy the shared artifacts (as-is)

```
.opencode/agents/good.md
.opencode/agents/ugly.md
.opencode/agents/bad.md
.opencode/context/workflow.md
opencode.json
```

The three agents and the workflow are deliberately generic: they make no
language or framework assumptions. Copy them without edits.

## 3. Customize the four injection points

### a) `openspec/config.yaml` — the project contract

Use the template in this repo. Fill in:

- **`context`**: project description, the fixed stack (frameworks, storage,
  auth, frontend approach, architecture layers, media handling, test
  runner, coverage threshold, quality gates, dev-data policy). Anything you
  declare here is *not renegotiated in proposals or designs* — write the
  decisions you do not want agents re-debating.
- **`rules`**: keep the methodology defaults (spec terminology, decision
  recording, validation boundaries, validation-closed groups, minimal
  wiring) and add project-specific rules (e.g. stack-idiomatic guidance for
  `design`, per-artifact conventions).
- **`operations`**: adjust the `apply`/`archive` guidance to your actual
  commands (e.g. "run pytest and ruff after each task").

### b) `AGENTS.md` — conduct rules

Copy from this repo. Edit only rule 4: it declares where requirements come
from — by default, the task definition included in the user's initial
prompt is the information from which the specification is created (plus
anything already living in `openspec/`, e.g. living specs to reach parity
with). This is the tie-breaker for agent-decided ambiguities.

### c) `opencode.json` — agent permissions

The `ugly`/`bad` agents run with the project's default permissions
(`edit: allow`, `bash: allow`) declared in `opencode.json`. There are no
per-agent permission blocks by default; if you need to restrict an agent,
declare a `permission` block under `agent.<name>` in `opencode.json`. The
agent files (`.opencode/agents/ugly.md`, `bad.md`) carry a commented
example of such a block. Note that `"*": "ask"` as a bash fallback makes
opencode prompt the user for every command not matching an allow rule.

Suggested shape when you do restrict:

- `ugly`: `edit: deny` + read-only tooling allowlist. It audits; it never
  writes.
- `bad`: may write test files, but every destructive and git-writing
  command stays denied. It attacks; it never repairs or commits.
- `good` (and the main agent): full write access.

### d) Project skills (optional but recommended)

Domain specialization lives in `.opencode/skills/` as project skills
(e.g. framework conventions, API patterns). The GBU agents are instructed
to load the relevant project skills; this is how you specialize them
without editing the agent files. Do **not** add skills that contradict
`config.yaml`.

## 4. First cycle checklist

- [ ] `openspec/` exists with `config.yaml` filled in (empty `specs/` and
      `changes/` are fine).
- [ ] `.opencode/context/workflow.md` present — the main agent reads it at
      the start of every request.
- [ ] `AGENTS.md` rule 4 declares your requirement inputs.
- [ ] `opencode.json` declares the project's default permissions; if you
  restrict `ugly`/`bad`, the per-agent blocks name your real commands.
- [ ] `openspec validate` passes on your first proposed change.
- [ ] After the first PAUSE-1, calibrate the GBU budgets in
      `gbu-state.json` if needed (defaults: 3 Good↔Ugly iterations per
      group, 2 Bad attacks, 6 minutes and 6 test-runner runs per Bad
      cycle).

## 5. Runtime behavior you should expect

- Commits happen automatically only at the agreed gates (planning,
  implementation, archive, regularization), grouped semantically. Anything
  else requires your explicit request.
- Ambiguities are resolved by the agent and flagged, never turned into
  questions mid-flow.
- The loop stops for you only at PAUSE-1, PAUSE-2, ESCALATION or
  BUDGET_EXHAUSTED.
- Every cycle ends with a spec audit; drift is either regularized through
  the same pipeline or consciously closed with findings documented.
