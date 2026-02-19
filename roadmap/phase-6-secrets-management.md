# Phase 6: Secrets & Credentials Management

## Overview
Proper secrets management is critical for security. Never hardcode credentials!

## Learning Objectives
- Use AWS Secrets Manager for secret storage
- Implement automatic secret rotation
- Use Parameter Store for configuration
- Secure application credentials

---

## Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| Cost | $0.40/secret/month | Free (Standard) |
| Rotation | Automatic | Manual |
| Use Case | DB credentials, API keys | App config |

---

## Store Secret in Secrets Manager

```bash
# Create secret
aws secretsmanager create-secret \
  --name prod/db/credentials \
  --secret-string '{"username":"admin","password":"MySecurePassword123!"}'

# Retrieve secret
aws secretsmanager get-secret-value \
  --secret-id prod/db/credentials
```

---

## Automatic Rotation

```bash
# Enable rotation (Lambda function required)
aws secretsmanager rotate-secret \
  --secret-id prod/db/credentials \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRotation \
  --rotation-rules AutomaticallyAfterDays=30
```

---

## Best Practices

- [ ] Never hardcode credentials
- [ ] Use IAM roles for AWS access
- [ ] Rotate secrets regularly (30-90 days)
- [ ] Use Secrets Manager for sensitive data
- [ ] Use Parameter Store for non-sensitive config
- [ ] Enable CloudTrail logging for secret access

---

[← Previous: Compliance](./phase-5-compliance.md) | [Back to Roadmap](../ROADMAP.md) | [Next: Compute Security →](./phase-7-compute-security.md)
