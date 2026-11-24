# What looks dead and is not

Search these before a cut. One hit is enough to mark the symbol keep or ask.

## Search

For a symbol `generateInvoicePdf`, search all of these, excluding vendored dependencies and VCS metadata:

```bash
git grep -n -E 'generateInvoicePdf|generate_invoice_pdf|generate-invoice-pdf' -- ':!node_modules' ':!.git'
```

Also search config, templates, CI, Docker, and workflow files specifically (`*.yml`, `*.yaml`, `*.json`, `*.toml`, `*.xml`, `*.html`, `*.sql`, `*.md`).

## Never delete as "unused"

- A name reached by `import()`, `require` of a variable, `importlib`, `getattr`, `eval`, or a string-keyed map.
- A file loaded by a glob or a path join (`./plugins/${name}`). Add it to the tool's entry list instead of deleting it.
- Framework conventions: file-system routes, `urls.py` views, Django management commands, Celery tasks, signal receivers, admin classes, middleware listed in settings, pytest fixtures (including autouse), Rails autoload paths, Next.js pages and route handlers.
- Package surfaces: `package.json` `main`, `types`, `bin`, and `exports`; Python entry points and console scripts; Cargo `[[bin]]`; Go `main` packages.
- The public API of a published library. Deprecate and let the owner remove it. This skill does not drop exported symbols of a package other repositories import.
- Side-effect modules: polyfills, CSS, global registration. No exports does not mean unused.
- Code behind a feature flag, build tag, or `if` on an env var, including a flag that appears always on. Retiring the flag is the owner's call.
- Applied database migrations. Never delete those.
- Generated code. Prefer fixing the generator. Deleting only the output brings it back on the next generate.
- Tests, fixtures, and snapshots that still describe live behaviour. Remove a test only when its subject was removed in this pass, or when another test already asserts the same behaviour. Do not weaken an assertion.
- Config files, `*.setup.*`, declaration files, lockfiles, licences, notices, version pins, and security checks.
- Files whose checksum is pinned, and scripts another repository or a scheduled job runs. If that is plausible and you cannot see the caller, mark ask.
- Code added in the last few weeks. It may be waiting for its caller.
- Anything this search cannot see: production logs, other services, notebooks, SQL outside the repo. Absence of a static caller is not absence of a caller.

## Registration trap

A function can be unused as an import and still be live because the module registers it on load (`register({ name: "togglePolygon" })`, a decorator, an `init()` import). Removing the word `export` can be safe when no one imports the name. Deleting the declaration removes the feature. Grep the registered string, not only the identifier.

## Kinds of dead, and the cut each allows

| Kind | Evidence | Cut |
| --- | --- | --- |
| Unreachable | No path from an entry, including dynamic and config searches | Delete the symbol and the imports that only served it |
| Unused | Callable, and every search is empty | Same, if it is not public API |
| Redundant | A live path already does this | Ask. Repoint callers, then delete, in one commit, with tests around the live path |
| Dormant flag | Flag is constantly off, or constantly on | Ask. Do not delete the branch in this skill unless the user confirms the flag is retired |

Risky leaves (a route, a job, a public handler) stay on the ask list even when searches are empty. Do not invent a tombstone metric unless the user asks for one.
