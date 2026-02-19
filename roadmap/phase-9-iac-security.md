# 🏗️ Phase 9: Infrastructure as Code Security

> **Secure your infrastructure code and CI/CD pipelines**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Infrastructure as Code (IaC) is essential for modern cloud operations, but it introduces new security challenges. This phase covers security best practices for Terraform, CloudFormation, policy-as-code tools, and integrating security into CI/CD pipelines.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Write secure Terraform code
- ✅ Write secure CloudFormation templates
- ✅ Use Checkov for policy-as-code
- ✅ Scan IaC with tfsec and cfn-nag
- ✅ Implement CloudFormation Guard
- ✅ Integrate security scanning into CI/CD
- ✅ Apply shift-left security principles
- ✅ Manage secrets in IaC

## 📚 Topics & Progress Tracker

### Week 1: Terraform Security

#### Terraform Best Practices
- [ ] Use remote state with encryption
- [ ] Enable state locking
- [ ] Never commit secrets to version control
- [ ] Use variables for sensitive data
- [ ] Pin provider versions
- [ ] Use modules for reusability

**Secure Terraform Configuration:**
```hcl
# Backend configuration with encryption
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:123456789012:key/xxxxx"
    dynamodb_table = "terraform-state-lock"
  }
  
  required_version = "~> 1.5"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Provider configuration
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Owner       = var.team_name
    }
  }
}
```

#### Secure S3 Bucket in Terraform
```hcl
resource "aws_s3_bucket" "secure_bucket" {
  bucket = "my-secure-bucket-${var.environment}"
  
  tags = {
    Name        = "SecureBucket"
    Environment = var.environment
  }
}

# Block public access
resource "aws_s3_bucket_public_access_block" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Enable versioning
resource "aws_s3_bucket_versioning" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.bucket_key.arn
    }
  }
}

# Enable logging
resource "aws_s3_bucket_logging" "secure_bucket" {
  bucket = aws_s3_bucket.secure_bucket.id

  target_bucket = aws_s3_bucket.log_bucket.id
  target_prefix = "access-logs/"
}
```

> **🔑 Key Takeaway:** Treat infrastructure code like application code - version control, code review, and security scanning are essential.

#### Managing Secrets in Terraform
- [ ] Use AWS Secrets Manager data source
- [ ] Use environment variables
- [ ] Avoid hardcoding sensitive values
- [ ] Use Terraform Cloud/Enterprise for secure variable storage

```hcl
# Fetch secret from AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/master-password"
}

locals {
  db_credentials = jsondecode(
    data.aws_secretsmanager_secret_version.db_password.secret_string
  )
}

# Use in RDS configuration
resource "aws_db_instance" "main" {
  identifier        = "mydb"
  engine            = "mysql"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  
  username = local.db_credentials.username
  password = local.db_credentials.password
  
  # Security settings
  storage_encrypted         = true
  kms_key_id               = aws_kms_key.rds.arn
  enabled_cloudwatch_logs_exports = ["error", "slowquery"]
  
  skip_final_snapshot = false
  final_snapshot_identifier = "mydb-final-${formatdate("YYYY-MM-DD-hhmm", timestamp())}"
}
```

### Week 2: CloudFormation Security

#### CloudFormation Best Practices
- [ ] Use CloudFormation Guard for policy validation
- [ ] Enable termination protection
- [ ] Use stack policies
- [ ] Implement change sets
- [ ] Use nested stacks for modularity

**Secure CloudFormation Template:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Secure S3 bucket with encryption and logging'

Parameters:
  BucketName:
    Type: String
    Description: Name of the S3 bucket
  
  KMSKeyArn:
    Type: String
    Description: ARN of KMS key for encryption

Resources:
  SecureBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
              KMSMasterKeyID: !Ref KMSKeyArn
      VersioningConfiguration:
        Status: Enabled
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      LoggingConfiguration:
        DestinationBucketName: !Ref LogBucket
        LogFilePrefix: s3-access-logs/
      Tags:
        - Key: Environment
          Value: Production
        - Key: ManagedBy
          Value: CloudFormation

  SecureBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref SecureBucket
      PolicyDocument:
        Statement:
          - Sid: DenyInsecureTransport
            Effect: Deny
            Principal: '*'
            Action: 's3:*'
            Resource:
              - !GetAtt SecureBucket.Arn
              - !Sub '${SecureBucket.Arn}/*'
            Condition:
              Bool:
                'aws:SecureTransport': 'false'

  LogBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${BucketName}-logs'
      AccessControl: LogDeliveryWrite
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

Outputs:
  BucketName:
    Description: Name of the created bucket
    Value: !Ref SecureBucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'
```

#### Stack Policies
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "Update:Replace",
        "Update:Delete"
      ],
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

### Week 3: Policy-as-Code Tools

#### Checkov - Policy-as-Code
- [ ] Install Checkov
- [ ] Scan Terraform code
- [ ] Scan CloudFormation templates
- [ ] Create custom policies
- [ ] Integrate into CI/CD

**Install and Run Checkov:**
```bash
# Install Checkov
pip install checkov

# Scan Terraform
checkov -d ./terraform --framework terraform

# Scan CloudFormation
checkov -f template.yaml --framework cloudformation

# Output as JSON
checkov -d ./terraform --output json

# Skip specific checks
checkov -d ./terraform --skip-check CKV_AWS_19,CKV_AWS_20

# Run only specific checks
checkov -d ./terraform --check CKV_AWS_18
```

**Custom Checkov Policy:**
```python
from checkov.common.models.enums import CheckResult, CheckCategories
from checkov.terraform.checks.resource.base_resource_check import BaseResourceCheck

class S3BucketEncrypted(BaseResourceCheck):
    def __init__(self):
        name = "Ensure S3 bucket is encrypted"
        id = "CKV_CUSTOM_1"
        supported_resources = ['aws_s3_bucket']
        categories = [CheckCategories.ENCRYPTION]
        super().__init__(name=name, id=id, categories=categories, supported_resources=supported_resources)

    def scan_resource_conf(self, conf):
        """
        Checks if S3 bucket has encryption enabled
        """
        if 'server_side_encryption_configuration' in conf:
            return CheckResult.PASSED
        return CheckResult.FAILED

check = S3BucketEncrypted()
```

#### tfsec - Terraform Security Scanner
- [ ] Install tfsec
- [ ] Scan for security issues
- [ ] Understand severity levels
- [ ] Fix reported issues

```bash
# Install tfsec
brew install tfsec  # macOS
# or
curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

# Run scan
tfsec .

# Output as JSON
tfsec . --format json

# Ignore specific checks
tfsec . --exclude aws-s3-enable-versioning

# Set minimum severity
tfsec . --minimum-severity HIGH
```

#### cfn-nag - CloudFormation Security Scanner
- [ ] Install cfn-nag
- [ ] Scan templates
- [ ] Understand violations
- [ ] Suppress false positives

```bash
# Install cfn-nag
gem install cfn-nag

# Scan template
cfn_nag_scan --input-path template.yaml

# Output as JSON
cfn_nag_scan --input-path template.yaml --output-format json

# Scan directory
cfn_nag_scan --input-path ./cloudformation/
```

#### CloudFormation Guard
- [ ] Write Guard rules
- [ ] Validate templates
- [ ] Use AWS managed rules
- [ ] Create custom rule sets

**CloudFormation Guard Rule:**
```
# S3 bucket must have encryption enabled
rule s3_bucket_encryption {
  AWS::S3::Bucket {
    Properties {
      BucketEncryption exists
      BucketEncryption {
        ServerSideEncryptionConfiguration exists
        ServerSideEncryptionConfiguration[*] {
          ServerSideEncryptionByDefault exists
          ServerSideEncryptionByDefault {
            SSEAlgorithm in ["AES256", "aws:kms"]
          }
        }
      }
    }
  }
}

# S3 bucket must block public access
rule s3_bucket_public_access {
  AWS::S3::Bucket {
    Properties {
      PublicAccessBlockConfiguration exists
      PublicAccessBlockConfiguration {
        BlockPublicAcls == true
        BlockPublicPolicy == true
        IgnorePublicAcls == true
        RestrictPublicBuckets == true
      }
    }
  }
}
```

**Run CloudFormation Guard:**
```bash
# Install cfn-guard
cargo install cfn-guard

# Validate template
cfn-guard validate \
  --rules my-rules.guard \
  --data template.yaml

# Use AWS managed rules
cfn-guard validate \
  --rules https://github.com/aws-cloudformation/aws-guard-rules-registry \
  --data template.yaml
```

### Week 4: CI/CD Security Integration

#### Secure CI/CD Pipeline
- [ ] Scan IaC in pull requests
- [ ] Fail builds on critical issues
- [ ] Require code review for infra changes
- [ ] Use branch protection
- [ ] Audit pipeline access

**GitHub Actions Security Workflow:**
```yaml
name: Infrastructure Security Scan

on:
  pull_request:
    paths:
      - 'terraform/**'
      - 'cloudformation/**'

jobs:
  terraform-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          soft_fail: false
          
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          framework: terraform
          soft_fail: false
          
      - name: Terraform Format Check
        run: |
          cd terraform
          terraform fmt -check -recursive
          
      - name: Terraform Validate
        run: |
          cd terraform
          terraform init -backend=false
          terraform validate

  cloudformation-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.0'
          
      - name: Install cfn-nag
        run: gem install cfn-nag
        
      - name: Scan CloudFormation
        run: cfn_nag_scan --input-path cloudformation/
        
      - name: Run CloudFormation Linter
        run: |
          pip install cfn-lint
          cfn-lint cloudformation/**/*.yaml

  security-gate:
    needs: [terraform-security, cloudformation-security]
    runs-on: ubuntu-latest
    steps:
      - name: Security Check Passed
        run: echo "All security checks passed!"
```

**GitLab CI/CD Security Pipeline:**
```yaml
stages:
  - validate
  - security-scan
  - plan
  - apply

terraform-validate:
  stage: validate
  image: hashicorp/terraform:latest
  script:
    - cd terraform
    - terraform init -backend=false
    - terraform fmt -check -recursive
    - terraform validate

tfsec-scan:
  stage: security-scan
  image: aquasec/tfsec:latest
  script:
    - tfsec terraform/ --format json > tfsec-report.json
  artifacts:
    reports:
      security: tfsec-report.json
  allow_failure: false

checkov-scan:
  stage: security-scan
  image: bridgecrew/checkov:latest
  script:
    - checkov -d terraform/ --output json > checkov-report.json
  artifacts:
    reports:
      security: checkov-report.json
  allow_failure: false

terraform-plan:
  stage: plan
  image: hashicorp/terraform:latest
  script:
    - cd terraform
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths:
      - terraform/tfplan
  only:
    - branches
  except:
    - main

terraform-apply:
  stage: apply
  image: hashicorp/terraform:latest
  script:
    - cd terraform
    - terraform init
    - terraform apply -auto-approve tfplan
  dependencies:
    - terraform-plan
  only:
    - main
  when: manual
```

#### Secret Scanning
- [ ] Use git-secrets
- [ ] Use gitleaks
- [ ] Use TruffleHog
- [ ] Pre-commit hooks
- [ ] Automated secret rotation

**Pre-commit Hook Configuration:**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/awslabs/git-secrets
    rev: master
    hooks:
      - id: git-secrets
        
  - repo: https://github.com/zricethezav/gitleaks
    rev: v8.16.0
    hooks:
      - id: gitleaks
        
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.77.0
    hooks:
      - id: terraform_fmt
      - id: terraform_validate
      - id: terraform_tfsec
      - id: terraform_checkov
```

**Install Pre-commit:**
```bash
# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run manually
pre-commit run --all-files
```

## 🛠️ Hands-On Labs

### Lab 1: Scan Terraform with Multiple Tools
```bash
#!/bin/bash
# Comprehensive Terraform security scan

echo "🔍 Running security scans on Terraform code..."

# tfsec
echo "\n=== tfsec ==="
tfsec . --format json > tfsec-results.json
tfsec .

# Checkov
echo "\n=== Checkov ==="
checkov -d . --framework terraform --output json > checkov-results.json
checkov -d . --framework terraform

# Terraform validate
echo "\n=== Terraform Validate ==="
terraform init -backend=false
terraform validate

# Terraform fmt check
echo "\n=== Terraform Format Check ==="
terraform fmt -check -recursive

echo "\n✅ Security scan complete!"
echo "Review results in tfsec-results.json and checkov-results.json"
```

### Lab 2: Create Security Policy Pipeline
```yaml
# security-pipeline.yml
name: Security Policy Enforcement

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  enforce-policies:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Tools
        run: |
          # Install tfsec
          curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash
          
          # Install Checkov
          pip install checkov
          
          # Install cfn-nag
          gem install cfn-nag
      
      - name: Scan Infrastructure Code
        id: scan
        run: |
          FAILED=0
          
          # Scan Terraform
          if [ -d "terraform" ]; then
            echo "Scanning Terraform..."
            tfsec terraform/ --soft-fail || FAILED=1
            checkov -d terraform/ --soft-fail || FAILED=1
          fi
          
          # Scan CloudFormation
          if [ -d "cloudformation" ]; then
            echo "Scanning CloudFormation..."
            cfn_nag_scan --input-path cloudformation/ || FAILED=1
          fi
          
          exit $FAILED
      
      - name: Comment on PR
        if: failure()
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '❌ Security scan failed. Please fix the issues and push again.'
            })
```

## ✅ Phase 9 Checklist

Before moving to Phase 10, ensure you can:
- [ ] Write secure Terraform configurations
- [ ] Write secure CloudFormation templates
- [ ] Use Checkov for policy enforcement
- [ ] Scan IaC with tfsec and cfn-nag
- [ ] Write CloudFormation Guard rules
- [ ] Integrate security scanning into CI/CD
- [ ] Implement pre-commit hooks for IaC
- [ ] Manage secrets securely in IaC

## 🎯 Success Criteria

You're ready for Phase 10 when you can:
1. Design a secure IaC workflow
2. Scan infrastructure code for security issues
3. Create custom security policies
4. Integrate security into CI/CD pipelines
5. Prevent secrets from being committed

> **💡 Pro Tip:** Shift security left! Catch issues in code before they reach production. Automate everything.

## 🔜 What's Next?

Great job securing your infrastructure code! Now let's secure compute workloads.

**Next:** [Phase 10: Compute Security](phase-10-compute-security.md) - Secure EC2, Lambda, and containers

---

**Remember:** "Secure infrastructure code = Secure infrastructure!" 🏗️🔒
