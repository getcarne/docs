---
title: NanoID
description: "URL-friendly unique ID generation."
keywords: ['carne', 'javascript', 'std', 'nanoid', 'id']
---

URL-friendly unique IDs. `size` must be `1..1024`, else the call
throws. `urlAlphabet` is the 64-char `A-Za-z0-9-_` default;
`defaultSize` is `21`.

```js
const {nanoid} = require("std/nanoid");
console.log(nanoid());    // "V1StGXR8_Z5jdHi6B-myT" (21 chars)
console.log(nanoid(10));  // 10 chars from the URL alphabet
```


## nanoid

`nanoid([size = 21]) -> string`

URL-safe id string.

```js
const {nanoid} = require("std/nanoid");
nanoid(10);
```

## customAlphabet

`customAlphabet(alphabet[, size = 21]) -> string`

Id string drawn from `alphabet`.

```js
const id = require("std/nanoid");
id.customAlphabet("abc123", 8);
```

## nanoidNonSecure

`nanoidNonSecure([size = 21]) -> string`

Same shape, non-secure PRNG (DOM keys, throwaway ids).

```js
const id = require("std/nanoid");
id.nanoidNonSecure();
```
