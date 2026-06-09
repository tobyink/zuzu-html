# Phase 7 Report: Fragment Parsing And Scripting Flag

## Summary

Phase 7 implements `HTML.parse_fragment` and
`HTMLParser.parse_fragment`, returning `HTMLDocumentFragment` instances
for context-sensitive HTML fragment parsing. The parser now accepts
`context`, `scripting`, and `strict` options for fragments, records the
last staging document and parse errors consistently with full-document
parses, and leaves `load`/`dump` as explicit placeholders.

## Completed Scope

- Added `HTMLTreeBuilder.parseFragment(input, context, scripting)`.
- Added fragment tree-builder state:
  - fragment parsing flag;
  - fragment context element;
  - returned `HTMLDocumentFragment`;
  - synthetic `html` element used as the open-stack root.
- Added `HTMLTreeConstructionResult.fragment()`.
- Added fragment-aware adjusted current node handling for context
  elements, including SVG and MathML contexts.
- Added fragment insertion-mode reset for table, table-body, row, cell,
  caption, colgroup, select, template, head, body, and frameset contexts.
- Added context-sensitive tokenizer initial states:
  - `title` and `textarea` use RCDATA;
  - `style`, `xmp`, `iframe`, `noembed`, and `noframes` use RAWTEXT;
  - `script` uses script data;
  - `noscript` uses RAWTEXT when scripting is enabled;
  - `plaintext` uses PLAINTEXT.
- Added scripting flag storage for fragment parsing.
- Added `script` handling through script-data text mode in head/body
  routing.
- Added `noscript` scripting-on/scripting-off behaviour.
- Added fragment form-pointer setup from the context element or its
  ancestors.
- Added `HTMLDocumentFragment.getElementsByTagName` for DOM-like
  fragment composition.
- Updated parser POD to document fragment options and return type.
- Added `tests/html/fragment.zzs`.
- Updated `tests/html/parser.zzs` to assert the implemented fragment
  API instead of the former placeholder.

## API Changes

- `HTML.parse_fragment(String html, ... options) -> HTMLDocumentFragment`
- `HTMLParser.parse_fragment(String html, ... options) -> HTMLDocumentFragment`
- `HTMLTreeBuilder.parseFragment(String input, context, Boolean scripting)`
- `HTMLTreeConstructionResult.fragment()`
- `HTMLDocumentFragment.getElementsByTagName(name)`

Fragment options:

- `context`: defaults to `"div"`; accepts a tag-name string or an
  `HTMLElement`/`HTMLTemplateElement`.
- `scripting`: defaults to `false`; controls `noscript` parsing.
- `strict`: defaults to `false`; throws after parsing if parse errors
  were recorded.

## Verification

All required Phase 1-7 focused suites passed:

- `tests/html/dom.zzs`: `1..120`
- `tests/html/tokenizer.zzs`: `1..87`
- `tests/html/parser.zzs`: `1..26`
- `tests/html/treebuilder.zzs`: `1..48`
- `tests/html/in-body.zzs`: `1..59`
- `tests/html/table.zzs`: `1..25`
- `tests/html/select.zzs`: `1..18`
- `tests/html/template.zzs`: `1..18`
- `tests/html/frameset.zzs`: `1..17`
- `tests/html/foreign-content.zzs`: `1..48`
- `tests/html/fragment.zzs`: `1..44`
- `git -C tobyink-dists/html diff --check`: passed with no output.

## Exit Criteria

- Fragment parsing returns a documented `HTMLDocumentFragment` shape that
  can be used with DOM traversal and mutation APIs.
- String and DOM context elements are accepted.
- Context-sensitive insertion mode, tokenizer state, scripting, form
  pointer, SVG, and MathML behaviours have focused coverage.
- Parser reuse after fragment parsing is covered and does not leak
  fragment state into later full-document parses.

## Known Limitations

- Phase 8 html5lib `.dat` harness integration is intentionally deferred.
- Standard `#script-on`/`#script-off` fixture variants are not yet
  imported; focused tests cover the parser option until the Phase 8
  harness lands.
- The parser still does not execute scripts; `scripting` only affects
  parsing decisions such as `noscript`.
- `HTML.load` and `HTML.dump` remain placeholders for later work.
