---
title: CSV
description: "CSV parsing."
keywords: ['carne', 'javascript', 'std', 'csv']
---

CSV parsing and serialization. With `headers: true` the first row becomes keys and the result is an
array of objects.

```js
const csv = require("std/csv");

const text = [
  "name,age,city",
  "ada,36,London",
  '"grace, h.",85,New York'
].join("\n");

const rows = csv.parse(text);
console.log(rows.length); // 3 (header + 2 data rows)
for (const row of rows.slice(1)) {
  console.log(row[0] + " is " + row[1] + " from " + row[2]);
}
// ada is 36 from London
// grace, h. is 85 from New York

const people = csv.parse(text, {headers: true});
for (const p of people) {
  console.log(p.name + " -> " + p.city);
}

console.log(csv.stringify([["a", "b"], ["1", "2"]]));
console.log(csv.stringify([{name: "ada", age: 36}, {name: "grace", age: 85}]));
console.log(csv.parse("a;b\n1;2\n", {delimiter: ";"})); // custom delimiter
```


## csv.parse

`csv.parse(text[, {headers, delimiter, quote}]) -> array`

```js
const csv = require("std/csv");
const people = csv.parse(text, {headers: true}); // text above
for (const p of people) {
  console.log(p.name + " is " + p.age);
}
```

## csv.stringify

`csv.stringify(value[, {delimiter, quote, headers}]) -> string`

Array of arrays writes rows directly; array of objects writes the first
object's keys as the header row plus one row per object. Cells coerce
(numbers/bools to text, null to `""`); fields holding the delimiter,
quote, or a newline are quoted with doubled quotes.

```js
const csv = require("std/csv");
console.log(csv.stringify(people)); // header row + one row per person
```
