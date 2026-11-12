---
title: HTML
description: "HTML parsing and serialization."
keywords: ['carne', 'javascript', 'std', 'html']
---

HTML parsing and serialization. Elements map to `{tag, attrs, children}`
(same shape as `std/xml`); text nodes are bare strings. Comments are
dropped on parse.

```js
const html = require("std/html");
const doc = html.parse("<p>hi <b>ada</b></p>");
console.log(html.innerText(doc)); // "hi ada"
console.log(html.stringify(doc)); // "<p>hi<b>ada</b></p>"
console.log(html.isInline("br"), html.isBlock("div")); // true true
```


## html.parse

`html.parse(text) -> array of nodes ({tag, attrs, children}, text as strings)`

```js
const html = require("std/html");
html.parse("example");
```

## html.stringify

`html.stringify(nodes) -> string`

Accepts a node, a text string, or an array of those; void elements
(`br`, `img`, ...) emit without a close tag.

```js
const html = require("std/html");
html.stringify("example");
```

## html.innerText

`html.innerText(node) -> string`

```js
const html = require("std/html");
html.innerText("example");
```

## html.isInline

`html.isInline(tagName) -> bool`

```js
const html = require("std/html");
html.isInline("example");
```

## html.isBlock

`html.isBlock(tagName) -> bool`

```js
const html = require("std/html");
html.isBlock("example");
```
