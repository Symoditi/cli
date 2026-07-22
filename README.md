# Symoditi CLI

Command-line interface for the [Symoditi](https://symoditi.com) brand reputation platform.

This repository hosts installation instructions and a versioned archive of CLI releases. The CLI
source lives in a private monorepo; binaries are distributed as standalone tarballs with an
embedded Node.js runtime — **no prerequisites on your machine**.

## Install (macOS)

Apple Silicon and Intel are both supported:

```sh
curl -fsSL https://cli.symoditi.com/install | sh
```

The installer:

- downloads the latest `stable` build from `cli.symoditi.com`;
- unpacks it into `~/Library/Application Support/symoditi` (user-writable — no `sudo`, ever);
- symlinks two equivalent binaries, `symoditi` and the short alias `sym`, into a `PATH`
  directory (falls back to `~/.local/bin`; the installer prints a `PATH` hint if needed).

Verify:

```sh
symoditi --version
```

## Update

The CLI updates itself:

```sh
symoditi update
```

It also checks for new versions in the background about once a week and prints a reminder when
one is available.

## Getting started

You need a Symoditi API key (ask your Symoditi workspace administrator). Then:

```sh
symoditi config login     # save the API key as a named profile
symoditi config doctor    # verify config, permissions and connectivity
```

Explore from there — every command and topic is self-documenting:

```sh
symoditi --help
symoditi providers list
sym monitorings list      # `sym` is the same binary
```

Global flags on every command: `--profile` to switch between saved profiles, and
`--output json|table|yaml` for scripting.

## Uninstall

```sh
rm -rf "$HOME/Library/Application Support/symoditi" \
       "$HOME/.local/bin/symoditi" "$HOME/.local/bin/sym" \
       "$HOME/.config/symoditi"
```

## Releases

The canonical distribution channel is `https://cli.symoditi.com` (the self-updater and the
installer both use it). GitHub Releases in this repository mirror the versioned artifacts as a
browsable archive with release notes.

## Supported platforms

macOS 13+ on Apple Silicon (`arm64`) and Intel (`x64`). Linux builds are planned.

## License

Proprietary — © Symoditi. All rights reserved. The binaries are provided for use with the
Symoditi platform. See [LICENSE](LICENSE).
