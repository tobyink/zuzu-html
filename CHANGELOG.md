# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project roughly adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

### Fixed

- Script-data tokenizer: matched end tags now hand control back to the
  tree builder immediately, attributes on text-mode end tags are
  consumed up to ">", original case is preserved when an end-tag
  candidate is re-emitted as text, and EOF handling no longer
  duplicates "<" or leaks tag characters.
- Tokenizer: "/" inside an unquoted attribute value is treated as an
  ordinary character instead of a self-closing flag.
- Tree builder: stray end tags are ignored in "before head", "in
  head", and "before html" per spec; basefont/bgsound are handled in
  head; noframes/basefont/bgsound route from body to the in-head
  rules; textarea/xmp/iframe/noembed/noscript-with-scripting switch to
  RCDATA/RAWTEXT from "in body"; a stray </p> inserts the spec's
  phantom p element; the newline after pre/listing/textarea is
  skipped; hr is allowed inside select; ruby rb/rtc/rt/rp generate
  implied end tags; section end tags in "in cell" are ignored when not
  in table scope.
- Tree-test serializer no longer escapes quotes (html5lib format is
  raw).
- Full WHATWG named character reference table (2231 entities,
  generated from entities.json) with longest-prefix matching and the
  attribute-value legacy exception; numeric references preserve the
  case of "X" when no digits follow.
- Spec-conformant U+0000 handling across the data state, in body,
  in select, foreign content, and bogus comments.
- Relaxed select parsing (unknown start tags use the in-body rules);
  foreign attribute namespace adjustment is restricted to SVG/MathML
  elements.
- Foster parenting only relocates insertions when the target is a
  table-section element, and "in table" only buffers characters when
  the current node is one; foreign content end tags with no matching
  foreign element fall through to the HTML rules; the adoption agency
  and scope checks are namespace-aware (SVG/MathML integration points
  are special elements and scope boundaries, except in table scope);
  non-space characters after leading whitespace in a colgroup pop the
  colgroup; td/th and section start tags in "in cell" respect table
  scope (fixing an infinite reprocess loop).
- html5lib tree-construction expected failures reduced from 1021 to
  423.

## 0.0.1 - 2026-06-10

*First release.*