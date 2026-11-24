# Docs, comments, and junk files

Check claims against the code, the config, and a run of the command. Another document is not a source.

## Comments

| Decision | What it looks like |
| --- | --- |
| KEEP | Why the code is this way; invariants; units; safety; a link to a spec or ticket that still matters |
| FIX | A name, default, flag, or path that is wrong for the current code. Correct the comment. Do not "fix" it by changing the code |
| CUT | Restates the line; banner comments; a docstring longer than the function and with no extra fact; "was" / "used to" / "previously" / "old" / "TODO removed"; a reference to a symbol or file that is gone; commented-out code |

Commented-out code is recovered from git, not from the comment.

## Docs that earn their place

Keep a doc that states something the code cannot: a rule the compiler does not enforce, a trap, a decision and why, a supported format that is still real, how to build, run, and test. Human-facing runbooks stay if their steps still work.

Cut or shrink:

- A feature, flag, or path the code no longer has.
- A count, total, or "N tests pass" copied into prose. Delete the number and repair the sentence ("the four checks below" becomes "the checks below").
- The same fact in a second file. Keep the file a machine or a human actually follows, link from the other, and say in the report which file you treated as defining and why.
- History told as if it were the present: "after the review", "replaces the old", "no longer". A real changelog the release uses stays.
- An index of files, components, or hooks that repeats the tree. The tree is already there.

Generated docs and generated tables of contents stay. Check for a generator before deleting one.

A document whose premise is gone (it describes a removed product) is proposed for deletion, not patched into something internally consistent and still wrong. If the project already archives under a named directory, `git mv` it there after confirmation. Otherwise `git rm` so history keeps it. Do not leave a "superseded" banner in the reader's path.

When code and a requirements doc disagree, report both. Do not make the doc describe wished-for behaviour, and do not change the implementation inside a doc pass unless the user already asked for that fix.

## Verifying a doc

- Every path exists. Every command is run from a clean shell, or the report marks it unverified.
- Every symbol, env var, flag, port, and default matches the file that defines it.
- After a rename or a deletion, search the removed name repo-wide in `*.md` and in comments. A count baked into a sentence ("the four checks") will not match the name; re-read the surrounding docs in the same directory.

## Junk and temporary files

Candidates, not automatic deletes:

- Tracked session notes: `PLAN.md`, `HANDOFF.md`, `IMPLEMENTATION_SUMMARY.md`, `REVIEW.md`, `PROGRESS.md`, `NOTES.md`, `review-round-*.md`, and similar. A file is a candidate only when nothing living links to it, and it narrates a finished agent task, contradicts the code, or duplicates the README.
- Editor and merge litter: `*.bak`, `*.orig`, `*~`, `*.swp`, `*.tmp`.
- Tracked build output, caches, and generated media that the build recreates.

Not junk because of the name: a plan the README links, an ADR, a changelog, a design note that still matches the code, measurements next to the script that reproduces them.

Untracked and gitignored disk use is a list (path, size, age), grouped apart from tracked cuts. Ask before moving any of it. Credentials, `.env`, databases, uploads, datasets, and model weights are never on that list.
