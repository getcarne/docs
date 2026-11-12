---
title: CSV
description: "CSV parsing."
keywords: ['carne', 'javascript', 'std', 'csv']
---

# CSV

CSV parsing and serialization. With `headers: true` the first row becomes keys and the result is an
array of objects.

```js
const csv = require("std/csv");
console.log(csv.parse("a,b\n1,2\n"));                  // [["a","b"],["1","2"]]
console.log(csv.parse("a,b\n1,2\n", {headers: true})); // [{a: "1", b: "2"}]
csv.parse("a;b\n1;2\n", {delimiter: ";"});              // custom delimiter/quote
console.log(csv.stringify([["a","b"],["1","2"]]));      // "a,b\n1,2"
console.log(csv.stringify([{a: "1", b: "2"}]));         // "a,b\n1,2" (header row + rows)
```


## csv.parse

`csv.parse(text[, {headers, delimiter, quote}]) -> array`

```js
const csv = require("std/csv");
csv.parse("example", {});
```

## csv.stringify

`csv.stringify(value[, {delimiter, quote, headers}]) -> string`

Array of arrays writes rows directly; array of objects writes the first
object's keys as the header row plus one row per object. Cells coerce
(numbers/bools to text, null to `""`); fields holding the delimiter,
quote, or a newline are quoted with doubled quotes.

```js
const csv = require("std/csv");
csv.stringify("example", {});
```
