---
title: DB driver & ORM
description: "SQLite/Postgres driver plus fluent query builder, models and migrations."
keywords: ['carne', 'javascript', 'std', 'db', 'sqlite', 'postgres', 'orm']
---

Two layers: `std/db/driver` is the raw executor (sqlite3 system lib,
libpq lazily loaded); `std/db` is the JS ORM on top (query builder,
models, relations, transactions, migrations). Placeholders only (`?`
for sqlite, `$1..$n` for postgres) — values bind separately, never
interpolated. Every terminal comes in async (`Promise`) + `Sync`
twins; `Sync` twins block and return directly (for boot scripts).

```js
const db = require("std/db");
const s = await db.connect("sqlite::memory:");
const User = s.defineModel("User", {table: "users", columns: {
  id: {type: "integer", primaryKey: true, autoIncrement: true},
  name: {type: "text", notNull: true}
}});
await User.createTable();
await User.create({name: "ada"});
console.log((await User.first()).name); // "ada"
await s.close();
```


## db.connect

`db.connect(spec) -> Promise<Database>` / `db.connectSync(spec) -> Database`

`"sqlite::memory:"`, `"sqlite:/path/app.db"`, `"sqlite:relative.db"`,
`"postgres://user:pass@host:5432/dbname"`, or
`{dialect: "sqlite", path}` / `{dialect: "postgres", host?, port?,
user?, password?, database, connectTimeout?}` (host defaults to
`127.0.0.1`, port to `5432`). Same shapes on `std/db/driver`
(`driver.connect`/`connectSync` return the raw handle).

```js
const db = require("std/db");
const s = await db.connect("sqlite::memory:");
```

## handle.query / exec

`query(text[, params]) -> {rows, rowCount, changes, lastInsertRowId?}` (same for `exec`)

`params` is an array of `string|number|boolean|null` (`undefined`
throws — pass `null` for NULL). `rows` is `[{col: val}]`; BLOBs arrive
as `Uint8Array`. Available raw (`s.raw`, driver handles) and on
`Database`, each with a `Sync` twin. Invalid SQL throws before
executing.

```js
await s.exec("CREATE TABLE t(a TEXT)");
await s.query("SELECT * FROM t WHERE a > ?", ["x"]);
```

## s.table

`s.table(name) -> QueryBuilder`

Fluent builder: `where({col: val|{eq,ne,gt,gte,lt,lte,like,ilike,in,notIn}})`,
`where("raw fragment", [params])` (AND-ed in order),
`select(cols?)`, `orderBy(col[, dir])`, `limit(n)`, `offset(n)`,
`join(kind, table, on)`, `insert(row|[rows])`, `update(patch)`,
`delete()` (refuses without `WHERE`). Terminals: `all()`,
`first()`, `run()` (+ `Sync` twins); `toSQL(dialect)` previews.

```js
await s.table("users").insert([{name: "ada", age: 36}]).run();
const rows = await s.table("users").where({age: {gte: 18}})
  .orderBy("age", "desc").select(["name"]).all();
```

## s.transaction

`s.transaction(async (tx) => ...) -> fn return`

`BEGIN`/`COMMIT`, rollback on throw; nested calls use savepoints.

```js
await s.transaction(async (tx) => {
  await tx.table("t").insert({a: 1}).run();
  throw new Error("rolls back");
});
```

## s.migrate

`s.migrate(migrations) -> [appliedVersions]` (+ `migrateSync`)

`[{version: int>=1, up: "SQL"|[SQL...]}]`, sorted by version, tracked
in `_carne_migrations`; applied-once.

```js
await s.migrate([{version: 1, up: "CREATE TABLE t(a TEXT)"}]);
```

## s.defineModel

`s.defineModel(name, {table?, columns}) -> Model`

Column types: `integer|real|text|blob|boolean|datetime`, with
`primaryKey`, `autoIncrement`, `notNull`, `unique`, `default`
(exactly one primary key required).

```js
const User = s.defineModel("User", {table: "users", columns: {
  id: {type: "integer", primaryKey: true, autoIncrement: true}
}});
```

## Model

`createTable()`, `create(data)`, `find(id)`, `all()`, `first()`,
`where(cond[, params]) -> {first, all}`, `get()` (raw builder),
`hasMany(Other[, {foreignKey?}])` (adds `inst.get<Plural>()`),
`belongsTo(Other[, {foreignKey?}])` (adds `inst.get<Model>()`).
Instances carry `save()` (UPDATE by pk) and `remove()`.

```js
User.hasMany(Post);
const ana = await User.where({name: "ada"}).first();
await ana.getPosts();
ana.age = 37; await ana.save();
```

## handle.close

`close() -> true`

Idempotent; use-after-close throws.
