# Phase 2 Report: Input Stream And Tokenizer

## Summary

Phase 2 adds a pure ZuzuScript tokenizer layer for the `html`
distribution. The new `html/tokenizer` module provides an input stream,
token objects, parse-error objects, named character-reference helpers,
and a reusable tokenizer. It emits tokenizer-level tokens only; DOM tree
construction and `HTML.parse` document output remain intentionally
unimplemented until Phase 3.

## Completed Scope

- Added `HTMLInputStream` with CRLF/CR newline normalization, source
  offset tracking, line/column tracking, `consume`, `reconsume`, `peek`,
  `match`, and `eof`.
- Added `HTMLToken` for `DOCTYPE`, `StartTag`, `EndTag`, `Comment`,
  `Character`, and `EOF` tokens, including ordered attributes,
  self-closing flags, doctype identifiers, force-quirks, attribute
  lookup, and debug strings.
- Added `HTMLParseError` with code, message, line, column, offset,
  tokenizer state, and stringification.
- Added `HTMLTokenizer` with `tokenize`, `nextToken`, `errors`,
  `state`, `setState`, `setLastStartTagName`, `lastStartTagName`, and
  `reset`.
- Implemented focused Phase 2 tokenizer behaviour for data, RCDATA,
  RAWTEXT, script data, PLAINTEXT, tags, attributes, comments, bogus
  comments, doctypes, character references, EOF handling, parse errors,
  duplicate attributes, and reusable/resettable operation.
- Updated `html/parser` to re-export tokenizer classes while preserving
  the parser placeholder behaviour.
- Added `tests/html/tokenizer.zzs` with focused tokenizer coverage.

## API Added

- Module: `html/tokenizer`
- Classes: `HTMLTokenizer`, `HTMLToken`, `HTMLParseError`,
  `HTMLInputStream`, `HTMLNamedCharacterReferences`
- Main-module re-exports from `html/parser`: the same five tokenizer
  classes above
- Metadata: `zuzu-distribution.json` now declares the `std/string`
  dependency used by the DOM and tokenizer modules.

## Commands Run

```bash
env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/tokenizer.zzs
# pass: 1..87

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/dom.zzs
# pass: 1..114

env ZUZULIB=tobyink-dists/html/modules:stdlib/modules zuzu-perl/bin/zuzu tobyink-dists/html/tests/html/parser.zzs
# pass: 1..8

git -C tobyink-dists/html diff --check
# pass: no output
```

## Spec Compliance Notes

- Tokenization is deterministic and stateful enough for Phase 3 tree
  builder integration: tokens expose structured data rather than debug
  strings, parse errors are ordered and non-fatal, and tokenizer state
  can be set/reset externally.
- Character tokens are coalesced for contiguous runs of text. This is
  tokenizer-equivalent for tree construction because the tree builder can
  append the character data in order.
- End tags with attributes and trailing solidus are emitted as end-tag
  tokens while recording the expected parse errors.
- Duplicate attributes record `duplicate-attribute`; the first value is
  preserved and later duplicates are discarded.
- CDATA/foreign-content handling is not complete in Phase 2 because the
  tokenizer does not yet receive namespace/tree-builder context.

## Character Reference Table Status

The named character-reference table is partial. It covers `amp`, `lt`,
`gt`, `quot`, `apos`, `nbsp`, `copy`, `reg`, and `not`, including
semicolon forms and legacy no-semicolon forms needed by the focused
Phase 2 tests. Numeric decimal and hexadecimal references are
implemented, including null, surrogate, out-of-range, missing semicolon,
and Windows-1252 C1 replacement handling.

## Exit Criteria Assessment

- Focused tests for comments, doctypes, start/end tags, attributes, raw
  text, RCDATA, character references, and EOF handling: met by
  `tests/html/tokenizer.zzs`.
- Tokenizer state transitions needed by the tree builder are exposed:
  met through `state`, `setState`, `setLastStartTagName`,
  `lastStartTagName`, and `reset`.
- Parse errors include useful context: met through `HTMLParseError`
  code, state, line, column, and offset.

## Known Limitations

- No tree construction or insertion-mode parser exists yet.
- `HTML.parse`, `HTMLParser.parse`, fragment parsing, load, and dump
  remain placeholders.
- The named character-reference table is partial and must be expanded
  before full html5lib compliance work.
- Script-data escaped and double-escaped substates are represented by a
  practical Phase 2 script-data scanner that preserves comment-like text
  and finds the appropriate `</script>` end tag; it is not yet a full
  state-by-state WHATWG implementation.
- No html5lib `.dat` harness exists until Phase 8.
