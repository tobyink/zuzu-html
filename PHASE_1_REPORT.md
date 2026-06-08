# Phase 1 Report: DOM Core

## Summary

Phase 1 implemented a pure ZuzuScript mutable DOM core in `html/dom`.
The `html/parser` module remains intentionally unimplemented and still
throws clear placeholder errors.

## Completed Scope

- Implemented `HTMLNode`, `HTMLDocument`, `HTMLElement`, `HTMLText`,
  `HTMLComment`, `HTMLDoctype`, and DOM-compatible aliases including
  `DOMDoctype`.
- Added DOM metadata, traversal, mutation, text content, normalisation,
  cloning, identity, equality, containment, and owner-document handling.
- Added document factories and document constraints: one root element,
  one doctype, comments allowed, and direct document text rejected.
- Added element attributes, simple selector support, tag/id/class
  lookup, tree walking helpers, and clear unsupported errors for
  `findnodes` and `findvalue`.
- Added compact HTML serialization with doctype, comments, void
  elements, escaped text, escaped attributes, `toXML`, and `to_String`.
- Replaced the export-only DOM smoke test with behavioural tests adapted
  from the `std/data/xml` DOM surface.

## API Parity Notes

- The implemented construction, traversal, mutation, attributes,
  cloning, walking, and serialization APIs are close to the practical
  `std/data/xml` DOM-like API.
- XPath methods are intentionally unsupported in Phase 1 and throw
  precise errors instead of returning misleading empty results.
- `querySelector` and `querySelectorAll` are intentionally limited to
  simple tag, id, class, and universal selectors.
- `createCDATASection` is provided for compatibility and returns an
  `HTMLText` node because this HTML DOM core has no separate CDATA node.

## Tests

These checks were run from `/home/tai/src/zuzuscript`:

```bash
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/dom.zzs
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/parser.zzs
git -C tobyink-dists/html diff --check
```

All three checks passed.

## Known Limitations

- No tokenizer or parser has been implemented.
- CSS selector support is deliberately minimal.
- Namespace support is storage-only through `namespaceURI`; MathML and
  SVG integration remain later-phase work.
- `pretty` serialization is accepted but currently ignored.
- Text inside raw-text elements such as `script` and `style` is still
  escaped by the simple Phase 1 serializer.

## Exit Criteria Assessment

- DOM construction, traversal, mutation, attributes, simple selectors,
  cloning, and serialization are implemented and covered by tests: met.
- API parity with `std/data/xml` is practical for tree manipulation:
  met, with XPath explicitly deferred.
- Parser remains unimplemented: met.
- Behavioural tests and parser placeholder test pass: met.
