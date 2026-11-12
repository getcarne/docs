---
title: URL
description: "URL parsing, formatting and percent-encoding."
keywords: ['carne', 'javascript', 'std', 'url']
---

RFC 3986 URL parsing and formatting. Query decodes treat `+` as space;
empty username/password/path/fragment stay absent, `port` is a number
when present. `format` also accepts `host` for `hostname` and `anchor`
(with optional leading `#`) for `fragment`.

```js
const url = require("std/url");
const u = url.parse("https://user:pw@host:8080/p?q=1&r=#frag");
console.log(url.format(u)); // "https://user:pw@host:8080/p?q=1&r=#frag"
console.log(url.stringifyQuery({a: "1", b: "x y"})); // "a=1&b=x+y"
```


## url.parse

`parse(text) -> {scheme, username?, password?, hostname, port?, path, query, fragment?, isAbsolute}`

`query` is `[{key, value}]`.

```js
const url = require("std/url");
url.parse("https://host:8080/p?q=1");
```

## url.format

`format(obj) -> string`

Inverse of `parse`. `obj.query` also accepts a string, `[[k, v]]`
pairs, or a `{k: v}` object; `port` validates as `1..65535`.

```js
const url = require("std/url");
url.format({scheme: "https", hostname: "h", path: "/p"});
```

## url.parseQuery

`parseQuery(text) -> [{key, value}]`

Raw query string (leading `?`/`#` ok) to decoded pairs.

```js
const url = require("std/url");
url.parseQuery("a=1&b=");
```

## url.stringifyQuery

`stringifyQuery([{key, value}] | [[k, v]] | {k: v} | string) -> string`

Query pairs to an encoded query string.

```js
const url = require("std/url");
url.stringifyQuery({a: "1", b: "x y"});
```

## url.encode / decode

`encode(text) -> string` / `decode(text) -> string` (plus
`encodeComponent`/`decodeComponent` aliases). Space encodes as `%20`.

```js
const url = require("std/url");
url.encode("a b/c"); // "a%20b%2Fc"
url.decode("a%20b");  // "a b"
```
