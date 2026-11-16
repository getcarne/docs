---
title: HTTP Server
description: "Async HTTP server for Carne."
keywords: ['carne', 'javascript', 'std', 'http', 'server']
---

Async HTTP server on the shared event loop — handlers run on the loop
thread, no threads unless you opt into workers. `serve` borrows the loop
and holds it alive until `close`, like a long-lived process.

Built on top of [PowPow](https://github.com/openpeeps/powpow), a
high-performance event notification library for Nim.

```js
const http = require("std/http");

const srv = http.serve(3000, req => ({body: "hi " + req.path}));
http.close(srv.port);
```

## Request object

Handlers receive one argument carrying `{method, path, query, url,
headers, body, clientIp}` plus a lazy `multipart` field (below). The
object is lazy: fields convert on first access, so handlers that ignore
the request pay almost nothing — but fields first read *after* the
handler returns answer `""` (`{}` for headers), because the pooled
request is recycled. Assignments to `req` fields stick.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18401, req => ({body: JSON.stringify(
    {m: req.method, p: req.path, q: req.query, ip: req.clientIp})}));
  const res = await http.get("http://127.0.0.1:18401/a/b?x=1");
  console.log(res.body); // '{"m":"GET","p":"/a/b","q":"x=1","ip":"127.0.0.1"}'
  http.close(srv.port);
})();
```

`req.headers` names are lowercased — always read them lowercase.

## Handler return values

- a string → `200 text/plain`.
- `{status?, headers?, body?}` → status (default 200), custom headers,
  and a string body (anything else JSON-serializes). `undefined`/`null`
  → `200` with an empty body.
- a `Uint8Array` body → binary bytes, `Content-Type` defaulting to
  `application/octet-stream`.
- `{file: path, attachment?, stream?, headers?}` → zero-copy file
  serving from disk (below).
- a throw → `500`, recorded as an uncaught error (the run still exits
  non-zero — see below).

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18402, req =>
    req.path === "/created" ? {status: 201, headers: {"X-Team": "red"}, body: "made"} :
    req.path === "/empty" ? undefined :
    {status: 404, body: "nope"});
  for (const p of ["/created", "/empty", "/gone"]) {
    const r = await http.get("http://127.0.0.1:18402" + p);
    console.log(p + " -> " + r.status + " " + JSON.stringify(r.body));
  }
  http.close(srv.port);
})();
// "/created -> 201 \"made\""
// "/empty -> 200 \"\""
// "/gone -> 404 \"nope\""
```

A throw answers `500` but is also recorded as uncaught, so the process
exits `1` with the error on stderr — reserve throws for real bugs, and
return `{status: 500, ...}` for expected failures:

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18403, req => {
    if (req.path === "/boom") throw new Error("kaput");
    return "fine";
  });
  const bad = await http.get("http://127.0.0.1:18403/boom");
  console.log("boom -> " + bad.status); // "boom -> 500"
  http.close(srv.port);
})();
// stderr: Error: kaput, exit code 1
```

## Content-Type defaults

The object form picks `Content-Type` when `headers` does not set one:
`text/plain` for string bodies, `application/json` for serialized ones.
`204` (and the other null-body statuses) carries no body at all.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18411, req =>
    req.path === "/t" ? {body: "hi"} :
    req.path === "/j" ? {body: 42} : {status: 204, body: "x"});
  const t = await http.get("http://127.0.0.1:18411/t");
  const j = await http.get("http://127.0.0.1:18411/j");
  const n = await http.get("http://127.0.0.1:18411/gone");
  console.log(t.headers["content-type"]); // "text/plain; charset=utf-8"
  console.log(j.headers["content-type"]); // "application/json"
  console.log(n.status + " " + JSON.stringify(n.body)); // '204 ""'
  http.close(srv.port);
})();
```

## Binary bodies

Return `{headers?, body: Uint8Array}` to send exact bytes (NUL-safe,
unlike strings). Unset `Content-Type` defaults to
`application/octet-stream` for non-empty bodies.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18404, req => ({body: new Uint8Array([72, 105, 195, 169])}));
  const res = await http.get("http://127.0.0.1:18404/");
  console.log(res.headers["content-type"] + " | " + res.body);
  // "application/octet-stream | Hié"
  http.close(srv.port);
})();
```

## Repeated headers

Header values accept arrays of strings, sent as repeated headers (e.g.
multiple `Set-Cookie`). Anything else as a value answers `500`.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18405, req => (
    {headers: {"X-Multi": ["a", "b"]}, body: "x"}));
  const res = await http.fetch("http://127.0.0.1:18405/");
  console.log(res.headers["x-multi"]); // "b"
  http.close(srv.port);
})();
```

## Empty bodies

HEAD requests and 101/204/205/304 statuses answer with an empty body —
HEAD sends the same headers with `Content-Length: 0`.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18306, req => ({body: "never seen"}));
  const res = await http.head("http://127.0.0.1:18306/");
  console.log(res.status + " body=" + JSON.stringify(res.body)); // '200 body=""'
  http.close(srv.port);
})();
```

## File serving

`{file: path, ...}` streams the file zero-copy via sendfile — no body
buffering, no JS heap copy. It owns ETag, Last-Modified, Range/206 and
304/416 handling plus the bare Content-Type; extras in `headers` pass
through except those owned names. Missing files answer 404; a malformed
`file` value answers 500.

```js
const http = require("std/http");
const os = require("std/os");

(async () => {
  os.writeFile("/tmp/carne_docs_hello.txt", "hello\n");
  const srv = http.serve(18406, req => ({file: "/tmp/carne_docs_hello.txt"}));
  const full = await http.get("http://127.0.0.1:18406/");
  const part = await http.fetch("http://127.0.0.1:18406/",
    {headers: {Range: "bytes=0-3"}});
  console.log(full.status + " " + JSON.stringify(full.body)); // '200 "hello\n"'
  console.log(part.status + " " + JSON.stringify(part.body) +
    " " + part.headers["content-range"]); // '206 "hell" bytes 0-3/6'
  http.close(srv.port);
  os.remove("/tmp/carne_docs_hello.txt");
})();
```

`attachment: true` downloads under the file's basename, a string names
the download; `stream: true` (or a chunk size) streams media in 1 MB
chunks with a 206-first response.

```js
const http = require("std/http");
const os = require("std/os");

(async () => {
  os.writeFile("/tmp/carne_docs_report.txt", "quarterly numbers");
  const srv = http.serve(18407, req => (
    {file: "/tmp/carne_docs_report.txt", attachment: "report.txt"}));
  const res = await http.get("http://127.0.0.1:18407/");
  console.log(res.headers["content-disposition"]);
  // 'attachment; filename="report.txt"'
  http.close(srv.port);
  os.remove("/tmp/carne_docs_report.txt");
})();
```

## Multipart uploads

Powpow auto-streams `multipart/form-data` uploads to tmpfiles from the
first body byte — nothing is copied twice and binary stays binary.
`req.multipart` exposes `{fields, files}` where files carry
`{field, filename, contentType, size, path}`. Tmpfiles are deleted right
after the handler returns, so read or copy them during handling
(`os.readFileBytes` gives binary-safe Uint8Arrays). File parts need a
filename and a Content-Type header (what browsers send); filename-only
parts read as text fields. Per-file/per-field/total caps come from the
serve `maxBodySize` opt, enforced at the protocol level (413/400 before
dispatch).

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18408, req => {
    const f = req.multipart.files[0];
    return {body: f.filename + ":" + f.size +
      " fields=" + JSON.stringify(req.multipart.fields)};
  });
  const res = await http.post("http://127.0.0.1:18408/u",
    "--b26\r\nContent-Disposition: form-data; name=\"t\"\r\n\r\nhi\r\n" +
    "--b26\r\nContent-Disposition: form-data; name=\"f\"; filename=\"a.txt\"\r\n" +
    "Content-Type: text/plain\r\n\r\n0123456789\r\n--b26--\r\n",
    {headers: {"Content-Type": "multipart/form-data; boundary=b26"}});
  console.log(res.status + " " + res.body); // '200 a.txt:10 fields={"t":"hi"}'
  http.close(srv.port);
})();
```

## Request limits

`maxBodySize` caps uploads (413 past it); `keepAliveMs` and
`readTimeoutMs` tune idle sockets. Malformed opts throw synchronously.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18409, {maxBodySize: 16},
    req => ({body: "got " + req.body.length}));
  const small = await http.post("http://127.0.0.1:18409/", "tiny");
  console.log("small -> " + small.status + " " + small.body);
  // "small -> 200 got 4"
  const big = await http.post("http://127.0.0.1:18409/", "this body is way too long");
  console.log("big -> " + big.status); // "big -> 413"
  http.close(srv.port);
})();
```

## http.serve

`http.serve(port[, host][, opts], handler) -> {port, host}`

Port is 1..65535, host defaults to `127.0.0.1`. The opts object takes
`{keepAliveMs?, readTimeoutMs?, maxBodySize?, workers?}`.

```js
const http = require("std/http");

(async () => {
  const srv = http.serve(18410, "127.0.0.1", {keepAliveMs: 1000},
    req => ({body: "hi"}));
  console.log(srv.host + ":" + srv.port); // "127.0.0.1:18410"
  console.log("closed: " + http.close(srv.port)); // "closed: true"
})();
```

## http.close

`http.close(port) -> bool`

Stops the server on `port` (`true` when one was live). With workers it
stops every worker; process exit joins them.

## Workers

`workers` (1..64, default 1) scales serving across threads. Above 1,
each extra worker is a full runtime (own JS context, loop, server)
re-running the script, sharing the port (a `SO_REUSEPORT` group on
Linux, one adopted listen socket on macOS). JS heaps are isolated per
worker: no shared state, and module-level side effects run once per
worker. Gate driver code on `process.isWorker` (`false` on main,
`true` on workers); `close(port)` stops every worker and process exit
joins them. `ws.serve` is not supported in workers.

```js
const http = require("std/http");

http.serve(18412, {workers: 2}, req => "hello worker");
if (!process.isWorker) {
  setTimeout(async () => {
    const res = await http.get("http://127.0.0.1:18412/");
    console.log(res.status + " " + res.body); // "200 hello worker"
    http.close(18412);
  }, 500);
}
```
