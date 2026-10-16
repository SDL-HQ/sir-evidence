# Verifying the exhibits

Every command below runs offline against a file in this repository. Nothing
calls a model provider, and nothing needs a credential.

## Three commands per exhibit, and the third is the one that answers

The first two verify one property each: the signed document, and the archive's
own manifest. `verify_evidence.py` runs those and the rest, and prints one
verdict over all of them. It is the command the archive of record tells a
reader to prefer, and an exit code of 0 from the first two is not the same
thing as its verdict:

- **ESTABLISHED** every applicable property established, none unknown
- **NOT ESTABLISHED** nothing failed, but at least one property is unknown
- **FAILED** at least one property actively failed

Not established is not a failure and it is not a pass. A certificate no
evidence contract governs has not failed a contract; a ledger that predates the
fields needed to recompute the signed counters has not disagreed with them.
Both are unknown, and unknown has its own verdict.

## Why each command names its ledger

A certificate is bound to the ledger of the run it signs, so verifying one
means checking it against that ledger. The verifier can usually find the
ledger by itself, from the run id inside the signed payload.

**Older certificates carry no run id.** They predate the field. For those the
verifier has nothing to key on, finds no ledger however complete the directory
is, and reports that the binding was not checked. That is a true answer and a
confusing one, so every command here names the ledger explicitly, and the
archive that ships no ledger at all passes `--no-ledger` instead.

We could have hidden this in a one-word wrapper. We have not, because the
point of this repository is that you can see exactly what is being checked.

## The commands

### 1. an archive that establishes every property

signature, ledger binding, signed counters and archive custody all check

```
python3 tools/verify_certificate.py \
    exhibits/20261012-065620-681856-gh38214991880-2b04aed6f784/audit.json \
    --ledger exhibits/20261012-065620-681856-gh38214991880-2b04aed6f784/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20261012-065620-681856-gh38214991880-2b04aed6f784
python3 tools/verify_evidence.py exhibits/20261012-065620-681856-gh38214991880-2b04aed6f784
```

Document verifier exits `0`, receipt verifier exits `0`, and the verdict is **ESTABLISHED**.
### 2. a signature that does not verify

a published certificate signed by a key the registry does not hold

```
python3 tools/verify_certificate.py \
    exhibits/20260405-040319-000000-702b336916d3/audit.json \
    --ledger exhibits/20260405-040319-000000-702b336916d3/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260405-040319-000000-702b336916d3
python3 tools/verify_evidence.py exhibits/20260405-040319-000000-702b336916d3
```

Document verifier exits `5`, receipt verifier exits `2`, and the verdict is **FAILED**.

This archive is listed individually under **E2** in the archive of record's errata.
### 3. an archive from before the 2.3.4 correction

a documented format boundary is not tampering, and the verifier says which

```
python3 tools/verify_certificate.py \
    exhibits/20260418-153112-000000-gh24607820167-f86793d88202/audit.json \
    --ledger exhibits/20260418-153112-000000-gh24607820167-f86793d88202/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260418-153112-000000-gh24607820167-f86793d88202
python3 tools/verify_evidence.py exhibits/20260418-153112-000000-gh24607820167-f86793d88202
```

Document verifier exits `9`, receipt verifier exits `0`, and the verdict is **NOT ESTABLISHED**.

This archive's defect is **E4** in the archive of record's errata, which describes the class and does not enumerate its members, so searching that document for this run id finds nothing.
### 4. an archive with no ledger and no receipt

verified, and less than "verified" sounds: nothing bound it and nothing vouches for the files

```
python3 tools/verify_certificate.py \
    exhibits/20260111-023031-211ac186f758/audit.json \
    --no-ledger
python3 tools/verify_archive_receipt.py exhibits/20260111-023031-211ac186f758
python3 tools/verify_evidence.py exhibits/20260111-023031-211ac186f758
```

Document verifier exits `0`, receipt verifier exits `3`, and the verdict is **NOT ESTABLISHED**.

This archive is listed individually under **E3** in the archive of record's errata.
### 5. a complete-looking archive that is not

the receipt catches what the certificate cannot

```
python3 tools/verify_certificate.py \
    exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee/audit.json \
    --ledger exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee
python3 tools/verify_evidence.py exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee
```

Document verifier exits `0`, receipt verifier exits `2`, and the verdict is **FAILED**.

This archive is listed individually under **E1** in the archive of record's errata.
### 6. a baseline record

a certificate verifier refuses it, and its own verifier accepts it

```
python3 tools/verify_baseline_record.py \
    exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/baseline.json \
    --ledger exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2
python3 tools/verify_evidence.py exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2
```

Document verifier exits `0`, receipt verifier exits `0`, and the verdict is **ESTABLISHED**.

And the part this exhibit exists for, the wrong verifier refusing it rather
than reporting something about it:

```
python3 tools/verify_certificate.py \
    exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/baseline.json \
    --ledger exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/proofs/itgl_ledger.jsonl
```

Exits `12`, naming the document kind it found.

## Break one and watch it get caught

Every exhibit above is an archive we published and a defect we disclosed. None
of them is a detection, and detection is the whole proposition, so here is one
you perform yourself. Copy exhibit 1, change one field in one ledger row, and
re-run the command that passed:

```
cp -r exhibits/20261012-065620-681856-gh38214991880-2b04aed6f784 /tmp/tampered
python3 - <<'EOF'
import json, pathlib
p = pathlib.Path("/tmp/tampered/proofs/itgl_ledger.jsonl")
rows = p.read_text().splitlines()
row = json.loads(rows[75])
row["expected"] = "allow" if row["expected"] == "block" else "block"
rows[75] = json.dumps(row, separators=(",", ":"), sort_keys=True)
p.write_text("\n".join(rows) + "\n")
EOF
python3 tools/verify_certificate.py /tmp/tampered/audit.json --ledger /tmp/tampered/proofs/itgl_ledger.jsonl
```

Exits `7`, and names the row and both hashes:

```
ERROR: ledger binding verification failed: Entry #76 has invalid ledger_hash: stored='a8cccdba5dbe2035267859644d72eefb3b4541fb0dd15842904197700e092b9f', computed='d46ce36f31b3dd34814b4ae198d39e6c00eb24cdd3cd0f22aba9ecdaf6cfac0e'
```

One field in one row of 150, and the chain does not re-derive.

## How to read the exit codes

Certificate verifier. Two rows share exit 0, because 82 archives verified with their binding checked and 39 verified without a ledger to check it against, and those are different facts:

| Exit | Meaning | Archives |
|---|---|---|
| `0` | verified | 82 |
| `0` | verified; no ledger in the archive, so no binding was checked | 39 |
| `5` | SIGNATURE VERIFICATION FAILED | 6 |
| `7` | ledger chain or certificate-binding verification failed | none published |
| `9` | ledger binding not established (no row count, or predates the 2.3.4 correction) | 223 |
| `12` | the document is not a SIR audit certificate, and nothing about it was verified, including its signature | none published |

Archive receipt verifier:

| Exit | Meaning | Archives |
|---|---|---|
| `0` | verified | 208 |
| `2` | archive incomplete or signature failed | 105 |
| `3` | no archive receipt (legacy archive) | 39 |

Baseline record verifier:

| Exit | Meaning | Archives |
|---|---|---|
| `0` | verified | 2 |

Every meaning and count above is generated from the archive of record's own
published figures, except the two exit codes no published archive produces,
which are generated from the verifier's own documentation.
