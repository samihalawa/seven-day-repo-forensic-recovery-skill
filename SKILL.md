---
name: seven-day-repo-forensic-recovery-skill
description: "Run a strict 7-day multi-source repo recovery across current thread, local assistant histories, clipboard stores, terminal evidence, repo docs, git, remote branches, and GitHub PRs; classify what was kept, lost, replaced, or still missing; then manually port only the still-valid gaps onto the current codebase."
---

# Seven Day Repo Forensic Recovery Skill

Use this skill when the user wants a full recent-history reconstruction and reconciliation, not a lightweight summary.

This skill is for cases like:

- "check whether the last days of work got reverted"
- "look across Codex, Claude, Kimi, Gemini, commits, PRs, branches, worktrees, clipboard, terminal"
- "tell me what survived, what was replaced, and restore only what still makes sense now"
- "do not trust previous DONE claims, prove what is in the current code"

Default window is the last 7 days unless the user explicitly asks for another range.

## Modes

Choose one before touching code:

- `forensic-analysis`
  - analysis only
  - no code edits
  - produce the keep/lost/replaced/restored matrix
- `recovery-execution`
  - run the forensic analysis first
  - verify the current repo next
  - manually implement only the still-valid missing work against the current codebase

If the user asks to "fix", "continue", "restore", or "implement", use `recovery-execution`.
Otherwise use `forensic-analysis`.

## Core Principles

- Current thread is the highest-priority instruction source.
- Current repo state beats old narratives.
- Original sources beat summaries.
- A prior `done` claim is false until the current repo proves it.
- Never blindly replay an old branch, stash, or patch.
- Restore only work that is both:
  - clearly evidenced in the last 7 days, and
  - still valid for the current architecture and product rules

## Hard Rules

- Default to the last 7 days across all available sources.
- Prove access with one real sample per source family before broad analysis.
- If a source is missing, mark it `missing-source` and continue.
- Read selected source files sequentially before final judgment.
- Distinguish:
  - `kept`
  - `lost`
  - `replaced intentionally`
  - `restored now`
  - `not restored on purpose`
- Never equate "not ancestor" with "should restore".
- Never equate "was claimed" with "was actually shipped".
- Never overwrite hot dirty files without checking whether they are active local work.
- In execution mode, do not cherry-pick forensic branches wholesale.
- Prefer manual current-code ports over revert/reapply cycles.
- Run the repo’s required verification after each logical change group.

## Source Coverage

Sweep all discoverable relevant sources in this order.

### 1. Active thread

- current conversation
- current repo path
- active user constraints

### 2. Current repo and local state

- `git status --short`
- `git log --oneline -20`
- `git rev-list --left-right --count origin/main...HEAD` when a remote exists
- `git worktree list`
- `git stash list`
- relevant owner files such as `AGENTS.md`, `CLAUDE.md`, `DESIGN.md`, `README.md`, `package.json`

### 3. Local assistant histories

Read recent matching files for the current repo from:

- `~/.codex/sessions/**/*.jsonl`
- `~/.claude/projects/<derived-project>/*.jsonl`
- `~/.copilot/session-state/*.jsonl`
- `~/.copilot/logs/*.log`
- `~/.auto-claude/memories/*`
- `~/.kimi/**`
- `~/Library/Application Support/KimiCode/**`
- `~/.gemini/sessions/**`
- `~/.gemini/antigravity/**`
- `~/.cursor/**`
- adjacent worktree-style assistant folders when present

Take the most recent 20 relevant sessions per source family by default.

### 4. Clipboard and pasted prompt evidence

Inspect if present:

- CopyClip
- VeloxClip
- plaintext clipboard exports or backups

Prefer long clips and dedupe near-identical content.

### 5. Repo-local exported notes

Inspect:

- `docs/codex-messages*`
- `*audit*`
- `*report*`
- `*critique*`
- `*analysis*`

Treat them as leads, not truth.

### 6. Terminal and shell evidence

Inspect when available:

- current thread terminal output
- `~/.zsh_history`
- `~/.bash_history`
- session-local shell logs or PTY transcripts
- command fragments found inside assistant session files

Use terminal evidence to confirm what was actually run, not just what an agent claimed.

### 7. Git and remote collaboration evidence

Inspect both local and remote:

- recent commits on all branches in the time window
- candidate branches
- remote branches
- merged and closed PRs
- PR commit lists
- PR changed-file lists
- branch ancestry against current `HEAD`

If GitHub access is available, inspect:

- `gh pr list --state all --limit 50`
- `gh pr view <number> --json ...`
- `gh repo view`

## Required Analysis Workflow

### Phase 0. Evidence Ledger

Build an evidence ledger with:

- mode
- time window
- repo path
- total local conversation files selected
- total clipboard hits selected
- total repo docs selected
- total commits reviewed
- total PRs/remote branches reviewed
- missing sources

### Phase 1. Ingestion

For each selected source:

- open one real sample first
- confirm format
- read sequentially
- mark coverage as `full` or `partial`

Normalize typo-heavy user requests, but preserve the original wording when it affects meaning.

### Phase 2. Conversation and Change Analysis

For each relevant ask or claim, record:

- original request
- normalized request
- constraints
- files or flows touched
- claimed outcome
- real evidence
- final classification

Tag each claim:

- `[OK]`
- `[PARTIAL]`
- `[FAIL]`
- `[CORRECTED]`
- `[IGNORED]`

Escalation rule:

- if a user re-asked after a prior completion claim, the earlier claim becomes `[FAIL]`

### Phase 3. Reconciliation Matrix

Build an asymmetric matrix with one row per workstream:

- workstream
- source evidence
- status now
- current proof
- action

Allowed statuses:

- `kept`
- `lost`
- `replaced`
- `restored-now`
- `not-restored-on-purpose`
- `unclear`

### Phase 4. Current Repo Verification

Before editing anything in execution mode:

- read the current source files
- inspect sibling surfaces
- inspect dirty worktree
- check whether the historical fix already exists
- verify whether the historical approach still fits current architecture

Classify every candidate gap as one of:

- `missing and valid`
- `missing but stale`
- `present already`
- `replaced intentionally`
- `unsafe to restore`

### Phase 5. Manual Recovery

Only restore items classified `missing and valid`.

Rules:

- port manually onto current files
- preserve current architecture
- do not restore stale UX or deleted flows just because they existed
- prefer shared helpers over raw duplicated logic
- preserve newer, better patterns when old work conflicts
- do not revive destructive data or monetization paths without explicit current evidence

### Phase 6. Verification

After each logical group:

- run required repo verification, at minimum the local typecheck/build step if the repo uses one
- capture file-and-line proof for every restored item
- re-check for sibling instances of the same failure class

## Mistakes To Avoid

- trusting continuation notes over original sessions
- restoring a non-ancestor commit without reading the current file
- confusing "not on current branch" with "missing"
- reviving passive prompts that violate newer product rules
- restoring destructive DB or monetization experiments because they once existed
- touching dirty hot files that belong to ongoing local work
- finishing with only an analysis when the user explicitly asked for implementation

## Output Format

Return results in this order unless the user wants another shape:

1. `Evidence Ledger`
2. `Coverage`
3. `Analysis Summary`
4. `Reconciliation Matrix`
5. `Regression Register`
6. `Current Repo Verification`

In `recovery-execution`, append:

7. `Implemented In This Turn`
8. `Verification`
9. `Not Restored And Why`
10. `Remaining Risks`

## Success Standard

This skill is complete only when:

- all major 7-day sources were either read or explicitly marked missing
- each major workstream has a keep/lost/replaced/restored classification
- the current repo is used as the final source of truth
- only still-valid missing work is restored
- every restored item has concrete file proof
- verification was run after the recovery edits
