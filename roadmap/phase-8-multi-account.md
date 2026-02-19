# Phase 8: Multi-Account & Organizational Security

## Overview
Multi-account architecture is AWS best practice for security, compliance, and operational efficiency.

## Learning Objectives
- Design multi-account organization structure
- Implement Service Control Policies (SCPs)
- Use AWS Control Tower
- Set up centralized security logging
- Configure delegated administration

---

## Multi-Account Strategy

```
Management Account (Root)
├── Security OU
│   ├── Security Tooling Account (GuardDuty, Security Hub)
│   ├── Log Archive Account (CloudTrail, Config)
│   └── Audit Account (Read-only auditors)
├── Workloads OU
│   ├── Production Account
│   ├── Staging Account
│   └── Development Account
└── Infrastructure OU
    ├── Network Account (Transit Gateway)
    └── Shared Services Account (AD, DNS)
```

---

## Service Control Policies (SCPs)

### Prevent Security Service Disablement

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "guardduty:DeleteDetector",
        "securityhub:DisableSecurityHub",
        "config:DeleteConfigurationRecorder",
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

### Restrict Regions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["us-east-1", "us-west-2"]
        }
      }
    }
  ]
}
```

---

## AWS Control Tower

### Benefits
- Pre-configured multi-account environment
- Automated account provisioning (Account Factory)
- Guardrails (preventive and detective)
- Dashboard for compliance
- Integrated with AWS Organizations

### Enable Control Tower

1. Navigate to AWS Control Tower console
2. Click "Set up landing zone"
3. Specify:
   - Home region
   - Log Archive account email
   - Audit account email
4. Review and confirm

---

## Centralized Logging

### CloudTrail to Central S3 Bucket

```bash
# In Log Archive account, create S3 bucket with policy allowing organization
aws s3api create-bucket \
  --bucket org-cloudtrail-logs-123456789012 \
  --region us-east-1

# Create organization trail in management account
aws cloudtrail create-trail \
  --name organization-trail \
  --s3-bucket-name org-cloudtrail-logs-123456789012 \
  --is-multi-region-trail \
  --is-organization-trail
```

---

## Best Practices

- [ ] Never use management account for workloads
- [ ] Implement SCPs at OU level
- [ ] Use AWS Control Tower for governance
- [ ] Centralize logs in dedicated account
- [ ] Enable all security services at organization level
- [ ] Use cross-account roles with MFA
- [ ] Automate account creation

---

[← Previous: Compute Security](./phase-7-compute-security.md) | [Back to Roadmap](../ROADMAP.md) | [Next: Incident Response →](./phase-9-incident-response.md)
