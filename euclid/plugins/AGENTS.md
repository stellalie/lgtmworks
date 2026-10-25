# Euclid plugins

> This is the plugin guide Euclid installs at `~/.config/euclid/plugins/AGENTS.md`, published for coding agents.
> Source: https://lgtm.ninja/euclid/plugins/AGENTS.md

A plugin is a folder here with a `plugin.toml` and the scripts it runs. Euclid reloads it when
you save, so write it, try it, and it's live. This file is the whole format.

## Make one (people and coding agents)

1. Make a folder: `~/.config/euclid/plugins/<id>/` (lowercase, no spaces; the id is the folder name).
2. Write `plugin.toml` and a script (any language; make it executable with `chmod +x`).
3. Try it: `/Applications/Euclid.app/Contents/MacOS/euclid-bar --try-plugin <id> [query]`.
   It runs every part once and prints the rows, the badge text or the exact error. Fix and repeat.
4. It's already live: open Euclid (⌥Space). Problems show in the footer and in `--check-config`.

## plugin.toml

```toml
name = "GitHub"                    # shown as the section title
description = "What it does, one line"

[source]                           # items in search, refreshed in the background
run = "./items.sh"                 # runs in this folder, in your login shell
refresh_minutes = 5                # 1–1440, default 10

[answer]                           # replies to what you type, below the instant results
run    = "./tz.sh"                 # gets the text as $1 (and $EUCLID_QUERY)
prefix = "tz"                      # `tz 3pm` runs it with "3pm"
match  = '^\d+(am|pm)$'            # and/or: runs on the whole query when it matches
                                   # one of the two is required; 2 s limit

[badge]                            # a short status chip on the search bar
run   = "./today.sh"               # first line of output, up to 40 characters; empty = hidden
icon  = "gauge"                    # optional SF Symbol name
refresh_minutes = 5                # default 5
click = 'ghostty-run tab ccusage'  # optional: runs when you click the badge

[[action]]                         # added to ⌘K, after your own actions
name = "Search GitHub code"
run  = 'open "https://github.com/search?type=code&q=$EUCLID_QUERY_URL"'

[settings]                         # defaults; users override in config.toml: [plugin.<id>]
limit = 30                         # scripts get $EUCLID_SETTING_LIMIT
```

Every part is optional, but a plugin needs at least one of `[source]`, `[answer]`, `[badge]`
or `[[action]]`.

## What scripts print

One JSON object per line (or a single JSON array). Only `title` is required.

```json
{"title":"Fix login on Safari","subtitle":"acme/web · #412","open":"https://github.com/acme/web/pull/412","label":"Review","suggest":true}
```

| Field | Meaning |
|---|---|
| `title` | The row's text (searched) |
| `subtitle` | Second line (searched) |
| `open` | `↵` opens this URL or file path |
| `paste` | `↵` pastes this text into the app you were in |
| `copy` | `↵` copies this text (and `⌘C` copies it) |
| `run` | `↵` runs this shell command in the plugin folder |
| `icon` | SF Symbol name, e.g. `arrow.triangle.pull` |
| `label` | Shown on the right; default: the plugin's name |
| `preview` | Text for the preview pane (`→`) |
| `id` | Stable identity, so Euclid learns what you pick; default: `open`, then `title` |
| `suggest` | `true` = also shown when the search box is empty (3 per plugin) |

`↵` does the first of `open`, `paste`, `copy`, `run` that is set. All fields are strings except
`suggest`. Print errors to stderr and exit non-zero: Euclid shows the first stderr line.

## What scripts get

- Working directory: the plugin folder. `$EUCLID_PLUGIN_DIR` is its path.
- `$1` and `$EUCLID_QUERY`: the text (answers and actions); `$EUCLID_QUERY_URL`: URL-encoded.
- `$EUCLID_SETTING_<KEY>` for each setting, uppercased.
- `~/.config/euclid/bin` first on PATH: `ghostty-run tab|split|window <cmd>` and
  `herdr-run tab|split <cmd>` open a terminal running a command.
- `[env]` from config.toml. Scripts run in a login shell, so Homebrew tools are on PATH.

The text you type is only ever passed as `$1` and environment variables, never pasted into a
command, so quote it: `"$1"`.

## Rules of thumb

- Keep sources fast and quiet: they run in the background with a 30 s limit, answers with 2 s.
- Put secrets in the Keychain and read them in the script:
  `security find-generic-password -s my-service -w`. Never in plugin.toml.
- At most 2,000 items per source.
- A team can share plugins from a repo: `[plugins] paths = ["~/work/team/euclid-plugins"]`.
- Turn one off without deleting it: `[plugins] disabled = ["<id>"]`.

## Examples

`euclid-bar --plugin add <name>` writes a working example into this folder. Read them, copy them,
change them:

- `github`: your PRs, review requests and issues, through the `gh` CLI (a source).
- `linear`: Linear issues assigned to you, through the GraphQL API (a source, key in the Keychain).
- `slack`: jump to channels, and `slack <words>` searches messages (a source and an answer).
- `tokens`: today's Claude Code tokens and cost on the bar, through `ccusage` (a badge).
- `define`: `define <word>` from the macOS Dictionary, in JavaScript for Automation (an answer).
- `currency`: `100 usd to eur` with daily rates, cached (an answer with a `match`).
- `contacts`: people in Contacts, through JavaScript for Automation (a source).
- `shortcuts`: run your Shortcuts, through the `shortcuts` CLI (a source and an action).
