# Phase 8 Report: html5lib Tree-Construction Tests

## Summary

Phase 8 adds a committed html5lib tree-construction fixture set and a
ZuzuScript TAP harness at `tests/html/tree-construction.zzs`.

The harness reads `.dat` fixtures as state-machine records, preserving
blank lines inside `#data`, handling `#errors`, `#new-errors`,
`#document-fragment`, `#script-on`, `#script-off`, and `#document`, and
running both scripting modes when a fixture does not specify one.

## Vendored Fixtures

Source URL:
`https://github.com/html5lib/html5lib-tests/tree/9fb614afaa42ce8787840f057b32084308e76549/tree-construction`

Pinned commit: `9fb614afaa42ce8787840f057b32084308e76549`

Retrieved: 2026-06-09

The upstream MIT licence text is included at `tests/html5lib/LICENSE`.
Provenance is recorded in `tests/html5lib/UPSTREAM.md`. The upstream
tree-construction README is included at
`tests/html5lib/tree-construction/README.md`.

Vendored file counts:

- 60 `.dat` tree-construction fixtures.
- 61 files under `tests/html5lib/tree-construction/`, including README.
- 63 files under `tests/html5lib/`, including licence and provenance.

No submodule was created.

## Harness Behaviour

Full documents are parsed with `HTMLParser.parse(data, scripting: mode)`.
Fragments are parsed with `HTMLParser.parse_fragment(data, context:
context, scripting: mode)`.

Fragment contexts are handled as follows:

- HTML contexts pass the local-name string to `parse_fragment`.
- `svg NAME` contexts create a temporary `HTMLDocument` and pass
  `createElementNS(SVG_NAMESPACE_URI, NAME)`.
- `math NAME` contexts create a temporary `HTMLDocument` and pass
  `createElementNS(MATHML_NAMESPACE_URI, NAME)`.

The harness imports `std/io::Path`, calls `requires_capability("fs")`,
and assumes tests run from `/home/tai/src/zuzuscript`, matching the
workspace test command. `zuzu-distribution.json` now declares `std/io`
because the committed harness uses `Path` to read fixtures.

## Serializer And Parser Fixes

`HTMLTreeTestSerializer` was tightened for html5lib-style output:

- Document and fragment nodes serialize only their children.
- SVG and MathML element labels use `svg local` and `math local`.
- Attribute labels include `xlink`, `xml`, and `xmlns` namespace labels.
- Attributes are sorted by html5lib attribute label.
- Doctypes include public and system IDs when present.
- HTML template content is serialized under `content`; foreign
  `template` elements are treated as ordinary foreign elements.

Small parser/tree-builder fixes were added for cases uncovered while
making the complete suite terminate:

- Empty doctype names are representable in the DOM.
- Foreign fragment breakout tokens no longer reprocess forever.
- `</html>` in column-group mode is ignored as specified.
- `</html>` body close reprocessing now depends on whether body closing
  advanced the insertion mode.
- Synthetic table-cell fragment contexts consume table-structure tokens
  that would otherwise try to close a context element not on the stack.

## Parse-Error Policy

Tree shape is the hard gate.

The harness records actual `parser.errors().length()` and compares it
with the number of legacy `#errors` lines. Mismatches are counted and
reported in a final diagnostic, but otherwise passing tree cases do not
fail solely because of parse-error count mismatch. Error code names are
not compared in Phase 8.

Final tree-construction run:

- Parsed cases: 1,795.
- Test variants: 3,551.
- Passing non-xfailed variants: 2,531.
- Xfailed variants: 1,020.
- TODO passes: 0.
- Parse-error-count mismatches: 2,245.
- Unexpected failures: 0.

## Expected Failures

Manifest path: `tests/html/tree-construction-xfails.zzm`

The manifest keys expected failures by `relative-file#case#script-mode`.
Every xfailed case is still run and compared; there are no silent skips.

Expected-failure categories:

- `parser algorithm gap`: 1,012 variants.
- `pending upstream spec-change fixture`: 4 variants.
- `script execution not implemented`: 4 variants.

Scripted fixtures are xfailed because script execution and DOM mutation
during parsing are not implemented.

## Commands Run

Direct Phase 8 harness:

```sh
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/tree-construction.zzs
```

Result:

```text
# html5lib tree-construction variants: 3551; parsed cases: 1795; xfailed variants: 1020; TODO passes: 0; parse-error-count mismatches: 2245; unexpected failures: 0
1..3551
Failed 1020 tests (1020 todo, 0 true fails)
```

Focused Phase 1-7 suites plus new harness:

```sh
for suite in dom tokenizer parser treebuilder in-body table select template frameset foreign-content fragment tree-construction; do
  env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu "tobyink-dists/html/tests/html/$suite.zzs"
done
```

Results:

```text
dom: 1..120
tokenizer: 1..87
parser: 1..26
treebuilder: 1..48
in-body: 1..59
table: 1..25
select: 1..18
template: 1..18
frameset: 1..17
foreign-content: 1..48
fragment: 1..44
tree-construction: 1..3551; 1020 todo, 0 true fails
```

Diff hygiene:

```sh
git -C tobyink-dists/html diff --check
```

Result: exit status 0, no output.

## Known Limitations And Phase 9 Risks

- The parser still has substantial html5lib tree-construction algorithm
  gaps, tracked by the xfail manifest.
- Parse-error counts are diagnostic only; Phase 9 should decide whether
  to stabilise count parity before comparing error code names.
- Scripted tree-construction tests remain expected failures until script
  execution and parser-time DOM mutation exist.
- Harness runtime is roughly 85 seconds under the current local
  `zuzu-perl/bin/zuzu` command, so future CI integration may need a
  separate extended-test gate or runtime optimisation.
- Phase 9 documentation should explain the current compatibility level,
  xfail manifest, parse-error policy, and standard-test coverage.
