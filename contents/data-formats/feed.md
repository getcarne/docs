---
title: Feed
description: "RSS and Atom feed parsing."
keywords: ['carne', 'javascript', 'std', 'feed', 'rss', 'atom']
---

RSS and Atom feed parsing and serialization.

```js
const feed = require("std/feed");
const rss = feed.parseRss('<rss version="2.0"><channel><title>T</title><item><title>Hi</title></item></channel></rss>');
console.log(rss.items.length); // 1 (feed object with items[])
console.log(feed.stringifyRss(rss)); // RSS 2.0 XML
const atom = feed.parseAtom(text); // feed object with entries[]
console.log(feed.stringifyAtom(atom)); // Atom XML
```


## feed.parseRss

`feed.parseRss(text) -> feed object with items[]`

```js
const feed = require("std/feed");
feed.parseRss("example");
```

## feed.stringifyRss

`feed.stringifyRss(feed) -> string (RSS 2.0 XML)`

```js
const feed = require("std/feed");
feed.stringifyRss("example");
```

## feed.parseAtom

`feed.parseAtom(text) -> feed object with entries[]`

```js
const feed = require("std/feed");
feed.parseAtom("example");
```

## feed.stringifyAtom

`feed.stringifyAtom(feed) -> string (Atom XML)`

```js
const feed = require("std/feed");
feed.stringifyAtom("example");
```
