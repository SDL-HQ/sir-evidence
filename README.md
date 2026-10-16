# SIR, in six commands and one you break yourself

SIR is a deterministic pre-inference gate. It reads a request before a model
does, decides `PASS` or `BLOCKED` by rules alone with no model in the decision
path, and writes a signed record of what it saw and what it decided. The record
is the point: it is designed to be checked by someone who does not trust us.

**Read [LIMITS.md](LIMITS.md) second.** It says what SIR does not do, in the
project's own words, and most of what is below is a limit made concrete.

This repository is generated from the archive of record, pinned at `v2.5.0`. It
holds 6 real published archives, each chosen because running our
verifiers against it produces a different answer.

**Our own verdict tool establishes 2 of the 6.**
`verify_evidence.py` prints one verdict over every property of an archive, and
exit 0 from a single-property verifier is not the same thing. 4 of
these 6 come back NOT ESTABLISHED or FAILED, and
4 of them are defects this project has already written up in its
own errata, named in the table.

| # | Exhibit | Document exit | Receipt exit | Verdict | Errata |
|---|---|---|---|---|---|
| 1 | an archive that establishes every property | `0` | `0` | **ESTABLISHED** | - |
| 2 | a signature that does not verify | `5` | `2` | **FAILED** | E2 |
| 3 | an archive from before the 2.3.4 correction | `9` | `0` | **NOT ESTABLISHED** | E4 |
| 4 | an archive with no ledger and no receipt | `0` | `3` | **NOT ESTABLISHED** | E3 |
| 5 | a complete-looking archive that is not | `0` | `2` | **FAILED** | E1 |
| 6 | a baseline record | `0` | `0` | **ESTABLISHED** | - |

[VERIFY.md](VERIFY.md) has the exact command for each, explains why each one
names its ledger, and ends with a recipe for changing one field in one ledger
row so you can watch the chain refuse to re-derive. That last one is the only
item here that is a detection rather than a disclosure, and it is the reason
the evidence plane exists.

The archive of record is at [https://github.com/SDL-HQ/sir-firewall](https://github.com/SDL-HQ/sir-firewall). It holds
353 runs and 351 certificates.

Nothing here is a sample chosen for how it looks. Each exhibit is selected from
a category by a tool in the main repository, a test there refuses any category
with fewer than two eligible archives, and another refuses any exhibit drawn
from the 54 ungated baseline runs this project published as audit
certificates. The first version of this page failed that last check, which did
not exist: its lead exhibit was one of the 54.

## This repository is generated

Every file here except the workflow that rebuilds it is written by
`tools/generate_shop_window.py` in the archive of record, from the latest
release tag. Nothing is edited in place. An edit made here would be discarded
by the next rebuild rather than read, so issues and changes belong in the
archive of record. `GENERATED` records the tag, the commit it was built from,
and when.

The build is not a copy. It runs every command on this page inside the
directory it has just written, including the tampering recipe, and stops
without publishing if any exit code or verdict differs from the one the page
gives for it.
