---
name: sniffer
description: Get a quick exploratory snapshot of an unfamiliar repo or folder — what it is, what tooling/stack it uses, plus diagrams (data flow, user request flow) and any gotchas called out in its README/CLAUDE.md/AGENTS.md files. A supporting skill for Sentinel (useful as a fast first look before a full audit), but usable standalone any time you land in a codebase cold. Trigger on "sniff <path>", "what is this repo/folder", "give me a snapshot of X", "explore this codebase", "what tooling does X use", or the user landing in an unfamiliar repo and asking what they're looking at.
---

# Sniffer — Quick Repo Snapshot

## Invocation

Natural language works, not just the slash form:
- `/sniffer <path>`
- "sniff `<path>`"
- "what is this repo/folder about"
- "give me a snapshot of `<path>`"
- "explore this codebase"

`<path>` defaults to the current working directory if the user doesn't name one.

## What this is

A single-pass, read-only exploratory task — no roster proposal, no dispatch, no fixing, no
findings ranking. It exists to answer "what am I looking at?" fast, not to audit it. If the user
wants a full audit-and-fix pass, point them at `/sentinel` instead — sniffer is a good thing to run
first, but it is not sentinel and doesn't hand off to it automatically.

## Steps

Run these yourself in the current session — don't spawn a subagent for this, it's meant to be
fast and cheap. If asked to sniff several unrelated paths at once, then dispatch one
`general-purpose` Agent per path in parallel.

### 1. Identify what it is

- `git -C <path> log --oneline -5` and `git -C <path> remote -v` (if it's a git repo) — name,
  recent activity, where it lives
- Read the manifest: `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` / equivalent —
  name, description, scripts, dependencies
- Read `README.md` if present — the maintainers' own account of purpose and usage
- `Glob` the top-level structure to spot the shape (monorepo vs single app, `apps/`/`packages/`
  layout, frontend/backend split, etc.)

### 2. Identify tooling

From the manifest(s) and config files found, list:
- Language(s) and version constraints
- Framework(s) — frontend, backend, test runner
- Build/package manager (npm/pnpm/yarn/uv/poetry/cargo/etc.)
- CI config if present (`.github/workflows/`, etc.)
- Lint/type-check setup

### 3. Read agent/instruction files for gotchas

Read, if present: `CLAUDE.md` (root and any nested ones), `AGENTS.md`, `.claude/agents/*`,
`.cursor/rules/*`, or equivalent. Pull out anything that reads as a gotcha or constraint the
maintainers explicitly flagged — non-obvious build steps, "don't do X" warnings, known-fragile
areas, environment quirks. Quote or closely paraphrase these; don't invent gotchas that aren't
actually stated somewhere.

### 4. Draw diagrams

Produce two lightweight diagrams from what was actually found — don't speculate beyond the code
and docs read in steps 1-3:

- **Data flow** — how data moves through the system's main components (e.g. request → handler →
  service → DB/external API → response). Keep it to the components you actually found evidence
  for.
- **User request flow** — for anything with a UI or API surface, the path a typical user
  action takes end to end (e.g. click → frontend route → API call → backend route → response
  rendered). Skip this diagram if the repo has no user-facing surface (e.g. a pure library).

Use plain ASCII/Mermaid-style text diagrams in the chat response — this is a snapshot task, not
a deliverable that needs a published artifact. Only offer to publish as an Artifact if the user
asks for something shareable.

## Output format

Keep it tight — this is a snapshot, not a report:

```
## <repo/folder name>

**What it is:** one or two sentences.
**Stack:** language(s), framework(s), build tool, test runner.
**Structure:** short bullet list of top-level layout.

**Data flow:**
<diagram>

**User request flow:** (omit if not applicable)
<diagram>

**Gotchas:** (omit section if none found)
- <gotcha> — source: <file>
```

## What NOT to do

- Don't propose an auditor roster or dispatch parallel agents for a single path — that's
  Sentinel's job, not this skill's.
- Don't fix, flag severity, or open findings — sniffer is read-only and descriptive.
- Don't fabricate gotchas — only report what's actually written in a README/CLAUDE.md/AGENTS.md/
  agent config file, cited with its source file.
- Don't publish an Artifact by default — chat output is enough unless asked.
