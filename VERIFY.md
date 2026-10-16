# Verifying the exhibits

Every command below runs offline against a file in this repository. Nothing
calls a model provider, and nothing needs a credential.

## Why each command names its ledger

A certificate is bound to the ledger of the run it signs, so verifying one
means checking it against that ledger. The verifier can usually find the
ledger by itself, from the run id inside the signed payload.

**Older certificates carry no run id.** They predate the field. For those the
verifier has nothing to key on, finds no ledger however complete the directory
is, and reports that the binding was not checked. That is a true answer and a
confusing one, so every command here names the ledger explicitly, and the two
archives that ship no ledger at all pass `--no-ledger` instead.

We could have hidden this in a one-word wrapper. We have not, because the
point of this repository is that you can see exactly what is being checked.

## The commands

### 1. a current certificate

signature, ledger binding and row count all check, and the archive is complete

```
python3 tools/verify_certificate.py \
    exhibits/20260921-134707-359329-gh35607761858-a8a5ade8f6ed/audit.json \
    --ledger exhibits/20260921-134707-359329-gh35607761858-a8a5ade8f6ed/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260921-134707-359329-gh35607761858-a8a5ade8f6ed
```

Document verifier exits `0`. Receipt verifier exits
`0`.
### 2. a signature that fails

a real signature that really does not verify, published anyway

```
python3 tools/verify_certificate.py \
    exhibits/20260405-040319-000000-702b336916d3/audit.json \
    --ledger exhibits/20260405-040319-000000-702b336916d3/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260405-040319-000000-702b336916d3
```

Document verifier exits `5`. Receipt verifier exits
`2`.
### 3. a pre-2.3.4 archive

a documented format boundary is not tampering, and the verifier says which it is

```
python3 tools/verify_certificate.py \
    exhibits/20260418-152959-000000-gh24607820167-1ebb76ec1902/audit.json \
    --ledger exhibits/20260418-152959-000000-gh24607820167-1ebb76ec1902/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260418-152959-000000-gh24607820167-1ebb76ec1902
```

Document verifier exits `9`. Receipt verifier exits
`0`.
### 4. an archive shipping no ledger

verified, and less than "verified" sounds

```
python3 tools/verify_certificate.py \
    exhibits/20260111-023031-211ac186f758/audit.json \
    --no-ledger
python3 tools/verify_archive_receipt.py exhibits/20260111-023031-211ac186f758
```

Document verifier exits `0`. Receipt verifier exits
`3`.
### 5. an archive with no receipt

legacy archives predate the receipt scheme, disclosed

```
python3 tools/verify_certificate.py \
    exhibits/20260111-023031-211ac186f758/audit.json \
    --no-ledger
python3 tools/verify_archive_receipt.py exhibits/20260111-023031-211ac186f758
```

Document verifier exits `0`. Receipt verifier exits
`3`.
### 6. a complete-looking archive that is not

the receipt catches what the certificate cannot

```
python3 tools/verify_certificate.py \
    exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee/audit.json \
    --ledger exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20260921-134234-688792-gh35607304829-b3a0326b0dee
```

Document verifier exits `0`. Receipt verifier exits
`2`.
### 7. a baseline record

a certificate verifier refuses it, and its own verifier accepts it

```
python3 tools/verify_baseline_record.py \
    exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/baseline.json \
    --ledger exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/proofs/itgl_ledger.jsonl
python3 tools/verify_archive_receipt.py exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2
```

Document verifier exits `0`. Receipt verifier exits
`0`.

And the part this exhibit exists for, the wrong verifier refusing it rather
than reporting something about it:

```
python3 tools/verify_certificate.py \
    exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/baseline.json \
    --ledger exhibits/20261015-092557-107124-gh38712792514-670d9d4222c2/proofs/itgl_ledger.jsonl
```

Exits `12`, naming the document kind it found.

## How to read the exit codes

Certificate verifier:

| Exit | Meaning |
|---|---|
| `0` | verified; no ledger in the archive, so no binding was checked |
| `5` | SIGNATURE VERIFICATION FAILED |
| `9` | ledger binding not established (no row count, or predates the 2.3.4 correction) |

Archive receipt verifier:

| Exit | Meaning |
|---|---|
| `0` | verified |
| `2` | archive incomplete or signature failed |
| `3` | no archive receipt (legacy archive) |

These meanings are generated from the archive of record's own published
figures, not written for this page.
