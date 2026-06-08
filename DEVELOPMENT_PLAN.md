# HTML Parser Development Plan

## Summary

This distribution will provide a pure ZuzuScript HTML5 parser and a
mutable DOM-like tree API. The main parser module is `html/parser`; the
DOM classes live in `html/dom`. The DOM API should stay as close as
practical to the DOM-like classes exposed by `std/data/xml`, while using
HTML parsing and serialization semantics.

The parser must implement the parsing algorithm defined in chapter 13 of
the WHATWG HTML specification:

<https://html.spec.whatwg.org/multipage/parsing.html>

Compliance should be driven by the html5lib standard tree-construction
tests:

<https://github.com/html5lib/html5lib-tests/tree/master/tree-construction>

## Phase 1: DOM Core

Scope: implement mutable in-memory tree classes in `html/dom`.

Description: build `HTMLDocument`, `HTMLNode`, `HTMLElement`,
`HTMLText`, `HTMLComment`, and supporting classes. Match the practical
surface of `std/data/xml`: `documentElement`, `createElement`,
`createTextNode`, `createComment`, traversal, mutation, attributes,
selectors, `toHTML`, `to_String`, `visitEach`, and `findFirst`.

Exit criteria:

- Adapted XML DOM parity tests pass for construction, traversal,
  mutation, attributes, cloning, selectors, and serialization.
- Document, element, text, comment, and doctype nodes have stable node
  type, node name, parent, owner document, and sibling behaviour.
- DOM methods either work or fail with clear unsupported-method errors.

## Phase 2: Input Stream And Tokenizer

Scope: implement the HTML5 input stream and tokenizer states from
WHATWG HTML chapter 13.

Description: accept Zuzu strings as already-decoded Unicode input,
normalize line endings, emit doctype, start-tag, end-tag, comment,
character, and EOF tokens, and implement named and numeric character
references. Record parse errors without making them fatal unless a
future parser option requests strict behaviour.

Exit criteria:

- Focused tokenizer tests pass for comments, doctypes, start/end tags,
  attributes, raw text, RCDATA, character references, and EOF handling.
- Tokenizer state transitions needed by the tree builder are exposed
  through a small internal interface.
- Parse errors include enough context for test diagnostics.

## Phase 3: Tree Builder Framework

Scope: implement the parser state machine, stack of open elements,
active formatting elements, insertion modes, document mode, and
parse-error collection.

Description: cover the initial, before html, before head, in head, after
head, in body, text, after body, and after after body insertion modes.
This phase establishes the shape of the parser without attempting every
specialised branch.

Exit criteria:

- Selected html5lib tree-construction tests pass for basic documents,
  head/body handling, text nodes, comments, doctypes, and implied
  `html`, `head`, and `body` elements.
- The produced tree can be serialized into the html5lib expected-tree
  format by the test harness.
- Parser state can be reset and reused without leaking tree-builder
  state between parses.

## Phase 4: In-Body Completeness

Scope: finish the main `in body` rules.

Description: implement implied end tags, formatting element
reconstruction, the adoption agency algorithm, void elements, headings,
lists, paragraphs, forms, buttons, and plaintext handling.

Exit criteria:

- Broad html5lib `in body` tree-construction cases pass without
  expected-tree drift.
- Formatting-element edge cases are covered by focused tests in addition
  to the imported html5lib tests.
- Void elements and implied end-tag behaviour match the standard test
  suite.

## Phase 5: Tables, Select, Template, And Frameset

Scope: implement specialised insertion modes.

Description: cover table and foster-parenting rules, captions, column
groups, table bodies, rows, cells, select/select-in-table, template
modes, and frameset handling.

Exit criteria:

- html5lib table, select, template, and frameset tree-construction files
  pass.
- Foster-parenting behaviour is covered by small regression tests because
  it is easy to break while changing stack handling.
- Template contents are represented in a documented DOM shape.

## Phase 6: Foreign Content

Scope: implement SVG and MathML integration points.

Description: handle namespace-aware element creation, adjusted tag
names, foreign attributes, HTML integration points, and transitions back
into HTML parsing.

Exit criteria:

- html5lib foreign-content tree-construction tests pass.
- DOM exposes namespace data consistently with the documented API.
- Serialization preserves the expected SVG and MathML tag and attribute
  spellings.

## Phase 7: Fragment Parsing And Scripting Flag

Scope: implement `HTML.parse_fragment`.

Description: support context elements, form pointer setup,
fragment-specific insertion mode selection, and scripting-enabled versus
scripting-disabled behaviour.

Exit criteria:

- html5lib fragment tests pass.
- `#script-on` and `#script-off` variants in the standard test data are
  represented in the test harness and parser options.
- Fragment parsing returns a documented fragment or node-list shape that
  composes cleanly with the DOM mutation API.

## Phase 8: Standard Test Suite Integration

Scope: vendor or submodule `html5lib-tests/tree-construction`.

Description: add a Zuzu test harness that reads `.dat` files, parses
`#data`, honours `#document-fragment`, `#script-on`, and `#script-off`,
and compares the produced tree with the expected html5lib tree format.

Exit criteria:

- The full standard tree-construction suite runs from
  `tests/html/tree-construction.zzs`.
- Unsupported cases are explicitly tracked in a manifest or TODO list;
  tests are not silently skipped.
- Failures show the input case, expected tree, actual tree, and parse
  mode.

## Phase 9: Documentation, Compatibility, And Release Readiness

Scope: finish public docs and distribution validation.

Description: document API parity and intentional gaps versus
`std/data/xml`, parser options, parse errors, serialization semantics,
and standard-test coverage.

Exit criteria:

- Package validation succeeds as a ZDF distribution.
- Smoke tests, DOM parity tests, and the html5lib harness pass at the
  claimed support level.
- `status` remains `unstable` until compliance is documented well enough
  to justify a more stable status.

## Testing Strategy

- Keep smoke tests small while the distribution is a scaffold.
- Add DOM parity tests before parser logic so parser bugs are not mixed
  with tree API bugs.
- Use html5lib tree-construction tests as the main compliance gate.
- Keep parse-error assertions focused and deterministic; tree shape is
  the primary acceptance signal.

## Assumptions And Defaults

- The parser is intended to be pure ZuzuScript unless a later limitation
  proves runtime support is unavoidable.
- The initial scaffold does not vendor html5lib tests; they belong in
  Phase 8.
- HTML DOM classes should follow the `std/data/xml` DOM-like API first
  and add browser-style conveniences only when they do not conflict with
  that compatibility goal.
