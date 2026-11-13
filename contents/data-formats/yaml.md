---
title: YAML
description: "YAML parsing and serialization."
keywords: ['carne', 'javascript', 'std', 'yaml']
---

YAML parsing and serialization. Single-document in v1: trailing documents after `---` are ignored.

```js
const yaml = require("std/yaml");

const src = [
  "server:",
  "  host: localhost",
  "  port: 8080",
  "features:",
  "  - logging",
  "  - metrics",
  "retries: 3",
  ""
].join("\n");

const config = yaml.parse(src);
console.log(config.server.host); // "localhost"
for (const f of config.features) {
  console.log("feature: " + f);
}

const emitted = yaml.stringify({app: "carne", debug: false, tags: ["js", "nim"]});
console.log(emitted);
console.log(yaml.parse(emitted).app); // "carne", roundtrip
```


## yaml.parse

`yaml.parse(text) -> value (objects, arrays, scalars)`

```js
const yaml = require("std/yaml");
const doc = yaml.parse("name: ada\ntags:\n  - nim\n  - js\n");
for (const t of doc.tags) {
  console.log("tag: " + t);
}
```

## yaml.stringify

`yaml.stringify(value) -> string (single YAML document)`

```js
const yaml = require("std/yaml");
console.log(yaml.stringify({name: "ada", retries: 3}));
```
