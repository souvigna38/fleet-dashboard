# Fleet Dashboard (Backup)

Static GitHub Pages site that mirrors the Flowboard `github-fleet` board.
Use this when [Flowboard](https://task-swarm-keeper.lovable.app) is down or
unreachable.

## Live site

**https://souvigna38.github.io/fleet-dashboard/**

## How it works

1. Each machine runs `fleet-flowboard-sync` every 15 min (launchd agent).
2. That script reads the live git state of every repo, pushes cards to the Flowboard board, **and** commits a `dashboard.json` snapshot to this repo.
3. GitHub Pages serves `index.html` which fetches `dashboard.json` and renders the same health-column view as Flowboard.

## Data freshness

The `generated_at` field at the top of `dashboard.json` (and in the page header) shows when the snapshot was last written. Each machine's sync agent writes its own cards to the snapshot, so the freshness is bounded by the slowest machine's 15-min launchd interval.

## Health columns

| Column     | Meaning                                                          |
|------------|------------------------------------------------------------------|
| `clean`    | Working tree clean, on manifest branch, up-to-date with origin. |
| `ahead`    | Local has unpushed commits.                                      |
| `behind`   | Local is behind origin (will fast-forward on next `fleet-sync`). |
| `dirty`    | Working tree has uncommitted changes.                            |
| `missing`  | Repo not cloned on this machine.                                 |

## Failover procedure

If Flowboard is down:

1. Open https://souvigna38.github.io/fleet-dashboard/ — this is the backup.
2. The data shown is at most 15 minutes stale (whichever machine last synced).
3. To force a fresh snapshot right now, run on any machine:
   ```bash
   python3 ~/.fleet/manifest/tools/fleet-flowboard-sync
   ```
   That will push to Flowboard (if it's back up) AND commit a fresh `dashboard.json` here.

When Flowboard comes back up, the next scheduled sync run will reconcile any drift automatically — the `fleet_id` (machine:repo) is the stable key, so cards just update in place.

## Local development

```bash
git clone git@github.com:souvigna38/fleet-dashboard.git
cd fleet-dashboard
python3 -m http.server 8000
# open http://localhost:8000
```

## Files

- `index.html` — the single-page dashboard (no build step, no framework, no external deps)
- `dashboard.json` — the data snapshot, committed by `fleet-flowboard-sync`
- `README.md` — this file

## Source

The sync script that generates `dashboard.json` lives at
`~/.fleet/manifest/tools/fleet-flowboard-sync` (also in
[souvigna38/fleet-manifest](https://github.com/souvigna38/fleet-manifest)).
