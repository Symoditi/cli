# Symoditi CLI

Command-line interface for the [Symoditi](https://symoditi.com) brand reputation platform.

Symoditi tracks how your brand shows up across search engines, AI assistants, news and review
sites. The CLI is the machine-readable way into all of it: every command can print JSON, every
failure is a non-zero exit code, and authentication is a pair of environment variables. That makes
it a natural fit for an AI agent working on your behalf — and perfectly pleasant to drive by hand.

> **Want an agent to set this up for you?** Copy [PROMPT.md](PROMPT.md) into Claude Code, Cursor,
> Codex, or any coding agent with shell access. It installs the CLI, gets you authenticated, and
> verifies the result.

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
# @symoditi/cli/0.1.1 darwin-arm64 node-v24.18.0
```

## Your first five minutes

Four steps take you from a fresh install to the pages driving negative sentiment about your
brand. The outputs below are illustrative, but the shapes are exactly what the API returns.

### 1. Authenticate

Mint an API key in the Symoditi app under **Settings → API Keys**
(<https://app.symoditi.com/settings/api-keys>), then save it as a named profile:

```sh
symoditi config login
```

It prompts for a profile name (default `prod`), an API URL (default `https://api.symoditi.com`)
and the key, verifies the key against the API, and only then writes `~/.symoditi/config.json`
with `600` permissions. Non-interactively — for agents, CI, or a scripted setup — pass everything
as flags:

```sh
symoditi config login --name prod --url https://api.symoditi.com --key "$SYMODITI_KEY"
```

Confirm the setup is healthy:

```sh
symoditi config doctor
# Config file: OK (/Users/you/.symoditi/config.json)
# Permissions: 600
# Profiles: prod
# Default profile: prod
# Health check: OK
```

### 2. Choose your workspace and brand

Symoditi data is scoped to a **workspace** (your organization) and a **brand** inside it. One
command lists everything your key can reach, with brands nested under each workspace:

```sh
symoditi brand-analytics get-workspaces -o json
```

```json
[
  {
    "id": "019e1234-0000-7000-8000-0000000000aa",
    "name": "Acme Inc",
    "slug": "acme",
    "role": "admin",
    "isActive": true,
    "brands": [
      { "id": "019e1234-0000-7000-8000-0000000000bb", "name": "Acme", "slug": "Acme" }
    ]
  }
]
```

Analytics commands take those two IDs as positional arguments, so keep them in shell variables:

```sh
WS=019e1234-0000-7000-8000-0000000000aa
BRAND=019e1234-0000-7000-8000-0000000000bb
```

Your account also carries a server-side **active context** — the workspace and brand the web app
opens on. Read it, and reuse it as your defaults:

```sh
symoditi auth get-app-state -o json
```

```json
{
  "activeWorkspaceId": "019e1234-0000-7000-8000-0000000000aa",
  "activeWorkspaceSlug": "acme",
  "activeBrandId": "019e1234-0000-7000-8000-0000000000bb",
  "activeBrandSlug": "Acme",
  "appState": {}
}
```

Switching it also changes what the web app opens on:

```sh
symoditi app-state update-app-state \
  --data "{\"activeWorkspaceId\":\"$WS\",\"activeBrandId\":\"$BRAND\"}"
```

### 3. Read the brand overview

The overview is the data behind the dashboard: how sentiment around your brand splits across
monitored search results, day by day.

```sh
symoditi brand-analytics get-overview "$WS" "$BRAND" \
  --dateFrom 2026-06-22 --dateTo 2026-07-22 -o json
```

```json
{
  "sentiment": {
    "graph_data": [
      { "date": "2026-07-01", "positive": 20.52, "negative": 19.65,
        "neutral": 33.21, "official": 13.23, "no_info": 13.39 },
      { "date": "2026-07-20", "positive": 9.7, "negative": 17.4,
        "neutral": 64.17, "official": 3.04, "no_info": 5.69 }
    ]
  },
  "meta": { "date_from": "2026-06-22", "date_to": "2026-07-22", "total_days": 4 }
}
```

Each number is that day's share of monitored links, in percent. The response also carries the
absolute counts behind those percentages, split by relevance. Narrow the picture with
`--countries`, `--keywords`, `--searchType`, `--mode links|news|ai` or `--monitoringId`;
`symoditi brand-analytics get-overview --help` lists every filter.

### 4. Find the pages driving negative sentiment

This is the question most people actually arrive with: *what is making us look bad, and how much
of the damage does each page account for?*

```sh
symoditi brand-analytics get-sentiment-drivers "$WS" "$BRAND" --view pool --limit 10 -o json
```

```json
{
  "view": "pool",
  "meta": {
    "group_by": "url",
    "run": { "run_date": "2026-07-20", "total_rows": 1546 },
    "total_neg_occurrences": 269
  },
  "pool": [
    {
      "normalized_url": "example-reviews.com/review/acme",
      "domain": "example-reviews.com",
      "occ_neg": 53,
      "share_of_neg_pool": 0.197
    },
    {
      "normalized_url": "reddit.com/r/example/comments/abc123/acme_thread",
      "domain": "reddit.com",
      "occ_neg": 13,
      "share_of_neg_pool": 0.048
    }
  ]
}
```

Read the first entry as: this page showed up as a negative result **53 times** across the
monitored keyword-and-engine grid, which is **19.7%** of every negative occurrence for the brand.
That ranking is your work queue — the top few entries usually account for most of the problem.

The same command has a second mode. `--view drivers` compares the two most recent monitoring runs
and reports what *changed*, which is what you want for a weekly check-in rather than a snapshot:

```sh
symoditi brand-analytics get-sentiment-drivers "$WS" "$BRAND" \
  --view drivers --direction down --limit 10 -o json
```

```json
{
  "view": "drivers",
  "meta": {
    "last_run": { "run_date": "2026-07-20", "total_rows": 1546 },
    "prev_run": { "run_date": "2026-07-17", "total_rows": 1390 },
    "net_pct_delta": -3.87
  },
  "drivers": [
    {
      "normalized_url": "example-reviews.com/review/acme",
      "domain": "example-reviews.com",
      "occ_last": 26,
      "occ_prev": 19,
      "pos_last": 0,
      "pos_prev": 19,
      "neg_last": 0,
      "neg_prev": 0,
      "contrib_delta_pp": -1.37,
      "mechanism": "receding_positive"
    }
  ]
}
```

Sentiment dropped 3.87 points between the two runs, and this page contributed 1.37 of them — not
by turning negative, but by losing the positive coverage it used to carry (`mechanism`). Pass
`--periodAFrom/--periodATo/--periodBFrom/--periodBTo` to compare arbitrary date ranges instead of
consecutive runs.

## Driving the CLI from an agent

The design goal is that an agent needs no special adapter — the CLI *is* the adapter.

- **Auth without a config file.** `SYMODITI_API_KEY` and `SYMODITI_API_URL` are read directly, so
  a container or CI job never has to run `config login`:

  ```sh
  export SYMODITI_API_KEY=symo_...
  export SYMODITI_API_URL=https://api.symoditi.com
  symoditi brand-analytics get-workspaces
  ```

  `SYMODITI_PROFILE` picks a saved profile by name instead. Prefer environment variables over the
  `--api-key` flag, which lands in shell history.
- **Structured output where it matters.** Output is a human-friendly table on a terminal and
  switches to JSON automatically when piped, so `| jq` works without passing a flag. Use
  `-o json|table|yaml` to force a format either way.
- **Exit codes mean what they should.** Any failure — unknown profile, rejected key, API error —
  exits non-zero, so `set -e` and `if ! symoditi …; then` behave the way you expect.
- **`--quiet`** suppresses progress logs and leaves only data on stdout.
- **Self-documenting.** `symoditi --help` lists topics, `symoditi <topic> --help` lists its
  commands, and `symoditi <topic> <command> --help` lists every flag — an agent can discover the
  whole surface without this README.

Beyond analytics, the CLI covers monitorings and their runs, tracked links and link rules,
keywords, probes, prompts, schedules, crawl routing, and provider inspection. Start at
`symoditi --help`.

## Update

The CLI updates itself:

```sh
symoditi update
```

It also checks for new versions in the background about once a week and prints a reminder when
one is available.

## Uninstall

```sh
rm -rf "$HOME/Library/Application Support/symoditi" \
       "$HOME/.local/bin/symoditi" "$HOME/.local/bin/sym" \
       "$HOME/.symoditi"
```

The installer puts the two symlinks in the first `PATH` directory it finds writable, so if it
reported something other than `~/.local/bin` when you installed, adjust those two paths to match.
`command -v symoditi` tells you where they actually live.

## Releases

The canonical distribution channel is `https://cli.symoditi.com` (the self-updater and the
installer both use it). GitHub Releases in this repository mirror the versioned artifacts as a
browsable archive with release notes.

## Supported platforms

macOS 13+ on Apple Silicon (`arm64`) and Intel (`x64`). Linux builds are planned.

## License

Proprietary — © Symoditi. All rights reserved. The binaries are provided for use with the
Symoditi platform. See [LICENSE](LICENSE).
