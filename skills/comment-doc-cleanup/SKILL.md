---
name: "comment-doc-cleanup"
description: "Use when asked to remove outdated, wrong or bloated code comments and docs in any repository, proving no code changed."
---

# Comment and documentation cleanup

Goal: comments and docs that are short, true for the current code, and helpful to the next developer (human or AI). Code behaviour must not change.

## 1. Scope and coordinate first
- Confirm the target branch and which trees are in scope (e.g. one crate, docs/, READMEs). If a name in the ask is ambiguous (typo, abbreviation), state your reading in the PR.
- List files other open PRs or in-flight work touch (`gh pr list`, `git diff --name-only base..branch`). Put them in a SKIP SET and tell those owners. Clean them in a later pass, rebased on the base that contains their work. Never edit a file another branch is changing.
- Leave agent/contributor rule files (AGENTS.md, CLAUDE.md, CONTRIBUTING) alone unless asked.

## 2. Build a baseline and safety net before editing
- Build the base and record the artefact (binary sections, bundle, wheel) with non-deterministic info off (e.g. Rust: panic locations/debug paths off, fixed codegen units).
- Write a comment-only TOKEN CHECK: strip comments from base and new file with the language's lexer and require identical token streams. Prove it on a deliberate comment-only probe edit.
- Write a STALE-REFERENCE SCANNER: comments that name paths, functions, settings/env vars, flags or files that no longer exist; history notes ("was", "used to", "old", "TODO removed", dates, ticket ids); claims about defaults.

## 3. Review in chunks, with a second reviewer
- Split by folder into chunks (~20-50 files). For each comment decide: KEEP (true, useful), FIX (wrong for current code: correct it against the code), CUT (history, restating the code, dead references, noise).
- Check every claim against the code itself, not against other comments or docs.
- Each chunk gets an independent second review against the code before it is accepted.
- Prefer the WHY over the WHAT; keep invariants, units, safety notes, links to source references.
- Do not touch code. Real code issues you spot (dead code, stale log text, wrong names) go into a code_issues.tsv (file, line, issue) for a separate change.

## 4. Docs
- Trim READMEs, build/toolchain notes, third-party notes and fixture docs to a short accurate minimum: what it is, how to build/run/test, where things live. Delete docs describing removed features or old plans.
- Verify each command and path in a doc actually works.

## 5. Prove nothing changed
- Token check passes on every changed code file.
- Rebuild: artefact identical to baseline (compare loaded sections or a hash).
- Run the full test suite.

## 6. Land
- Draft PR into the target branch, summarising: files touched, lines removed, comments corrected, false claims fixed, skip set, identity proof, tests. Attach code_issues.tsv.
- After blocking PRs land, do a follow-up pass over the skip set the same way.