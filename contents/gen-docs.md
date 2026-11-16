---
title: Generate Documentation
description: "Generate API reference docs from JavaScript sources with carne gen docs."
keywords: ["carne", "gen", "docs", "api reference", "jsdoc", "documentation"]
---

`carne gen docs` reads the current package and writes an API reference
into `carnedocs/` — a `docgen.json` feed for json-based documentations
plus a static `index.html` page. Sources are parsed with
[sweetsyntax](https://github.com/openpeeps/sweetsyntax); only exported
symbols are documented, with their JSDoc when available. Code coloring in
the HTML is baked in at generation time, so the pages need no client-side
highlighter.

## Commands

```text
carne gen docs                          # json + html into ./carnedocs/
carne gen docs --format:json            # only docgen.json
carne gen docs --format:html            # only index.html + assets/
carne gen docs --out:reference          # write into ./reference/
carne gen docs --entry:lib/util.js      # document one file only
```

Run it where a `package.json` resolves (cwd upward). Without `--entry`
every `.js`/`.mjs`/`.cjs` file counts, except tooling and non-API
directories (`node_modules`, `tests/`, `examples/`, the output dir
itself). `--format` takes `json`, `html` or `all` (the default) and
`--out` defaults to `carnedocs`.

`gen` groups generator subcommands, so future generators will slot in
beside `docs`.

## What gets documented

Both module systems work, including mixed ones:

```js
/** Says hello. */
export function hello(name = "world") {
  return "Hello, " + name + "!";
}

/** A counter. */
export class Counter {}

/** The format version. */
export const VERSION = "1.0";
```

`export {a, b as c}`, `export default`, `module.exports = {...}`,
`exports.name = ...` and `module.exports.name = ...` are all picked up;
anything not exported is skipped, even with a JSDoc block above it.
`export * from` is listed without symbols — re-exports are not followed
in v1. Names starting with `_` are still documented but flagged
`private` in the JSON and badged in the HTML.

Each symbol records its name, kind (`function`, `class`, `variable`),
file, line, a readable signature (`hello(name = "world")`), the JSDoc
fields below, and pre-highlighted HTML fragments (`signatureHtml`,
`examplesHtml`) that json-based consumers can embed directly.

## JSDoc

The leading description plus these tags are understood; everything else
is ignored:

```js
/**
 * Say hello.
 * @param {string} [name] - who to greet
 * @returns {string} the greeting
 * @example
 * hello("ada")
 */
function hello(name = "world") {
  return "Hello, " + name + "!";
}
```

`@param` accepts `{type}`, `[optional]` and `- description` parts;
`@returns` (or `@return`) takes an optional `{type}` plus text;
`@example` keeps its code verbatim and may repeat; `@deprecated`
optionally carries a migration note. Tags also work inline on one line
(`/** Farewell. @param {string} name - who leaves */`). Symbols without
JSDoc are still listed, with empty description and params.

## Output

```text
carne gen docs
```

```text
Success: documented 1 symbol from 1 file in /tmp/greet/carnedocs
```

`docgen.json` carries the package identity and the symbol list:

```json
{
  "generator": "carne gen docs",
  "package": {"name": "greet", "version": "1.2.0", "description": "Say hi"},
  "symbols": [
    {
      "name": "hello",
      "kind": "function",
      "file": "index.js",
      "line": 12,
      "signature": "hello(name = \"world\")",
      "description": "Say hello.",
      "params": [
        {"name": "name", "type": "string", "optional": true,
         "description": "who to greet"}
      ],
      "returnsType": "string",
      "returns": "the greeting",
      "examples": ["hello(\"ada\")"]
    }
  ]
}
```

`index.html` renders the same model with per-symbol signatures, param
tables, examples and source locations, styled by `assets/style.css`.
Parse failures never stop the run: the file is skipped with a warning,
and only when nothing parses at all does the command exit non-zero.
