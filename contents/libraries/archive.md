---
title: Archive
description: "Archive and compression: tar, zip, rar, gzip, zlib, lz4, bzip2, brotli."
keywords: ['carne', 'javascript', 'std', 'archive', 'tar', 'zip', 'compression']
---

Synchronous archiving and compression (pure Nim, blocks like
`std/os`). Formats are case-insensitive (`".tar.gz"` works as
`"tar"` + `{compression: "gz"}`). Binary data crosses as `Uint8Array`
(use `os.writeFileBytes`/`os.readFileBytes` to persist); entry names
must be relative (no `..`, no leading `/`).

```js
const archive = require("std/archive");
const blob = archive.create("tar", [
  {name: "hello.txt", data: "hello archive"},
  {name: "data/", dir: true}
], {compression: "gz"});
console.log(archive.identify("/tmp/x.tar.gz")); // {archive: "tar", compression: "gz"}
console.log(archive.list("/tmp/x.tar.gz"));     // ["hello.txt", "data/"]
```


## archive.compress

`archive.compress(format, data[, level]) -> Uint8Array`

Compresses one blob (`gz|gzip`, `zz|zlib`, `lz4`, `bz2|bzip2`,
`br|brotli`). `data` is a string or `Uint8Array`.

```js
const archive = require("std/archive");
archive.compress("gz", "example");
```

## archive.decompress

`archive.decompress(format, data) -> Uint8Array`

```js
const archive = require("std/archive");
archive.decompress("gz", "example");
```

## archive.compressFile

`archive.compressFile(format, src, dst[, level]) -> undefined`

File-to-file compression.

```js
const archive = require("std/archive");
archive.compressFile("gz", "/tmp/a.txt", "/tmp/a.txt.gz");
```

## archive.decompressFile

`archive.decompressFile(format, src, dst) -> undefined`

```js
const archive = require("std/archive");
archive.decompressFile("gz", "/tmp/a.txt.gz", "/tmp/a.txt");
```

## archive.create

`archive.create(format, entries[, {compression}]) -> Uint8Array`

Builds `tar`/`zip`/`rar` in memory (`{compression}` makes e.g.
`tar.gz`). Entries mix `{name, data}`, `{name, path}` (on-disk file,
name defaults to basename), `{name, dir: true}`, and directories
(recurse automatically). Cap: 100,000 entries.

```js
const archive = require("std/archive");
archive.create("tar", [{name: "a.txt", data: "x"}], {compression: "gz"});
```

## archive.createFile

`archive.createFile(format, entries, out[, {compression}]) -> undefined`

Same as `create` but writes `out`.

```js
const archive = require("std/archive");
archive.createFile("zip", [{name: "a.txt", data: "x"}], "/tmp/a.zip");
```

## archive.archiveDir

`archive.archiveDir(format, srcDir, dst[, {compression, root}]) -> undefined`

Archives a whole directory tree; `root` is the virtual prefix inside
the archive (`""` by default).

```js
const archive = require("std/archive");
archive.archiveDir("tar", "/tmp/tree", "/tmp/tree.tar");
```

## archive.extract

`archive.extract(path, outDir) -> undefined`

Extracts any supported archive (compressed variants sniffed).
Zip-slip safe.

```js
const archive = require("std/archive");
archive.extract("/tmp/a.tar.gz", "/tmp/restored");
```

## archive.list

`archive.list(path) -> [name]`

Entry names as strings.

```js
const archive = require("std/archive");
archive.list("/tmp/a.zip");
```

## archive.read

`archive.read(path) -> [{name, data}]`

Reads all entries into memory; `data` is a `Uint8Array` per file.

```js
const archive = require("std/archive");
archive.read("/tmp/a.zip");
```

## archive.identify

`archive.identify(path) -> {archive, compression}`

Sniffs the format (`{archive: "tar"|"zip"|"rar"|null, compression:
"gz"|"zz"|"lz4"|"bz2"|"br"|null}`).

```js
const archive = require("std/archive");
archive.identify("/tmp/a.tar.gz");
```
