# Compliance Policies

Rego policies that check a Terraform plan (`terraform show -json`) against specific controls before it's ever applied. An empty `deny` set means compliant; any message means the plan is rejected.

| File | Control | Severity | Remediation |
|---|---|---|---|
| `sc28_encryption.rego` | SC-28 (Encryption at Rest) | High | Add an `encryption { default_kms_key_name = ... }` block referencing a `google_kms_crypto_key` you control. |
| `ac3_no_public.rego` | AC-3 (Access Enforcement) | Critical | Set `uniform_bucket_level_access = true`, `public_access_prevention = "enforced"`. For firewalls, narrow `source_ranges` or remove the rule. |
| `cm6_required_tags.rego` | CM-6 (Configuration Settings) | Medium | Add the four required labels (`project`, `environment`, `managed_by`, `compliance_scope`) to the resource. |

Tests for each policy live in `tests/`, run with:

```bash
opa test -v policies/
```
