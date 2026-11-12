---
title: Installation
description: "Install Carne from GitHub Releases or build from source."
keywords: ["carne", "install", "installation", "releases", "build"]
---

## Prebuilt binaries

Prebuilt binaries for macOS and Linux are published on
[GitHub Releases](https://github.com/getcarne/carne/releases). Each version
ships `carne_<os>-<arch>.tar.gz` (`linux`/`macos` × `x86_64`/`arm64`);
Windows is not supported. The tarball contains the `carne` binary plus
`LICENSE` and `README.md`:

```bash
curl -sL https://github.com/getcarne/carne/releases/download/0.1.2/carne_macos-arm64.tar.gz -o carne.tar.gz
tar xzf carne.tar.gz
./carne_macos-arm64/carne --version
```

In GitHub Actions, the [getcarne/action](https://github.com/getcarne/action)
setup action installs Carne from the same releases (and installs the Linux
system libraries for you).

## Build from source

Requirements:

- Nim >= 2.2.0
- `nimble` (comes with Nim)
- On Linux: `libjavascriptcoregtk-4.1-dev`

```bash
nimble install carne
```
Or with `clue`:

```bash
clue install carne --build
```

Clone it, and build it via Nimble. then find the binary in `bin/carne`
```bash
git clone https://github.com/openpeeps/carne.git
cd carne && nimble install -Y
```

Verify the install:

```text
carne --version
```

## Full CLI reference

```text
usage: carne <command> [args] [options]

Scripting:
  run <script.js> [args]    run a .js file
  test [file] [...]         run tests/**/*.test.js (or the given file)

Standard library:
  std init                  download JS stdlib (std/*) into ~/.carne/std
  std update                refresh all JS stdlib files
  std list                  show installed JS stdlib version and files

Packages:
  pkg init [dir]            scaffold a new package (wizard, -Y skips it)
    --yes, -Y               skip the wizard, use defaults
  pkg install <spec>        install a package by name, git URL, gh:user/repo alias, name#ref, or name + source URL
    --url:<url>             pin a source URL
  pkg remove <name> [ver]   uninstall one version, or all
    --yes                   skip confirmation prompts
  pkg update [name]         update one package, or all
  pkg list                  list installed packages
  pkg develop [path]        editable install from a local directory
  pkg prune                 remove orphaned packages
    --yes                   skip confirmation prompts
  pkg refresh               re-download registry data
  pkg publish               open a PR adding this package to the registry
    --tags:<list>           comma-separated tags
    --web:<url>             package website URL
    --url:<url>             pin a source URL
    --dry-run               no network, git, or secret changes
    --yes                   skip confirmation prompts

Versioning:
  bump [semver]             bump version in package.json (or a direct dep)
    --level:<level>         major, minor, or patch
    --dep:<name>            bump a direct dependency spec (prefix kept)
  upgrade [version]         self-update carne from GitHub releases (into ~/.carne/bin)
    --dry-run               print resolved URLs only, no changes

Inspect:
  dump [pkg[@ref]]          print package.json (cwd or an installed package)

options:
  -h, --help                show this help
  --version                 show version
```

Scripts run with `carne run <script.js> [args]` (extra args land in
`env.args()`); test files run with `carne test` (see the
[Testing](testing) page). Packages install into one shared global store at
`~/.carne` — see the [Package manager](package-manager) page.

## Platform notes

- macOS: supported out of the box.
- Linux: requires `libjavascriptcoregtk-4.1-dev`.
- Windows: not supported yet.
