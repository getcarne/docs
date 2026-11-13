---
title: XML
description: "XML parsing and serialization."
keywords: ['carne', 'javascript', 'std', 'xml']
---

XML parsing and serialization. Elements map to `{tag, attrs, children}`. Comments, prologs and doctypes
are dropped on parse.

```js
const xml = require("std/xml");

const doc = xml.parse(
  '<catalog store="main">' +
  '<book id="1" lang="en"><title>Nim in Action</title><price>30</price></book>' +
  '<book id="2"><title>JS Good Parts</title><price>25</price></book>' +
  '</catalog>');

console.log(doc.attrs.store); // "main"
for (const book of doc.children) {
  const title = book.children[0].children[0];
  const price = book.children[1].children[0];
  console.log(book.attrs.id + ": " + title + " (" + price + ")");
}
// 1: Nim in Action (30)
// 2: JS Good Parts (25)

console.log(xml.parse(xml.stringify(doc)).children.length === 2); // true, roundtrip
```


## xml.parse

`xml.parse(text) -> {tag, attrs, children}`

```js
const xml = require("std/xml");
const node = xml.parse("<note><to>Ada</to></note>");
console.log(node.children[0].children[0]); // "Ada"
```

## xml.stringify

`xml.stringify(node) -> string`

```js
const xml = require("std/xml");
console.log(xml.stringify(node)); // "<note><to>Ada</to></note>"
```
