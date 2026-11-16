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

const s = udp.bind(19002, (msg, rinfo) => s.sendTo(msg.data, rinfo));
const eph = udp.bind(0, (msg) => console.log(msg.data));
console.log("ephemeral port: " + (eph.port > 0));
eph.close();
s.close();
```

## udp.connect

`udp.connect(host, port[, onMessage]) -> socket`

Connected socket with `onMessage(msg[, rinfo])` (optional). Supports
both `send(text)` (to the peer) and `sendTo(...)` (any target).

```js
const udp = require("std/udp");

const s = udp.bind(19007, (msg, rinfo) => s.sendTo("pong:" + msg.data, rinfo));
const c = udp.connect("127.0.0.1", 19007, (msg) => {
  console.log(msg.data); c.close(); s.close();
});
c.send("ping"); // client gets "pong:ping"
```

## sock.sendTo

`sock.sendTo(text, rinfo) | sock.sendTo(text, host, port) -> true`

Sends to `{address, port}` or a positional host/port pair.

```js
const udp = require("std/udp");

const s = udp.bind(19005, (msg, rinfo) => s.sendTo("back:" + msg.data, rinfo));
const c = udp.connect("127.0.0.1", 19005, (msg) => {
  console.log(msg.data); c.close(); s.close();
});
c.sendTo("hi", {address: "127.0.0.1", port: 19005});
c.sendTo("hi", "127.0.0.1", 19005); // positional form
```

## sock.send

`sock.send(text) -> true`

Connected (`connect`) sockets only; `bind` sockets must use `sendTo`.

```js
const udp = require("std/udp");

const s = udp.bind(19006, (msg, rinfo) => s.sendTo("pong:" + msg.data, rinfo));
const c = udp.connect("127.0.0.1", 19006, (msg) => {
  console.log(msg.data); c.close(); s.close();
});
c.send("ping"); // client gets "pong:ping"
```

## sock.close

`sock.close() -> true`

Final and idempotent; unknown handles are a no-op. Sockets also carry
the local `address`/`port` (`bind`) or the remote peer (`connect`).

```js
const udp = require("std/udp");

const s = udp.bind(0, (msg) => {});
console.log("ephemeral port: " + (s.port > 0));
console.log("closed: " + s.close() + "," + s.close()); // idempotent
```
