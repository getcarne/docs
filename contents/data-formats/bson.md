---
title: BSON
description: "BSON parsing and serialization."
keywords: ['carne', 'javascript', 'std', 'bson']
---

BSON parsing and serialization. Raw BSON bytes travel with an opaque mapping (do not read the string
as text); `stringify` output always parses back.

```js
const bson = require("std/bson");

const doc = {
  hello: "world",
  n: 42,
  active: true,
  nothing: null,
  tags: ["a", "b"],
  nested: {x: 1}
};

const blob = bson.stringify(doc);
console.log("bytes: " + blob.length);

const back = bson.parse(blob);
for (const key of Object.keys(back)) {
  console.log(key + " = " + JSON.stringify(back[key]));
}
console.log(JSON.stringify(back) === JSON.stringify(doc)); // true, roundtrip
```


## bson.parse

`bson.parse(data) -> value`

```js
const bson = require("std/bson");
const back = bson.parse(blob); // blob above
console.log(back.hello); // "world"
```

## bson.stringify

`bson.stringify(value) -> data (opaque, do not read as text)`

```js
const bson = require("std/bson");
console.log(typeof bson.stringify({n: 1})); // "string" (opaque bytes)
```
