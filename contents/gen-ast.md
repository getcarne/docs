---
title: Generate AST
description: "Dump a JavaScript AST to JSON or binary .ast with carne gen ast."
keywords: ["carne", "gen", "ast", "sweetsyntax", "javascript", "fbe"]
---

`carne gen ast <file>` parses one JavaScript file with
[sweetsyntax](https://github.com/openpeeps/sweetsyntax) and writes its
abstract syntax tree beside the input (or into `--out`): `<base>.json`
(minified JSON, the same shape sweetsyntax's own `ast` command prints)
or `<base>.ast` (a compact binary framing built on
[openparser/fbe](https://github.com/openpeeps/openparser)). One output
per run; `--format` picks `json` (the default) or `bin`.

## Commands

```text
carne gen ast app.js                   # app.json beside app.js
carne gen ast app.js --format:bin      # app.ast instead
carne gen ast app.js --out:dump        # write into ./dump/
```

Only JavaScript counts
(`.js`/`.mjs`/`.cjs`/`.jsx`); anything else exits non-zero, as do a
missing file and a parse failure.

```text
carne gen ast greet.js --out:out
```

```text
Success: wrote AST of greet.js (2 top-level nodes): greet.json in /tmp/astdoc/out
```

## JSON

Every node records its kind, source position and payload. `const x = 1;`
dumps as one minified line:

```json
{"nodes":[{"kind":"nkStatement","ln":1,"col":1,"children":[{"kind":"nkIdent","ln":0,"col":0,"name":"const"},{"kind":"nkIdentDefs","ln":0,"col":0,"children":[{"kind":"nkIdent","ln":1,"col":7,"name":"x"},{"kind":"nkEmpty","ln":0,"col":0},{"kind":"nkLitInt","ln":1,"col":11,"valInt":1}]}]},{"kind":"nkEmpty","ln":1,"col":12}]}
```

Leaf payloads keep their kind: `valBool`, `valInt`, `valFloat`,
`valStr`, `valBigInt`, `valImag` and `name` for identifiers.
Statements, calls, functions, blocks and literals nest under
`children`.

## Binary .ast

The `.ast` file starts with the magic `CAST`, then an FBE root struct
(version 1; field 1 is the source path, field 2 the top-level nodes).
Each node is an inner struct holding the kind byte, line, column, an
optional leaf payload and an optional children vector. Unknown field
ids are skipped so newer writers stay readable; magic and version are
checked strictly. For the `greet.js` above the two outputs are close in
size (1295 vs 1212 bytes): JSON for inspection, `.ast` for
field-addressed machine reads without parsing text.
