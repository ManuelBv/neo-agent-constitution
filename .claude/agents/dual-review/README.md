# Dual-Review Pattern — How It Works

Two independent reviewer agents, on different models, review the same narrow claims in parallel and report back. No debate, no third tie-breaker — agreement stands, disagreement escalates.

## Pipeline

```
Reviewer A (opus) ─┐
                    ├─→ Arbiter reconciles → agreement: verdict stands
Reviewer B (sonnet)─┘                      → disagreement: tool-grounded check, or escalate to human
```

- **Reviewer A and Reviewer B run in parallel, never seeing each other's output.** Neither reads the other's verdict file before writing its own, even on a re-run.
- **One round only.** No back-and-forth, no "here's what the other reviewer said, do you agree?" Multi-round debate between LLM agents is a documented failure mode — it drifts from the original question ("problem drift") and can converge both agents onto the same confidently-wrong answer ("echo chamber" convergence). Capping it at one independent pass each avoids both.
- **Different models on purpose.** Reviewer A runs opus-tier, Reviewer B runs sonnet-tier. Two calls to the identical model/prompt tend to reproduce the same blind spot rather than adding real independent perspective.

## Why agreement isn't automatically proof

Research on LLM judge panels found that even panels of *different* model families have heavily correlated errors — models tend to fail on the same items for the same reasons, not randomly. One study found a 9-judge panel across 7 model families carried only about 2 independent votes' worth of real information, and that two judges can agree on the *wrong* answer a majority of the time on some benchmarks.

This means: **two reviewers agreeing is meaningful evidence, but not proof.** The Arbiter's reconciliation rule treats agreement as sufficient to let a verdict stand, but for high-stakes rows where a deterministic check is available, running that check even on an agreement row is worth the small extra cost — see `arbiter.md`'s reconciliation rule, item 1.

This is also why the pattern insists on **narrow, specific, mechanically-checkable-when-possible claims** rather than broad "is this good" judgments — correlated-error risk is highest on genuinely ambiguous, subjective calls and lowest on narrow factual ones.

## Disagreement handling — the load-bearing design decision

1. **Deterministic check exists** (a test, a linter, an accessibility scan, a type-checker) → run it, let its output decide. Always preferred over more LLM opinions.
2. **No deterministic check exists** (genuinely subjective — e.g. spec-fidelity nuance) → escalate to the human with both reviewers' full reasoning shown side by side. Do not spawn a third LLM to "break the tie" — a third opinion doesn't resolve genuine ambiguity, it just manufactures false consensus that can be mistaken for resolution.

## When to use this vs. a single reviewer

Reserve this for the highest-stakes, most ambiguity-prone individual verdicts — a single reviewer's mistake here is expensive and the judgment call is genuinely non-mechanical (e.g. "does this meet WCAG criterion X," "does this scenario's acceptance criteria actually hold"). It costs roughly 2x a single-reviewer pass on the rows it's applied to, so scope it narrowly — a handful of hard rows, not a whole review. If a deterministic check already exists for the question, skip voting entirely and just run the check.

## Usage

1. Write a `vote-rubric.md` for the task — narrow, specific, checkable claims, one per row. Keep it small on purpose.
2. Dispatch `reviewer-a` and `reviewer-b` in parallel (two Task calls in one message).
3. Dispatch `arbiter` once both verdicts exist — it reconciles mechanically per the rule above and writes `vote-result.md`.
4. If any rows are escalated, stop and present them to the user — don't resolve them yourself.

## Files

```
.claude/agents/dual-review/
├── reviewer-a.md   (opus-tier)
├── reviewer-b.md   (sonnet-tier)
└── arbiter.md      (sonnet-tier, mechanical reconciliation only)
```

Per-task working files (`vote-rubric.md`, `reviewer-a-verdict.md`, `reviewer-b-verdict.md`, `vote-result.md`) live wherever the task's docs live — see the `tiered` pattern's `docs/features/[name]/` convention for an example of where to put them in a feature-pipeline context.
