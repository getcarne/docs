---
title: Feed
description: "RSS and Atom feed parsing."
keywords: ['carne', 'javascript', 'std', 'feed', 'rss', 'atom']
---

RSS and Atom feed parsing and serialization. Parse once, then iterate
`items` (RSS) or `entries` (Atom) like any array.

```js
const feed = require("std/feed");

const rssText = [
  '<?xml version="1.0"?>',
  '<rss version="2.0"><channel>',
  '<title>Demo Blog</title>',
  '<link>http://blog.example</link>',
  '<description>Posts and updates</description>',
  '<item><title>First post</title><link>http://blog.example/1</link></item>',
  '<item><title>Second post</title><link>http://blog.example/2</link></item>',
  '</channel></rss>'
].join("\n");

const rss = feed.parseRss(rssText);
console.log(rss.title + " (" + rss.items.length + " items)"); // Demo Blog (2 items)
for (const item of rss.items) {
  console.log(" - " + item.title + " -> " + item.link);
}
console.log(feed.parseRss(feed.stringifyRss(rss)).title); // roundtrip

const atomText = [
  '<?xml version="1.0"?>',
  '<feed xmlns="http://www.w3.org/2005/Atom">',
  '<id>urn:demo</id><title>Demo Feed</title>',
  '<updated>2026-01-01T00:00:00Z</updated>',
  '<author><name>Ann</name></author>',
  '<entry><id>urn:demo:1</id><title>Hello Atom</title>',
  '<updated>2026-01-02T00:00:00Z</updated>',
  '<link href="http://blog.example/a1"/></entry>',
  '<entry><id>urn:demo:2</id><title>Second entry</title>',
  '<updated>2026-01-03T00:00:00Z</updated>',
  '<link href="http://blog.example/a2"/></entry>',
  '</feed>'
].join("\n");

const atom = feed.parseAtom(atomText);
console.log(atom.title.value + " by " + atom.authors[0].name); // Demo Feed by Ann
for (const entry of atom.entries) {
  console.log(" - " + entry.title.value + " (" + entry.links[0].href + ")");
}
console.log(feed.parseAtom(feed.stringifyAtom(atom)).id); // "urn:demo", roundtrip
```


## feed.parseRss

`feed.parseRss(text) -> feed object with items[]`

```js
const feed = require("std/feed");
const rss = feed.parseRss(rssText); // rssText above
for (const item of rss.items) {
  console.log(item.title + " -> " + item.link);
}
```

## feed.stringifyRss

`feed.stringifyRss(feed) -> string (RSS 2.0 XML)`

```js
const feed = require("std/feed");
console.log(feed.stringifyRss(rss).includes("<title>Demo Blog</title>")); // true
```

## feed.parseAtom

`feed.parseAtom(text) -> feed object with entries[]`

```js
const feed = require("std/feed");
const atom = feed.parseAtom(atomText); // atomText above
for (const entry of atom.entries) {
  console.log(entry.id + ": " + entry.title.value);
}
```

## feed.stringifyAtom

`feed.stringifyAtom(feed) -> string (Atom XML)`

```js
const feed = require("std/feed");
console.log(feed.stringifyAtom(atom).includes("<feed")); // true
```
