---
title: Package manager
description: "The Carne DFS package manager with one shared global store."
keywords: ["carne", "package manager", "pkg", "packages", "npm"]
---

# Package manager

A DFS (depth-first search) package manager with one shared global store
(`~/.carne`, no `node_modules`), built on
[pkg/datpkgr](https://github.com/openpeeps/datpkgr). It installs from both git
and npm-style sources.

## Commands

```text
carne pkg init [dir]
carne pkg install <spec>
carne pkg remove <name> [ver]
carne pkg update [name]
carne pkg list
carne pkg develop <path>
carne pkg prune
carne pkg refresh
carne pkg publish [opts]
```

## Scaffolding a package

```text
carne pkg init              # wizard in the current directory (must be empty)
carne pkg init <dir>        # wizard into ./<dir>/ (created when missing)
carne pkg init -Y [dir]     # skip the wizard, scaffold with defaults
```

The wizard asks for the package name (lowercase, validated before anything
is written), a semver version, description, author (defaults to the git user
name), license and keywords. `-Y` / `--yes` takes the git author, `0.1.0`,
`MIT` and `["carne"]` keywords with no prompts; without it, stdin must be a
TTY. An existing `package.json` is never overwritten.

The scaffold writes:

- `package.json` — name, version, description, license, keywords, `main`
- `index.js` — a documented `hello` starter
- `tests/<name>.test.js` — a `std/test` starter, found by `carne test`

See the [Testing](testing) page for the test runner and matchers.

## Installing from git

```text
carne pkg install <git-url>
carne pkg install gh:user/repo#v1
```

Supported sources include `https://` URLs, `git+`/`git://` transports, `git@`
and `.git` URLs, plus forge aliases (`gh:`, `github:`, `gl:`, `cb:`,
`srht:`). A `#ref` suffix pins a branch or tag.

The cloned repo must contain a valid `package.json` at its root.

## Installing from the registry

```text
carne pkg install <name>[@range]
carne pkg install npm:name@range
```

Resolved through the public registry at
[openpeeps/carnepkgs](https://github.com/openpeeps/carnepkgs) (source `carne`,
cached at `~/.carne/registries/carne.json`; `carne pkg refresh` re-downloads it).

## Package imports

Installed packages load through `require("pkg/<name>")` as CommonJS:

```js
const demo = require("pkg/carne_demo_pkg");
console.log(demo.hello("ada")); // hi ada
```

A `@version` suffix pins an installed version
(`require("pkg/greet@0.1.0")`), and `pkg/<name>/sub/path` reaches inside a
package. `./...` requires work between package files.

Only CommonJS v1 is supported: ESM (`import`/`export`) packages fail with a
clear error, as do packages with no loadable entry.
