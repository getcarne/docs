---
title: Watcher
description: "File watching with event callbacks."
keywords: ['carne', 'javascript', 'std', 'watcher', 'fs', 'events']
---

File watching over the run's shared event loop. Events are
`modified`, `created`, `deleted`, `renamed`, `attrib`, `linkCount`,
`revoke`. Watches hold the loop alive until `close()` (Node
semantics). Not supported in HTTP worker threads.

```js
const watcher = require("std/watcher");
const w = watcher.watch("/tmp/log.txt", (events) => {
  console.log(events); // e.g. ["modified"]
  w.close();           // final, idempotent
});
console.log(w.path); // "/tmp/log.txt"
```


## watcher.watch

`watch(path, onEvent) -> {close(), path}`

Async watch; `onEvent(events: string[])` fires per batch. `path` must
be non-empty and watchable.

```js
const watcher = require("std/watcher");
watcher.watch("/tmp/log.txt", (events) => console.log(events));
```

## watcher.watchOnce

`watchOnce(path[, timeoutMs = 10000]) -> string[]`

Blocks until the first event batch or timeout, then returns the events
array (`timeoutMs` must be positive).

```js
const watcher = require("std/watcher");
const events = watcher.watchOnce("/tmp/log.txt", 1000);
```

## handle.close

`close() -> true`

Final and idempotent; releases the loop slot.
