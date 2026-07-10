# neo-agent-constitution

`.claude/CLAUDE.md` is the universal Claude Code configuration for everything under my `neo` workspace — one canonical set of behavioral rules (honesty, evidence-based claims, TDD, etc.) instead of the same text copy-pasted and drifting across a dozen per-project config files. `.claude/skills/` holds the handful of skills (agent-browser, markitdown, session-archive, tdd) that were identical across those projects, promoted here as the single source of truth. This repo tracks only `.claude/` — the actual project folders each have their own independent git history and are deliberately ignored here.

## Rules

Each rule below started as something duplicated across multiple per-project `CLAUDE.md` files, or a lesson learned in one project that turned out to generalize. This list is the "why," not a restatement of the "what" — see `.claude/CLAUDE.md` for full rule text.

| Rule | Why it exists |
|---|---|
| Intellectual Honesty | Prevents sycophantic agreement with incorrect statements — an assistant that never disagrees is unreliable, not polite. |
| Communication Style | Keeps output signal-dense; hedging and apologies were consistently flagged as noise that buries the actual answer. |
| Evidence-Based Status Claims | Bans confident claims like "fully functional" that weren't backed by an actual build/test/log — traced to a specific incident where a claim was made without substantiation. |
| Test-claim verification (part of Evidence-Based Status Claims) | The AI's sandboxed bash session can silently diverge from the user's real environment (browser rendering, GUI, hardware timing) — a test "passing" there isn't proof it passes for the user. |
| CTFCV Framework | Forces explicit Context/Task/Format/Constraints/Verification assumptions up front, so ambiguity gets caught before work starts rather than after. |
| Implementation Protocol | Directly enforces CTFCV for coding tasks — stops implementation from starting before assumptions are actually confirmed. |
| Proactive Action Mapping | Added after a real miss: a recruiter reply was sent without the freshly-tailored CV attached, because the correlated next step wasn't surfaced before finalizing. |
| Quantitative Rigor | Any numeric conclusion must show its working, so a wrong assumption in step 2 is visible instead of buried inside a final number. |
| Code Quality Protocol | Codifies "read before you propose changes" — guessing at code that hasn't been read produces confidently wrong suggestions. |
| Large Files | Corrects a false assumption (imported from a project skill) that large files must always be pre-chunked — Grep and offset/limit Read make that unnecessary in this environment. |
| Coding-Specific Rules | Keeps changes scoped to what was asked — no speculative abstractions, no dead code left "just in case," no comments narrating unchanged code. |
| TDD (Red-Green-Refactor) | Was already the standing default across most active projects; made universal instead of re-declared per project. |
| Research Protocol | Primary sources and recency prevent answers built on stale or secondhand summaries, while skipping search on settled theory avoids wasted lookups. |
| Session Management (prompt review) | Catches ambiguity or missing context in the very first message, before any work is based on a misreading of it. |
| markitdown skill + Windows App Control workaround | The `markitdown` executable is blocked by this machine's App Control policy; the Python-module invocation is the only path that actually works here. |
| agent-browser skill (headed mode, never auto-close) | Headed mode lets the user watch automation happen instead of trusting an opaque headless run; not auto-closing avoids destroying a session the user may still need. |
| Session Archival Protocol | Standardizes the format so archived conversations are consistently searchable later, instead of each project inventing its own layout. |
| Explicitly Dropped: token-usage tracking | Proactive `/context` display was judged as more noise than value and was deliberately removed — kept here so it isn't silently reintroduced. |
