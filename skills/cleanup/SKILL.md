---
name: cleanup
description: "Finds and removes dead code, junk or temporary files, and outdated, duplicated, or bloated comments and docs without changing product behaviour. Use when asked to clean up a repository, delete unused code, prune stale or AI-generated markdown, or prove a comment-only edit. Defaults to a dry-run report and waits for confirmation before any delete. Triggers include cleanup, dead code, unused exports, stale docs, 清理, 死代码, 过时文档."
---

# Cleanup

Goal: code, comments, and docs that are short and true for the current tree, with the dead code, junk files, and leftover plans an AI session leaves behind taken out. Product behaviour, public API, commands, flags, file formats, and data stay as they are.

Default is a report. A request that only says "clean up" changes nothing. Delete or rewrite only the items the user confirms. "Apply the safe set" confirms rows marked safe, and nothing marked ask.

If the ask is only comments or docs, do not delete code. If the ask is only dead code, do not rewrite docs except references the deletion broke.

## 1. Scope and coordinate first

- Confirm the target branch and which trees are in scope. On a feature branch, default to files this branch changed, including uncommitted work, and report pre-existing findings outside that diff without editing them. A request to clean the repository widens to the whole tree. If a name in the ask is ambiguous, state your reading in the report.
- List files other open PRs or in-flight work touch (`gh pr list`, `git diff --name-only base...branch`). Put them in a SKIP SET. Clean them in a later pass, rebased on the base that contains their work. Never edit a file another branch is changing.
- Leave agent and contributor rule files (AGENTS.md, CLAUDE.md, CONTRIBUTING, `.cursor/rules`, skill files) alone unless asked. Stale rules in them are reported, not fixed, in this pass.
- Do not start while the build or tests are red, during an incident, or when nobody owns the area. Say so and stop.

## 2. Build a baseline and safety net before editing

- `git status`. Work on a new branch from a clean tree. Leave other people's uncommitted work alone. Do not install a tool the repo does not already have without an explicit OK, including one-off `npx`, `uvx`, or `go run`.
- Read the repo's own instructions and follow their build, test, and PR rules. Record the exact commands.
- Run the build and the full test suite. Code removal needs a green baseline. If it is red, or it cannot be run, label every code finding "not verified by a build", and limit later edits to comments, docs, and junk files.
- For a pass that must not change code tokens, prepare the comment-only token check in [references/proof.md](references/proof.md) and prove it on one deliberate comment-only probe before trusting it.
- Record a short before-count: tracked files, and Markdown lines if docs are in scope. The report compares against it. Do not write that count into the repo.

## 3. Review in chunks, with a second pass

Split by folder into chunks of about 20–50 files. Tool output and filename guesses are leads, not a delete list. Read [references/detectors.md](references/detectors.md) for which tool to run, and [references/keep.md](references/keep.md) before calling any symbol dead. Search every lead yourself, including string, config, and framework uses. If `git log -S` or `git blame` shows the line exists to fix a bug, keep it.

A second pass re-reads each proposed cut against the code before it is accepted. A tool, another comment, or another doc is not the authority. If a read-only subagent is available, it may only reject leads; it may not add cuts the second pass has not checked.

For each candidate decide:

- **KEEP** — true, useful, or not proven dead.
- **FIX** — wrong for the current code. Correct it against the code. A doc whose premise is gone is not fixed in place; propose removal.
- **CUT** — unused, junk, or noise, and then only as **safe** or **ask** below.

| Mark | Bar | Examples |
| --- | --- | --- |
| safe | Zero references after the keep-list search, no side effects, not a public API, entry point, migration, fixture, or generated output, and inside the confirmed scope | Unused import; commented-out code; a tracked `*.bak` or editor swap file; an unused internal file nothing names |
| ask | Could change behaviour, is costly to recreate, or the evidence is incomplete | Public export, feature flag, dynamic registry, file another repo might call, anything added recently, human-written docs, untracked data |
| keep | A trap in [references/keep.md](references/keep.md), or doubt | When unsure, keep it. A missed dead symbol is acceptable. Deleting live code is not |

Out-of-scope defects (behaviour bugs, stale log strings, renames that are not required by a deletion) go in the report's follow-ups. They are not fixed in this change.

Do not mix kinds in one commit later: dead code, a dependency, comments, docs, and junk files are separate commits. No formatter pass, no refactor, no "while I'm here".

## 4. Docs, comments, and junk files

Read [references/docs.md](references/docs.md). In short:

- Comments: prefer WHY over WHAT. Keep invariants, units, safety notes, and links to sources. Cut history ("was", "used to", "previously", "TODO removed"), comments that restate the line, and names of paths, symbols, flags, or settings that no longer exist.
- Docs: trim to what it is, how to build, run, and test, and where things live. Verify each command and path. Code wins for current behaviour. A requirements doc that disagrees with the code is a follow-up, not a silent edit.
- Apply the drift test before keeping a sentence: if the value is computed from source, delete it and repair the sentence; if it is copied from the one defining file, link instead of restating; if it is a real historical measurement or decision, keep that atom. A date does not save a derived count.
- Same fact in two files: keep the defining file, point the other at it. Fixing one copy and leaving the rest is a failed pass. Search the fact repo-wide, not only in files the diff touched.
- Junk is a tracked file linked from nowhere, when it narrates a finished agent session, contradicts the code, or duplicates the README. Names alone (`PLAN.md`, `HANDOFF.md`, review-round notes, implementation summaries) are not enough. A changelog, ADR, or living plan stays.
- Untracked caches and build output are listed, not deleted, until the user agrees. Never delete `.env`, credentials, databases, uploads, datasets, or model weights. Never `rm -rf` them.

## 5. Prove nothing users can observe has changed

After each confirmed batch, and again at the end:

- Revert the batch immediately if the build or tests go red. Do not patch the test or the production code to force green. Re-run the detector: a deletion often orphans the next file. Stop when a pass finds nothing new, or after five passes.
- Comment-only code edits: the token check in [references/proof.md](references/proof.md) passes on every changed code file. When the build can be made deterministic, the artefact hash matches the baseline.
- Dead-code edits: build and tests green is required and is not proof that production never calls the symbol. Say which evidence you had (static search, tool, tests) and which you did not (runtime hits, other repositories).
- Docs: every command you kept has been run, or you mark it unverified. No remaining link points at a file this pass removed.
- The diff contains no drive-by formatting or behaviour change.

## 6. Land

Open a draft PR into the target branch. The body states: files touched, lines removed, comments corrected, false claims fixed, skip set, what was proven, tests and commands run, and the ask-list left undone. Follow-ups stay in the PR body. Do not add a findings markdown file to the tree.

After blocking PRs land, do a follow-up pass over the skip set the same way.

One theme per commit, messages that say what went and why it was safe. Do not amend or rewrite history. `git revert` of that commit is the rollback.
