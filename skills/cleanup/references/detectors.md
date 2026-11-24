# Detectors

Run the tool the repo already uses. If none is installed, ask before adding one, and otherwise use the grep method at the bottom. Every hit is a lead. Read [keep.md](keep.md) before proposing a cut.

Work unused files first, then unresolved imports, then unused exports, then unused dependencies. Later kinds are often fallout from the earlier ones. After a deletion, run the tool again.

Do not add a blanket ignore, and do not raise a confidence threshold, to make the report empty. An ignore entry needs a comment that says why the symbol is live.

## By ecosystem

| Ecosystem | Unused code, files, exports | Unused dependencies | How to read the result |
| --- | --- | --- | --- |
| JavaScript / TypeScript | [knip](https://knip.dev) | knip (depcheck and ts-prune are unmaintained; do not add them) | Knip reports what it cannot reach from entry files. A surprising file is often a missing entry, not a delete. Fix config hints before trusting the rest. `knip --fix` may drop an `export` keyword; deleting the declaration is a separate decision, because the module may register itself on load. Do not pass `--allow-remove-files` until the file list has been through the keep-list. |
| Python | [vulture](https://github.com/jendrikseipp/vulture); ruff F401 and F841 for unused imports and locals (this covers unimport; do not add both) | [deptry](https://github.com/fpgmaas/deptry) | Confidence is the kind of syntax, not the chance it is dead. 100% is an unused argument or unreachable code, 90% is an import, 60% is a function, class, or attribute. Treat 60% as a lead. Do not switch on `--min-confidence 80` or `100` to hide framework code; whitelist the live name instead. pytest fixtures, Django views, signals, admin classes, and template-only methods look unused. |
| Go | `deadcode` (`golang.org/x/tools/cmd/deadcode`) with `-test`; staticcheck U1000 | `go mod tidy`, and only delete what the diff shows is unused | `deadcode` starts from `main` and one GOOS/GOARCH/tag set. Re-run per build tag. Exported library API and `//go:linkname` targets are not dead because this binary cannot see them. |
| Rust | rustc `dead_code`, clippy | [cargo-machete](https://github.com/bnjbvr/cargo-machete) or cargo-udeps | cargo-machete misses build-script deps and renamed crates. `dead_code` does not see a bin, example, or bench you did not build. |
| Java / Kotlin | IDE unused inspection, detekt | Gradle dependency-analysis, if the build already has it | Reflection, DI, and annotations dominate. Grep the class name as a string before any cut. |
| Any | compiler and linter warnings | lockfile diff | Warnings are leads. A warning about a public re-export is usually keep. |

## Grep method when no tool is available

1. List top-level definitions (functions, classes, files) in the scoped tree.
2. For each name, `git grep -n -F` the identifier, its snake_case form, and its kebab-case form, excluding `node_modules`, `.git`, and build output.
3. A count of one (the definition only) is a lead, not a finding. Continue with the searches in [keep.md](keep.md).

## Dependencies

Remove a dependency only when nothing imports it and no config, CLI, plugin, or `@types` package needs it. Update the lockfile in the same commit (`npm uninstall`, `uv remove`, the repo's equivalent). A dependency used only by a config file (eslint, babel, a test runner) is live.

## Duplication and links

- Duplicated code: a structural copy tool the project already has, or a direct read of near-copy helpers. Copies inside tests and fixtures are often intentional. Merging live duplicates is a refactor: mark it ask, and do it in its own commit only after confirmation. Repoint callers first, then delete the copy.
- Doc links: the repo's link checker if it has one, otherwise resolve relative links and path literals yourself. External URL checks need network permission; skip them rather than guessing.

## What the tools miss on purpose

Static reachability does not see `import()`, `require(variable)`, `importlib`, `getattr`, string keys, decorators that register on import, file-based routes, or a second repository. Knip leaving entry exports out of the report is intentional: those names are the package's public surface. Turning on `includeEntryExports` in an application (no external consumers) is a lead generator; in a published library it is a breaking-change list, which this skill does not apply.
