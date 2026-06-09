# Phase 6 Report: Foreign Content

## Summary

Phase 6 adds namespace-aware SVG and MathML foreign-content support to the
full-document parser. The parser now creates SVG and MathML DOM elements,
adjusts selected SVG/MathML tag and attribute names, maps XLink/XML/XMLNS
attributes, exits foreign content at HTML breakouts and integration
points, and allows CDATA sections only while tokenizing foreign content.

## Completed Scope

- Added namespace-aware DOM APIs: namespace constants, `createElementNS`,
  namespaced attribute methods, `attributeRecords`, namespace-preserving
  clone/equality, and namespace-aware serialization.
- Added tokenizer `allowCDATA` state so `<![CDATA[` emits character data in
  SVG/MathML contexts and remains parse-error/comment-compatible in HTML.
- Added tree-builder foreign-content dispatch, SVG/MathML insertion,
  adjusted SVG names, adjusted SVG/MathML attributes, foreign attribute
  mapping, HTML breakouts, MathML text integration points, SVG/MathML HTML
  integration points, and foreign end-tag recovery.
- Updated `HTMLTreeTestSerializer` to emit namespace-qualified foreign
  element and attribute labels while preserving existing HTML/template
  output.
- Added focused Phase 6 coverage in `tests/html/foreign-content.zzs`.

## API Changes

- `html/dom` now exports `HTML_NAMESPACE_URI`, `SVG_NAMESPACE_URI`,
  `MATHML_NAMESPACE_URI`, `XLINK_NAMESPACE_URI`, `XML_NAMESPACE_URI`, and
  `XMLNS_NAMESPACE_URI`.
- `HTMLDocument.createElementNS(namespaceURI, qualifiedName)` creates
  namespace-aware elements.
- `HTMLElement` and `HTMLTemplateElement` now support `setAttributeNS`,
  `getAttributeNS`, `hasAttributeNS`, `removeAttributeNS`, and
  `attributeRecords`.
- Existing `setAttribute`, `getAttribute`, `hasAttribute`,
  `removeAttribute`, `attributeNames`, and `attributes` remain available
  using serialized qualified attribute names.
- `HTMLTokenizer` now exposes `setAllowCDATA(boolean)` and `allowCDATA()`
  for tree-builder context control.

## Commands Run

```bash
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/dom.zzs
# pass: 1..120

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/tokenizer.zzs
# pass: 1..87

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/parser.zzs
# pass: 1..25

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/treebuilder.zzs
# pass: 1..48

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/in-body.zzs
# pass: 1..59

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/table.zzs
# pass: 1..25

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/select.zzs
# pass: 1..18

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/template.zzs
# pass: 1..18

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/frameset.zzs
# pass: 1..17

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/foreign-content.zzs
# pass: 1..48

git -C tobyink-dists/html diff --check
# pass: no output
```

## Exit Criteria Assessment

- Namespace-aware DOM nodes and attributes: met.
- Adjusted SVG/MathML names and foreign attributes: met for the covered
  Phase 6 adjustment tables.
- HTML and MathML integration points and exits back to HTML parsing: met
  for focused SVG, MathML, and `annotation-xml` cases.
- CDATA in foreign content: met, including EOF recovery.
- Serialization and html5lib-style test serializer support: met for
  namespace-qualified focused cases.

## Known Limitations

- Fragment parsing, scripting behaviour, file `load`/`dump`, and the
  html5lib `.dat` harness remain deferred.
- The tokenizer still uses the Phase 2 partial named-character-reference
  table.
- SVG/MathML adjustment tables cover the spec names needed by current
  focused tests plus common `fe*` and camelCase names; Phase 8 compliance
  work may expand them when running the full standard suite.
