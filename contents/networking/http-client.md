---
title: HTTP Client
description: "Async HTTP client for Carne."
keywords: ['carne', 'javascript', 'std', 'http', 'client']
---

The HTTP client is fully async and Promise-based. `fetch`, `get`,
`post`, `put`, `patch`, `delete`, `head` and `options` each return a
Promise that resolves to `{status, statusText, headers, body}`. Client
and server multiplex on one event loop together with the timers — truly
async, no threads.

Built on top of [PowPow](https://github.com/openpeeps/powpow), a
high-performance event notification library for Nim.

Scripts need an async wrapper for `await` (top-level await is out of
scope in v1):

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18301, req => ({body: req.method + " " + req.path}));
  const res = await http.get("http://127.0.0.1:18301/hello");
  console.log(res.status + " " + res.body); // "200 GET /hello"
  const posted = await http.post("http://127.0.0.1:18301/items", "ping");
  console.log(posted.status + " " + posted.body); // "200 POST /items"
  http.close(srv.port);
})();
```

## Response shape

Every method resolves to the same object: numeric `status`, `statusText`
(`"OK"`, `"Created"`, …), `headers` with lowercased names (read
`res.headers["content-type"]`, never the original case), and the body as
a string. Bodies arrive as raw wire bytes decoded lossy (U+FFFD), so any
payload converts instead of rejecting the fetch.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18302, req => (
    {status: 201, headers: {"X-Team": "red"}, body: "ok"}));
  const res = await http.fetch("http://127.0.0.1:18302/");
  console.log(res.status + " " + res.statusText); // "201 Created"
  console.log(res.headers["x-team"] + " | " + res.body); // "red | ok"
  http.close(srv.port);
})();
```

## Client methods

- `fetch(url[, {method, headers, body, timeoutMs}])` — full control.
- `get`/`delete`/`head`/`options` take `(url[, {headers, timeoutMs}])`;
  `method`/`body` are rejected there, use `fetch` instead.
- `post`/`put`/`patch` take `(url, body[, {headers, timeoutMs}])`.
- `timeoutMs` aborts a slow request; network failures reject the
  Promise while bad option shapes throw synchronously.

## http.fetch

`http.fetch(url[, {method, headers, body, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18303, req => ({body: req.method + ":" + req.body}));
  const res = await http.fetch("http://127.0.0.1:18303/",
    {method: "PUT", body: "v2", headers: {"X-A": "1"}});
  console.log(res.status + " " + res.body); // "200 PUT:v2"
  http.close(srv.port);
})();
```

## http.get

`http.get(url[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18304, req => ({body: "q=" + req.query}));
  const res = await http.get("http://127.0.0.1:18304/search?x=1&y=2");
  console.log(res.body); // "q=x=1&y=2"
  http.close(srv.port);
})();
```

## http.post

`http.post(url, body[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18301, req => ({body: req.method + ":" + req.body}));
  const res = await http.post("http://127.0.0.1:18301/items", "ping");
  console.log(res.status + " " + res.body); // "200 POST:ping"
  http.close(srv.port);
})();
```

## http.put

`http.put(url, body[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18303, req => ({body: req.method + ":" + req.body}));
  const res = await http.put("http://127.0.0.1:18303/items/1", "v2");
  console.log(res.status + " " + res.body); // "200 PUT:v2"
  http.close(srv.port);
})();
```

## http.patch

`http.patch(url, body[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18305, req => ({body: req.method + ":" + req.body}));
  const res = await http.patch("http://127.0.0.1:18305/items/1", "fix");
  console.log(res.status + " " + res.body); // "200 PATCH:fix"
  http.close(srv.port);
})();
```

## http.delete

`http.delete(url[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18305, req => ({body: req.method + ":" + req.path}));
  const res = await http.delete("http://127.0.0.1:18305/items/1");
  console.log(res.status + " " + res.body); // "200 DELETE:/items/1"
  http.close(srv.port);
})();
```

## http.head

`http.head(url[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}> (body is always "")`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18306, req => ({body: "never seen"}));
  const res = await http.head("http://127.0.0.1:18306/");
  console.log(res.status + " body=" + JSON.stringify(res.body)); // "200 body=\"\""
  http.close(srv.port);
})();
```

## http.options

`http.options(url[, {headers, timeoutMs}]) -> Promise<{status, statusText, headers, body}>`

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18305, req => ({body: req.method}));
  const res = await http.options("http://127.0.0.1:18305/items");
  console.log(res.status + " " + res.body); // "200 OPTIONS"
  http.close(srv.port);
})();
```

## Errors

Connection failures reject the Promise (catch them like any async
error); malformed options — a `method` on `get`, a non-string header —
throw synchronously before anything hits the wire.

```js
const http = require("std/http");

(async () => {
  try {
    await http.get("http://127.0.0.1:18399/", {timeoutMs: 500});
  } catch (e) {
    console.log("rejected: " + String(e).split("\n")[0].slice(0, 60));
    // "rejected: Error: get: connect() failed on all resolved addresses for 1"
  }
})();
```
