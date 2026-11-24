# Report

The report goes in the chat, and later in the PR body. Do not add a markdown or tsv file to the repository to hold it.

```markdown
# Cleanup report

Scope: <branch, trees, or "this branch's diff">
Mode: report | apply
Baseline: <build and test commands, pass/fail>
Skip set: <files left alone, and why>

## Safe (waiting for confirmation)
- `path:line` — <what> — <evidence: tool + grep with no other hits>

## Ask
- `path:line` — <what> — <why it might be live> — recommendation: keep | cut

## Fix
- `path:line` — claimed <X>, code says <Y>

## Keep (sampled traps)
- `path:line` — <the dynamic, config, or public use that saved it>

## Follow-ups (not in this change)
- `path:line` — <bug or behaviour change observed>

## Proof (fill in after apply)
- Commits: <hash, theme>
- Token check: pass | not applicable | not available
- Artefact hash: match | expected change | not deterministic
- Tests: <command and result>
- Not proven: <runtime traffic, other repos, commands not run>
```

Lead with counts, then the rows. A row without a file and a reason does not go in. Empty sections are omitted, except Proof after an apply.
