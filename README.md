# ergonia-witness

The public daily checkpoint of the [Ergonia](https://ergonia.works)
chain head. One JSON line per UTC day, committed by the steward's
scheduled run.

Ergonia's `/api/attest` recomputes the entire event chain on every
request and returns its `count` and `head.hash`. That works fine if
you trust the Worker. This repository is what a reader can compare
against when they do not: each day's line records the same figures
in a place the Worker cannot rewrite.

## Format

`HEADS.jsonl` is append-only. One line per day, in UTC order,
oldest first. Each line is a compact JSON object:

```json
{"date":"2026-08-31","count":42,"head_id":42,"head_hash":"a1b2c3...","captured_at":"2026-08-31T07:35:12Z"}
```

- `date` : the UTC date the checkpoint was taken (YYYY-MM-DD)
- `count` : the number of events in the chain at capture time
- `head_id` : the id of the newest event at capture time
- `head_hash` : the SHA-256 of that event (as returned by /api/attest)
- `captured_at` : the ISO-8601 UTC timestamp the request was made

The append order is guaranteed by the workflow: any line committed
under a given date is committed in a single push per day. A day
without a line means the checkpoint job did not run (workflow
disabled, GitHub outage, network error). A day with two lines means
the same job ran twice, which is worth flagging in the steward's
report.

## How to check the chain

If today's line is `{count: N, head_hash: H}`, then hitting
`https://ergonia.works/api/attest` right now should return `ok: true`
with `count >= N` and, if `count == N`, `head.hash == H`. If those
numbers do not line up, the chain has been rewritten between the
checkpoint and now.

The checkpoint does not sign anything. It relies on GitHub's own
commit history to make the timestamp and the file contents
non-repudiable. If you distrust GitHub too, the same JSONL is a fine
target for an independent notary; the format is stable on purpose.

## What lives here

- `HEADS.jsonl` : the append-only checkpoint log
- `LICENSE` : AGPL-3.0-or-later, same as the platform

Nothing else. This repo is a witness; if it grows features, the
witness role gets diluted.
