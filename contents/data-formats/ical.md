---
title: iCalendar
description: "iCalendar parsing and serialization."
keywords: ['carne', 'javascript', 'std', 'ical', 'calendar']
---

iCalendar parsing and serialization. Build the source as an indented
line array joined on `\r\n`, parse it once, then walk the resulting
`events` / `todos` arrays like normal objects.

```js
const ical = require("std/ical");

const src = [
  "BEGIN:VCALENDAR",
  "VERSION:2.0",
  "PRODID:-//carne//demo//EN",
  "BEGIN:VEVENT",
  "UID:standup-1",
  "DTSTAMP:20260101T000000Z",
  "DTSTART:20260102T100000Z",
  "DTEND:20260102T101500Z",
  "SUMMARY:Team standup",
  "LOCATION:Room 3",
  "END:VEVENT",
  "BEGIN:VEVENT",
  "UID:review-2",
  "DTSTAMP:20260101T000000Z",
  "DTSTART:20260103T140000Z",
  "DTEND:20260103T150000Z",
  "SUMMARY:Design review",
  "LOCATION:Room 3",
  "END:VEVENT",
  "BEGIN:VTODO",
  "UID:todo-1",
  "DTSTAMP:20260101T000000Z",
  "DUE:20260105T170000Z",
  "SUMMARY:Write release notes",
  "END:VTODO",
  "END:VCALENDAR",
  ""
].join("\r\n");

const cal = ical.parse(src);
console.log(cal.events.length); // 2
console.log(cal.todos.length);  // 1

for (const ev of cal.events) {
  console.log(ev.dtstart.value + " - " + ev.summary + " @ " + (ev.location || "no room"));
}
// 20260102T100000Z - Team standup @ Room 3
// 20260103T140000Z - Design review @ Room 3

const inRoom3 = cal.events.filter(function (ev) { return ev.location === "Room 3"; });
console.log("in Room 3: " + inRoom3.length); // 2

for (const todo of cal.todos) {
  console.log("todo: " + todo.summary + " due " + todo.due.value);
}

console.log(ical.parse(ical.stringify(cal)).events.length); // 2, roundtrip
```

Shape: `{events, todos, journals, timezones, others}`. Absent options
stay absent; dates serialize as `{value, tzid?}`, durations as `"P1DT2H"`
strings. Unknown properties ride along in `extraProps` and survive
the roundtrip.


## ical.parse

`ical.parse(text) -> {events, todos, journals, timezones, others}`

```js
const ical = require("std/ical");
const cal = ical.parse(src); // src is the line array above
for (const ev of cal.events) {
  console.log(ev.summary + " starts " + ev.dtstart.value);
}
```

## ical.stringify

`ical.stringify(cal) -> string (a VCALENDAR document)`

```js
const ical = require("std/ical");
const text = ical.stringify(cal); // CRLF document
console.log(ical.parse(text).todos[0].summary); // "Write release notes"
```
