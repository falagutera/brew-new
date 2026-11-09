# AGENTS.md — brew-new

Notes for coding agents (and people) changing brew-new. The [README](README.md) says what it does
and how to install it; this file says how to change it without breaking the people who installed it.

## What it is

One Bash script, `brew-new`, that lists the Homebrew formulae and casks added in a time window. It
clones `Homebrew/homebrew-core` and `homebrew-cask` (blobless, shallow), diffs the tip against the
last commit before the window, and takes descriptions (and, for `--oss`, licenses) from the
`formulae.brew.sh` JSON API and the GitHub API. It is distributed through the Homebrew formula
`brew-new` in `open/gcn/homebrew-tap`, by `curl` from the raw file, and by cloning this repo.

## Commands

```sh
bash -n brew-new                 # syntax
shellcheck brew-new              # lint (advisory in CI)
bash brew-new --version          # must print "brew-new <VERSION>"
bash brew-new bogus              # must fail with "bad period" (the formula's test relies on it)
bash brew-new 7d                 # a real run: needs git, curl, jq and network
~/storage/bin/sec-check .        # the CI security gates, locally (if you have it)
```

## Map

| File | What it holds |
|---|---|
| `brew-new` | the whole tool: options (`usage`), period parsing, tap clones (`ensure_clone`), API caches (`desc_file`), the added-file diff (`scan_tap`), GitHub license lookups (`gh_license*`, `resolve_cask_licenses`), output (`emit_group`) |
| `VERSION` | the released version, `X.Y.Z` |
| `CHANGELOG.md` | one entry per version |
| `.gitlab-ci.yml` | shared security stage + `changelog`, `test` and `shellcheck` jobs |

## Rules a change must keep

1. **`VERSION=` in `brew-new` and the `VERSION` file agree.** The `test` job checks it, and the
   Homebrew formula's test asserts `brew-new <version>` from `--version`.
2. **A bad period fails with "bad period"** and exit 1, before any network or dependency check
   (the formula's `test do` block depends on that message).
3. **Portable Bash on macOS and Linux.** No GNU-only flags, no date arithmetic (git parses
   "N weeks ago"); Bash 3.2 on macOS runs it too. Dependencies stay git, curl, jq and the usual
   coreutils; Homebrew itself is not required.
4. **Diff two real commits.** Names come from `git diff --diff-filter=A OLD HEAD`, never from the
   shallow-clone boundary (which makes every file look added).
5. **Tokens never on argv.** The GitHub token (`GITHUB_TOKEN`, `GH_TOKEN` or `gh auth token`) is
   held in `BN_TOKEN` and handed to `curl` as a config on stdin (`-K -`, see `gh_license`), so it
   does not show in `ps`.
6. **Caches live under `${XDG_CACHE_HOME:-~/.cache}/brew-new`** (plus `mktemp` files); nothing
   else is written. Failed license lookups are not cached.
7. The options in `usage`, the README's Usage block and the CHANGELOG move together.

## Releasing

- Fix = patch, new option = minor, a changed output format or removed option = major. Docs, CI
  or tests alone need no release.
- Bump `VERSION` **and** `VERSION=` in `brew-new`, add the `## X.Y.Z — YYYY-MM-DD` entry to
  `CHANGELOG.md` (the `changelog` job fails without it), commit, then tag `vX.Y.Z` and push the tag.
- The formula in `open/gcn/homebrew-tap` (`Formula/brew-new.rb`) downloads the tag tarball from
  the GitHub mirror (`github.com/falagutera/brew-new`). Once the tag is there, update `url` and
  `sha256` in the formula (`curl -fsSL <url> | shasum -a 256`) and push the tap.
- Code, comments, commits and the README in English.

## Security checks

CI runs the shared security stage from `open/gcn/ci-templates` (`security.yml`) on `main` and on
merge requests: gitleaks and trivy block; semgrep, yamllint and poutine only report (the Go and
Dockerfile jobs skip this repo). The shared shellcheck job looks only at `*.sh`, so this repo has
its own advisory `shellcheck` job for `brew-new`. The `test` and `changelog` jobs block.
