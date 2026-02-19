# Phase 7: Compute Security

## Overview
Securing compute resources (EC2, Lambda, containers) is essential for overall cloud security.

## Learning Objectives
- Secure EC2 instances
- Use IMDSv2 to prevent SSRF attacks
- Implement SSM Session Manager
- Secure Lambda functions
- Protect containerized applications

---

## EC2 Security Best Practices

### Use IMDSv2

```bash
# Require IMDSv2 on instance
aws ec2 modify-instance-metadata-options \
  --instance-id i-1234567890abcdef0 \
  --http-tokens required \
  --http-put-response-hop-limit 1
```

### Use SSM Session Manager (No SSH Keys!)

```bash
# Connect to instance via SSM
aws ssm start-session --target i-1234567890abcdef0
```

**Benefits:**
- No SSH keys to manage
- All sessions logged in CloudTrail
- Can restrict to specific users
- Works through private subnets

---

## Systems Manager Patch Manager

```bash
# Create patch baseline
aws ssm create-patch-baseline \
  --name "Production-Baseline" \
  --approval-rules "PatchRules=[{PatchFilterGroup={PatchFilters=[{Key=CLASSIFICATION,Values=Security}]},ApproveAfterDays=7}]"

# Create maintenance window
aws ssm create-maintenance-window \
  --name "Production-Patching" \
  --schedule "cron(0 2 ? * SUN *)" \
  --duration 4 \
  --cutoff 1
```

---

## Lambda Security

### Secure Lambda Function

```python
import boto3
import os
import json

# Get secret from Secrets Manager (NOT environment variables!)
secrets_client = boto3.client('secretsmanager')

def lambda_handler(event, context):
    # Retrieve secret
    secret_response = secrets_client.get_secret_value(
        SecretId=os.environ['SECRET_ARN']
    )
    secret = json.loads(secret_response['SecretString'])
    
    # Use least privilege IAM role
    # Lambda execution role should have minimal permissions
    
    return {
        'statusCode': 200,
        'body': 'Success'
    }
```

### Lambda Security Checklist

- [ ] Use least privilege execution role
- [ ] Store secrets in Secrets Manager
- [ ] Enable VPC configuration only when needed
- [ ] Set appropriate timeout and memory limits
- [ ] Enable X-Ray tracing
- [ ] Validate all inputs
- [ ] Use Lambda layers for dependencies

---

## Container Security (ECS/EKS)

### ECR Image Scanning

```bash
# Enable scan on push
aws ecr put-image-scanning-configuration \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true

# Get scan findings
aws ecr describe-image-scan-findings \
  --repository-name my-app \
  --image-id imageTag=latest
```

### EKS Security Best Practices

- [ ] Use IAM Roles for Service Accounts (IRSA)
- [ ] Implement Pod Security Standards
- [ ] Enable audit logging
- [ ] Use Network Policies
- [ ] Scan images before deployment
- [ ] Use minimal base images (distroless)

---

## Best Practices

- [ ] Disable IMDSv1, use IMDSv2 only
- [ ] Use SSM Session Manager instead of SSH
- [ ] Implement automated patching
- [ ] Create hardened Golden AMIs
- [ ] Enable detailed monitoring
- [ ] Use IAM roles for applications

---

[← Previous: Secrets](./phase-6-secrets-management.md) | [Back to Roadmap](../ROADMAP.md) | [Next: Multi-Account →](./phase-8-multi-account.md)
