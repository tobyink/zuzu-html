# html

Pure ZuzuScript HTML5 parser and mutable DOM-like tree classes.

The `html` distribution provides:

- `html/parser`, a document and fragment parser for HTML input.
- `html/dom`, a DOM-like tree API modelled on the practical surface of
  `std/data/xml`.
- `html/tags`, shortcut functions for HTML generation.
- Focused parser and DOM tests plus a committed html5lib tree-construction
  harness.

The parser follows the WHATWG HTML parsing algorithm where currently
implemented, with remaining conformance gaps tracked in
`tests/tree-construction-xfails.zzm`.

## Installation

Install from a ZDF archive with `zuzuzoo`:

```sh
zuzuzoo install html
```

Import the parser facade for most use:

```zzs
from html/parser import HTML, HTMLParser;
```

Import DOM classes directly when constructing trees by hand:

```zzs
from html/dom import HTMLDocument, SVG_NAMESPACE_URI, MATHML_NAMESPACE_URI;
```

Or use `html/tags` for simpler tree construction:

```zzs
from html/tags import *;
```

## Full Document Parsing

Use `HTML.parse` or its alias `HTML.parse_string` for full documents.
Both return an `HTMLDocument`.

```zzs
from html/parser import HTML;

let doc := HTML.parse("<!doctype html><title>Example</title><p>Hello</p>");
let title := doc.getElementsByTagName("title")[0].textContent();
let body_text := doc.getElementsByTagName("body")[0].textContent();
```

The parser inserts omitted `html`, `head`, and `body` elements according
to the tree-builder rules it implements. The returned document can be
traversed and serialized through the DOM API:

```zzs
let root := doc.documentElement();
let html := doc.toHTML();
```

## Reusable Parser

`HTMLParser` is reusable. Each parse resets parser state, stores the
last returned staging document, and records the parse errors for that
operation.

```zzs
from html/parser import HTMLParser;

let parser := new HTMLParser();
let first := parser.parse("<p>one</p>");
let first_errors := parser.errors();

let second := parser.parse("<!doctype html><p>two</p>");
let last_doc := parser.document();
```

`HTMLParser.document()` returns the last full document, or the internal
staging document used for the last fragment parse. `errors()` returns a
copy of the last parse's error array. `parseErrors()` is an alias for
`errors()`.

## Parse Errors And Strict Mode

HTML parse errors are non-fatal by default. They are collected as
`HTMLParseError` objects and exposed through `HTMLParser.errors()` and
`HTMLParser.parseErrors()`. This matches normal HTML parsing, where tree
construction recovers from many malformed inputs.

Pass `strict: true` to throw after parsing if any parse errors were
recorded:

```zzs
let parser := new HTMLParser();
parser.parse( "<!doctype html><p>ok</p>", strict: true );
```

Strict mode does not change the tree-building algorithm. It only turns a
non-empty parse-error list into an exception after the parser has
attempted to build the tree.

The html5lib harness treats tree shape as the hard gate. Parse-error
counts are reported diagnostically; error-code parity with html5lib is
not currently part of the pass/fail contract.

## Fragment Parsing

Use `HTML.parse_fragment` or `HTMLParser.parse_fragment` to parse an HTML
fragment. The return value is an `HTMLDocumentFragment`, not a full
document.

```zzs
from html/parser import HTML;

let fragment := HTML.parse_fragment("<tr><td>x", context: "table");
let cell_text := fragment.getElementsByTagName("td")[0].textContent();
```

The `context` option defaults to `"div"` and accepts:

- A tag-name string, such as `"div"`, `"table"`, `"tbody"`, `"tr"`,
  `"td"`, `"template"`, `"title"`, `"textarea"`, `"style"`,
  `"script"`, `"plaintext"`, or `"noscript"`.
- The string `"svg"` for an SVG fragment context.
- The string `"math"` for a MathML fragment context.
- An `HTMLElement` or `HTMLTemplateElement`, including elements created
  with `createElementNS` for SVG or MathML contexts.

Fragment parsing uses context-sensitive tokenizer states for RCDATA,
RAWTEXT, script data, plaintext, table insertion modes, select modes,
template content, SVG, and MathML.

## Scripting Flag

The `scripting` option defaults to false. It affects the parsing of
`noscript` content and the html5lib scripting-mode variants:

```zzs
let parsed := HTML.parse_fragment(
	"<b>x</b>",
	context: "noscript",
	scripting: false,
);

let raw := HTML.parse_fragment(
	"<b>x</b>",
	context: "noscript",
	scripting: true,
);
```

The scripting flag does not execute scripts. Parser-time script
execution and script-driven DOM mutation are intentionally not
implemented.

## DOM Creation And Mutation

`html/dom` provides `HTMLDocument`, `HTMLElement`, `HTMLText`,
`HTMLComment`, `HTMLDoctype`, `HTMLTemplateElement`, and
`HTMLDocumentFragment`.

```zzs
from html/dom import HTMLDocument;

let doc := new HTMLDocument();
let root := doc.createElement("html");
let body := doc.createElement("body");
let p := doc.createElement("p");

doc.appendChild(doc.createDoctype());
doc.appendChild(root);
root.appendChild(body);
body.appendChild(p);
p.setTextContent("Hello");
```

Supported tree operations include `appendChild`, `prependChild`,
`insertBefore`, `replaceChild`, `removeChild`, `remove`, `childNodes`,
`children`, `firstChild`, `lastChild`, `nextSibling`, `previousSibling`,
`parentNode`, `ownerDocument`, `contains`, `cloneNode`, `isSameNode`,
`isEqualNode`, `normalize`, `textContent`, and `setTextContent`.

Supported lookup and traversal helpers include `documentElement`,
`getElementsByTagName`, `getElementById`, `querySelector`,
`querySelectorAll`, `visitEach`, and `findFirst`. Selectors are limited
to tag names, `#id`, `.class`, and `*`.

Attributes are available through `id`, `setId`, `getAttribute`,
`setAttribute`, `hasAttribute`, `removeAttribute`, `attributeNames`,
`attributes`, `getAttributeNS`, `setAttributeNS`, `hasAttributeNS`,
`removeAttributeNS`, and `attributeRecords`.

`toHTML`, `toXML`, and `to_String` serialize compact HTML. The `pretty`
argument is accepted but currently does not enable pretty printing.

### Using `html/tags`

The `html/tags` module provides shortcuts for constructing a DOM.

```zzs
from html/dom import HTMLDocument;
from html/tags import *;

let doc := new HTMLDocument();
doc.appendChild( doc.createDoctype() );
doc.appendChild(
	HTML(
		BODY(
			P("Hello")
		)
	)
);
```

## SVG And MathML Namespaces

Use `createElementNS` for namespace-aware DOM creation:

```zzs
from html/dom import HTMLDocument, SVG_NAMESPACE_URI, MATHML_NAMESPACE_URI;

let doc := new HTMLDocument();
let svg := doc.createElementNS( SVG_NAMESPACE_URI, "svg" );
let circle := doc.createElementNS( SVG_NAMESPACE_URI, "circle" );
svg.appendChild(circle);

let math := doc.createElementNS( MATHML_NAMESPACE_URI, "math" );
```

The parser creates SVG and MathML elements in their namespaces during
foreign-content parsing. It also handles adjusted SVG tag names,
MathML/SVG integration points, and the XLink, XML, and XMLNS attribute
namespaces needed by the html5lib tree-construction fixtures.

## Template Content

`HTMLDocument.createElement("template")` returns an
`HTMLTemplateElement`. Template children are stored in
`template.content()`, an `HTMLDocumentFragment`.

```zzs
let doc := HTML.parse("<body><template><p>x</p></template></body>");
let template := doc.getElementsByTagName("template")[0];
let content := template.content();
```

`template.childNodes()` is empty for parsed template contents; use
`template.content().childNodes()` to inspect or mutate the template body.
Serialization emits the content between the template start and end tags.

## Compatibility With std/data/xml

The DOM API intentionally tracks the useful parts of `std/data/xml`:
node factories, traversal, mutation, attributes, text content, cloning,
simple selector helpers, `visitEach`, `findFirst`, and serialization.

Compatibility aliases are exported as `DOMNode`, `DOMDocument`,
`DOMElement`, `DOMText`, `DOMComment`, and `DOMDoctype`.

Known differences are intentional:

- HTML parsing, HTML namespaces, void elements, and template content use
  HTML semantics rather than XML semantics.
- `createCDATASection` returns an `HTMLText` node because this DOM core
  does not expose CDATA section nodes.
- `findnodes` and `findvalue` throw clear unsupported-method errors.
- `querySelector` and `querySelectorAll` support only simple selectors.
- `HTML.load` and `HTML.dump` are not implemented. Use `HTML.parse` and
  `toHTML` with explicit `std/io` handling in application code for now.

## Intentional Limitations

This release is marked `trial` in `zuzu-distribution.json`. The status is
validator-supported and signals that the parser is usable but not yet a
stable conformance claim.

Known gaps:

- The full html5lib tree-construction suite still has expected failures
  listed in `tests/ree-construction-xfails.zzm`.
- Parse-error count and parse-error code parity are diagnostic only.
- Script execution during parsing is not implemented.
- `HTML.load` and `HTML.dump` are not implemented.
- CSS selector support is intentionally small.
- XPath/ZPath-style `findnodes` and `findvalue` are not implemented.
- Pretty serialization is not implemented.

## html5lib Coverage

The committed harness is `tests/tree-construction.zzs`. It reads the
vendored html5lib tree-construction fixtures and runs both scripting
modes when a fixture does not specify one.

Current claimed support level:

- Parsed cases: 1,795.
- Test variants: 3,551.
- Passing non-xfailed variants: 2,531.
- Expected-failure variants: 1,020.
- TODO passes: 0.
- Unexpected failures: 0.
- Parse-error-count mismatches: diagnostic only.

### Largest Unintentional Gaps Behind Expected Failures

The current xfail manifest still uses broad reasons for many cases, but
the largest failure clusters point to these implementation issues. Counts
below are approximate variant counts from the current manifest, grouped
by the dominant fixture files that expose each behaviour.

1. Script-data tokenization edge cases, about 164 variants. The tokenizer
   does not yet fully match the HTML script data, escaped script data,
   incomplete end-tag, and EOF states. This shows up heavily in
   `tests16.dat`, `scriptdata01.dat`, and `domjs-unsafe.dat`; examples
   include preserving `</SCRIPT` as text at EOF and not prematurely
   ending `<script>` when script text contains nested-looking tags.
2. Root, head, body, and post-body insertion modes, about 148 variants.
   Several edge cases around comments, stray end tags, misplaced
   `title`, and content after `html`/`body` still use the wrong insertion
   mode or reopen the wrong element. The biggest clusters are
   `tests19.dat`, `tests1.dat`, `comments01.dat`, and `doctype01.dat`.
3. Low-level character and token edge cases, about 140 variants. NULL
   replacement/removal, carriage-return normalization, entity edge cases,
   and unquoted attributes followed by a self-closing slash are not fully
   spec-compatible. These failures are concentrated in
   `plain-text-unsafe.dat`, `webkit02.dat`, `entities01.dat`, and
   `entities02.dat`.
4. Template interactions with special insertion modes, about 80 variants.
   Template content works for the focused tests, but the full algorithm
   for `template` inside `select`, table, head, and related insertion
   modes is incomplete. The failures are concentrated in `template.dat`;
   for example `<select><template></template></select>` currently drops
   the expected template node.
5. Ruby element autoclosing and scope rules, about 32 variants. The tree
   builder does not yet implement the ruby-specific rules which close
   `rb`, `rt`, `rtc`, and related elements at the right points. These are
   tracked by `ruby.dat`; the current tree often nests the next ruby child
   where html5lib expects a sibling.

Expected failures are not skipped. They are run, compared, and reported
as TODO failures by the TAP harness. If an expected failure starts
passing, the harness reports a TODO pass so the manifest can be reduced.

## Fixture Provenance

html5lib fixtures are committed under `tests/html5lib/`.

- Source: <https://github.com/html5lib/html5lib-tests/tree/9fb614afaa42ce8787840f057b32084308e76549/tree-construction>
- Pinned commit: `9fb614afaa42ce8787840f057b32084308e76549`
- Retrieved: 2026-06-09
- Upstream licence: `tests/html5lib/LICENSE`
- Provenance note: `tests/html5lib/UPSTREAM.md`

No generated archives or caches are part of the package payload.
