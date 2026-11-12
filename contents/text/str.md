---
title: Strings
description: "String utilities: search, case, split, wrap, validators."
keywords: ['carne', 'javascript', 'std', 'str', 'strings']
---

Free-function string utilities: the subject string is always the first
argument. Semantics match the JS builtins wherever one exists (UTF-16
indices, locale-independent case). Argument types are checked strictly
(non-strings throw); validators return `false` on non-strings instead.

```js
const str = require("std/str");
console.log(str.camelCase("hello-world_foo")); // "helloWorldFoo"
console.log(str.slugify("Hello, World!"));     // "hello-world"
console.log(str.levenshtein("kitten", "sitting")); // 3
console.log(str.isEmail("a@b.com"), str.isPort("443")); // true true
```


## Search

| Function | Description |
|---|---|
| `indexOf(s, needle[, from]) -> number` | First UTF-16 offset at/after from, -1 when absent |
| `lastIndexOf(s, needle[, from]) -> number` | Last offset at/before from, -1 when absent |
| `includes(s, needle[, from]) -> bool` | True when needle occurs at/after from |
| `count(s, needle) -> number` | Non-overlapping occurrences; empty needle gives length + 1 |
| `equals(a, b) -> bool` | True for an exact (`===`) match |
| `startsWith(s, prefix) -> bool` | Prefix check |
| `endsWith(s, suffix) -> bool` | Suffix check |

## Case and trim

| Function | Description |
|---|---|
| `toUpperCase(s) -> string` | Upper-cases (JS semantics) |
| `toLowerCase(s) -> string` | Lower-cases (JS semantics) |
| `trim(s) -> string` | Strips leading and trailing whitespace |
| `trimStart(s) -> string` | Strips leading whitespace |
| `trimEnd(s) -> string` | Strips trailing whitespace |
| `isAscii(s) -> bool` | True when every UTF-16 unit is below 0x80 |
| `isBlank(s) -> bool` | True when empty or all whitespace |

## Split and join

| Function | Description |
|---|---|
| `lines(s) -> string[]` | Splits on `\n`, stripping one trailing `\r` per line (`""` gives `[""]`) |
| `split(s, sep) -> string[]` | Splits on a separator (`""` splits into UTF-16 units) |
| `join(parts[, sep = ","]) -> string` | Joins a string array (every element must be a string) |
| `words(s) -> string[]` | Splits on whitespace runs, dropping empties (`""` gives `[]`) |

## Replace

| Function | Description |
|---|---|
| `replace(s, search, replacement) -> string` | First occurrence (`$`-patterns stay literal) |
| `replaceAll(s, search, replacement) -> string` | Every non-overlapping occurrence |

## Shape

| Function | Description |
|---|---|
| `slice(s[, start[, end]]) -> string` | JS slice (negatives count from the end) |
| `padStart(s, width[, pad]) -> string` | Pads left to width |
| `padEnd(s, width[, pad]) -> string` | Pads right to width |
| `padCenter(s, width[, pad]) -> string` | Centers (extra padding goes right) |
| `repeat(s, n) -> string` | Repeats (native RangeError on bad counts) |
| `reverse(s) -> string` | Reverses by code points (astral characters intact) |
| `truncate(s, maxLen[, ellipsis]) -> string` | Cuts to maxLen code points, appending ellipsis when cut |
| `chomp(s) -> string` | Drops one trailing newline |
| `collapseWhitespace(s) -> string` | Collapses whitespace runs to one space, trims ends |

## Case converters

Word split is non-alphanumeric runs plus camel boundaries:

| Function | Description |
|---|---|
| `camelCase(s) -> string` | `hello-world_foo` → `helloWorldFoo` |
| `snakeCase(s) -> string` | `helloWorld` → `hello_world` |
| `kebabCase(s) -> string` | `helloWorld` → `hello-world` |
| `titleCase(s) -> string` | `hello world` → `Hello World` |
| `slugify(s) -> string` | Lower-cases, maps runs to single dashes |

## HTML, distance, layout

| Function | Description |
|---|---|
| `escapeHtml(s) -> string` | Escapes `&<>"'` to entities |
| `unescapeHtml(s) -> string` | Unescapes named/numeric entities; unknown pass through |
| `levenshtein(a, b) -> number` | Code-point edit distance (throws past 4M DP cells) |
| `wordWrap(s, width) -> string` | Greedy wrap keeping blank-line paragraphs |
| `indent(s, prefix) -> string` | Prefixes every line |
| `dedent(s) -> string` | Drops the common space/tab margin |
| `stripAnsi(s) -> string` | Drops CSI/OSC/ESC escape sequences |

## Validators

Non-strings are `false`, not throws (`isPort` also accepts numbers):

| Function | Description |
|---|---|
| `isEmail(s) -> bool` | One `@`, dot-atom local, LDH domain, < 256 chars |
| `isDomain(s) -> bool` | LDH labels, alpha TLD |
| `isIPv4(s) -> bool` | Strict dotted quad (no leading zeros) |
| `isIPv6(s) -> bool` | Strict IPv6 (8 groups or one `::`) |
| `isIP(s) -> bool` | IPv4 or IPv6 |
| `isWebUrl(s) -> bool` | Practical http(s) URL check |
| `isUUID(s) -> bool` | Canonical 8-4-4-4-12 hex |
| `isPort(v) -> bool` | Integer port 0–65535 (numeric strings ok, floats rejected) |
