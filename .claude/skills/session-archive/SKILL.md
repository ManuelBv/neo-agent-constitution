---
name: session-archive
description: Create a comprehensive session archive file capturing the full conversation, technical execution details, code changes, and tool outputs. Use when the user asks to "save this conversation", "store this session", "archive this conversation", "save this to conversations", "create a session file", or "store this in [folder]".
---

# Session Archival Protocol

When the user requests to store or save a conversation, follow this protocol to create a comprehensive session archive file.

---

## Archive File Naming

Format: `[YYYY-MM-DD]-CLAUDE-[subject].md`

Example: `2026-02-01-CLAUDE-diffusion-encoding-research.md`

---

## Required Content Structure

Generate a complete session archive file with the following sections:

### 1. Header Metadata
- Date (YYYY-MM-DD)
- Session Subject (descriptive title)
- Duration estimate
- Status (Completed/In Progress)
- Completeness indicator (100% - All conversation + technical details)

### 2. Conversation Flow Section
For each user message, include:
- **User Request**: Exact user message text in blockquote format
- **Claude Response**: Summary of what Claude responded
- **Actions Taken**: Bullet list of what was executed
- **Sample Results**: Key outputs or file changes

### 3. Technical Execution Details Section
For each major action/file modification, include:
- **File Path**: Absolute path to the file
- **File Details**: Total lines, language, type
- **Original Content**: Code/text before changes (in code blocks)
- **New Content**: Code/text after changes (in code blocks)
- **Changes Made**: Bullet list of modifications
- **Status**: indicator

### 4. Tool Calls & Outputs Section
For each bash/tool command executed:
- **Command**: Exact command with proper formatting
- **Output**: Complete command output
- **Exit Code**: Success/failure indicator
- **Impact**: What changed as a result

### 5. Git History Section (if applicable)
- Commit hashes
- Commit messages
- Files changed
- Insertions/deletions
- Push status and results

### 6. Summary & Metrics Section
Include:
- Table of all actions performed with status
- File statistics (paths, lines, actions)
- Total counts (files modified, created, renamed)
- Timeline breakdown by phase
- Key achievements checklist

### 7. Complete File Listing (if applicable)
For bulk operations (file renames, creations):
- Complete numbered list of all files
- Before/after names for renames
- Status for each file

### 8. Timeline Section
Table with columns:
- Step/Phase
- Action description
- Duration
- Status

---

## Quality Standards

**Completeness**: Must include ALL conversation details and technical execution
- No summarization that omits details
- Full code diffs for every edit
- Complete command outputs
- All user questions and Claude responses

**Accuracy**: All information must be factual
- Exact file paths (not approximations)
- Actual code snippets (not paraphrased)
- Real command outputs (not examples)
- Correct timestamps and dates

**Organization**: Logical, easy-to-navigate structure
- Clear section headings
- Numbered or bulleted lists
- Code blocks for technical content
- Tables for metrics and comparisons

**Comprehensiveness**: Nothing important is left out
- Every file edited is documented
- Every command executed is shown
- Every user interaction is captured
- Full before/after for all changes

---

## Minimum Sections Required
1. Conversation flow with all user messages
2. Detailed technical execution for each major action
3. Complete code/content changes with diffs
4. All command outputs and results
5. Summary metrics and timeline
6. Complete status indicators

---

## Special Instructions
- **Timestamps**: Include session date (today's date in YYYY-MM-DD format)
- **Tool Calls**: Show every bash command, read operation, edit operation
- **Outputs**: Capture actual tool outputs, not summaries
- **Code Context**: Show surrounding code when possible
- **Completeness Check**: Verify every action in conversation is documented

---

## Output Location

Save archive files to the `conversations/` folder within the current project directory:

```
<project>/conversations/[YYYY-MM-DD]-CLAUDE-[subject].md
```
