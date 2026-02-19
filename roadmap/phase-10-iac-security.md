# Phase 10: Infrastructure as Code (IaC) Security

## Overview
Secure IaC practices ensure infrastructure is deployed securely and consistently.

## Learning Objectives
- Scan IaC for security issues
- Implement policy as code
- Secure CI/CD pipelines
- Manage secrets in IaC
- Detect configuration drift

---

## IaC Security Scanning Tools

| Tool | Language | Features |
|------|----------|----------|
| **Checkov** | Python | Multi-cloud, 1000+ policies |
| **tfsec** | Go | Terraform-specific, fast |
| **cfn-nag** | Ruby | CloudFormation scanning |
| **Semgrep** | Python | Custom rules |

---

## Scan Terraform with tfsec

```bash
# Install tfsec
brew install tfsec

# Scan current directory
tfsec .

# Output as JSON
tfsec --format json . > tfsec-results.json
```

## Scan with Checkov

```bash
# Install Checkov
pip install checkov

# Scan Terraform
checkov -d . --framework terraform

# Scan CloudFormation
checkov -f template.yaml --framework cloudformation
```

---

## Common IaC Security Issues

### 1. Hardcoded Secrets ❌

```hcl
# BAD - Never do this!
resource "aws_db_instance" "database" {
  username = "admin"
  password = "SuperSecret123!"
}
```

```hcl
# GOOD - Use Secrets Manager
data "aws_secretsmanager_secret_version" "db_creds" {
  secret_id = "prod/db/credentials"
}

resource "aws_db_instance" "database" {
  username = jsondecode(data.aws_secretsmanager_secret_version.db_creds.secret_string)["username"]
  password = jsondecode(data.aws_secretsmanager_secret_version.db_creds.secret_string)["password"]
}
```

### 2. Public S3 Buckets ❌

```hcl
# BAD
resource "aws_s3_bucket" "data" {
  bucket = "my-data"
  acl    = "public-read"  # Dangerous!
}
```

```hcl
# GOOD
resource "aws_s3_bucket" "data" {
  bucket = "my-data"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### 3. Overly Permissive Security Groups ❌

```hcl
# BAD
resource "aws_security_group" "web" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

```hcl
# GOOD
resource "aws_security_group" "web" {
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS from internet"
  }
}
```

---

## Secure CI/CD Pipeline

```yaml
# GitHub Actions example
name: Terraform Security Scan

on:
  pull_request:
    paths:
      - '**.tf'

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          soft_fail: false
      
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
```

---

## Terraform State Security

```hcl
# Secure S3 backend
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:123456789012:key/xxxxx"
    dynamodb_table = "terraform-locks"
  }
}
```

**State File Security:**
- [ ] Store in S3 with encryption
- [ ] Enable versioning
- [ ] Use DynamoDB for locking
- [ ] Restrict access to state files
- [ ] Never commit state to version control

---

## Best Practices

- [ ] Scan IaC before deployment
- [ ] Use modules with security built-in
- [ ] Implement peer review
- [ ] Store secrets properly
- [ ] Enable drift detection
- [ ] Use CI/CD for deployments
- [ ] Separate environments (dev, staging, prod)

---

[← Previous: Incident Response](./phase-9-incident-response.md) | [Back to Roadmap](../ROADMAP.md)
