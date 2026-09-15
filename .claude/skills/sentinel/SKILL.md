---
name: sentinel
description: Run a full audit-and-fix pass over an already-cloned local repo using the Sentinel dynamic multi-agent pattern — dynamic per-domain auditors (security, perf, RAG patterns, a11y, product/UX, dev tooling, etc.), a severity-ranked issue list, and sequential surgical fixes with atomic commits. Trigger on natural-language phrasing, not just the slash form — "get sentinel to check/audit/review/investigate <path>", "have sentinel look at X", "run sentinel on X", "/sentinel <path>", or the user preparing for/running a timed whole-codebase audit-and-fix exercise. The verb varies (check, audit, review, investigate, scan) — what matters is "sentinel" + a repo/path target.
---

# Sentinel — Whole-Repo Audit & Fix

## Invocation

Natural language works, not just the slash form — any of these should trigger this skill:
- `/sentinel <local-path>`
- "get sentinel to check/audit/review/investigate `<local-path>`"
- "have sentinel look at `<local-path>`"
- "run sentinel on `<local-path>`"
- "sentinel, audit `<local-path>`"

The trigger word is "sentinel" plus a repo/path target — the verb around it (check, audit, review,
investigate, scan, look at) doesn't change what happens. Extract `<local-path>` from whatever the
user actually said; ask for it if they said "get sentinel to check this" without naming a path and
none is obvious from context (e.g. no repo currently open/discussed).

`<local-path>` must already be cloned locally (this skill does not clone — the user is expected
to have already run `git clone` themselves, e.g. via their own GitHub SSH setup).

## On Trigger — Show This First

Before doing anything else (before validating the path, before dispatching the orchestrator),
output this banner as-is so the user sees Sentinel has been engaged:

```
      (@)                                  ,======.
       \\                                //  ((@))  \
        \\~~~~~~                    ~~~~//   \\==// |
              \\~~~~              ~~~~//       `--'
                 \\~~  ,-------.  ~~//
                   \\ /  o . O  \ //
        (@)==~~~~==| .o ((@)) o. |==~~~~==(@)
                    | o  o   o  o |
                     \  o     o  /
                      `.--___--.'
                       /|  ||  |\
                    ~~/ |  ||  | \~~
                 ~~~/    /  \    \~~~
              ~~~/      /    \      \~~~
            //         /      \         \\
         (@)          '        '          (@)

           S E N T I N E L  //  scanning target...
```

Then proceed with the rest of this skill's steps normally.

## Communication Rule — Concise, Always

Every message sent to the user during a Sentinel run — by this skill, the orchestrator, or any
dispatched agent — must be short. This is not optional flavor, it's a hard constraint:

- Status updates: one sentence, prefixed with the bracketed marker from `README.md`'s "Status
  markers" section (`[SCAN]`, `[FOUND]`, `[WAIT]`, etc.). No preamble, no restating what was just
  asked, no "I'll now proceed to...". This applies to relaying a subagent's hand-back too — do not
  re-narrate its findings in prose; give the one-sentence headline and link the file.
- Findings: the fixed what/severity/steps/where/why format only — no extra narrative wrapped
  around it, no restating the finding in prose before or after showing it.
- Roster proposals, ranked lists, commit summaries: tables or short bullet lines, not paragraphs.
- If a longer explanation is genuinely needed (e.g. justifying a root-cause diagnosis), it belongs
  in the written docs/sentinel/*.md file — not repeated back to the user in the chat turn. Point to
  the file instead of duplicating its content in prose.

Default to fewer words. If a status update can be said in five words instead of twenty, use five.

## What this does

Delegates to the Sentinel agent pattern at `.claude/agents/sentinel/`. Read
`.claude/agents/sentinel/README.md` first if you haven't already — it explains the full pipeline,
model-tier rationale, and docs structure this skill produces.

1. Validate `<local-path>` exists and is a git repo (`git -C <path> status`). If it isn't, stop
   and tell the user — do not attempt to clone it yourself.
2. Hand off to the **orchestrator** (`.claude/agents/sentinel/orchestrator.md`) via the Agent tool,
   passing the validated path. The orchestrator owns everything from here: repo profiling, dynamic
   roster proposal, dispatch, ranking, and sequential implementation.
3. Stay available for live steering for the rest of the session — if the user interjects with a
   tip ("check X, I saw this behavior"), relay it to the running orchestrator rather than starting
   a separate parallel process.

## Output

All output lands in `<local-path>/docs/sentinel/` — `main.md` as the live index/tracker, one
`<domain>.md` per auditor. Fixes land as atomic commits directly in the target repo.

## Prerequisites check (only if not already confirmed this session)

The assessment this pattern was built for expects Node v22+ and pnpm installed. If either command
fails, tell the user rather than silently attempting to install project dependencies globally:
```bash
node --version   # expect v22+
pnpm --version
```

## When NOT to use this

- A single-file or single-function review — use `/code-review` instead, this is for whole-repo,
  multi-domain audits.
- The repo isn't cloned yet — tell the user to clone it first (with their own auth setup), then
  re-invoke.
