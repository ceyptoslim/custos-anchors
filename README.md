# custos-anchors

Append-only anchor records for the frozen red-proof observation sets of
[ceyptoslim/CUSTOS-CORE](https://github.com/ceyptoslim/CUSTOS-CORE).

Each record pins the digest of every frozen run directory as it stood at a named commit. The
records are chained: a record names the hash of the record before it, so a rewrite of history
leaves a break rather than a replacement.

## Records

| File | Epoch | Source commit | Aggregate | Anchor hash |
|---|---|---|---|---|
| `anchors/epoch-0.json` | 0 | `f06571b0f2ac` | `96219e9491556295df407a42a1ea9ad219d6bb18aa8a283e4ebf39527d107419` | `1c650575c2f2997d0af8d8fd566388270e42eb485fe57b7f485f812ec6e8fddf` |
| `anchors/epoch-1.json` | 1 | `e86b72c907d8` | `96219e9491556295df407a42a1ea9ad219d6bb18aa8a283e4ebf39527d107419` | `25bb62cd891c422e1aa22164517077722261bb1cf47713d34cbb9fdf4fd6cfdc` |

Both records carry the same eleven run digests and the same aggregate. Epoch 0 has an empty
`prev_anchor_hash` because it starts the chain, and epoch 1 links to it. Epoch 1 exists because the
digest script changed, not because any observation set moved: the script now takes its sha256 from
`shasum -a 256` where `sha256sum` is absent, which macOS requires.

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

## The records are checked where they land

Every push to this repository runs `.github/workflows/verify-anchors.yml`. It walks the chain,
checks that each record links to the one before it, and fails if an epoch drops or changes a digest
that an earlier epoch anchored. It needs no secret and reads nothing but the records here, so a bad
append is caught in this repository rather than only by whoever reads CUSTOS-CORE later.

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

Writes from the CUSTOS-CORE side use a fine-grained token scoped to this repository alone, held in
a GitHub Environment named `anchor-write` restricted to `main`, never in ordinary repository
secrets, and the writer runs only on pushes to `main`. That token exists as of 2026-11-19, and its
ability to write was proven by a manual dry run that pushed a throwaway branch here and deleted it,
so the first real epoch did not depend on an untested permission.

## License

The records, this README and the verification workflow are dedicated to the public domain under
[CC0 1.0 Universal](LICENSE). Anyone may copy, quote, mirror or republish them without asking
permission.

That is deliberate. An anchor only helps if a party who does not trust this project can hold their
own copy of it, and a license that required permission or attribution would put a lawyer between
the reader and the check.
