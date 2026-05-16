# iceland-results-live

Live election-night results channel for **lydraedisveislan.is**.

This repo exists only to serve one file — `results.json` — fast.

## Why a separate repo?

The main site (`iceland-elections-2026`) has ~6,800 tracked files. GitHub
Pages rebuilds and redeploys the **entire** site on every push, which takes
1–5 minutes — far too slow for live results updating every few minutes.

This repo contains just `results.json` + `.nojekyll`, so its GitHub Pages
deploy finishes in ~10–20 seconds. The website fetches the file
cross-origin at runtime:

```js
fetch('https://halldorberg.github.io/iceland-results-live/results.json?t=' + Date.now())
```

GitHub Pages serves static files with `Access-Control-Allow-Origin: *`, and
Cloudflare fronts it with a short edge TTL.

## How it's updated (election night)

1. Results are entered in the local backend → `data/live-results.draft.json`
   in the main repo.
2. `python scripts/publish_live_results.py` (in the main repo) validates the
   draft, computes D'Hondt seats, appends a timestamped snapshot per
   municipality, and writes `results.json` **here**.
3. `git add results.json && git commit && git push` — one small file.

Total edit-to-live ≈ 20–45 s. The main repo is never touched on the night.

## Schema

```jsonc
{
  "updatedAt": "2026-05-16T22:14:30Z",
  "munis": {
    "reykjavik": {
      "totalSeats": 23,
      "snapshots": [            // append-only, oldest → newest
        {
          "at": "2026-05-16T22:14:30Z",
          "votesCounted": 12345,
          "parties": {
            "D": { "pct": 28.7, "seats": 7 },
            "S": { "pct": 20.4, "seats": 5 }
          }
        }
      ]
    }
  }
}
```

`snapshots` is append-only so the site's carousel can scroll back through
earlier (lower vote-count) results. Generated/validated entirely by
`scripts/publish_live_results.py` — do not hand-edit.
