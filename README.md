# FIRELINE pick ledger

Append-only, publicly verifiable record of every checkpoint decision the FIRELINE model makes — picks **and** passes. Each registry is a YAML file timestamped via [OpenTimestamps](https://opentimestamps.org), with the proof anchored to the Bitcoin blockchain.

## Two-phase commit

Every record goes through a commit → reveal cycle:

1. **Commit** — the checkpoint fires. The complete registry is hashed and stamped; the `.ots` proof lands here immediately under `proof/` — the commitment is public while the content stays sealed.
2. **Reveal** — the next morning the proof upgrades (Bitcoin confirmation) and the YAML content publishes under `revealed/` at the identical path.

A `proof/` entry with no `revealed/` twin is a **sealed commitment** — it exists, timestamped, but its pick can't be read yet.

## Layout

```
proof/<league>/<yy-mm-dd>/<timing>/<HOME>-vs-<AWAY>.ots
revealed/<league>/<yy-mm-dd>/<timing>/<HOME>-vs-<AWAY>.yaml
```

- `<yy-mm-dd>` — the ET slate day
- `<timing>` — the checkpoint: `pre6h`, `pre3h`, `pre1h`, `tipoff`, `q1end`, `q2end`, `q3end`, `q4last5m`
- `<HOME>`/`<AWAY>` — team abbreviations

## Registry format

```yaml
id: nba/26-11-12/pre3h/PHI-vs-TOR
timestamp: 2026-11-12T18:05:12.331Z
salt: 4f1a9c2d77b3e001
timing: pre3h
model: graphite-26-11-10
match:
  league: nba
  gameId: "401909092"
  date: 2026-11-12
  start: 2026-11-13T00:00Z
  home: PHILADELPHIA 76ERS
  away: TORONTO RAPTORS
  homeAbbr: PHI
  awayAbbr: TOR
bets:
  - market: moneyline
    pick: PHILADELPHIA 76ERS
    odds: 2.05
    prob: 0.5312
    stake: 0.0159
note: ml bet; hc no market
```

- `salt` — random per record; makes the committed hash unguessable before reveal
- `odds` — decimal price the checkpoint evaluated at (carried book odds pregame and at `tipoff`, live quote from `q1end` on); `prob` the model's fair probability; `stake` the bankroll fraction
- `note` — each leg's outcome: `bet`, `no bet` (market existed, strategy declined), or `no market` (no usable price at the checkpoint)
- A record with **no `bets:`** is a committed no-bet — the model passed at that timing. The pass is as permanent as a pick: the sealed `proof/` tree shows one commitment per game per checkpoint, so nothing can be retroactively claimed or hidden.

## Verify

Any revealed record verifies directly against Bitcoin — no trust in this repo required:

```bash
pip install opentimestamps-client
ots verify proof/nba/26-11-12/pre3h/PHI-vs-TOR.ots revealed/nba/26-11-12/pre3h/PHI-vs-TOR.yaml
```

`ots upgrade` refreshes a pending proof once its calendar attestation confirms (~hours after stamp). The proof attests the file's *exact bytes* — any edit, including the salt, invalidates it.
