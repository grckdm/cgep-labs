# Compliance Policies

Rego policies that check a Terraform plan (`terraform show -json`) against specific controls before it's ever applied. An empty `deny` set means compliant; any message means the plan is rejected.

| File | Cloud | Control | Severity | Remediation |
|---|---|---|---|---|
| `sc28_encryption.rego` | GCP | SC-28 (Encryption at Rest) | High | Add an `encryption { default_kms_key_name = ... }` block referencing a `google_kms_crypto_key` you control. |
| `ac3_no_public.rego` | GCP | AC-3 (Access Enforcement) | Critical | Set `uniform_bucket_level_access = true`, `public_access_prevention = "enforced"`. For firewalls, narrow `source_ranges` or remove the rule. |
| `cm6_required_tags.rego` | GCP | CM-6 (Configuration Settings) | Medium | Add the four required labels (`project`, `environment`, `managed_by`, `compliance_scope`) to the resource. |
| `sc28_encryption_aws.rego` | AWS | SC-28 (Encryption at Rest) | High | Add an `aws_s3_bucket_server_side_encryption_configuration` resource referencing the bucket. |
| `ac3_no_public_aws.rego` | AWS | AC-3 (Access Enforcement) | Critical | Add an `aws_s3_bucket_public_access_block` referencing the bucket with all four flags set to `true`. |
| `cm6_required_tags_aws.rego` | AWS | CM-6 (Configuration Settings) | Medium | Add the four required tags (`Project`, `Environment`, `ManagedBy`, `ComplianceScope`), or set them via provider `default_tags`. |

A control ID means the same thing on every cloud; the rule that enforces it does not. GCP and AWS resources are checked by separate files so each rule stays short and readable, while the control ID keeps the library organized around controls rather than clouds.

Tests for each policy live in `tests/`, run with:

```bash
opa test -v policies/
```

The AWS variants are wired into `scripts/policy-gate.sh`, which runs all three `*_aws` namespaces against a Terraform plan and exits non-zero on any violation.
