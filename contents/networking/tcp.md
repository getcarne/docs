---
title: TCP
description: "Async TCP client and server."
keywords: ['carne', 'javascript', 'std', 'tcp', 'networking', 'tls']
---

Async TCP client and server over the run's shared event loop. All
payloads are UTF-8 strings in v1. Servers hold the loop alive until
`close()` (Node semantics); throws become `CarneError`.

```js
const tcp = require("std/tcp");
const srv = tcp.serve(9000, {onData: (c, m) => c.write("echo:" + m)});
const c = await tcp.connect("127.0.0.1", 9000, {
  onData: (m) => { console.log(m); c.close(); tcp.close(srv.port); }
});
c.write("hello"); // client gets "echo:hello"
```


## tcp.serve

`tcp.serve(port[, host], {onOpen?, onData?, onClose?, onError?, certFile?, keyFile?}) -> {port, host}`

`port` is `1..65535`; `host` defaults to `"127.0.0.1"`. Handlers are
all optional: `onOpen(conn)`, `onData(conn, chunk)`, `onClose(conn)`,
`onError(conn, {message})`. TLS servers pass `certFile` + `keyFile`
(PEM, both required together). Not supported in HTTP worker threads.

```js
const tcp = require("std/tcp");
tcp.serve(9000, {onData: (c, m) => c.write("echo:" + m)});
```

## tcp.connect

`tcp.connect(host, port[, {tls?, serverName?, rejectUnauthorized?, onOpen?, onData?, onClose?, onError?}]) -> Promise<conn>`

Resolves to the connection handle on open (after the TLS handshake
when `tls: true`); rejects on refuse, TLS failure, or close before
open. `serverName` defaults to `host`; `rejectUnauthorized` defaults
to `true` (set `false` for self-signed fixtures).

```js
const tcp = require("std/tcp");
const c = await tcp.connect("127.0.0.1", 9000, {
  tls: true, rejectUnauthorized: false, onData: (m) => console.log(m)
});
```

## tcp.close

`tcp.close(port) -> bool`

Closes the listener on `port` (plus its accepted connections). Throws
when nothing listens there. Safe inside a callback.

```js
const tcp = require("std/tcp");
tcp.close(9000);
```

## conn.write

`conn.write(text) -> true`

Sends on the handle from `connect`/`serve`. Throws when the
connection is closed (or pre-TLS-open).

```js
c.write("hello");
```

## conn.close

`conn.close() -> true`

Final and idempotent; unknown handles are a no-op. Handles also carry
`remoteAddress` and `remotePort` (server side: the peer).
