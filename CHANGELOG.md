# Changelog

All notable changes to brew-new. Versions follow [SemVer](https://semver.org/); each release is
a `vX.Y.Z` tag, and the Homebrew formula in `open/gcn/homebrew-tap` installs that tag.

## Unreleased

- Code clean-up so shellcheck passes; no change in behavior.

## 1.2.0 — 2026-11-09

What brew-new provides today (1.2.0 added `--oss`; this file starts here):

- Lists the Homebrew formulae and casks added in a time window (`7d`, `2w`, `3mo`, `1y`; default
  30 days), in the `==> New Formulae` / `==> New Casks` style, with descriptions from the
  official Homebrew API.
- Filters: only formulae or only casks, a name regex, and `--oss` for open-source entries only
  (formulae by SPDX license, casks by the license of their GitHub repository).
- Output as names only, with the date each entry was added, JSON or CSV.
- Caches tap clones, descriptions and license lookups under `~/.cache/brew-new`; `--refresh`
  forces a fresh fetch.
- Bash, git, curl and jq; works on macOS and Linux without Homebrew installed.
