# FIRELINE proof ledger

A permanent, publicly verifiable record of FIRELINE's decisions. Every checkpoint — from hours before tip-off to quarter ends — is committed as a sealed YAML registry, including both selections and passes. Each registry is timestamped using [OpenTimestamps](https://opentimestamps.org), with its cryptographic proof anchored to the Bitcoin blockchain.

## Two-phase commit

Every record goes through a commit → reveal cycle:

1. **Commit** — When a checkpoint fires, the complete registry is hashed and timestamped. Its `.ots` proof is published under `proof/`, making the commitment public while its contents remain sealed.
2. **Reveal** — The following morning, the YAML registry is published under `revealed/` at the corresponding path. The `.ots` proof is upgraded once its Bitcoin attestation becomes available.

A `proof/` entry without a matching `revealed/` file is a **sealed commitment** — its existence is recorded, but its contents remain hidden. An old unrevealed proof is an anomaly worth reporting, not an indefinitely pending disclosure.

## Layout

```text
proof/<league>/<yy-mm-dd>/<timing>/<HOME>-vs-<AWAY>.ots
revealed/<league>/<yy-mm-dd>/<timing>/<HOME>-vs-<AWAY>.yaml
```

- `<yy-mm-dd>` — the ET slate day
- `<timing>` — the checkpoint: `pre6h`, `pre3h`, `pre1h`, `tipoff`, `q1end`, `q2end`, `q3end`, `q4last5m`
- `<HOME>` / `<AWAY>` — team abbreviations

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

- `salt` — random per record, preventing the committed hash from revealing its contents before disclosure.
- `odds` — decimal market baseline evaluated at the checkpoint: carried market prices pregame and at `tipoff`, live quotes from `q1end` onward.
- `prob` — the model's probability estimate committed at that checkpoint.
- `stake` — the theoretical bankroll exposure calculated by the model.
- `note` — the decision context for each market: `bet` (selection made), `no bet` (market available, strategy declined), or `no market` (no usable price available).
- A record without `bets:` represents a **committed pass**, preserved with the same cryptographic guarantees as a selection. Prediction estimates are recorded only when a selection is made.
- The `proof/` tree is intended to contain one commitment per game per checkpoint. Missing commitments can be identified by comparing the repository against independent game schedules.
- **Schema evolution:** Each record is preserved under the schema in effect at its checkpoint. Historical records are never migrated; missing fields in older records represent unavailable evidence, not necessarily defects.

Historical field names are preserved for compatibility with the original sealed records.

## Verify

Revealed records can be independently verified against Bitcoin without trusting the repository's contents.

Install the OpenTimestamps client:

```bash
pip install opentimestamps-client
```

Verify a revealed registry against its proof:

```bash
ots verify proof/nba/26-11-12/pre3h/PHI-vs-TOR.ots revealed/nba/26-11-12/pre3h/PHI-vs-TOR.yaml
```

Use `ots upgrade` to refresh pending proofs when their Bitcoin attestations become available.

OpenTimestamps authenticates the **exact bytes** of a committed file. Any modification, including changes to its salt, invalidates the original proof.

For reliable verification, use the canonical repository bytes (`git show HEAD:<path>`) rather than potentially normalized working-tree files. Git checkout settings may alter line endings and produce false digest mismatches.

**Timing limitation:** A confirmed Bitcoin attestation establishes that the committed content existed no later than the attesting block's timestamp. Proving that a decision preceded its checkpoint may also require examining public Git history and independent timing evidence.

## Audit

Independent verification should examine more than individual cryptographic proofs. A valid timestamp establishes the integrity of a particular record, but does not by itself guarantee that every expected decision was published.

The full auditing methodology is documented in [AUDIT.md](AUDIT.md), covering:

- Repository identity and publication history
- Cryptographic verification and Bitcoin attestations
- Commitment timing against actual sporting events
- Git history continuity and possible rewrites
- Completeness against independent game schedules
- Missing, delayed, or unrevealed commitments
- Evidence requirements and reporting scope

The goal is to make FIRELINE's recorded decisions **independently verifiable, historically accountable, and resistant to undetected manipulation**.