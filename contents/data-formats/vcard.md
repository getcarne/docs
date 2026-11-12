---
title: vCard
description: "vCard contact parsing."
keywords: ['carne', 'javascript', 'std', 'vcard', 'contacts']
---

vCard contact parsing and serialization.

```js
const vcard = require("std/vcard");
const contacts = vcard.parse("BEGIN:VCARD\r\nVERSION:4.0\r\nFN:Ada\r\nEND:VCARD\r\n");
console.log(contacts.length); // 1 (one object per VCARD)
console.log(vcard.stringify(contacts)); // VCARD documents (CRLF)
```


## vcard.parse

`vcard.parse(text) -> contact[] (one object per VCARD)`

```js
const vcard = require("std/vcard");
vcard.parse("example");
```

## vcard.stringify

`vcard.stringify(contacts) -> string (VCARD documents, CRLF)`

Accepts one contact or an array of contacts; absent options stay absent.

```js
const vcard = require("std/vcard");
vcard.stringify("example");
```
