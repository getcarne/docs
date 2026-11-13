---
title: vCard
description: "vCard contact parsing."
keywords: ['carne', 'javascript', 'std', 'vcard', 'contacts']
---

vCard contact parsing and serialization. Contacts come back as objects
with `fn`, `n`, `emails[]`, `tels[]`, `nicknames[]` and `org`; parse a
whole address book at once, then loop over it.

```js
const vcard = require("std/vcard");

const src = [
  "BEGIN:VCARD",
  "VERSION:4.0",
  "FN:Ada Lovelace",
  "N:Lovelace;Ada;;;",
  "NICKNAME:Queen of Numbers",
  "EMAIL;TYPE=work:ada@example.com",
  "TEL;TYPE=voice:+1000",
  "ORG:Analytical Engines;Research",
  "END:VCARD",
  "BEGIN:VCARD",
  "VERSION:4.0",
  "FN:Grace Hopper",
  "EMAIL:grace@example.com",
  "TEL;TYPE=cell:+2000",
  "END:VCARD",
  ""
].join("\r\n");

const contacts = vcard.parse(src);
console.log(contacts.length); // 2

for (const c of contacts) {
  const email = c.emails.length > 0 ? c.emails[0].value : "no email";
  console.log(c.fn + " <" + email + ">");
  for (const t of c.tels) {
    console.log("  tel: " + t.value + " [" + t.types.join(", ") + "]");
  }
}
// Ada Lovelace <ada@example.com>
// Grace Hopper <grace@example.com>

console.log(contacts[0].n.given + " " + contacts[0].n.family); // "Ada Lovelace"
console.log(vcard.parse(vcard.stringify(contacts))[0].fn);   // roundtrip
```


## vcard.parse

`vcard.parse(text) -> contact[] (one object per VCARD)`

```js
const vcard = require("std/vcard");
const contacts = vcard.parse(src); // src is the line array above
for (const c of contacts) {
  console.log(c.fn + ": " + c.emails.length + " email(s)");
}
```

## vcard.stringify

`vcard.stringify(contacts) -> string (VCARD documents, CRLF)`

Accepts one contact or an array of contacts; absent options stay absent.

```js
const vcard = require("std/vcard");
const text = vcard.stringify(contacts);
console.log(text.includes("BEGIN:VCARD")); // true
```
