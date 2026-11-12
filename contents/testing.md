---
title: Testing
description: "Write and run tests with carne test and the std/test module."
keywords: ["carne", "test", "testing", "std/test", "expect", "assert"]
---

Tests live in `tests/**/*.test.js` and run with `carne test`. The
`std/test` module provides `test`/`it`, `describe`, `expect` matchers and a
Node-style `assert` — no dependencies needed.

## Running tests

```text
carne test                  # every tests/**/*.test.js, sorted
carne test local            # tests/local.test.js (name resolves under tests/)
carne test tests/foo.test.js
```

Each file runs in a fresh runtime, so state never leaks between files. A
failing run exits non-zero; passing output looks like:

```text
# tests/local.test.js
ok - local basic write/read/exists
pass 1/1 tests
Success: pass 1/1 files
```

Failures print as `fail - <name>` with the message indented below, followed
by a `fail x/y tests` summary. With no arguments and no `tests/` directory
(or no `*.test.js` files in it), `carne test` reports that instead of
passing silently.

## Writing tests

Registration is synchronous; execution is deferred until the file finishes
evaluating, so this works under both `carne run` and `carne test`. Test
functions may return a promise — async cases are awaited serially, one per
timer tick, so servers can open, serve and close in order.

```js
const { test, expect } = require("std/test");
const { hello } = require("../index.js");

test("greets by name", () => {
  expect(hello("ada")).toBe("Hello, ada!");
});
```

`it(name, fn)` is an alias of `test`. `describe(name, fn)` groups cases
under a shared `parent > child > case` prefix. Describe bodies run
immediately and must stay synchronous (grouping only, no assertions); a
throwing body becomes one failing `(setup)` case.

```js
const { describe, test, expect } = require("std/test");

describe("limiter", () => {
  test("passes under the limit", async () => {
    expect((await get("/data")).status).toBe(200);
  });

  test("limits over the quota", async () => {
    expect((await get("/data")).status).toBe(429);
  });
});
```

## expect matchers

| Matcher | Meaning |
|---|---|
| `toBe(v)` | Identity (`Object.is`: `NaN` equals `NaN`) |
| `toEqual(v)` | Structural equality (arrays, objects, `Date`, `RegExp`) |
| `toBeTruthy()` / `toBeFalsy()` | Truthiness |
| `toBeNull()` / `toBeUndefined()` / `toBeDefined()` / `toBeNaN()` | Nullish / NaN checks |
| `toContain(x)` | Array member (`Object.is`) or string substring |
| `toMatch(s\|re)` | String contains substring or matches `RegExp` |
| `toThrow([s\|re\|ctor])` | Function throws (optional message / pattern / class filter) |
| `toBeGreaterThan(n)` / `toBeLessThan(n)` | Numeric comparison |
| `toBeGreaterThanOrEqual(n)` / `toBeLessThanOrEqual(n)` | Numeric comparison |
| `toBeCloseTo(n[, precision])` | Float proximity (default 2 decimals) |
| `.not` | Negates any matcher: `expect(x).not.toBe(y)` |

```js
expect([0, 1, 2]).toEqual([0, 1, 2]);
expect("hello static\n").toMatch(/static/);
expect(() => JSON.parse("{oops")).toThrow("Unexpected token");
expect(r.headers["retry-after"] !== undefined).toBe(true);
```

## assert

Node-style alternative to `expect`. The bare call asserts truthiness with an
optional message override:

```js
const { assert } = require("std/test");

assert(body.length > 0, "body is not empty");
assert.ok(ip.length > 0);
assert.equal(code, 200);            // loose ==
assert.strictEqual(status, 200);    // Object.is
assert.deepEqual(order, ["a", "b"]);
assert.throws(() => JSON.parse("{oops"));
assert.doesNotThrow(() => JSON.parse("{}"));
assert.fail("unreachable");
```

## stats

`stats()` returns the runner counters once the deferred run settles:
`{total, passed, failed, done}`.

## Scaffolding

`carne pkg init -Y` generates a starter `tests/<name>.test.js` using
`std/test`, so new packages are test-ready. See the
[Package manager](package-manager) page.
