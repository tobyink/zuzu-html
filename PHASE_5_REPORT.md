# Phase 5 Report: Tables, Select, Template, And Frameset

## Summary

Phase 5 adds specialised HTML tree-construction modes for table
content, foster parenting, select/select-in-table recovery, template
content, and framesets. The parser still does not implement foreign
content, fragment parsing, or the html5lib `.dat` harness.

## Completed Scope

- Added insertion modes for `in table`, `in table text`, `in caption`,
  `in column group`, `in table body`, `in row`, `in cell`, `in select`,
  `in select in table`, `in template`, `in frameset`,
  `after frameset`, and `after after frameset`.
- Added table-scope stack helpers, table-context stack clearing,
  cell/caption close helpers, active-formatting markers, template mode
  stack state, pending table character buffering, and a foster-parenting
  insertion path shared by element, text, and comment insertion.
- Added minimal template DOM support: `HTMLTemplateElement` and
  `HTMLDocumentFragment`; `HTMLDocument.createElement("template")`
  returns a template element whose `content()` is the insertion target
  for template contents.
- Implemented focused handling for captions, column groups, implied
  table sections/rows/cells, nested-table recovery, hidden inputs and
  forms in tables, option/optgroup auto-closing, table tokens inside
  select, template EOF recovery, `frame` void insertion, nested
  framesets, and `noframes` raw-text handling.

## API Changes

- `html/dom` now provides `HTMLTemplateElement` and
  `HTMLDocumentFragment`.
- `html/parser` re-exports the new template DOM classes.
- `HTMLTreeTestSerializer` prints template content under a documented
  `content` line so Phase 5 tree-shape tests can assert fragment
  contents deterministically.

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

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/table.zzs
# pass: 1..25

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/select.zzs
# pass: 1..18

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/template.zzs
# pass: 1..18

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/frameset.zzs
# pass: 1..17

git -C tobyink-dists/html diff --check
# pass: no output
```

## Exit Criteria Assessment

- Tables, captions, column groups, table bodies, rows, cells, and
  foster-parented text are implemented and covered by structural tests.
- Select and select-in-table recovery are implemented and covered by
  option, optgroup, and table-token recovery tests.
- Template content insertion uses a document fragment and is covered by
  DOM and tree-builder tests, including EOF recovery.
- Frameset modes are implemented and covered by body-less frameset,
  nested frameset, `frame`, `noframes`, and body-content prevention
  tests.
- Parser reuse after table/select/template/frameset parses is covered.

## Known Limitations

- Foreign content for SVG and MathML remains Phase 6 scope.
- `HTML.parse_fragment` remains unimplemented until Phase 7.
- The html5lib `.dat` harness remains Phase 8 scope.
- The Phase 2 named-character-reference table is still partial.
- Table/select/template/frameset support is tested with focused
  structural cases; full standard-suite conformance is intentionally
  deferred to the html5lib integration phase.
