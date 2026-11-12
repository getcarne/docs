---
title: UDP
description: "Async UDP sockets."
keywords: ['carne', 'javascript', 'std', 'udp', 'networking']
---

Async UDP sockets over the run's shared event loop. Messages arrive as
`{data}` (UTF-8 string) with sender info as `{address, port}`. Sockets
hold the loop alive until `close()` (Node semantics). No DTLS in v1.

```js
const udp = require("std/udp");
const s = udp.bind(9001, (msg, rinfo) => s.sendTo("pong:" + msg.data, rinfo));
const c = udp.connect("127.0.0.1", 9001, (msg) => {
  console.log(msg.data); c.close(); s.close();
});
c.send("ping"); // client gets "pong:ping"
```


## udp.bind

`udp.bind(port[, host], onMessage) -> socket`

Listens for datagrams; `port 0` binds an ephemeral port (read it back
off `sock.port`). `onMessage(msg, rinfo)` is required. Bind sockets
must use `sendTo`, not `send`.

```js
const udp = require("std/udp");
const s = udp.bind(9001, (msg, rinfo) => s.sendTo(msg.data, rinfo));
const eph = udp.bind(0, (msg) => console.log(msg.data));
```

## udp.connect

`udp.connect(host, port[, onMessage]) -> socket`

Connected socket with `onMessage(msg[, rinfo])` (optional). Supports
both `send(text)` (to the peer) and `sendTo(...)` (any target).

```js
const udp = require("std/udp");
const c = udp.connect("127.0.0.1", 9001, (msg) => console.log(msg.data));
```

## sock.sendTo

`sock.sendTo(text, rinfo) | sock.sendTo(text, host, port) -> true`

Sends to `{address, port}` or a positional host/port pair.

```js
s.sendTo("hi", {address: "127.0.0.1", port: 9001});
s.sendTo("hi", "127.0.0.1", 9001);
```

## sock.send

`sock.send(text) -> true`

Connected (`connect`) sockets only; `bind` sockets must use `sendTo`.

```js
c.send("ping");
```

## sock.close

`sock.close() -> true`

Final and idempotent; unknown handles are a no-op. Sockets also carry
the local `address`/`port` (`bind`) or the remote peer (`connect`).
