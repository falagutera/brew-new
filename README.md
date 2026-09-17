# brew-new

List the Homebrew **formulae** and **casks added** in a time window — in the
familiar `==> New Formulae` / `==> New Casks` style, with descriptions.

Homebrew has no built-in "recently added" command. `brew-new` reads the git
history of the `homebrew-core` / `homebrew-cask` taps and diffs two **real**
commits (the tip vs. the last commit before the window). Descriptions come from
the official JSON API (`formulae.brew.sh`) — the same source `brew` uses.

```console
$ brew-new 2w -c
==> New Casks
bitmuse: Bit-perfect music player for local hi-res libraries
distill-web-monitor: Monitor webpages for changes
gitdifferent: Git client built around a three-way merge tool
network-weather: Network diagnostics tool
picmal: Converts and compresses images, video, audio and PDFs locally
tungsten-edge: Window-oriented taskbar that replaces the Dock
...
```

## Install

### Homebrew (tap)

```sh
brew tap gutocarvalho/tap https://gitlab.gutocarvalho.net/open/homebrew-tap.git
brew trust gutocarvalho/tap
brew install brew-new
```

`brew trust` is required once because Homebrew 7+ won't load formulae from a
third-party tap until you trust it.

### Manual

It's a single Bash script. Drop it on your `PATH`:

```sh
curl -fsSL https://gitlab.gutocarvalho.net/open/brew-new/-/raw/main/brew-new \
  -o ~/.local/bin/brew-new && chmod +x ~/.local/bin/brew-new
```

Or clone and symlink:

```sh
git clone https://gitlab.gutocarvalho.net/open/brew-new.git
ln -sf "$PWD/brew-new/brew-new" ~/.local/bin/brew-new
```

**Requirements:** `git`, `curl`, standard `awk`/`sed`/`sort`/`grep`. `jq` only
for `--json`. Homebrew itself is **not** required. Works on macOS and Linux.

## Usage

```
brew-new [PERIOD] [options]

PERIOD   7d | 2w | 3mo | 1y   (a bare number = days; default 30d)

  -p, --period P     time window (7d | 2w | 3mo | 1y)
  -d, --days N       time window in days
  -f, --formulae     only formulae (homebrew-core)
  -c, --casks        only casks (homebrew-cask)
  -a, --all          both (default)
  -g, --filter RE    keep only names matching regex RE (case-insensitive)
      --names        just the names (no descriptions), column layout
      --dates        append the date each entry was added
      --json         emit JSON  [{name,type,date,desc}]
      --csv          emit CSV   name,type,date,desc
  -r, --refresh      force fresh clone + description refresh
  -h, --help         help
  -v, --version      version
```

### Examples

```sh
brew-new                      # brew-style, everything new in the last 30 days
brew-new 2w -c                # casks added in the last 2 weeks
brew-new 3mo -g 'ai|mcp|agent'   # AI/agent stuff from the last 3 months
brew-new 1y --csv > new.csv   # a year of additions as CSV
brew-new -f --dates           # formulae, with the date each landed
```

## How it works

1. **Names** — a blobless, shallow clone (`--filter=blob:none --shallow-since`)
   of each tap, then `git diff --diff-filter=A OLD HEAD` where `OLD` is the last
   commit before the window. Diffing two real trees avoids the classic
   shallow-clone trap where every file looks "added" at the graft boundary.
2. **Dates** — walked from the in-window commit range (`OLD..HEAD`), which has
   real parents, so per-commit added-file detection is correct there.
3. **Descriptions** — the reduced `name → desc` map from
   `formulae.brew.sh/api/{formula,cask}.json`, cached locally.

Caches live under `${XDG_CACHE_HOME:-~/.cache}/brew-new` (tap clones refreshed
every 6h, descriptions every 24h; `--refresh` forces both).

## License

MIT — see [LICENSE](LICENSE).
