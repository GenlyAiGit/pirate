# pirate

Search Pirate Bay from the terminal, select results with `fzf`, and auto-import into rTorrent.

## Dependencies

- **bash** (≥4.0 for associative arrays)
- **curl** — API requests
- **jq** — JSON parsing
- **fzf** — interactive fuzzy finder
- **numfmt** — human-readable sizes (part of GNU coreutils)
- **md5sum** — cache key hashing (GNU coreutils)
- **awk**, **stat**, **date** — standard POSIX tools

## Installation

```bash
chmod +x pirate
# optionally put it on your PATH
cp pirate ~/.local/bin/
```

## Usage

```bash
pirate <search terms>
```

1. Enter a search query — results are fetched from Pirate Bay and cached for 5 minutes.
2. An `fzf` picker opens showing torrents sorted by seeders (highest first).
3. Use **Tab** to select multiple torrents, **Enter** to confirm.
4. `.torrent` files are written to the watch directory for rTorrent to pick up.

## Configuration

All settings are at the top of the script:

| Variable | Default | Description |
|---|---|---|
| `MAX` | `100` | Maximum results to display |
| `CACHE_TTL` | `300` | Cache lifetime in seconds |
| `WATCH_DIR` env var | `~/.rtorrent/watch` | rTorrent watch directory |
| `TRACKERS` | 8 trackers | Trackers appended to magnet links |

Set `WATCH_DIR` to change the output directory:

```bash
export WATCH_DIR=/path/to/watch
pirate "ubuntu 24.04"
```

## How it works

1. Queries the unofficial `apibay.org` API.
2. Sorts results by seeders (descending).
3. Displays in `fzf` with a detail preview panel.
4. On selection, generates a bencoded `.torrent` file containing a magnet URI with trackers.
5. Places the `.torrent` file in the rTorrent watch directory for automatic import.
