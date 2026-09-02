# OSCAL Layer

Authored with `trestle` (v5.0.0, OSCAL 1.2.1) under `.trestle-work/` (gitignored authoring
workspace), then copied here as the committed artifacts.

## Component definitions

`components/compliant-s3.json` describes `terraform/primitives/compliant-s3` — the S3 primitive
module with a dedicated access-log bucket. It claims four controls, each mapped to the exact
Terraform resource that enforces it:

| Control | Resource | Enforces |
|---|---|---|
| sc-28 | `aws_s3_bucket_server_side_encryption_configuration.primary` | AES-256 encryption at rest via a customer-managed KMS key |
| ac-3 | `aws_s3_bucket_public_access_block.primary` | Explicit deny on every public access vector |
| au-3 | `aws_s3_bucket_logging.primary` | Access logging to a dedicated log-delivery bucket |
| cm-6 | `aws_s3_bucket_versioning.primary` | Object versioning as a required configuration setting |

All four `implemented-requirements` link to the same signed evidence bundle:

```
s3://cgep-lab-grc-evidence-vault-1733a891/runs/33167465430/evidence-33167465430-7dd5e8f2571797f2ce24010c547c1b64d5f3360e.tar.gz
```

produced by the Lab 4.3/4.4 pipeline.

## Profiles

`profiles/cge-p-minimum.json` selects the same four controls (`sc-28`, `ac-3`, `au-3`, `cm-6`)
from the NIST SP 800-53 Rev 5 catalog. Resolving it (`trestle author profile-resolve`) produces a
self-contained catalog with full control text pulled in — the artifact an SSP would import.

## Evidence traversal result (Lab 6.1, Step 7)

Following the `sc-28` evidence link and running `scripts/verify-evidence.sh` against the bundle
above returned a partial result, not a clean `CHAIN INTACT`:

- **Integrity** — passed (SHA-256 matched).
- **Authenticity** — passed (`cosign verify-blob: Verified OK`).
- **Preservation** — **failed**: `FAIL: retention expired`. The bundle's S3 Object Lock
  (`GOVERNANCE` mode) was retained until 2026-08-29 per the Lab 4.4 write-up; this traversal ran
  after that date, so the lock has since lapsed.

This is left as-is rather than papered over: it's an honest demonstration of what OSCAL's
evidence links actually verify at the moment you check them, including a control (preservation)
that is explicitly time-bound and can expire. A fresh pipeline run would produce a new bundle with
a new retention window; that wasn't done here so the finding stands as recorded.

## Validation

`evidence/lab-6-1/trestle-validate.txt` — `trestle validate` output for both models, both `VALID`.
