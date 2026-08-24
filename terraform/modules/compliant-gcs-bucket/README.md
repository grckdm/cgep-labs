# compliant-gcs-bucket

Reusable Terraform module producing a CMEK-encrypted GCS bucket with a hardcoded compliance floor. Consumers control only project, environment, retention, and naming — every control below is enforced inside the module and cannot be disabled from a caller.

## Controls enforced

- **SC-12** (Cryptographic Key Establishment and Management) — bucket encryption key is a customer-managed `google_kms_crypto_key`, not Google's default key.
- **SC-13** (Cryptographic Protection) — data at rest is protected via CMEK (`encryption.default_kms_key_name`).
- **SC-28** (Protection of Information at Rest) — CMEK plus 90-day automatic key rotation (`rotation_period = "7776000s"`).
- **AU-11** (Audit Record Retention) — bucket-level `retention_policy`, minimum 365 days when `environment == "prod"` (enforced via variable validation).
- **CM-6** (Configuration Settings) — required labels (`project`, `environment`, `managed_by`, `compliance_scope`) merged onto every bucket; uniform bucket-level access and enforced public access prevention are hardcoded, not configurable.

## Output

`compliance_attestation` returns a machine-readable summary of the above, consumed as evidence by later labs' policy checks and OSCAL component definitions.
