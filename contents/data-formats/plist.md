---
title: Plist
description: "Apple property lists (XML and binary)."
keywords: ['carne', 'javascript', 'std', 'plist', 'apple']
---

Apple property lists: XML or binary in, XML out. Dictionaries become
plain objects, arrays stay arrays.

```js
const plist = require("std/plist");

const src =
  '<?xml version="1.0"?>' +
  '<plist version="1.0"><dict>' +
  '<key>CFBundleName</key><string>CarneDemo</string>' +
  '<key>CFBundleVersion</key><string>3</string>' +
  '<key>LSRequiresIPhoneOS</key><true/>' +
  '<key>URLs</key><array><string>https://a.example</string>' +
  '<string>https://b.example</string></array>' +
  '</dict></plist>';

const doc = plist.parse(src);
console.log(doc.CFBundleName); // "CarneDemo"
for (const url of doc.URLs) {
  console.log("url: " + url);
}

const back = plist.parse(plist.stringify({answer: 42, ok: true}));
console.log(back.answer); // 42, roundtrip
```


## plist.parse

`plist.parse(data) -> value (XML or binary plist)`

```js
const plist = require("std/plist");
const doc = plist.parse(src); // src above
console.log(doc.CFBundleVersion); // "3"
```

## plist.stringify

`plist.stringify(value) -> string (XML plist)`

```js
const plist = require("std/plist");
console.log(plist.stringify({name: "CarneDemo"})); // XML plist out
```
