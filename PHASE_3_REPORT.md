# Phase 3 Report: Tree Builder Framework

## Summary

Phase 3 implemented the first full-document HTML tree-builder framework.
`HTML.parse` and `HTMLParser.parse` now return `HTMLDocument` instances
for the basic insertion modes covered by this phase. Fragment parsing,
file loading, dumping, full in-body rules, tables, select/template,
foreign content, and the html5lib `.dat` harness remain deferred.

## Completed Scope

- Added `html/treebuilder` with `HTMLTreeBuilder`,
  `HTMLTreeConstructionResult`, and `HTMLTreeTestSerializer`.
- Implemented the `initial`, `before html`, `before head`, `in head`,
  `text`, `after head`, basic `in body`, `after body`, and
  `after after body` insertion modes.
- Connected tokenizer tokens to the Phase 1 DOM classes for doctypes,
  comments, elements, attributes, text nodes, and basic recovery.
- Preserved tokenizer parse errors in parser/tree-builder errors and
  added simple tree-builder errors for missing doctypes, EOF recovery,
  unmatched end tags, and misnested end tags.
- Updated `HTMLParser` as a reusable parser with `document()`,
  `errors()`, `parseErrors()`, `strict`, and stored `scripting` option.
- Updated the tokenizer so `nextToken()` can produce tokens
  incrementally, allowing tree construction to switch into RCDATA and
  RAWTEXT before subsequent text is tokenized.

## API Added Or Changed

- `HTML.parse(html, ...options)` and `HTMLParser.parse(html, ...options)`
  return `HTMLDocument`.
- `HTML.parse_string` and `HTMLParser.parse_string` delegate to `parse`.
- `HTMLParser.document()`, `errors()`, and `parseErrors()` expose the
  last parse result.
- `HTMLTreeConstructionResult.document()`, `errors()`, and
  `parseErrors()` expose tree-builder results.
- `HTMLTreeTestSerializer.serialize(doc)` emits a deterministic
  html5lib-style tree string for focused tests.
- `parse_fragment`, `load`, and `dump` remain explicit placeholders.

## Selected Cases Covered

- Explicit doctype/html/head/title/body documents.
- Implied `html`, `head`, and `body` elements.
- Comments before doctype, in head, in body, after body, and after html.
- Doctype public/system identifiers and element attribute copying.
- Text coalescing from adjacent character tokens.
- `title` RCDATA entity expansion and `style` RAWTEXT preservation.
- Basic nesting plus simple unmatched and misnested end-tag recovery.
- EOF in head, text mode, and body-like inputs.
- Parser reuse without leaked document, stack, mode, or errors.
- html5lib-style serializer output for representative trees.

## Commands Run

```bash
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/dom.zzs
# pass: 1..114

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/tokenizer.zzs
# pass: 1..87

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/parser.zzs
# pass: 1..25

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/treebuilder.zzs
# pass: 1..48
```

## Exit Criteria Assessment

- Full-document tree builder framework: met for Phase 3 scope.
- Required insertion modes: met.
- `HTML.parse` returns `HTMLDocument`: met.
- Parser reuse and strict mode: met.
- Tokenizer errors copied into parser/tree-builder errors: met.
- DOM-node and html5lib-style serializer tests: met.
- Phase 4+ behaviours excluded and documented: met.

## Known Limitations

- No adoption agency algorithm or full in-body special cases.
- No table, select, template, frameset, or foreign-content handling.
- No fragment parser, context element support, or scripting-flag effects.
- No file `load`/`dump` implementation.
- No html5lib `.dat` test harness yet.
- `HTMLTreeTestSerializer` is a focused Phase 3 assertion helper, not the
  full Phase 8 standard-test harness.
