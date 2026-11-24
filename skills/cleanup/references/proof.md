# Proving a comment-only edit

Use this when a changed code file is supposed to contain no token change. Docs and dead-code commits do not use it.

## Token check

Strip comments with the language's lexer and compare the remaining token stream to the baseline file. A regex is not a lexer: strings and regular expressions contain comment markers.

Prove the check before relying on it. Add a one-line comment to a copy of a scoped file, run the check, and require a pass. Then change one identifier, run the check, and require a failure. Discard the probe.

If the language has no lexer available without installing a tool, do not claim token identity. Say so, and rely on a diff that shows only comment lines plus a green build and tests.

### Python (stdlib)

```python
import tokenize, io, pathlib, sys

def tokens(path):
    src = pathlib.Path(path).read_bytes()
    out = []
    for tok in tokenize.tokenize(io.BytesIO(src).readline):
        if tok.type in (tokenize.COMMENT, tokenize.NL, tokenize.ENCODING, tokenize.ENDMARKER):
            continue
        out.append((tokenize.tok_name[tok.type], tok.string))
    return out

before, after = sys.argv[1], sys.argv[2]
raise SystemExit(0 if tokens(before) == tokens(after) else 1)
```

Compare against the file from the baseline commit (`git show BASE:path`), not against the editor's saved copy.

### Other languages

Do the equivalent with a real parser the project already depends on (TypeScript compiler API, `go/scanner`, `syn`, tree-sitter). Drop comment and trivia tokens only. Compare type and text of what remains.

## Artefact identity

When the build can be made deterministic (fixed codegen units, source paths remapped, no timestamps), hash the artefact before and after a comment-only commit. The hashes match. Record the command that turned non-determinism off.

A dead-code deletion is expected to change the artefact. Do not require a matching hash there. Require a green build and the full test suite instead, and say in the report that runtime call data was or was not available.
