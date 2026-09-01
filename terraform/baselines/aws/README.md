# AWS Security Services Baseline

Account-level continuous monitoring: CloudTrail records what happened, Security Hub normalizes
findings from across the account into NIST-mapped controls. Unlike the pipeline in Chapter 4,
this evidence isn't tied to a pull request — it's produced continuously, whether or not anyone
pushes code.

| Service | Resources | Controls | Notes |
|---|---|---|---|
| CloudTrail | `aws_cloudtrail.mgmt`, `aws_s3_bucket.trail` | AU-2 (audit events), AU-12 (audit generation), AU-10 (non-repudiation) | Multi-region, log-file validation enabled — the hourly signed digest is what makes AU-10 concrete: tampering with delivered logs is detectable. |
| Security Hub | `aws_securityhub_account.this`, standards subscriptions | RA-5 (vulnerability scanning), SI-4 (system monitoring) | Subscribed to NIST 800-53 Rev 5 and AWS Foundational Security Best Practices. `enable_default_standards` (a provider default) also auto-subscribed CIS AWS Foundations Benchmark v1.2.0 — three standards running, not two. |
| AWS Config | not deployed | CM-2 (baseline config), CM-6 (config settings), CM-8 (inventory) | Skipped for this lab. Security Hub reports the gap itself: finding `Config.1` — *"AWS Config should be enabled and use the service-linked role for resource recording"* (CRITICAL) — is the machine-readable evidence of this control's status. |

## Evidence captured

`evidence/lab-5-2/security-hub-findings.json` — 15 findings as of first capture: 14 LOW-severity
CIS log-metric-filter checks (missing CloudWatch alarms for things like root usage, console
sign-in without MFA, security group changes) and 1 CRITICAL (`Config.1`, above). A quiet,
freshly-baselined account producing exactly the finding profile the guide predicts.

## Verify

```bash
terraform output
aws cloudtrail get-trail-status --name "$(terraform output -raw trail_name)" --region us-east-1 --profile grcclub
aws securityhub describe-hub --region us-east-1 --profile grcclub
```

## Cost note

This baseline bills while it runs (Security Hub standards checks). Destroy promptly after
capturing evidence:

```bash
terraform destroy -auto-approve
```
