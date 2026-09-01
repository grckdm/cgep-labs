# GCP Security Services Baseline

GCP counterpart to the AWS baseline in Lab 5.2. Where CloudTrail and Security Hub are detective —
they flag a problem after the fact — Org Policy is preventive: a violating API call is rejected
outright, so there is nothing to remediate later because the forbidden action never happened.
Workload Identity Federation applies the same identity-first posture to CI/CD: GitHub Actions
authenticates with a short-lived OIDC token instead of a downloadable service-account key.

| Control | Resource | Maps to | Notes |
|---|---|---|---|
| Uniform bucket-level access | `google_org_policy_policy.uniform_bucket_access` | CM-6 | Rejects any new bucket created with legacy ACLs. |
| Disable service-account key creation | `google_org_policy_policy.disable_sa_keys` | AC-2 | Verified live — see below. |
| Require OS Login | `google_org_policy_policy.require_oslogin` | AC-3 | Rejects VM creation with metadata-based SSH keys. |
| Workload Identity Federation | `google_iam_workload_identity_pool.github`, `..._provider.github`, `google_service_account.gha` | AC-2, IA-2 | `attribute_condition` scopes trust to `grckdm/cgep-labs` only — no other GitHub repo can assume this identity. |
| Data Access audit logs | `google_project_iam_audit_config.{storage,kms,iam}` | AU-2 | `DATA_READ` + `DATA_WRITE` + `ADMIN_READ`, off by default per service — the single most common GCP audit finding is that nobody turns these on. |

## Lesson: Data Access logs are off by default

Admin Activity logs are always on and free, but Data Access logs — the ones that actually record
who read or wrote what — are off by default for every service and cost roughly $0.50/GB ingested.
An auditor checking AU-2 against a fresh GCP project will find logging *exists* but doesn't
actually capture data access unless someone deliberately enabled it, per service, as this lab does
for `storage`, `cloudkms`, and `iam`.

## Verified: Org Policy enforcement

After apply, key creation on the WIF service account was attempted and rejected at the API,
before propagation had even fully settled:

```
ERROR: (gcloud.iam.service-accounts.keys.create) FAILED_PRECONDITION:
Key creation is not allowed on this service account.
constraint: iam.disableServiceAccountKeyCreation
```

No key was ever created — there's no finding to write up, because the control acted at the moment
of the API call rather than surfacing the violation after the fact.

## Evidence captured

`evidence/lab-5-4/iam-policy.json` — project IAM policy `auditConfigs`, confirming
`DATA_READ`/`DATA_WRITE`/`ADMIN_READ` are enabled for `storage.googleapis.com`,
`cloudkms.googleapis.com`, and `iam.googleapis.com`.

## Verify

```bash
gcloud org-policies list --project=cgep-lab-5-4
gcloud iam workload-identity-pools list --location=global --project=cgep-lab-5-4
gcloud projects get-iam-policy cgep-lab-5-4 --format="json(auditConfigs)"

SA_EMAIL=$(terraform output -raw wif_service_account_email)
gcloud iam service-accounts keys create /tmp/k.json --iam-account="$SA_EMAIL" --project=cgep-lab-5-4
# Expect: FAILED_PRECONDITION
```

## Cleanup

```bash
terraform destroy -auto-approve \
  -var="gcp_project=cgep-lab-5-4" \
  -var="github_repo=grckdm/cgep-labs"
```

WIF pools enter a 30-day soft-delete on destroy — a pool with the same
`workload_identity_pool_id` (`github-actions`) can't be recreated until that expires, or is
`undelete`d and then deleted with `--purge`.
