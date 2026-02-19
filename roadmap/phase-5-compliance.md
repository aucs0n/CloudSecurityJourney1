# Phase 5: Vulnerability & Compliance Management

## Overview
Compliance and vulnerability management ensure your AWS infrastructure meets security standards and is free from known vulnerabilities.

## Learning Objectives
- Use Amazon Inspector for vulnerability scanning
- Implement AWS Config Rules for compliance
- Understand compliance frameworks (CIS, PCI-DSS, HIPAA, etc.)
- Use AWS Audit Manager for evidence collection
- Automate compliance remediation

---

## Amazon Inspector

### Inspector Scanning Types

1. **EC2 Scanning**: Software vulnerabilities, network reachability
2. **ECR Scanning**: Container image vulnerabilities
3. **Lambda Scanning**: Function code and dependencies

### Enable Inspector

```bash
# Enable Inspector
aws inspector2 enable --resource-types EC2 ECR LAMBDA

# List findings
aws inspector2 list-findings \
  --filter-criteria '{"severities":[{"comparison":"EQUALS","value":"CRITICAL"}]}'
```

---

## AWS Config Rules

### Common Config Rules

- `s3-bucket-public-read-prohibited`
- `encrypted-volumes`
- `iam-password-policy`
- `cloudtrail-enabled`
- `root-account-mfa-enabled`
- `vpc-flow-logs-enabled`
- `rds-encryption-enabled`

### Deploy Config Rules

```bash
# Enable Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/ConfigRole

# Start recording
aws configservice start-configuration-recorder \
  --configuration-recorder-name default

# Deploy conformance pack (CIS Benchmark)
aws configservice put-conformance-pack \
  --conformance-pack-name cis-aws-benchmark \
  --template-s3-uri s3://aws-configservice-conformance-packs/cis-aws-benchmark.yaml
```

---

## Compliance Frameworks

### CIS AWS Foundations Benchmark

**Key Controls:**
- 1.1: Avoid root account use
- 1.4: Ensure MFA for root account
- 2.1: Ensure CloudTrail is enabled in all regions
- 2.3: Ensure S3 bucket used for CloudTrail logs is not publicly accessible
- 4.1: Ensure no security groups allow 0.0.0.0/0 ingress to port 22

### PCI-DSS Requirements

For payment card data:
- Encryption of cardholder data (KMS)
- Network segmentation (VPC)
- Access control (IAM)
- Logging and monitoring (CloudTrail, CloudWatch)
- Regular vulnerability scans (Inspector)

### HIPAA Compliance

For healthcare data:
- Encryption at rest and in transit
- Access auditing (CloudTrail)
- Backup and recovery (snapshots)
- Incident response procedures

---

## Best Practices

- [ ] Enable Inspector in all accounts
- [ ] Use AWS Config Rules for continuous compliance
- [ ] Regular compliance audits
- [ ] Automate remediation where possible
- [ ] Document compliance controls

---

[← Previous: Monitoring](./phase-4-monitoring-logging.md) | [Back to Roadmap](../ROADMAP.md) | [Next: Secrets →](./phase-6-secrets-management.md)
