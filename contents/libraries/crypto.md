---
title: Crypto
description: "Hashing, AEAD sealing, signatures, passwords, key exchange and RSA."
keywords: ['carne', 'javascript', 'std', 'crypto', 'hash', 'signatures', 'aes', 'rsa']
---

Hashing, AEAD sealing, signatures, passwords, key exchange and public-key
encryption. Every binary value crosses the boundary as a lowercase hex
string (input accepts either case), and every key, seed, nonce, tag and
envelope is length-checked with an explicit error.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.sha256hex("abc")); // "ba7816bf8f01cfea..."

const stored = crypto.password.hash("s3cret");      // argon2id "salt:hash"
console.log(crypto.password.verify("s3cret", stored)); // true

const keys = crypto.sign.keypair(); // {publicKey, secretKey} hex (ed25519)
const sig = crypto.sign.sign(keys.secretKey, "msg");
console.log(crypto.sign.verify(keys.publicKey, "msg", sig)); // true

const keyHex = "00".repeat(32); // 32-byte key as hex
const env = crypto.aead.seal("secret", keyHex); // {nonce, mac, cipher} hex
console.log(crypto.aead.open(env, keyHex));     // "secret" (throws on MAC failure)
```

`aead.open`, `aes.open` and `rsa.decrypt` throw on failure (a bad MAC or
ciphertext is exceptional); `sign.verify`, `ecdsa.verify`, `rsa.verify`
and `password.verify` return `false` on mismatch instead. X25519/RSA/ECDSA
and AES modes are all covered below — nothing is deferred.


## hash.md5

`hash.md5(text) -> hex digest`

Legacy interop only (checksums against old systems) — never for
security. MD5 is broken for collision resistance.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.md5("abc")); // "900150983cd24fb0d6963f7d28e17f72"
```

## hash.sha256hex

`hash.sha256hex(text) -> hex digest`

The default content hash: file integrity, cache keys, content
addressing. Deterministic and fast.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.sha256hex("abc"));
// "ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad"
```

## hash.sha384hex

`hash.sha384hex(text) -> hex digest`

SHA-2 with a 384-bit digest, for protocols that mandate the wider hash.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.sha384hex("abc"));
// "cb00753f45a35e8bb5a03d699ac65007272c32ab0eded1631a8b605a43ff5bed8086072ba1e7cc2358baeca134c825a7"
```

## hash.sha512hex

`hash.sha512hex(text) -> hex digest`

The 512-bit SHA-2 digest — 128 hex chars. Useful where a longer digest
is required; otherwise prefer SHA-256.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.sha512hex("abc").length); // 128
```

## hash.blake2bhex

`hash.blake2bhex(text[, size]) -> hex digest (size 1..64, default 32)`

Fast modern hash with a tunable digest: `size` bytes out, so the hex is
twice as long. Short digests suit probabilistic structures (bloom
filters); keep 32+ bytes for security use.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.blake2bhex("abc"));
// "bddd813c634239723171ef3fee98579b94964e3bb1cb3e427262c8c068d52319"
console.log(crypto.hash.blake2bhex("abc", 16).length); // 32
```

## hash.hmacSha256hex

`hash.hmacSha256hex(keyHex, text) -> hex mac`

Keyed message authentication: proves the text came from someone holding
`keyHex` (webhook signatures, API request signing). Anyone can verify
with the same key, so keep it secret.

```js
const crypto = require("std/crypto");
console.log(crypto.hash.hmacSha256hex("6b6579", "The quick brown fox jumps over the lazy dog"));
// "f7bc83f430538424b13298e6aa6fb143ef4d59a14946175997479dbc2d1a3cd8"
```

## aead.seal

`aead.seal(text, keyHex) -> {nonce, mac, cipher}`

Seals `text` with XChaCha20-Poly1305 under a 32-byte key (64 hex chars).
Each call picks a fresh random 24-byte nonce, so sealing the same text
twice gives different envelopes. Store or transmit all three fields.

```js
const crypto = require("std/crypto");
const key = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f";
const env = crypto.aead.seal("order #42 ships at dawn", key);
console.log(env.nonce.length + "," + env.mac.length); // "48,32"
```

## aead.open

`aead.open({nonce, mac, cipher}, keyHex) -> text`

Opens an envelope sealed by `aead.seal`. Throws when the MAC check fails
(tampered ciphertext) or the key is wrong — never returns forged
plaintext.

```js
const crypto = require("std/crypto");
const key = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f";
const env = crypto.aead.seal("order #42 ships at dawn", key);
console.log(crypto.aead.open(env, key)); // "order #42 ships at dawn"
```

## sign.keypair

`sign.keypair() -> {publicKey, secretKey} hex`

Fresh random ed25519 signing pair (32-byte public, 64-byte secret key).
Publish the public key, guard the secret key.

```js
const crypto = require("std/crypto");
const keys = crypto.sign.keypair();
console.log(keys.publicKey.length + "," + keys.secretKey.length); // "64,128"
```

## sign.keypairFromSeed

`sign.keypairFromSeed(seedHex) -> {publicKey, secretKey} hex`

Deterministic pair from a 32-byte seed (64 hex chars): same seed, same
keys. Handy for tests, fixtures and deriving an identity from a master
secret.

```js
const crypto = require("std/crypto");
const seed = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f";
console.log(crypto.sign.keypairFromSeed(seed).publicKey);
// "f65333fa6303b6a23defd7de2af8aa461cb047ccbf12d4edd29ef3b1eba6706b"
```

## sign.sign

`sign.sign(secretHex, text) -> hex signature`

Signs `text` with an ed25519 secret key. Anyone with the public key can
verify; only the secret-key holder can sign (release artifacts, signed
URLs, audit log entries).

```js
const crypto = require("std/crypto");
const seed = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f";
const keys = crypto.sign.keypairFromSeed(seed);
const sig = crypto.sign.sign(keys.secretKey, "transfer $10 to ada");
console.log(crypto.sign.verify(keys.publicKey, "transfer $10 to ada", sig)); // true
```

## sign.verify

`sign.verify(publicHex, text, sigHex) -> bool`

Checks an ed25519 signature. Returns `false` — never throws — when the
text or signature does not match.

```js
const crypto = require("std/crypto");
const seed = "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f";
const keys = crypto.sign.keypairFromSeed(seed);
const sig = crypto.sign.sign(keys.secretKey, "transfer $10 to ada");
console.log(crypto.sign.verify(keys.publicKey, "transfer $11 to ada", sig)); // false
```

## password.hash

`password.hash(pw) -> "hex(salt):hex(hash)"`

Argon2id password hash with a fresh random salt per call — the same
password hashes differently every time. Store the whole string; the
salt travels with the hash so verification needs nothing else.

```js
const crypto = require("std/crypto");
const stored = crypto.password.hash("s3cret");
console.log(stored.split(":").length); // 2
```

## password.verify

`password.verify(pw, stored) -> bool`

Checks a password against a stored hash. Returns `false` for a wrong
password and for malformed stored strings (safe to pass user input
straight through).

```js
const crypto = require("std/crypto");
const stored = crypto.password.hash("s3cret");
console.log(crypto.password.verify("s3cret", stored)); // true
console.log(crypto.password.verify("wrong", stored));  // false
```

## x25519.keypair

`x25519.keypair([secretHex]) -> {secret, public} hex`

Diffie-Hellman key pair for agreeing on a shared secret without ever
sending it. With no argument a fresh random pair is made; pass a
32-byte secret to derive deterministically.

```js
const crypto = require("std/crypto");
const fixed = crypto.x25519.keypair("000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f");
console.log(fixed.public);
// "8f40c5adb68f25624ae5b214ea767a6ec94d829d3d7b5e1ad1ba6f3e2138285f"
```

## x25519.sharedKey

`x25519.sharedKey(mySecretHex, theirPublicHex[, infoHex]) -> hex`

The Diffie-Hellman shared secret: both sides compute the same value
from their own secret plus the peer's public key, then use it as a
symmetric key (e.g. for `aead.seal`). The optional `infoHex` binds the
secret to a context string.

```js
const crypto = require("std/crypto");
const alice = crypto.x25519.keypair(), bob = crypto.x25519.keypair();
const a = crypto.x25519.sharedKey(alice.secret, bob.public);
const b = crypto.x25519.sharedKey(bob.secret, alice.public);
console.log(a === b); // true
```

## aes.seal

`aes.seal(text, keyHex[, nonceHex[, adHex]]) -> {nonce, tag, cipher}`

AES-GCM with 16/24/32-byte keys (32/48/64 hex chars) — the choice for
interoperating with other platforms, databases and hardware that speak
AES. A random 12-byte nonce is picked unless `nonceHex` pins one
(deterministic vectors for tests); `adHex` adds authenticated
additional data that must match on open but is not encrypted.

```js
const crypto = require("std/crypto");
const e = crypto.aes.seal("hello", "000102030405060708090a0b0c0d0e0f", "000102030405060708090a0b", "");
console.log(e.tag + "," + e.cipher); // "892a5290e6a1cc5c447671d2fdb399f0,fb09cba209"
```

## aes.open

`aes.open({nonce, tag, cipher}, keyHex[, adHex]) -> text`

Opens an AES-GCM envelope. Throws on authentication failure (wrong key,
tampered tag/ciphertext, or mismatched additional data).

```js
const crypto = require("std/crypto");
const key = "00112233445566778899aabbccddeeff";
const box = crypto.aes.seal("session token", key);
console.log(crypto.aes.open(box, key)); // "session token"
```

## ecdsa.keypair

`ecdsa.keypair(curve) -> {privateKey, x, y, curve}`

Elliptic-curve pair on `P-256`, `P-384`, `P-521` or `secp256k1`
(coordinates are fixed-width hex: 64 chars for P-256/secp256k1, 96 for
P-384, 132 for P-521). Pick the curve your peer or chain requires —
P-256 for the web, secp256k1 for Bitcoin/Ethereum.

```js
const crypto = require("std/crypto");
const kp = crypto.ecdsa.keypair("P-256");
console.log(kp.privateKey.length + "," + kp.x.length + "," + kp.y.length); // "64,64,64"
```

## ecdsa.sign

`ecdsa.sign(privateHex, text, curve) -> hex signature (R||S)`

ECDSA signature over `text` (used for JWT/ES256, TLS, blockchains).
The output concatenates the R and S values, each coordinate-width.

```js
const crypto = require("std/crypto");
const sig = crypto.ecdsa.sign("00".repeat(31) + "01", "hello", "P-256");
console.log(sig.slice(0, 32)); // "fe485cf09056db3fec44540ce06a9abf8"
```

## ecdsa.verify

`ecdsa.verify(xHex, yHex, text, sigHex, curve) -> bool`

Checks an ECDSA signature against the signer's public coordinates.
Returns `false` on mismatch instead of throwing.

```js
const crypto = require("std/crypto");
const kp = crypto.ecdsa.keypair("P-256");
const sig = crypto.ecdsa.sign(kp.privateKey, "hello", "P-256");
console.log(crypto.ecdsa.verify(kp.x, kp.y, "hello", sig, "P-256")); // true
console.log(crypto.ecdsa.verify(kp.x, kp.y, "bye", sig, "P-256"));   // false
```

## ecdsa.sharedKey

`ecdsa.sharedKey(privHex, xHex, yHex, curve[, infoHex]) -> hex`

ECDH: same idea as `x25519.sharedKey` — both sides derive the same
secret from a private key plus the peer's public point. Use it when the
peer only speaks NIST/secp curves.

```js
const crypto = require("std/crypto");
const a = crypto.ecdsa.keypair("P-256"), b = crypto.ecdsa.keypair("P-256");
console.log(crypto.ecdsa.sharedKey(a.privateKey, b.x, b.y, "P-256") ===
  crypto.ecdsa.sharedKey(b.privateKey, a.x, a.y, "P-256")); // true
```

## rsa.keypair

`rsa.keypair([bits]) -> {privateKey, publicKey} (1024 is tests-only)`

RSA pair at 1024, 2048 (default), 3072 or 4096 bits. Key objects carry
hex `n`/`e`/`d`/`p`/`q`/`dp`/`dq`/`qinv` plus `k` and `bits`. Generate
1024-bit keys only for fast tests — ship 2048+.

```js
const rsa = require("std/crypto").rsa;
const kp = rsa.keypair(1024);
console.log(kp.privateKey.k + "," + kp.publicKey.bits); // "128,1024"
```

## rsa.sign

`rsa.sign(privateKey, text, hash[, padding]) -> hex signature`

RSA signature with `SHA-256`/`SHA-384`/`SHA-512` and `pss` (default)
or `pkcs1v15` padding — for certificates, signed tokens and interop
with RSA-based APIs.

```js
const rsa = require("std/crypto").rsa;
const kp = rsa.keypair(1024);
const sig = rsa.sign(kp.privateKey, "release v1.2.0", "SHA-256");
console.log(rsa.verify(kp.publicKey, "release v1.2.0", sig, "SHA-256")); // true
```

## rsa.verify

`rsa.verify(publicKey, text, sigHex, hash[, padding]) -> bool`

Checks an RSA signature. Padding defaults to `pss`; pass `"pkcs1v15"`
for legacy verifiers. Returns `false` on mismatch.

```js
const rsa = require("std/crypto").rsa;
const kp = rsa.keypair(1024);
const sig = rsa.sign(kp.privateKey, "release v1.2.0", "SHA-256");
console.log(rsa.verify(kp.publicKey, "release v1.2.1", sig, "SHA-256")); // false
```

## rsa.encrypt

`rsa.encrypt(publicKey, text, hash) -> hex cipher (OAEP)`

Encrypts `text` for the public-key holder with OAEP padding
(`SHA-256`/`SHA-384`/`SHA-512`; `SHA-1` is OAEP-only for decrypting
legacy traffic — never sign with it). Only small payloads fit: use it
to wrap a random symmetric key, then seal bulk data with `aead.seal`.

```js
const rsa = require("std/crypto").rsa;
const kp = rsa.keypair(1024);
const cipher = rsa.encrypt(kp.publicKey, "one-time code", "SHA-256");
console.log(typeof cipher); // "string"
```

## rsa.decrypt

`rsa.decrypt(privateKey, cipherHex, hash) -> text (OAEP)`

Opens an OAEP envelope. Throws on failure instead of returning
garbage.

```js
const rsa = require("std/crypto").rsa;
const kp = rsa.keypair(1024);
const cipher = rsa.encrypt(kp.publicKey, "one-time code", "SHA-256");
console.log(rsa.decrypt(kp.privateKey, cipher, "SHA-256")); // "one-time code"
```
