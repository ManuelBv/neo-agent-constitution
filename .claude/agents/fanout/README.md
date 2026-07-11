# Fanout Pattern — How It Works

**Fixed, named pairs of specialist agents, dispatched together in parallel, at most 2 at a time.** Not a dynamic N-worker decomposition — each pair has a stable identity and a fixed place in the pipeline. This is the pattern actually used in `purrfect-blocks`: research fans out into 2 specialists before implementation, verification fans out into 2 specialists after implementation.

## Usage

Invoke a pair by name: `fanout legal/technical` or `fanout coder/unittester`. Only 2 agents run per fanout call — if a task needs more than one pair's worth of parallel checking, run the pairs sequentially (e.g. `coder/unittester` first, then `correctness/security` on the same diff), not all at once.

## The pairs

### `legal/technical` — before implementation

**Agents**: `legal-researcher.md` + `technical-researcher.md`
**When**: during research, before any planning or implementation starts.
**Conditional**: `technical-researcher` always runs. `legal-researcher` only runs if the feature actually touches user data, third-party assets, monetization, or distribution — don't spawn it reflexively just because `technical-researcher` is running.
**Why paired**: both inform the same upcoming decision (what to build and how), but are orthogonal domains — one has no bearing on the other's findings, so there's nothing to synthesize; the next stage (planner or you) just reads both outputs directly.

### `coder/unittester` — after implementation

**Agents**: `code-reviewer.md` + `unit-tester.md`
**When**: right after implementation, before considering a feature done.
**Why paired**: verifies two different things about the same change — is the code good (review) and is it actually tested against the spec, including BDD scenario coverage (unit-tester). Independent concerns, same input (the diff), no synthesis needed.

### `correctness/security` — after implementation, for risk-sensitive changes

**Agents**: `correctness-checker.md` + `security-checker.md`
**When**: after implementation, in addition to (not instead of) `coder/unittester`, specifically when the change touches auth, input handling, data storage, or external I/O.
**Why paired**: a general code-reviewer skims both; when the stakes are high enough to want dedicated attention, splitting logic-correctness from security-vulnerability-hunting into two focused passes catches more than one generalist pass.

### `performance/a11y` — after implementation, for UI-heavy changes

**Agents**: `performance-checker.md` + `a11y-checker.md`
**When**: after implementation, for frontend-heavy features where both performance and accessibility carry real risk.
**Why paired**: same logic as `correctness/security` — two dimensions a generalist reviewer might skim, each worth a dedicated pass when the feature is UI-heavy enough to matter.

### `writer/critic` — sequential, not parallel

**Agents**: `writer.md` + `critic.md`
**When**: any time a single artifact (a doc, a spec, a piece of content — not code) needs an independent check before being finalized.
**Why it's here despite not running in parallel**: it's still a fixed, named 2-agent pair dispatched as a unit, same invocation shape as the others (`fanout writer/critic`) — just sequential internally because the critic needs the writer's finished output to review. Ideally on different models/tiers (writer: sonnet, critic: opus) to avoid same-model blind spots — same rationale as the `dual-review` pattern's reviewer-diversity requirement, but here it's one critic checking one artifact, not two reviewers voting.

## Why fixed pairs, not dynamic N-worker fanout

This is deliberately narrower than a generic "decompose into N independent angles" fanout. Every pair here has a **stable identity and a fixed place in the pipeline** — there's no decomposition step because the specialization is known in advance, and no synthesis step because each pair's two outputs are either orthogonal (nothing to reconcile) or sequential (critic already has final say). If you need genuine dynamic decomposition — an open-ended research question where you don't know the angles upfront — that's a different pattern, not this one.

## Adding a new pair

Only add a pair when there's a real recurring need for two specific, named specialists to run together — not preemptively. Each new pair needs: two agent files (or reuse of existing ones), a stated "when in the pipeline" placement, and a rationale for why splitting into two focused passes beats one generalist pass.
