# Capstone Write-up

## Lab 4.4: Evidence Management and Chain of Custody

Run `33167465430` (PR #3, `add-cosign-signing`) produced a signed evidence bundle stored at
`s3://cgep-lab-grc-evidence-vault-1733a891/runs/33167465430/`. The table below maps each of the
four chain-of-custody properties to the artifact that proves it, based on that run.

| Property | Proven by | How |
|---|---|---|
| **Authenticity** | `evidence-33167465430-*.tar.gz.sig.bundle` | `cosign verify-blob` checks the bundle against a Fulcio certificate tied to `https://token.actions.githubusercontent.com` — the signature could only have been produced by this repository's GitHub Actions workflow. |
| **Integrity** | `evidence-33167465430-*.tar.gz.sha256` | `scripts/verify-evidence.sh` recomputes the SHA-256 of the downloaded bundle and compares it to this sidecar. A single altered byte in the bundle changes the hash and fails the check. |
| **Timeliness** | Rekor transparency log entry (checked inside `cosign verify-blob`) | Sigstore's public log records the signature with a timestamp at signing time, independent of anything AWS or this repo controls. |
| **Preservation** | S3 Object Lock retention on the bundle object (`GOVERNANCE` mode, retained until `2026-08-29`) | Even after a new object version was written to the same key during the Lab 4.4 tamper test, the original version (`GMW1_Tjke2BBZhFKc9TDSdZnY9OwJlKC`) remained retrievable and unmodified by version ID — Object Lock protects existing versions from deletion or modification, not just the current file. |

### Tamper test result

Downloading the bundle, appending a byte, and re-hashing produced a mismatched SHA-256
(`1d910dac...` vs `43fd36fa...`). Both `cosign verify-blob` and `scripts/verify-evidence.sh`
independently rejected the tampered content:

```
$ EVIDENCE_VAULT=cgep-lab-grc-evidence-vault-1733a891 bash scripts/verify-evidence.sh 33167465430 --profile grcclub
FAIL: SHA mismatch
```

Re-uploading the tampered bundle to the vault did not overwrite the original — S3 versioning
created a new version instead, and the original version's content was independently confirmed
still intact by fetching it directly via its version ID. Chain of custody held throughout.
