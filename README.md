# SIR, in seven commands

SIR is a deterministic pre-inference gate. It reads a request before a model
does, decides `PASS` or `BLOCKED` by rules alone with no model in the decision
path, and writes a signed record of what it saw and what it decided. The record
is the point: it is designed to be checked by someone who does not trust us.

**Read [LIMITS.md](LIMITS.md) second.** It says what SIR does not do, in the
project's own words, and most of what is below is a limit made concrete.

This repository is generated from the archive of record, pinned at `v2.5.0`.
It holds seven real published archives, chosen because running our own
verifiers against them produces different answers.

**5 of the seven do not come back clean.** One has a signature that
genuinely does not verify. One is missing a file its own manifest lists. The
rest are boundaries and absences we have disclosed rather than tidied: an
archive too old for the current binding scheme, one that ships no ledger, and
one that predates the receipt format.

The seventh comes back clean twice over and is still worth running: it is a
baseline record, not a certificate, and pointing the certificate verifier at
it makes the verifier refuse the document rather than say something wrong
about it.

| # | Exhibit | Document exit | Receipt exit | What it shows |
|---|---|---|---|---|
| 1 | a current certificate | `0` | `0` | signature, ledger binding and row count all check, and the archive is complete |
| 2 | a signature that fails | `5` | `2` | a real signature that really does not verify, published anyway |
| 3 | a pre-2.3.4 archive | `9` | `0` | a documented format boundary is not tampering, and the verifier says which it is |
| 4 | an archive shipping no ledger | `0` | `3` | verified, and less than "verified" sounds |
| 5 | an archive with no receipt | `0` | `3` | legacy archives predate the receipt scheme, disclosed |
| 6 | a complete-looking archive that is not | `0` | `2` | the receipt catches what the certificate cannot |
| 7 | a baseline record | `0` | `0` | a certificate verifier refuses it, and its own verifier accepts it |

[VERIFY.md](VERIFY.md) has the exact command for each, and explains why each
one names its ledger on the command line.

The archive of record is at [https://github.com/SDL-HQ/sir-firewall](https://github.com/SDL-HQ/sir-firewall). It holds
349 runs and 347 certificates.

Nothing here is a sample chosen for how it looks. Each exhibit is selected
from a category by a tool in the main repository, and a test there refuses any
category with only one member, so an exhibit is always an example of a class
rather than a fact about one run.

## This repository is generated

Every file here except the workflow that rebuilds it is written by
`tools/generate_shop_window.py` in the archive of record, from the latest
release tag. Nothing is edited in place. An edit made here would be discarded
by the next rebuild rather than read, so issues and changes belong in the
archive of record. `GENERATED` records the tag, the commit it was built from,
and when.

The build is not a copy. It runs every command on this page inside the
directory it has just written, and stops without publishing if any exit code
differs from the one the page gives for it.
