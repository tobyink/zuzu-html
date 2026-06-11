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
- "In template" follows the spec dispatch: caption/colgroup and table
  sections re-enter "in table", col enters "in column group", stray
  end tags are ignored, and characters/comments use the in-body rules
  without losing the template insertion mode; template start tags
  inside colgroup are processed in head.
- Stray caption/col/colgroup/frame/head/tbody/td/tfoot/th/thead/tr
  start tags in body are ignored per spec.
- Full spec "adjust SVG attributes" table (62 camelCase attribute
  names); adjusting foreign attributes only matches the spec's exact
  xlink:*, xml:lang/xml:space, and xmlns:xlink names instead of any
  prefixed attribute.
- "In select": input/keygen/textarea start tags close the select per
  spec (ignored in the fragment case when no select is in select
  scope), and end tags use the in-body rules only when their element
  is open inside the select, so formatting elements outside the
  select stay intact.
- The search element is a block/special element: it closes an open p
  and participates in scope checks.
- Adoption Agency Algorithm spec fixes: the inner loop survives
  non-formatting nodes being removed from the stack (the element above
  is captured before removal); "insert last node" foster-parents when
  the common ancestor is a table part (and targets template content);
  the in-scope check tests the chosen formatting element itself rather
  than any element with that name; the active formatting element
  search stops at the last marker.
- A nobr start tag with a nobr in scope is a parse error that runs the
  adoption agency algorithm and reconstructs before inserting.
- Reconstructed active formatting clones are inserted at the
  appropriate place (foster-parenting in table contexts), and flushing
  non-whitespace pending table characters reconstructs the active
  formatting elements first. adoption01.dat and tests26.dat are fully
  green.
- html5lib tree-construction expected failures reduced from 1021 to
  188.
- Portability: the script-data tokenizer used `continue` (a switch
  fallthrough keyword) as loop control, which zuzu-rust and zuzu-js
  tolerate but zuzu.pl does not; it now uses `next`. The
  tree-construction harness no longer calls a method on a String
  (`length` operator instead) and no longer re-wraps the manifest Path
  in `new Path(...)`. The full tree-construction suite now passes on
  zuzu-rust, zuzu.pl, and zuzu-js.

## 0.0.1 - 2026-06-10

*First release.*