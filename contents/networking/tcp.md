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

(async () => {
  const srv = tcp.serve(9000, {onData: (c, m) => c.write("echo:" + m)});
  const c = await tcp.connect("127.0.0.1", 9000, {
    onData: (m) => { console.log(m); c.close(); tcp.close(srv.port); }
  });
  c.write("hello"); // client gets "echo:hello"
})();
```


## tcp.serve

`tcp.serve(port[, host], {onOpen?, onData?, onClose?, onError?, certFile?, keyFile?}) -> {port, host}`

`port` is `1..65535`; `host` defaults to `"127.0.0.1"`. Handlers are
all optional: `onOpen(conn)`, `onData(conn, chunk)`, `onClose(conn)`,
`onError(conn, {message})`. TLS servers pass `certFile` + `keyFile`
(PEM, both required together). Not supported in HTTP worker threads.

```js
const tcp = require("std/tcp");

(async () => {
  const srv = tcp.serve(19007, {onData: (c, m) => c.write("echo:" + m)});
  console.log(srv.port);
  const c = await tcp.connect("127.0.0.1", 19007, {
    onData: (m) => { c.close(); tcp.close(srv.port); }
  });
  c.write("hello"); // server echoes
})();
```

## tcp.connect

`tcp.connect(host, port[, {tls?, serverName?, rejectUnauthorized?, onOpen?, onData?, onClose?, onError?}]) -> Promise<conn>`

Resolves to the connection handle on open (after the TLS handshake
when `tls: true`); rejects on refuse, TLS failure, or close before
open. `serverName` defaults to `host`; `rejectUnauthorized` defaults
to `true` (set `false` for self-signed fixtures).

```js
const tcp = require("std/tcp");

(async () => {
  const srv = tcp.serve(19008, {onData: (c, m) => c.write("echo:" + m)});
  const c = await tcp.connect("127.0.0.1", 19008, {
    tls: false, onData: (m) => { console.log(m); c.close(); tcp.close(srv.port); }
  });
  console.log("peer " + c.remoteAddress + ":" + c.remotePort);
  c.write("hi");
})();
```

## tcp.close

`tcp.close(port) -> bool`

Closes the listener on `port` (plus its accepted connections). Throws
when nothing listens there. Safe inside a callback.

```js
const tcp = require("std/tcp");

const srv = tcp.serve(19009, {});
console.log("closed: " + tcp.close(srv.port)); // "closed: true"
```

## conn.write

`conn.write(text) -> true`

Sends on the handle from `connect`/`serve`. Throws when the
connection is closed (or pre-TLS-open).

```js
const tcp = require("std/tcp");

(async () => {
  const srv = tcp.serve(19003, {onData: (c, m) => c.write("echo:" + m)});
  const c = await tcp.connect("127.0.0.1", 19003, {
    onData: (m) => { console.log(m); c.close(); tcp.close(srv.port); }
  });
  console.log(c.write("hello")); // true, client gets "echo:hello"
})();
```

## conn.close

`conn.close() -> true`

Final and idempotent; unknown handles are a no-op. Handles also carry
`remoteAddress` and `remotePort` (server side: the peer).

```js
const tcp = require("std/tcp");

(async () => {
  const srv = tcp.serve(19004, {onData: (c, m) => {
    console.log(c.remoteAddress + " sent " + m);
    c.write("bye-ack");
  }});
  const c = await tcp.connect("127.0.0.1", 19004, {
    onData: (m) => { console.log("client closed: " + c.close()); tcp.close(srv.port); }
  });
  c.write("bye");
})();
```
