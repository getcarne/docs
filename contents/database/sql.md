---
title: SQL Validator
description: "SQL parsing and validation."
keywords: ['carne', 'javascript', 'std', 'sql', 'database']
---

SQL parsing, validation and serialization.

```js
const sql = require("std/sql");
const ast = sql.sqlite.parse("SELECT a FROM t"); // AST object ({kind, value?, quote?, children?})
console.log(sql.sqlite.stringify(ast)); // "SELECT a FROM t" (normalized)
console.log(sql.sqlite.format("select a from t")); // normalized SQL
console.log(sql.postgres.validate("SELECT $1"));      // {valid: true}
console.log(sql.mysql.validate("SELECT `a` FROM t")); // {valid: true}
```

One sub-namespace per dialect: `sqlite`, `postgres` (= `pgsql`), `mysql`.
Each has `parse(sql)` (throws a JS `Error` on syntax errors),
`stringify(ast)` and `format(sql)` (normalized SQL), and
`validate(sql)` (never throws, reports `{valid: false, error}` instead).


## sqlite.parse

`sqlite.parse(sql) -> AST object`

```js
const sql = require("std/sql");
sql.sqlite.parse("SELECT 1");
```

## sqlite.stringify

`sqlite.stringify(ast) -> string`

```js
const sql = require("std/sql");
sql.sqlite.stringify("example");
```

## sqlite.format

`sqlite.format(sql[, {upperCase}]) -> string (normalized)`

```js
const sql = require("std/sql");
sql.sqlite.format("example", {});
```

## sqlite.validate

`sqlite.validate(sql) -> {valid, error?}`

```js
const sql = require("std/sql");
sql.sqlite.validate("SELECT 1");
```

## postgres.parse

`postgres.parse(sql) -> AST object`

```js
const sql = require("std/sql");
sql.postgres.parse("SELECT 1");
```

## postgres.stringify

`postgres.stringify(ast) -> string`

```js
const sql = require("std/sql");
sql.postgres.stringify("example");
```

## postgres.format

`postgres.format(sql[, {upperCase}]) -> string (normalized)`

```js
const sql = require("std/sql");
sql.postgres.format("example", {});
```

## postgres.validate

`postgres.validate(sql) -> {valid, error?}`

```js
const sql = require("std/sql");
sql.postgres.validate("SELECT 1");
```

## mysql.parse

`mysql.parse(sql) -> AST object`

```js
const sql = require("std/sql");
sql.mysql.parse("SELECT 1");
```

## mysql.stringify

`mysql.stringify(ast) -> string`

```js
const sql = require("std/sql");
sql.mysql.stringify("example");
```

## mysql.format

`mysql.format(sql[, {upperCase}]) -> string (normalized)`

```js
const sql = require("std/sql");
sql.mysql.format("example", {});
```

## mysql.validate

`mysql.validate(sql) -> {valid, error?}`

```js
const sql = require("std/sql");
sql.mysql.validate("SELECT 1");
```
