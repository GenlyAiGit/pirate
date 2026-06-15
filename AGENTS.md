# AGENTS.md

## Project Overview

**pirate** is a single-file bash script (`pirate`) that searches Pirate Bay via the unofficial `apibay.org` API, presents results in an interactive `fzf` picker, and writes `.torrent` files to an rTorrent watch directory.

## Tech

- **Language**: Bash (single script, no build system)
- **Dependencies**: `curl`, `jq`, `fzf`, `md5sum`, `numfmt`, `awk`, `stat`
- **API**: `https://apibay.org/q.php` (unofficial Pirate Bay proxy)

## Key Variables

| Variable | Default | Purpose |
|---|---|---|
| `MAX` (hardcoded) | 100 | Max results to display |
| `CACHE_TTL` (hardcoded) | 300 | Cache expiry in seconds |
| `CACHE_DIR` | `~/.cache/pirate` | API response cache location |
| `WATCH_DIR` | `~/.rtorrent/watch` | Output directory for `.torrent` files |
| `TRACKERS` (hardcoded array) | 8 trackers | Trackers appended to magnet links |

## Architecture

1. **Cache check** — Query is MD5-hashed; if a cached API response exists and is <5 min old, it's reused.
2. **API fetch** — `curl` to `apibay.org/q.php` with a spinner animation.
3. **Parse & sort** — `jq` sorts results by seeders (desc), limits to `$MAX`, formats as TSV.
4. **Preview setup** — Generates a temp preview script used by `fzf`.
5. **Picker** — `fzf --multi` with preview window. User tabs to select, enter to confirm.
6. **Output** — For each selected hash, a bencoded `.torrent` file with magnet URI + trackers is written to `$WATCH`.

## Notable Technical Details

- `set -euo pipefail` is set — any command failure exits the script. This affects how arithmetic/increment operators behave.
- `((++count))` (pre-increment) is used intentionally over `((count++))` (post-increment) because the latter returns exit code 1 when count is 0, which would kill the script under `set -e`.
- Trackers are URL-encoded with `jq -sRr @uri` and appended to magnet links.
- Torrent files use the bencode format (`d10:magnet-uri<len>:<magnet>e`).

## Known Issues / Technical Debt

- Some hardcoded tracker domains are defunct (`coppersurfer.tk`, `leechers-paradise.org`, `9.rarbg.to`).
- `md5sum` is GNU-specific; macOS uses `md5` — script is Linux-only.
- No dependency validation at startup (will fail with cryptic errors if `jq`/`fzf`/`numfmt` are missing).
- `apibay.org` is an unofficial proxy and is frequently blocked or taken down — no fallback API.
- Spinner animation is slow (1s per full cycle).
- No `--help` flag beyond the usage error message.
- No tests or CI.

## Assumptions

- Linux environment with GNU coreutils.
- `jq` must be installed.
- `fzf` must be installed.
- `numfmt` (from GNU coreutils) must be installed.
- rTorrent watch directory exists or can be created.
