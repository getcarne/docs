---
title: TOML
description: "TOML parsing and serialization."
keywords: ['carne', 'javascript', 'std', 'toml']
---

TOML parsing and serialization. TOML datetimes surface as strings (the literal as written).

```js
const toml = require("std/toml");

const src = [
  'title = "carne demo"',
  'count = 3',
  '',
  '[server]',
  'host = "localhost"',
  'port = 3000',
  '',
  '[features]',
  'list = ["logging", "metrics"]',
  ''
].join("\n");

const cfg = toml.parse(src);
console.log(cfg.server.host + ":" + cfg.server.port); // "localhost:3000"
for (const key of Object.keys(cfg.server)) {
  console.log("server." + key + " = " + cfg.server[key]);
}
for (const f of cfg.features.list) {
  console.log("feature: " + f);
}

console.log(toml.stringify({app: {name: "carne", workers: 4}}));
```


## toml.parse

`toml.parse(text) -> object (document root must be a table)`

```js
const toml = require("std/toml");
const cfg = toml.parse(src); // src above
console.log(cfg.title); // "carne demo"
```

## toml.stringify

`toml.stringify(obj) -> string`

```js
const toml = require("std/toml");
console.log(toml.stringify(cfg)); // back to TOML text
```
