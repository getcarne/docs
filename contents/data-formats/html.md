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

const doc = html.parse("<article><h1>Hi</h1><p>hello <b>ada</b> and <b>grace</b></p></article>");
console.log(html.innerText(doc)); // "Hi hello ada and grace"

for (const node of doc) {
  if (typeof node === "string") continue;
  console.log("top: " + node.tag); // "article"
  for (const child of node.children) {
    console.log("  child: " + (typeof child === "string" ? "(text) " + child : child.tag));
  }
}

console.log(html.stringify(doc)); // same markup back
console.log(html.isInline("br"), html.isBlock("div")); // true true
```


## html.parse

`html.parse(text) -> array of nodes ({tag, attrs, children}, text as strings)`

```js
const html = require("std/html");
const doc = html.parse("<p>hi <b>ada</b></p>");
for (const node of doc[0].children) {
  console.log(typeof node === "string" ? "text: " + node : "tag: " + node.tag);
}
```

## html.stringify

`html.stringify(nodes) -> string`

Accepts a node, a text string, or an array of those; void elements
(`br`, `img`, ...) emit without a close tag.

```js
const html = require("std/html");
console.log(html.stringify(doc)); // "<article>...</article>"
```

## html.innerText

`html.innerText(node) -> string`

```js
const html = require("std/html");
console.log(html.innerText(doc)); // all text, tags stripped
```

## html.isInline

`html.isInline(tagName) -> bool`

```js
const html = require("std/html");
console.log(html.isInline("span"), html.isInline("div")); // true false
```

## html.isBlock

`html.isBlock(tagName) -> bool`

```js
const html = require("std/html");
console.log(html.isBlock("div"), html.isBlock("br")); // true false
```
