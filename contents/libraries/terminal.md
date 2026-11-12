---
title: Terminal
description: "Terminal styling, size, reads and CLI arg parsing."
keywords: ['carne', 'javascript', 'std', 'terminal', 'colors', 'args']
---

Styling, size, raw output, interactive reads, and CLI arg parsing.
Colors reach live terminals only (unless the CLI `--isatty` flag says
otherwise — see `colorEnabled()`); no env vars involved. `write`,
`readLine` and `readPassword` block the loop, so they suit scripts,
not servers.

```js
const t = require("std/terminal");
console.log(t.green("ok")); // green on TTYs, plain on pipes
console.log(t.columns(), t.rows()); // e.g. 80 24
const p = t.parseArgs(["--port", "3000"], {string: ["port"]});
console.log(p.port); // "3000" (values never coerce)
```


## Stylers

`green`, `red`, `yellow`, `blue`, `magenta`, `cyan`, `gray`, `bold`,
`dim`, `underline` — each `(text) -> string`, plain passthrough when
colors are off, one reset at the end (no nesting).

```js
const t = require("std/terminal");
t.red("x"); t.bold("x"); t.stripAnsi(t.red("x")); // "x"
```

## terminal.colorEnabled

`colorEnabled() -> bool`

Whether styling emits ANSI codes (`true` under `--isatty:true`).

```js
const t = require("std/terminal");
t.colorEnabled();
```

## terminal.stripAnsi

`stripAnsi(text) -> string`

Drops CSI escape sequences.

## terminal.write

`write(text) -> undefined`

Stdout write, no trailing newline.

## terminal.isTTY

`isTTY() -> bool`

Whether stdout is a terminal.

## terminal.columns / rows

`columns() -> int` / `rows() -> int`

Terminal size (80/24 when unknown).

## terminal.readLine

`readLine([prompt]) -> string | null`

One line, or null on EOF; blocks the loop.

## terminal.readPassword

`readPassword([prompt]) -> string | null`

Password with no echo; null on EOF; blocks the loop.

## terminal.parseArgs

`parseArgs([argv][, {boolean?, string?, alias?, default?, positional?}]) -> {_: [], ...}`

Parses CLI args; values never coerce (strings stay strings, flags are
booleans, repeats last-win). `argv` defaults to the script's own args
(past `carne script.js`). `--key val` / `-k val` consume the next
token unless it looks like a flag; `--no-x` reads `false`; `-abc`
splits into flags; `--` ends flag parsing; declared-but-absent
booleans read `false`. `alias` maps `{name: string|string[]}`,
`default` fills `{k: string|bool|number}`, `positional` names leading
positionals.

```js
const t = require("std/terminal");
t.parseArgs(["--port", "3000", "-v", "--", "--notflag"],
  {boolean: ["v"], string: ["port"]});
// {port: "3000", v: true, _: ["--notflag"]}
```
