---
title: Env
description: "Process info and environment variables."
keywords: ['carne', 'javascript', 'std', 'env', 'process', 'environment']
---

```js
const env = require("std/env");
console.log(env.args());            // e.g. ["carne", "app.js", "--port", "3000"]
console.log(env.getenv("HOME"));    // string | undefined
console.log(env.parse("A=1\nB=x $A\n")); // {A: "1", B: "x 1"} (expansion applied)
console.log(env.stringify({A: "1", B: "x y"})); // "A=1\nB=\"x y\"\n"
env.load();                         // reads ./.env into process env
console.log(env.cwd());             // current directory
env.exit(2);                        // terminate with exit code 2
```


## env.args

`env.args() -> process.argv as a fresh array`

```js
const env = require("std/env");
env.args();
```

## env.parse

`env.parse(text) -> object (with variable expansion applied)`

```js
const env = require("std/env");
env.parse("example");
```

## env.stringify

`env.stringify(obj) -> string (dotenv text, double-quoted when needed)`

```js
const env = require("std/env");
env.stringify("example");
```

## env.load

`env.load([path][, override]) -> undefined (writes into process env)`

```js
const env = require("std/env");
env.load("example", "example");
```

## env.getenv

`env.getenv("HOME") -> string | undefined`

```js
const env = require("std/env");
env.getenv("key");
```

## env.cwd

`env.cwd() -> string`

```js
const env = require("std/env");
env.cwd();
```

## env.exit

`env.exit(2) -> terminates the run with exit code 2`

```js
const env = require("std/env");
env.exit(1000);
```
