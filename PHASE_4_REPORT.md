# Phase 4 Report: In-Body Completeness

## Summary

Phase 4 extends `html/treebuilder` from the Phase 3 framework to a much
more complete `in body` implementation. `HTML.parse` now handles ordinary
HTML body construction rules for paragraphs, headings, lists, formatting
elements, forms, buttons, void elements, and plaintext while keeping later
phases out of scope.

## Completed Scope

- Added tree-builder state for active formatting elements, the form pointer,
  and cached head/body elements.
- Added scope and stack helpers for ordinary, button, and list-item scope,
  implied end tags, named popping, and element lookup.
- Implemented active-formatting reconstruction with duplicate pruning.
- Implemented adoption-agency recovery for `a`, `b`, `big`, `code`, `em`,
  `font`, `i`, `nobr`, `s`, `small`, `strike`, `strong`, `tt`, and `u`.
- Expanded `in body` start-tag handling for `html`/`body` attribute merging,
  supported head-ish tags, block elements, headings, `pre`/`listing`, `form`,
  `li`, `dd`/`dt`, `button`, anchors, formatting elements, void elements,
  `hr`, `plaintext`, and generic elements.
- Expanded `in body` end-tag handling for `body`/`html`, block elements,
  `p`, `li`, `dd`/`dt`, headings, `form`, `button`, formatting elements,
  `br`, and generic end tags.
- Added `tests/html/in-body.zzs` with structural DOM and
  html5lib-style serializer assertions for the required Phase 4 scenarios.

## API Changes

No public API was added. `HTML.parse`, `HTML.parse_string`, and
`HTMLParser.parse` retain the Phase 3 API but now cover more HTML body
tree-construction rules. `parse_fragment`, `load`, and `dump` remain clear
placeholders.

Internal tree-builder helpers were added for element classification, scope
checks, stack manipulation, active formatting reconstruction, adoption-agency
recovery, implied end tags, paragraph/list/heading/button closing, and
attribute merging.

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

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/in-body.zzs
# pass: 1..59

git -C tobyink-dists/html diff --check
# pass: no output
```

## Exit Criteria Assessment

- Implied end tags: met for `p`, `li`, `dd`, `dt`, headings, and generic
  supported in-body closing.
- Formatting reconstruction: met and covered by paragraph boundary tests.
- Adoption-agency recovery: met for the requested formatting element set and
  covered by representative `b`/`i` and nested-anchor cases.
- Void elements: met for body-level void insertion and immediate stack pop.
- Headings, lists, paragraphs, forms, buttons, and plaintext: met and covered
  by focused tests.
- Parser reuse: met; tests verify formatting/form/stack/mode/error state does
  not leak across parses.

## Known Limitations

- No tables, select, template, framesets, foster parenting, or table insertion
  modes; those remain Phase 5.
- No foreign content, namespaces beyond DOM storage, SVG, or MathML handling;
  those remain Phase 6.
- No fragment parsing or scripting-flag-specific fragment behaviour; those
  remain Phase 7.
- No html5lib `.dat` harness; standard test integration remains Phase 8.
- The tokenizer still has the Phase 2 partial named-character-reference table.
