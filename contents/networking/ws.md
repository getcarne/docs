---
title: WebSocket
description: "Async and sync WebSocket client and server."
keywords: ['carne', 'javascript', 'std', 'ws', 'websocket']
---

Async + sync WebSocket client and server, example:

```js
const ws = require("std/ws");

// Server (stays alive until closed, like Node)
const srv = ws.serve(8080, {
  onMessage: (conn, msg) => conn.send("hi " + msg.data)
});

// Async client (handshake on the loop; wrap in an async IIFE)
(async () => {
  const c = await ws.connect("ws://127.0.0.1:8080/chat", {
    onMessage: (msg) => {
      console.log(msg.data); // hi ada
      c.close();
      ws.close(srv.port);
    }
  });
  c.send("ada");
})();

// Sync client (blocks until open)
const c2 = ws.connectSync("ws://127.0.0.1:8080/chat");
c2.send("sync");
c2.close();
```

## connect

`connect(url[, opts])` returns a Promise for the
`{send, ping, protocol, close}` handle (`await ws.connect(...)` or
`.then(...)`).

Handshake failures reject the promise (an `onError` handler fires too
when given). Without a rejection path — pure handler use with no
await/catch — errors with no `onError` handler become the run's
uncaught error.

```js
const ws = require("std/ws");

(async () => {
  try {
    await ws.connect("ws://127.0.0.1:18599/");
  } catch (e) {
    console.log("rejected: " + String(e).split("\n")[0].slice(0, 70));
    // "rejected: Error: connect: connect() failed on all resolved addresses for 127.0.0"
  }
})();
```

## connectSync

`connectSync(url[, opts])` pumps the same loop until open (default 10s).

It blocks, so use it for scripts and fixtures rather than servers:
pumping the shared loop may run due timers and settle other in-flight
work early. Handshake failures throw synchronously.

```js
const ws = require("std/ws");

try {
  ws.connectSync("ws://127.0.0.1:18599/");
} catch (e) {
  console.log("threw: " + String(e).split("\n")[0].slice(0, 70));
  // "threw: Error: connectSync: connect() failed on all resolved addresses for 127"
}
```

## Options

Both connect forms take handshake opts in the same object as the
handlers: `{headers?, protocols?, timeoutMs?, pingIntervalMs?,
idleTimeoutMs?, maxFrameSize?, onOpen?, onMessage?, onClose?,
onError?}`. `timeoutMs` defaults to 10s; `pingIntervalMs`/`idleTimeoutMs`
default to 0 (off); `maxFrameSize` defaults to powpow's cap. `protocols`
offers subprotocols — the server picks, `protocol()` reports.

```js
const ws = require("std/ws");

(async () => {
  let done;
  const p = new Promise(r => { done = r; });
  const srv = ws.serve(18503, {onMessage: (c, m) => c.send("hi " + m.data)});
  const c = await ws.connect("ws://127.0.0.1:18503/", {
    headers: {"X-Test": "yes"}, protocols: ["chat"], timeoutMs: 5000,
    onMessage: m => {
      const pr = c.protocol(); c.close(); ws.close(srv.port); done(pr + ":" + m.data);
    }
  });
  c.send("ada");
  console.log(await p); // ":hi ada" (this server selects no subprotocol)
})();
```

## send

`send(text[, {binary}])` returns `true`. Text frames go out as strings;
`{binary: true}` sends the UTF-8 bytes as a binary frame (no TypedArrays
in v1). Either way `send` on a closed connection throws.

Messages arrive as `{data, isBinary}` — binary frames deliver their
bytes as a string with `isBinary: true`.

```js
const ws = require("std/ws");

(async () => {
  let done;
  const p = new Promise(r => { done = r; });
  const srv = ws.serve(18501, {
    onMessage: (c, m) => c.send("bin:" + m.data + ":" + m.isBinary)});
  const c = ws.connectSync("ws://127.0.0.1:18501/", {onMessage: m => {
    c.close(); ws.close(srv.port); done(m.data + ":" + m.isBinary);
  }});
  c.send("hi", {binary: true});
  console.log(await p); // "bin:hi:true:false"
})();
```

## ping

`ping([data])` sends a control ping and returns `true`; the peer's pong
is answered internally, so there is nothing to handle.

```js
const ws = require("std/ws");

(async () => {
  let done;
  const p = new Promise(r => { done = r; });
  const srv = ws.serve(18502, {onMessage: (c, m) => c.send("echo:" + m.data)});
  const c = ws.connectSync("ws://127.0.0.1:18502/", {onMessage: m => {
    c.close(); ws.close(srv.port); done(m.data);
  }});
  console.log("proto=" + JSON.stringify(c.protocol()) + " ping=" + c.ping("x"));
  // 'proto="" ping=true'
  c.send("still-open");
  console.log(await p); // "echo:still-open"
})();
```

## protocol

`protocol()` returns the negotiated subprotocol (`""` when none).

## Callbacks

Server callbacks get `(conn, payload)`, client callbacks just
`(payload)` (`onOpen` takes no arguments). Messages are `{data,
isBinary}`, closes `{code, reason}`, errors `{message}`.

`close()` is final: no `onClose` fires for locally-closed connections,
and every form of close is idempotent. Closing the server from inside a
callback is safe.

```js
const ws = require("std/ws");

(async () => {
  const srv = ws.serve(18504, "127.0.0.1", {
    onMessage: (conn, msg) => conn.send("hi " + msg.data),
    onClose: (conn, e) => console.log("server saw close " + e.code + " " + e.reason)
  });
  console.log(srv.host + ":" + srv.port); // "127.0.0.1:18504"
  const c = ws.connectSync("ws://127.0.0.1:18504/");
  c.send("ada");
  await new Promise(r => setTimeout(r, 200));
  c.close(1000, "done"); // server logs "server saw close 1000 done"
  await new Promise(r => setTimeout(r, 200));
  console.log("closed: " + ws.close(srv.port)); // "closed: true"
})();
```

## TLS

`wss://` clients use a shared TLS context with system CAs; serving
`wss://` needs cert loading and is out of scope in v1.


## conn.send

`conn.send(text[, {binary}]) -> true`

Sends on the handle from `connect`/`serve`. Throws when the connection
is closed.

```js
const ws = require("std/ws");

(async () => {
  let done;
  const p = new Promise(r => { done = r; });
  const srv = ws.serve(18501, {onMessage: (c, m) => c.send("got:" + m.data)});
  const c = ws.connectSync("ws://127.0.0.1:18501/", {onMessage: m => {
    c.close(); ws.close(srv.port); done(m.data);
  }});
  c.send("ada");
  console.log(await p); // "got:ada"
})();
```

## conn.close

`conn.close([code[, reason]]) -> true`

Final and idempotent; unknown handles are a no-op. `code` is
1000..4999 (default 1000). The pending slot releases when the close
lands (or at teardown).

```js
const ws = require("std/ws");

const srv = ws.serve(18506, {onMessage: (c, m) => c.send("hi " + m.data)});
const c = ws.connectSync("ws://127.0.0.1:18506/");
c.close(1000, "done"); // true
ws.close(srv.port);
```

## conn.ping

`conn.ping([data]) -> true`

Sends a control ping; the peer's pong is answered internally.

```js
const ws = require("std/ws");

const srv = ws.serve(18507, {onMessage: (c, m) => c.send("hi " + m.data)});
const c = ws.connectSync("ws://127.0.0.1:18507/");
console.log(c.ping("x")); // true
c.close();
ws.close(srv.port);
```

## conn.protocol

`conn.protocol() -> string`

The subprotocol the server selected (`""` when none).

```js
const ws = require("std/ws");

const srv = ws.serve(18508, {onMessage: (c, m) => c.send("hi " + m.data)});
const c = ws.connectSync("ws://127.0.0.1:18508/", {protocols: ["chat"]});
console.log(JSON.stringify(c.protocol())); // '""'
c.close();
ws.close(srv.port);
```

## ws.serve

`ws.serve(port[, host], handlers) -> {port, host}`

Handlers `{onOpen?, onMessage?, onClose?, onError?}` are all optional.
Not supported in HTTP worker threads. Closing from inside a callback is
deferred to the next loop turn, so the intro example's
`ws.close(srv.port)` inside `onMessage` is safe.

```js
const ws = require("std/ws");

const srv = ws.serve(18504, "127.0.0.1", {
  onMessage: (conn, msg) => conn.send("hi " + msg.data)
});
console.log(srv.host + ":" + srv.port); // "127.0.0.1:18504"
ws.close(srv.port);
```

## ws.connect

`ws.connect(url[, {headers?, protocols?, timeoutMs?, pingIntervalMs?, idleTimeoutMs?, maxFrameSize?, ...handlers}]) -> Promise<conn>`

The URL is `ws://` or `wss://`. Handshake failures reject; malformed
opts (a non-array `protocols`, a non-object `headers`) throw
synchronously.

```js
const ws = require("std/ws");

(async () => {
  let done;
  const p = new Promise(r => { done = r; });
  const srv = ws.serve(18503, {onMessage: (c, m) => c.send("hi " + m.data)});
  const c = await ws.connect("ws://127.0.0.1:18503/", {
    onMessage: m => { c.close(); ws.close(srv.port); done(m.data); }
  });
  c.send("ada");
  console.log(await p); // "hi ada"
})();
```

## ws.connectSync

`ws.connectSync(url[, opts]) -> established conn`

Blocks pumping the loop until the handshake completes (default 10s
`timeoutMs`), then returns the handle directly. Same opts shape as
`connect`.

```js
const ws = require("std/ws");

const srv = ws.serve(18501, {onMessage: (c, m) => c.send("hi " + m.data)});
const c = ws.connectSync("ws://127.0.0.1:18501/");
c.send("ada");
c.close();
ws.close(srv.port);
```

## ws.close

`ws.close(port) -> bool`

Stops the server on `port` (`true` when one was live). Safe inside a
callback — the close lands on the next loop turn.
