# sdd-adversarial-agents-loop-engineering

**Spec-driven development with adversarial agents, built on loop
engineering principles.**

This repository contains a reusable methodology: a
spec-driven development (SDD) lifecycle in which an AI agent implements
software exclusively from OpenSpec specifications, hardened by an
adversarial loop of fresh-context subagents, and audited after every cycle
so the specs and the code never silently diverge.

The human decision points are deliberate: validate the implementation, and
decide what to do with post-audit drift. Everything else — proposing,
implementing, reviewing, attacking, verifying, archiving, committing — is
an automatic pipeline stage.

## Two loops

The methodology runs on two nested loops:

1. **The general loop** — the SDD lifecycle itself: propose → implement →
   verify → archive → audit, with routing driven by persisted state
   (`tasks.md` checkboxes, `openspec status`, living specs, archived
   changes). One user request is one iteration of this loop.
2. **The GBU adversarial loop** — the loop inside implementation: per task
   group, Good implements and Ugly audits until approved (≤ 3 iterations);
   after the last group, Bad attacks the whole change surface. Fresh-context
   subagents, bounded budget.

```
                 ┌──────────────────────────────────────────────┐
                 │                    Cycle                     │
                 ▼                                              │
  Phase A: plan & implement (automatic)                         │
    propose → auto-resolve ambiguities → validate               │
      → commit(planning) → GBU apply loop                       │
      → quality gates → commit(implementation)                  │
    ── PAUSE-1: "Does the implementation look correct? YES/NO" ─┤
                                                                │
  Phase B: verify & archive (automatic, after SI)               │
    verify → archive → commit(archive)                          │
                                                                │
  Phase C: audit & close (automatic)                            │
    spec-audit ── IN SYNC ──► close cycle                       │
         │                                                      │
         └─ drift ── PAUSE-2: close or regularize ───────────────┘
```

Every phase, the routing table for user intents, and the exact artifacts
are declared in [`.opencode/context/workflow.md`](.opencode/context/workflow.md).
The project-wide conduct rules live in [`AGENTS.md`](AGENTS.md).

## The GBU adversarial loop

Implementation is not a single agent writing code: it is a loop.

| Agent | Role | Writes | Verdicts |
|---|---|---|---|
| 🤠 **The Good** (`good.md`) | Generator: implements one task group against the delta specs | workspace code + tests + task checkboxes | JSON summary with assumptions and validation |
| 👺 **The Ugly** (`ugly.md`) | Auditor (read-only, fresh context): audits the group against its delta-spec scenarios and its exercisability | nothing | `APPROVED / REJECTED / ESCALATION` with per-scenario evidence |
| 🌵 **The Bad** (`bad.md`) | Adversarial validator: attacks the whole change surface once with a bounded budget | only NEW test files | `SURVIVES / BROKEN / BUDGET_EXHAUSTED / ESCALATION` with structured bugs |

The orchestrator (the main agent) drives the loop: per task group,
Good implements and Ugly audits until approved (≤ 3 iterations); after the
last group, Bad attacks the whole change surface; blocking bugs loop back
to Good. ESCALATION and budget exhaustion always stop and ask the user —
exhausted budget is never treated as "the code is correct".

Loop state and evidence are first-class artifacts:

- `openspec/changes/<change>/gbu-state.json` — resume-capable loop state,
  updated after every step.
- `openspec/changes/<change>/gbu-report.md` — the Sheriff's decision
  artifact: requirement-by-requirement proof, quality-gate results,
  adversarial pass, deviations, and manual validation pointers. Append-only
  and archived with the change.

## Key principles

- **The spec is the source of truth.** Every observable behavior change
  lands as an OpenSpec change.
- **Exercisable surface.** Every change must leave each new capability
  exercisable end-to-end through the primary interface its delta spec
  declares.
- **Budgeted loops.** Adversarial pressure is bounded (iterations, wall
  clock, test-runner executions).
- **Gated commits.** Git writes happen only at the agreed gates, executed
  with the commit command (`.opencode/commands/commit.md`).

## Repository layout

```
AGENTS.md                        # conduct rules for the main agent
opencode.json                    # project permissions; per-agent restriction is optional
.opencode/
  agents/{good,ugly,bad}.md      # GBU subagent definitions (generic)
  commands/commit.md             # commit command (semantic grouping at the agreed gates)
  context/workflow.md            # the lifecycle pipeline (the core document)
  skills/openspec-*              # vendored openspec CLI skills
  skills/spec-audit/             # post-cycle spec-drift audit skill
openspec/
  config.yaml                    # template: project context + artifact rules
docs/adopting.md                 # how to adopt the methodology in a new project
```

## Requirements

- [opencode](https://opencode.ai) as the agent runtime.
- The [openspec CLI](https://github.com/Fission-AI/OpenSpec) to drive
  changes, artifacts, validation and archiving.
- A project that adopts this layout (see `docs/adopting.md`).
