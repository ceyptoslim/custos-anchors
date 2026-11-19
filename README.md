# custos-anchors

Append-only anchor records for the frozen red-proof observation sets of
[ceyptoslim/CUSTOS-CORE](https://github.com/ceyptoslim/CUSTOS-CORE).

Each record pins the digest of every frozen run directory as it stood at a named commit. The
records are chained: a record names the hash of the record before it, so a rewrite of history
leaves a break rather than a replacement.

## Records

| File | Epoch | Source commit | Aggregate |
|---|---|---|---|
| `anchors/epoch-0.json` | 0 | `f06571b0f2ac` | `96219e9491556295df407a42a1ea9ad219d6bb18aa8a283e4ebf39527d107419` |

Epoch 0 carries 11 run digests. Its `prev_anchor_hash` is empty because it is the start of the
chain. Its anchor hash is
`1c650575c2f2997d0af8d8fd566388270e42eb485fe57b7f485f812ec6e8fddf`.

## How to check a record

The digest scheme is fixed in a script, not in prose: `scripts/anchor_digest.sh` in CUSTOS-CORE.
The canonical form is sha256 over sorted `digest  relpath` lines, `LC_ALL=C`, LF, two spaces
between digest and path, one trailing newline. Only `docs/red-proof-records/run-*/` is hashed, so
the living documents in that folder cannot move a digest.

To reproduce the aggregate from a clone of CUSTOS-CORE at the recorded commit:

    git clone https://github.com/ceyptoslim/CUSTOS-CORE
    cd CUSTOS-CORE && git checkout f06571b0f2ac
    bash scripts/anchor_digest.sh aggregate docs/red-proof-records

To check that the tree still matches the chain, from CUSTOS-CORE:

    bash scripts/check_anchors.sh <newest record> docs/red-proof-records --origin external --chain <records dir> --chain-source custos-anchors

That exits 0 when every anchored run digest is unchanged and no anchored run has disappeared, 1
when one has moved, and 2 when the chain does not link or a record is not the tip. A run that is
not in the anchor is allowed: new observation sets are anchored at the next epoch.

## What these records do and do not establish

A record shows that the frozen set now published matches the set that was anchored on a date. It
does not show that what was frozen is what executed on a runner, it is not third-party
corroboration of the observation sets, and `custody_verified` remains False in CUSTOS-CORE.

## Provenance of this repository

Created 2026-11-19 and seeded with epoch 0 by the owner through the GitHub API. The separation
this repository provides is against edits that arrive through a pull request to CUSTOS-CORE, not
against the owner: the same account holds both repositories, so a compromise of that account
reaches both. Moving these records to an account or organization that cannot write CUSTOS-CORE is
the stronger arrangement, and it has not been done.

Writes from the CUSTOS-CORE side are meant to use a fine-grained token scoped to this repository
alone, held in a GitHub Environment named `anchor-write` restricted to `main`. That token does not
exist yet.
