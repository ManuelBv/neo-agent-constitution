# Sentinel Pattern — How It Works

A dynamic, N-worker fan-out audit pipeline for auditing an unfamiliar codebase end to end —
security (including supply chain), performance, reliability, accessibility, product/UX, API
design, documentation, dev tooling/CI-CD, and stack-specific domains like RAG-for-TTS vs
RAG-for-LLM — then fixing prioritized issues with minimal, surgical, test-backed commits. Built
for the Speechify Platform "Refactoring LLM Assessment" format (90 minutes, TypeScript monorepo,
pnpm), but usable for any timed or untimed whole-repo audit.

## Entry point

Triggered via the `/sentinel <local-path>` skill (`.claude/skills/sentinel/`), not invoked as a
bare agent — the skill is the one thing you type to kick off an audit. `<local-path>` is a
directory you've already cloned yourself (the real assessment requires your own GitHub SSH setup
regardless, so the skill doesn't duplicate that).

## Pipeline

```
/sentinel <path>
      │
      ▼
1. Orchestrator (sonnet)
   - profiles the repo (stack, frameworks, folder layout, test setup)
   - proposes a dynamic auditor roster with a one-line rationale each
   - WAITS for your explicit go-ahead or edits — never launches unapproved
      │
      ▼ (parallel dispatch, one Task call per approved auditor)
2. Dynamic auditors (sonnet by default; orchestrator can downgrade specific
   ones to haiku only if you ask it to)
   - one per domain identified at runtime — not a fixed roster
   - each: reads code skeptically + runs deterministic tooling for its
     domain (eslint, tsc, react-doctor, etc. via Bash) + WebSearches
     current best practices/vuln classes to ground findings
   - writes one file: docs/sentinel/<domain>.md
      │
      ▼
3. Ranker (sonnet)
   - merges every docs/sentinel/<domain>.md
   - ranks findings against the static severity rubric (see ranker.md),
     refined/overridden per-run via WebSearch against current OWASP/WCAG
     guidance
   - writes docs/sentinel/main.md — the index + live status tracker
      │
      ▼ (one dispatch per approved issue, sequential — each commit must
      │  land before the next fix starts, so history stays atomic)
4. Implementer (haiku, fed a full near-final diff by the ranker — not
   asked to design the fix itself)
   - applies the diff, writes/updates the test in the same pass, runs the
     full test suite, commits atomically, updates docs/sentinel/main.md
     status to "resolved in commit <hash>"

Live steering: since entry is a skill you type directly, you can interject at any point —
"go check X, I saw this behavior" — and the orchestrator spins up a scoped one-off auditor
immediately without pausing the others, folding its output into the same docs/sentinel/ structure.
```

## Known cost factor outside this pattern's control

As of this writing, the Claude Code Agent SDK has prompt caching disabled by default for
subagent requests (tracked upstream: anthropics/claude-code#29966) — each auditor's tool
definitions and system prompt get billed as fresh uncached input on every one of its own tool
calls, not just once per agent. With 7 auditors making 20-50+ calls each, this compounds. This is
an SDK-level gap, not something fixable from these agent definition files — don't attempt a
workaround here (e.g. hand-rolling a cache key) until the upstream issue is resolved; re-check
its status before assuming it's still open.

## Why sonnet auditors, haiku implementers

- **Auditors default to sonnet.** Diagnostic depth — root cause, not symptom — is the top-scored
  criterion in the assessment rubric this pattern was built for, and Sonnet's reasoning is worth
  the small extra cost (~$2–15 total for 30 parallel auditors at current API pricing; on a Pro
  plan the real constraint is the 5-hour rolling usage window, not dollars). The orchestrator may
  switch specific auditors to haiku, but only when explicitly asked to — never as a default
  cost-saving move.
- **Implementers default to haiku, but only because the ranker hands them a near-final diff.**
  Haiku is not asked to design the fix — it applies a fully-specified patch, runs tests, and
  writes the commit. This is safe specifically because the risky reasoning (what to change and
  why) already happened at sonnet tier in step 3. If a future issue needs the implementer to make
  a judgment call the diff didn't cover, escalate that one issue to sonnet rather than loosening
  the spec haiku receives.

## Priority rubric — why static-with-web-refinement, not pure LLM judgment

LLMs don't have reliable causal/severity judgment across unrelated domains (a security hole vs an
accessibility gap vs a performance regression aren't commensurable without an external anchor).
`ranker.md` carries a fixed baseline order (security/PII leakage > supply chain > reliability/
data-loss > exceptional-condition mishandling > accessibility > performance > API contract >
style/maintainability) sourced from OWASP Risk Rating (2025 Top 10) and WCAG conformance-level
conventions, so ranking is reproducible even if a WebSearch mid-run fails or returns thin results.
The per-run WebSearch step refines or overrides specific rows using current guidance — it does not
replace the backbone.

## Docs structure

```
docs/sentinel/
├── main.md              ← index: every issue, severity, status (open | fixed), commit hash
├── <domain>.md          ← one auditor's findings, domain-level (not one file per issue)
├── <domain>.md          ← one file per domain the orchestrator identified for this repo —
│                            the actual set varies per repo, decided at runtime, not fixed
└── <ad-hoc-domain>.md   ← anything the orchestrator adds dynamically, or you request live
```

Each domain file is one auditor's complete output — findings for that domain only, in the fixed
format (what / severity / steps to replicate / why the fix is right, short sentences, no lengthy
prose). `main.md` never duplicates finding detail — it links out and tracks status only.

## Finding format (every auditor, every domain file)

```markdown
### [SEV] Short title
**What:** one or two plain-language sentences — no jargon a generalist engineer wouldn't know.
**Severity:** critical | high | medium | low — per ranker.md's rubric, not the auditor's own call.
**Steps to replicate:** numbered, concrete, plain language — no file/line references here.
**Where:** file paths, line numbers, symbol names — implementation pointers, kept separate from
the narrative above so it stays skimmable.
**Why this fix is right:** one or two sentences, no lengthy rationale.
```

Auditors propose a severity as a starting signal, but the ranker has final say — this keeps
individual auditors from grading their own domain as more urgent than it is.

## Status markers — how Sentinel communicates back

The full ASCII banner (`.claude/skills/sentinel/SKILL.md`) prints once, on trigger. For everything
after that — phase transitions, findings, fixes — every Sentinel agent (orchestrator, auditors,
ranker, implementer) prefixes its status lines with a short bracketed marker instead of plain
prose, so Sentinel's voice stays visually distinct from ordinary Claude output without reprinting
full art every time:

| Marker | Meaning | Used by |
|--------|---------|---------|
| `[SCAN]` | profiling / reading code, nothing concluded yet | orchestrator, auditors |
| `[FOUND]` | a finding has been written to a domain file | auditors |
| `[RANK]` | ranking/prioritization in progress or complete | ranker |
| `[FIX]` | a diff is being applied | implementer |
| `[TEST]` | test suite running | implementer |
| `[COMMIT]` | a commit has landed | implementer |
| `[WAIT]` | blocked on explicit user go-ahead | orchestrator |

Example: `[SCAN] Profiled apps/ai-chat-test — FastAPI + React, no CI wiring found.`

Keep the message after the marker to the same one-or-two-sentence brevity already required
elsewhere in this pattern — the marker replaces a preamble like "Status update:", it doesn't
invite a longer one.

## Commit message register

Concise, technical, senior-full-stack register — describe the defect and the fix precisely, no
filler, no marketing language ("improved", "enhanced"), no restating the diff. See
`implementer.md` for the exact template.

## Files

```
.claude/agents/sentinel/
├── README.md          (this file)
├── orchestrator.md     (sonnet — entry point, repo profiling, roster proposal, dispatch, live steering)
├── auditor.md          (sonnet default — template persona, orchestrator instantiates one per domain)
├── ranker.md           (sonnet — merges findings, applies severity rubric, writes main.md)
└── implementer.md      (haiku — applies near-final diff, tests, commits)
```

`auditor.md` is a template, not a single fixed agent — the orchestrator fills in `{{DOMAIN}}`,
`{{DETERMINISTIC_TOOLING_HINTS}}`, and `{{WEB_SEARCH_FOCUS}}` per domain when it dispatches each
Task call, rather than there being a separate `.md` file per possible domain.
