---
title: Dotenv
description: ".env file parsing and loading."
keywords: ['carne', 'javascript', 'std', 'dotenv', 'env']
---

`.env` file parsing, exposed as `std/env` (there is no separate
`std/dotenv` module). Values support `$VAR` expansion against earlier
keys.

```js
const env = require("std/env");

const src = [
  "PORT=3000",
  "HOST=localhost:$PORT",
  "DEBUG=true",
  ""
].join("\n");

const cfg = env.parse(src);
for (const key of Object.keys(cfg)) {
  console.log(key + "=" + cfg[key]);
}
// PORT=3000
// HOST=localhost:3000
// DEBUG=true

console.log(env.stringify({PORT: "3000", HOST: "localhost"}));
env.load();            // reads ./.env into process env
env.load(".env", true); // ...overwriting existing vars
```


## dotenv.parse

`env.parse(text) -> object (with variable expansion applied)`

```js
const env = require("std/env");
const cfg = env.parse(src); // src above
console.log(cfg.HOST); // "localhost:3000"
```

## dotenv.load

`env.load([path][, override]) -> undefined (writes into process env)`

```js
const env = require("std/env");
env.load("/tmp/demo.txt", true);
```
