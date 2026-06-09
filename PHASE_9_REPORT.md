# Phase 9 Report: Production Hardening And Release Readiness

## Summary

Phase 9 replaced the scaffold-facing public documentation with
production documentation for the implemented parser, DOM API,
metadata, packaging, test harness, and known limitations. No parser
rewrite was attempted.

## Documentation Completed

- Rewrote `README.md` to cover purpose, source-checkout usage,
  imports, full-document parsing, reusable `HTMLParser`, parse errors,
  strict mode, fragment parsing, the scripting flag, DOM construction,
  mutation, traversal, namespace-aware SVG/MathML creation, template
  content, `std/data/xml` compatibility, intentional limitations,
  html5lib coverage, and fixture provenance.
- Updated `modules/html/parser.zzm` POD to describe current public APIs:
  `HTML.parse`, `HTML.parse_string`, `HTML.parse_fragment`,
  `HTML.load`, `HTML.dump`, `HTMLParser.document()`,
  `HTMLParser.errors()`, and `HTMLParser.parseErrors()`.
- Updated `modules/html/dom.zzm` POD to describe the current DOM classes,
  node operations, document factories, attribute APIs, template content,
  namespace APIs, compatibility aliases, and known API limits.
- Made `HTML.load` and `HTML.dump` status explicit: both still throw
  clear unimplemented errors.

## Metadata And Packaging Decisions

- `zuzu-distribution.json` keeps the required name, version, author,
  repository, licence, abstract, status, and dependencies.
- Repository is `https://github.com/tobyink/zuzu-html`.
- Licence remains `Artistic-1.0 OR GPL-2.0-or-later`.
- Dependencies remain:
  - `std/string`, required by the DOM, tokenizer, tree builder, and tests.
  - `std/io`, required by the committed html5lib fixture harness.
- Status was changed from `unstable` to `trial`. The current `zuzuzoo`
  validator and website validator accept only `stable` and `trial`; the
  parser still has documented html5lib expected failures, so `trial` is
  the correct validator-supported non-stable status.
- Packaging validation was run from a `/tmp` tarball built only from
  `git ls-files`, so generated caches and archives were excluded from
  the package payload.

## Compatibility Statement

The DOM API is intentionally close to the useful surface of
`std/data/xml`: document factories, traversal, mutation, attributes,
text content, simple selector helpers, cloning, equality, `visitEach`,
`findFirst`, and serialization.

Intentional differences remain:

- HTML parsing, namespaces, void elements, and template content follow
  HTML semantics, not XML semantics.
- `createCDATASection` returns `HTMLText`.
- `findnodes` and `findvalue` throw unsupported-method errors.
- `querySelector` and `querySelectorAll` support only tag, `#id`,
  `.class`, and `*` selectors.
- `toHTML(pretty)` and `toXML(pretty)` accept `pretty` but serialize
  compact HTML.
- `HTML.load` and `HTML.dump` are not implemented.

## html5lib Coverage And Parse-Error Policy

The committed html5lib tree-construction fixture set is pinned to
upstream commit `9fb614afaa42ce8787840f057b32084308e76549`. Fixture
licence and provenance are committed under `tests/html5lib/`.

Latest harness result:

- Parsed cases: 1,795.
- Test variants: 3,551.
- Passing non-xfailed variants: 2,531.
- Expected-failure variants: 1,020.
- TODO passes: 0.
- Unexpected failures: 0.
- Parse-error-count mismatches: 2,245.

Tree shape is the hard gate. Expected failures are still run and
compared; they are not silently skipped. Parse-error counts are reported
as diagnostics and are not currently pass/fail criteria. Error-code
parity with html5lib is not claimed.

## Commands Run

Stale-term scan:

```sh
rg -n "scaffold|placeholder|planned|Phase [0-9]|TODO|FIXME|deferred|unsupported|unstable" tobyink-dists/html
```

Result: public README/changelog/parser DOM/tokenizer/treebuilder POD no
longer contain stale scaffold or phase-progress wording. Remaining hits
are historical phase reports, `DEVELOPMENT_PLAN.md`, explicit
limitations, test names, TODO-pass harness output, and implementation
error codes.

Focused suites:

```sh
for suite in dom tokenizer parser treebuilder in-body table select template frameset foreign-content fragment; do
  printf '%s\n' "## $suite"
  env ZUZULIB=tobyink-dists/html/modules:stdlib/modules \
    zuzu-perl/bin/zuzu "tobyink-dists/html/tests/html/$suite.zzs"
done
```

Result: exit status 0.

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
```

Full html5lib tree-construction harness:

```sh
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules \
  zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/tree-construction.zzs
```

Result: exit status 0.

```text
Failed 1020 tests (1020 todo, 0 true fails)
# html5lib tree-construction variants: 3551; parsed cases: 1795; xfailed variants: 1020; TODO passes: 0; parse-error-count mismatches: 2245; unexpected failures: 0
1..3551
```

Because TODO passes were 0, the xfail manifest was left unchanged.

Diff hygiene:

```sh
git -C tobyink-dists/html diff --check
```

Result: exit status 0, no output.

Package archive dry-run install:

```sh
tmp=$(mktemp -d /tmp/zuzu-html-phase9.XXXXXX)
mkdir -p "$tmp/html-0.0.1"
git -C tobyink-dists/html ls-files -z \
  | tar --null -C tobyink-dists/html -T - -cf - \
  | tar -C "$tmp/html-0.0.1" -xf -
tar -C "$tmp" -czf "$tmp/html-0.0.1.tar.gz" html-0.0.1
ZUZULIB=tobyink-dists/html/modules:stdlib/modules \
  zuzuzoo install --dry-run --no-test \
  --lib-dir "$tmp/lib" --bin-dir "$tmp/bin" \
  --meta-dir "$tmp/meta" --cache-dir "$tmp/cache" \
  "$tmp/html-0.0.1.tar.gz"
status=$?
rm -rf "$tmp"
exit $status
```

Result: exit status 0.

```text
zuzuzoo: planning install for /tmp/zuzu-html-phase9.ry7Jfs/html-0.0.1.tar.gz
zuzuzoo: using local archive /tmp/zuzu-html-phase9.ry7Jfs/html-0.0.1.tar.gz
Install target:
  lib: /tmp/zuzu-html-phase9.ry7Jfs/lib
  bin: /tmp/zuzu-html-phase9.ry7Jfs/bin
  meta: /tmp/zuzu-html-phase9.ry7Jfs/meta

Removals:
  none

Installs:
  - html 0.0.1
    module html/dom.zzm -> /tmp/zuzu-html-phase9.ry7Jfs/lib/html/dom.zzm
    module html/parser.zzm -> /tmp/zuzu-html-phase9.ry7Jfs/lib/html/parser.zzm
    module html/tokenizer.zzm -> /tmp/zuzu-html-phase9.ry7Jfs/lib/html/tokenizer.zzm
    module html/treebuilder.zzm -> /tmp/zuzu-html-phase9.ry7Jfs/lib/html/treebuilder.zzm

Conflicts:
  none
Dry run complete
```

## Known Limitations

- Substantial html5lib tree-construction gaps remain tracked in
  `tests/html/tree-construction-xfails.zzm`.
- Parse-error count and error-code parity are diagnostic only.
- Script execution and parser-time script-driven DOM mutation are not
  implemented.
- `HTML.load` and `HTML.dump` are not implemented.
- Selector support is intentionally limited.
- XPath/ZPath-style `findnodes` and `findvalue` are not implemented.
- Pretty serialization is not implemented.

## Exit Criteria Assessment

- README no longer describes a scaffold or placeholders: met.
- Public parser and DOM APIs documented with working examples: done.
- `zuzu-distribution.json` validates under current tooling: met.
- Archive dry-run install succeeds from a `/tmp` tarball: met.
- Focused html tests pass: met.
- Full html5lib harness has zero unexpected failures at the claimed
- support level: met.
- `git diff --check` passes: met.
- `PHASE_9_REPORT.md` exists and is specific: done.
