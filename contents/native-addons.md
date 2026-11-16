---
title: Native Addons
description: "Write high-performance native addons for Carne in Nim: newAddon, carne build, and the JS wrapper pattern."
keywords: ["carne", "native addon", "addon", "nim", "shared library", "newAddon", "carne build"]
---

Native addons are Nim shared libraries (`.addon`) callable from
JavaScript — the Carne answer to Node's N-API packages. A package
wraps a prebuilt binary with a JS entry that requires it relatively;
the wrapper stays plain JS while the hot path runs as compiled Nim.
Libraries load once per process (fresh exports per JS context) on the
[pluginkit](https://github.com/openpeeps/pluginkit) ABI, so any
pluginkit host can load them too.

## Requirements

Building needs carne itself as a Nim package — via nimble or clue —
plus a C toolchain behind `nim c`. Checkouts work too (`carne build`
also honors `CARNE_SRC`).

## Authoring

`carne gen addon hello` scaffolds a wrapper package
(`<name>.nimble` requiring `carne`, `index.js`, `native/<name>.nim`,
`native/build.sh`). The Nim side uses `newAddon` from
`carne/api/addon`, with the same `registerFn` header types and arity
checks as std modules:

```nim
import carne/api/addon

newAddon("hello", "0.1.0"):
  exportObject:
    registerFn hello(name: default(string, "world")):
      ## hello([name]) -> greeting string
      return toJS(ctx, "Hello, " & name & "!")
    registerFn add(a: int, b: int):
      ## add(a, b) -> number
      return toJS(ctx, a + b)
```

One addon per module. Supported parameter types: `string`, `int` /
`int64`, `float`, `bool`, `NimValue` (eager snapshot of anything),
`JSObjectRef` (live object, arrays rejected), `JSValueRef` (live ref,
unchecked) and a trailing `RestArgs` tail. `name?: Type` and
`name: default(Type, value)` mark a parameter optional. No Nim
exception may escape a callback — report failures through
`throwError` like std modules do.

## The native boundary

Type checking happens at the boundary, in Nim, so wrappers need no
manual `typeof` guards:

- Wrong argument types and wrong arity throw real JS `TypeError`s.
  Messages keep the member's `##` doc comment (rendered as JSDoc)
  after the mismatch line, so `carne run` prints full context.
- Optional parameters follow JS default-parameter semantics:
  omitted *or* explicit `undefined` takes the default.
- `carne build` compiles with `-d:carneAddon`, so addon sources can
  branch with `when defined(carneAddon)` when needed.

## Building

```text
carne build <input.nim> <output>   # output gains .addon when extensionless
```

`carne build` drives `nim c --app:lib --threads:on -d:carneAddon`.
Carne sources resolve as `CARNE_SRC` first, then the clue develop
checkout, then a nimble-installed `carne`; transitive `--path`
entries come from the nearest `clue.lock`. Without any resolvable
carne it fails closed and tells you to install it as a Nim package.

## The wrapper pattern

The `.addon` filename is platform-independent — one wrapper loads it
everywhere with no platform switch:

```js
// index.js — scalar checks live at the native boundary, so the
// wrapper only adapts shapes and composes behavior.
const native = require("./native/hello.addon");

const { hello, add } = native;

function greetAll(names) {
  if (!Array.isArray(names)) {
    throw new TypeError("greetAll: names must be an array");
  }
  return names.map((name) => hello(name));
}

module.exports = { hello, add, greetAll };
```

```js
const hello = require("./index.js");
console.log(hello.hello("ada")); // Hello, ada!
```

`require("./x.addon")` works from any JS file — package or script —
and entry scripts accept `module`/`exports` too, so a wrapper also
runs directly via `carne index.js`. `require("pkg/<name>")` flows
through unchanged: the package's JS entry pulls in its bundled
binary. `.so` / `.dylib` / `.node` files load as-is for
platform-specific layouts.

One library loads per process (`plugin_init` runs the Nim runtime a
single time); every JS context gets fresh exports. There is no
unload. Version safety is enforced at load: the loader reads the
addon's built-against carne version and errors on major mismatch
(minor drift warns). See `examples/addon/` in the carne repo for the
complete working package.
