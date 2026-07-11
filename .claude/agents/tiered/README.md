# Tiered Pattern — How It Works

Five agents, run in sequence, with a docs structure that tracks project state across sessions and per-feature history. This is the real pipeline from `ceiling-designer`, generalized for reuse in any project.

## The two routes

The Speccer decides which route a feature takes, based on how well-understood it is:

```
Full pipeline (new interaction pattern, architectural decision, unknown territory):
Speccer → Researcher → Planner [human approval] → Implementer → Reviewer
                                                                    ↕ (max 5 rounds)

Short pipeline (small, well-understood feature):
Speccer → Implementer → Reviewer
                            ↕ (max 5 rounds)
```

Don't pick the route yourself — the Speccer's interview surfaces enough context to decide, and states its rationale explicitly in its output ("Skipping researcher and planner — rationale: ..."). Use the same pattern for both small and large features; the routing handles the distinction.

## The docs structure — plan, progress, and per-feature briefs

This pattern is not just a list of agents — it's a doc-tracking discipline that gives every agent (and every future session) the context to pick up where the last one left off.

```
docs/
├── general/
│   ├── plan.md        ← architecture decisions, tech stack, step list
│   └── progress.md    ← live step tracker
└── features/
    └── YYYY-MM-DD-NNN-feature-name/
        ├── 1-speccer-output.md      ← spec: domain terms + BDD scenarios + pipeline routing decision
        ├── 2-researcher-output.md   ← options + comparison table (Full pipeline only)
        ├── 3-planner-output.md      ← chosen approach + implementation plan + Review Runs Log (Full pipeline only)
        ├── [2 or 4]-implementer-output.md  ← implementation notes, runs appended (never overwritten)
        └── [3 or 5]-reviewer-output.md      ← checklist results, PASS/FAIL per run
```

### `plan.md`

Written **collaboratively, once, in the first working session** on the project — architecture decisions, tech stack choices, and the overall step list for building the thing. This is the single source of truth for "why is it built this way," so every agent that needs to stay consistent with existing decisions (Researcher, Planner) reads it before doing anything else.

Not a living document that changes every feature — only revisit it when an architectural decision actually changes.

### `progress.md`

The **live step tracker**. Read at the start of every session, before anything else happens — this is how an agent (or a human returning after days away) knows what's done, what's in flight, and what's next without re-deriving it from git history or asking the user to re-explain. Updated once per feature, after the user has done a manual check and confirmed the Reviewer's PASS.

### Per-feature folders (`docs/features/YYYY-MM-DD-NNN-feature-name/`)

One folder per feature, numbered sequentially (`001`, `002`, ...) so ordering is unambiguous regardless of when features land relative to each other. Each numbered file is one agent's output:

- **`1-speccer-output.md`** — the spec: domain terms (DDD ubiquitous language), acceptance criteria (BDD Given/When/Then), constraints, out-of-scope, and the Full/Short routing decision with rationale.
- **`2-researcher-output.md`** *(Full pipeline only)* — options investigated, a comparison table, and a recommendation — but no final decision. That's the Planner's job.
- **`3-planner-output.md`** *(Full pipeline only)* — the chosen approach, rationale, trade-offs, a step-by-step implementation plan, a Definition of Done, and the **Review Runs Log** (appended to by the Reviewer on every failed round — never overwritten).
- **Implementer output** (`4-implementer-output.md` on Full, `2-implementer-output.md` on Short) — what was built, per run. Run 1 is the initial implementation; each subsequent run is a correction appended as a new section, never overwriting the previous run's notes. This preserves the full history of what changed and why across correction cycles.
- **Reviewer output** (`5-reviewer-output.md` on Full, `3-reviewer-output.md` on Short) — the full checklist result (BDD scenarios, DDD language, Farley's 8 quality gates, type-checking/linting, tests, accessibility, performance) with PASS/FAIL per item and specific, actionable failures if it fails.

**Files are updated in place — sections are appended per run, never overwritten.** This is deliberate: the Review Runs Log and the Implementer's run history are the audit trail for how a feature actually got built, including every failed attempt and correction. Losing that by overwriting would make it impossible to see whether the Implementer is making the same mistake twice.

## Why this matters beyond "5 agents talking to each other"

The plan/progress/brief structure is what makes this pattern usable across sessions and across a growing project, not just within one conversation:

- A new session starts by reading `progress.md` — no re-explaining where things stand.
- A stuck feature's full failure history is in its Review Runs Log — no re-deriving what's already been tried.
- Architectural consistency across dozens of features is enforced by every agent reading `plan.md` before acting, not by hoping each session remembers past decisions.

Skipping this structure and just running the 5 agents ad hoc loses all of that — it becomes 5 agents doing one task well, not a system that scales across a real project's lifetime.

## Model tiering and bounded loop

See the agent files themselves (`speccer.md`, `researcher.md`, `planner.md`, `implementer.md`, `reviewer.md`) for each role's protocol, and `apps/agent-smith/examples/01-tiered-pipeline-reviewer/README.md` for the rationale behind the model tier assigned to each agent and the 5-round correction cap.
