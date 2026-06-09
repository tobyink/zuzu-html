# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project roughly adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## 0.0.1 - 2026-06-08

### Added

- Added `html/dom`, a pure ZuzuScript mutable DOM-like tree API with
  documents, elements, text, comments, doctypes, template elements,
  document fragments, traversal, mutation, attributes, namespace-aware
  SVG/MathML creation, simple selector helpers, cloning, equality, and
  compact HTML serialization.
- Added `html/parser`, including full-document parsing, fragment
  parsing, reusable `HTMLParser` instances, parse-error collection,
  strict mode, scripting-flag handling for `noscript`, tokenizer
  re-exports, and tree-builder test helpers.
- Implemented tokenizer and tree-builder support for core HTML document
  structure, in-body recovery, formatting reconstruction, adoption-agency
  recovery, tables, select, template, frameset, SVG, MathML, foreign
  attributes, integration points, and context-sensitive fragments.
- Added focused TAP suites for DOM, tokenizer, parser, tree builder,
  in-body rules, tables, select, template, frameset, foreign content,
  and fragments.
- Added the html5lib tree-construction fixture harness with a committed
  expected-failure manifest and fixture provenance.
- Added production README documentation and public module POD for the
  implemented parser and DOM APIs.

### Changed

- Set distribution status to `trial`, which is the validator-supported
  non-stable status for this release while html5lib conformance gaps
  remain tracked.
