# 🔒 Phase 4: Data Security

> **Protect your data at rest and in transit**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Data is your most valuable asset. This phase focuses on encryption strategies, key management, S3 security, and secrets management to ensure your data remains confidential and secure throughout its lifecycle.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Implement encryption at rest and in transit
- ✅ Master AWS Key Management Service (KMS)
- ✅ Understand AWS CloudHSM use cases
- ✅ Secure S3 buckets comprehensively
- ✅ Use AWS Certificate Manager (ACM)
- ✅ Manage secrets with Secrets Manager and Parameter Store
- ✅ Apply data classification and DLP strategies

## 📚 Topics & Progress Tracker

### Week 1: Encryption Fundamentals & AWS KMS

#### Encryption Basics
- [ ] Understand encryption at rest vs in transit
- [ ] Symmetric vs Asymmetric encryption review
- [ ] Envelope encryption concept
- [ ] Key rotation strategies
- [ ] Compliance requirements (FIPS 140-2)

**Encryption Architecture:**
```
┌─────────────────────────────────────────────────────┐
│         ENCRYPTION IN AWS                           │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Data at Rest:                                      │
│  ├─ EBS volumes (KMS)                              │
│  ├─ S3 objects (SSE-S3, SSE-KMS, SSE-C)           │
│  ├─ RDS databases (KMS)                            │
│  ├─ DynamoDB tables (KMS)                          │
│  └─ EFS file systems (KMS)                         │
│                                                     │
│  Data in Transit:                                   │
│  ├─ TLS/SSL (Certificate Manager)                  │
│  ├─ VPN (Site-to-Site, Client VPN)                │
│  ├─ API Gateway (TLS)                              │
│  └─ CloudFront (HTTPS, Field-level encryption)     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

> **🔑 Key Takeaway:** Always encrypt sensitive data. Use AWS KMS for key management - never manage encryption keys manually.

#### AWS KMS (Key Management Service)
- [ ] Customer Master Keys (CMKs) - now called KMS keys
- [ ] AWS managed keys vs Customer managed keys
- [ ] Key policies and grants
- [ ] Multi-region keys
- [ ] Automatic key rotation
- [ ] Key deletion and recovery

**KMS Key Types:**
```
KMS Keys:
├─ AWS Managed Keys
│  └─ Created automatically (aws/s3, aws/ebs, etc.)
│  └─ Free, automatic rotation every year
│
├─ Customer Managed Keys
│  └─ You create and manage
│  └─ $1/month + usage costs
│  └─ Full control over key policies
│
└─ AWS Owned Keys
   └─ Used by AWS services
   └─ Not visible to you
   └─ No cost
```

**Creating and Using KMS Keys:**
```bash
# Create KMS key
aws kms create-key \
  --description "My application encryption key" \
  --key-policy file://key-policy.json

# Create alias
aws kms create-alias \
  --alias-name alias/my-app-key \
  --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab

# Enable automatic key rotation
aws kms enable-key-rotation \
  --key-id 1234abcd-12ab-34cd-56ef-1234567890ab

# Encrypt data
aws kms encrypt \
  --key-id alias/my-app-key \
  --plaintext "sensitive data" \
  --output text \
  --query CiphertextBlob

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted-data \
  --output text \
  --query Plaintext | base64 --decode
```

**KMS Key Policy Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key for encryption",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyAppRole"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow attachment of persistent resources",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyAppRole"
      },
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": "*",
      "Condition": {
        "Bool": {
          "kms:GrantIsForAWSResource": "true"
        }
      }
    }
  ]
}
```

#### Envelope Encryption
- [ ] Understand how envelope encryption works
- [ ] Data encryption keys (DEK)
- [ ] Key encryption keys (KEK)
- [ ] Performance benefits

**Envelope Encryption Flow:**
```
┌─────────────────────────────────────────────────┐
│        ENVELOPE ENCRYPTION PROCESS              │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. Request Data Key from KMS                   │
│     ┌──────┐          ┌─────────┐             │
│     │ App  │─────────►│   KMS   │             │
│     └──────┘          └─────────┘             │
│                                                 │
│  2. KMS Returns:                                │
│     • Plaintext Data Key                        │
│     • Encrypted Data Key (with CMK)             │
│                                                 │
│  3. App Encrypts Data with Plaintext Key        │
│     [Data] + [Plaintext Key] = [Encrypted Data] │
│                                                 │
│  4. Store:                                      │
│     • Encrypted Data                            │
│     • Encrypted Data Key                        │
│                                                 │
│  5. Delete Plaintext Key from Memory            │
│                                                 │
└─────────────────────────────────────────────────┘
```

#### AWS CloudHSM
- [ ] Understand HSM vs KMS differences
- [ ] FIPS 140-2 Level 3 compliance
- [ ] Dedicated hardware security module
- [ ] Use cases for CloudHSM
- [ ] Cluster management

**KMS vs CloudHSM:**
| Feature | AWS KMS | AWS CloudHSM |
|---------|---------|--------------|
| **Multi-tenancy** | Shared | Dedicated |
| **FIPS 140-2** | Level 2 | Level 3 |
| **Key Control** | AWS + You | You only |
| **Price** | $1/key/month | $1.45/hour per HSM |
| **AWS Integration** | Native | Custom |
| **Use Case** | General purpose | Regulatory requirements |

### Week 2: S3 Security

#### S3 Bucket Security Checklist
- [ ] Block public access (account and bucket level)
- [ ] Enable versioning
- [ ] Enable MFA Delete
- [ ] Configure bucket policies
- [ ] Use S3 Access Points
- [ ] Enable server access logging
- [ ] Enable AWS CloudTrail for S3 data events

**S3 Security Layers:**
```
┌──────────────────────────────────────────────┐
│      S3 SECURITY DEFENSE IN DEPTH            │
├──────────────────────────────────────────────┤
│ 1. Account-Level Block Public Access        │
│    └─ Prevents all public access settings   │
│                                              │
│ 2. Bucket-Level Block Public Access         │
│    └─ Per-bucket public access control      │
│                                              │
│ 3. Bucket Policy                             │
│    └─ Resource-based permissions             │
│                                              │
│ 4. IAM Policies                              │
│    └─ Identity-based permissions             │
│                                              │
│ 5. S3 Access Control Lists (ACLs)           │
│    └─ Object and bucket-level ACLs           │
│                                              │
│ 6. Encryption (SSE-S3, SSE-KMS, SSE-C)      │
│    └─ Protect data at rest                  │
│                                              │
│ 7. TLS/SSL                                   │
│    └─ Protect data in transit               │
└──────────────────────────────────────────────┘
```

**Block Public Access (CLI):**
```bash
# Enable block public access at account level
aws s3control put-public-access-block \
  --account-id 123456789012 \
  --public-access-block-configuration \
    BlockPublicAcls=true,\
    IgnorePublicAcls=true,\
    BlockPublicPolicy=true,\
    RestrictPublicBuckets=true

# Enable block public access at bucket level
aws s3api put-public-access-block \
  --bucket my-secure-bucket \
  --public-access-block-configuration \
    BlockPublicAcls=true,\
    IgnorePublicAcls=true,\
    BlockPublicPolicy=true,\
    RestrictPublicBuckets=true
```

#### S3 Encryption Options
- [ ] SSE-S3 (Server-Side Encryption with S3 managed keys)
- [ ] SSE-KMS (Server-Side Encryption with KMS)
- [ ] SSE-C (Server-Side Encryption with Customer-provided keys)
- [ ] Client-side encryption
- [ ] Default encryption settings

**S3 Encryption Comparison:**
| Type | Key Management | Use Case | Cost |
|------|----------------|----------|------|
| **SSE-S3** | AWS managed | General purpose | Free |
| **SSE-KMS** | KMS managed | Audit trail, key control | KMS charges apply |
| **SSE-C** | Customer managed | Full control | Transfer overhead |
| **Client-side** | Customer managed | Encrypt before upload | App complexity |

**Enforce Encryption in Transit:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-secure-bucket",
        "arn:aws:s3:::my-secure-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

**Enforce SSE-KMS Encryption:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-secure-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```

#### S3 Versioning and MFA Delete
- [ ] Enable versioning for data protection
- [ ] Understand version lifecycle
- [ ] Configure MFA Delete for critical buckets
- [ ] Implement lifecycle policies

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Enable MFA Delete (requires root account)
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled,MFADelete=Enabled \
  --mfa "arn:aws:iam::123456789012:mfa/root-account-mfa-device 123456"
```

#### S3 Access Points
- [ ] Simplify data access management
- [ ] Create access points with different policies
- [ ] VPC-only access points
- [ ] Cross-account access points

```bash
# Create S3 Access Point
aws s3control create-access-point \
  --account-id 123456789012 \
  --name my-access-point \
  --bucket my-bucket \
  --vpc-configuration VpcId=vpc-xxxxx
```

### Week 3: Secrets Management

#### AWS Secrets Manager
- [ ] Store database credentials
- [ ] Automatic rotation
- [ ] Integration with RDS
- [ ] Cross-account access
- [ ] Audit with CloudTrail

**Creating and Using Secrets:**
```bash
# Create secret
aws secretsmanager create-secret \
  --name prod/db/password \
  --description "Production database password" \
  --secret-string '{"username":"admin","password":"SuperSecretPass123!"}'

# Retrieve secret
aws secretsmanager get-secret-value \
  --secret-id prod/db/password \
  --query SecretString \
  --output text

# Enable automatic rotation
aws secretsmanager rotate-secret \
  --secret-id prod/db/password \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRotation \
  --rotation-rules AutomaticallyAfterDays=30
```

**Python Example - Using Secrets Manager:**
```python
import boto3
import json
from botocore.exceptions import ClientError

def get_secret(secret_name):
    """Retrieve secret from AWS Secrets Manager"""
    client = boto3.client('secretsmanager', region_name='us-east-1')
    
    try:
        response = client.get_secret_value(SecretId=secret_name)
        secret = json.loads(response['SecretString'])
        return secret
    except ClientError as e:
        raise Exception(f"Error retrieving secret: {e}")

# Usage
db_credentials = get_secret('prod/db/password')
username = db_credentials['username']
password = db_credentials['password']
```

#### AWS Systems Manager Parameter Store
- [ ] Store configuration data and secrets
- [ ] Standard vs Advanced parameters
- [ ] Parameter tiers and pricing
- [ ] Integration with other AWS services
- [ ] Parameter policies

**Parameter Store vs Secrets Manager:**
| Feature | Parameter Store | Secrets Manager |
|---------|----------------|-----------------|
| **Cost** | Free (standard) / $0.05/param (advanced) | $0.40/secret/month |
| **Size Limit** | 4 KB (standard) / 8 KB (advanced) | 64 KB |
| **Rotation** | Manual | Automatic |
| **RDS Integration** | No | Yes |
| **Use Case** | Config + secrets | Secrets only |

**Parameter Store Commands:**
```bash
# Create parameter
aws ssm put-parameter \
  --name "/prod/app/db-connection" \
  --value "mongodb://prod-db:27017" \
  --type "SecureString" \
  --tier "Standard"

# Retrieve parameter
aws ssm get-parameter \
  --name "/prod/app/db-connection" \
  --with-decryption \
  --query "Parameter.Value" \
  --output text

# Get parameters by path
aws ssm get-parameters-by-path \
  --path "/prod/app/" \
  --with-decryption
```

> **💡 Pro Tip:** Use Secrets Manager for credentials that need rotation (like database passwords). Use Parameter Store for general configuration and secrets that don't require rotation.

#### Secrets Management Best Practices
- [ ] Never hardcode secrets in code
- [ ] Use environment variables or secret services
- [ ] Rotate secrets regularly (30-90 days)
- [ ] Audit secret access with CloudTrail
- [ ] Use least privilege for secret access
- [ ] Implement secret versioning
- [ ] Test rotation procedures

### Week 4: Certificate Management & Data Protection

#### AWS Certificate Manager (ACM)
- [ ] Request public certificates (free)
- [ ] Import third-party certificates
- [ ] Automatic renewal
- [ ] Integration with CloudFront, ALB, API Gateway
- [ ] Certificate transparency logging

```bash
# Request certificate
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com \
  --validation-method DNS

# List certificates
aws acm list-certificates

# Describe certificate
aws acm describe-certificate \
  --certificate-arn arn:aws:acm:us-east-1:123456789012:certificate/xxxxx
```

#### Data Classification
- [ ] Public vs Internal vs Confidential vs Restricted
- [ ] Tagging strategy for classification
- [ ] Automated classification with Macie
- [ ] Data handling procedures

**Data Classification Levels:**
```
┌────────────────────────────────────────────────┐
│      DATA CLASSIFICATION FRAMEWORK             │
├────────────────────────────────────────────────┤
│ PUBLIC                                         │
│  └─ Marketing materials, public docs           │
│     Tagging: data-class:public                 │
│                                                │
│ INTERNAL                                       │
│  └─ Internal docs, non-sensitive data          │
│     Tagging: data-class:internal               │
│                                                │
│ CONFIDENTIAL                                   │
│  └─ Business data, customer info (encrypted)   │
│     Tagging: data-class:confidential           │
│                                                │
│ RESTRICTED                                     │
│  └─ PII, PHI, financial data (encrypted+MFA)   │
│     Tagging: data-class:restricted             │
└────────────────────────────────────────────────┘
```

#### Data Loss Prevention (DLP)
- [ ] Implement access controls
- [ ] Enable encryption
- [ ] Monitor data access patterns
- [ ] Use Macie for sensitive data discovery
- [ ] Audit data transfers

#### RDS Encryption
- [ ] Enable encryption at rest
- [ ] SSL/TLS for connections
- [ ] IAM database authentication
- [ ] Encrypted snapshots
- [ ] Read replica encryption

```bash
# Create encrypted RDS instance
aws rds create-db-instance \
  --db-instance-identifier encrypted-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --allocated-storage 20 \
  --storage-encrypted \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/xxxxx \
  --master-username admin \
  --master-user-password password
```

## 🛠️ Hands-On Labs

### Lab 1: Implement S3 Bucket Security
```bash
#!/bin/bash
# Comprehensive S3 bucket security setup

BUCKET_NAME="my-secure-app-bucket"

# Create bucket
aws s3api create-bucket \
  --bucket $BUCKET_NAME \
  --region us-east-1

# Block all public access
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,\
    BlockPublicPolicy=true,RestrictPublicBuckets=true

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration \
    '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms"}}]}'

# Enable logging
aws s3api put-bucket-logging \
  --bucket $BUCKET_NAME \
  --bucket-logging-status \
    '{"LoggingEnabled":{"TargetBucket":"my-logs-bucket","TargetPrefix":"s3-access-logs/"}}'

# Apply bucket policy
cat > bucket-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::$BUCKET_NAME",
        "arn:aws:s3:::$BUCKET_NAME/*"
      ],
      "Condition": {
        "Bool": {"aws:SecureTransport": "false"}
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket $BUCKET_NAME \
  --policy file://bucket-policy.json

echo "✅ Secure S3 bucket created: $BUCKET_NAME"
```

### Lab 2: KMS Key Rotation Automation
```python
#!/usr/bin/env python3
"""
Check and enable KMS key rotation for all customer-managed keys
"""
import boto3

def enable_key_rotation():
    kms = boto3.client('kms')
    
    # List all customer-managed keys
    paginator = kms.get_paginator('list_keys')
    
    for page in paginator.paginate():
        for key in page['Keys']:
            key_id = key['KeyId']
            
            try:
                # Get key metadata
                metadata = kms.describe_key(KeyId=key_id)
                
                # Only process customer-managed keys
                if metadata['KeyMetadata']['KeyManager'] == 'CUSTOMER':
                    # Check rotation status
                    rotation = kms.get_key_rotation_status(KeyId=key_id)
                    
                    if not rotation['KeyRotationEnabled']:
                        print(f"⚠️  Rotation disabled for key: {key_id}")
                        print(f"   Enabling rotation...")
                        
                        kms.enable_key_rotation(KeyId=key_id)
                        print(f"✅ Rotation enabled for key: {key_id}")
                    else:
                        print(f"✓ Rotation already enabled for key: {key_id}")
            
            except Exception as e:
                print(f"❌ Error processing key {key_id}: {str(e)}")

if __name__ == "__main__":
    enable_key_rotation()
```

### Lab 3: Secrets Rotation Lambda
```python
import boto3
import json

def lambda_handler(event, context):
    """
    Lambda function to rotate RDS database password
    """
    service_client = boto3.client('secretsmanager')
    
    # Get the token from the event
    token = event['Token']
    
    # Get secret metadata
    metadata = service_client.describe_secret(
        SecretId=event['SecretId']
    )
    
    if not metadata['RotationEnabled']:
        raise ValueError("Secret is not enabled for rotation")
    
    # Implement rotation steps
    step = event['Step']
    
    if step == "createSecret":
        create_secret(service_client, event['SecretId'], token)
    elif step == "setSecret":
        set_secret(service_client, event['SecretId'], token)
    elif step == "testSecret":
        test_secret(service_client, event['SecretId'], token)
    elif step == "finishSecret":
        finish_secret(service_client, event['SecretId'], token)
    
    return {'statusCode': 200}

def create_secret(service_client, secret_id, token):
    """Generate new secret value"""
    # Implementation here
    pass

def set_secret(service_client, secret_id, token):
    """Update the secret in the service"""
    # Implementation here
    pass

def test_secret(service_client, secret_id, token):
    """Test the new secret"""
    # Implementation here
    pass

def finish_secret(service_client, secret_id, token):
    """Finalize the rotation"""
    # Implementation here
    pass
```

## ✅ Phase 4 Checklist

Before moving to Phase 5, ensure you can:
- [ ] Explain the difference between encryption at rest and in transit
- [ ] Create and manage KMS keys with proper policies
- [ ] Secure S3 buckets using multiple layers of protection
- [ ] Implement secrets rotation with Secrets Manager
- [ ] Choose between Secrets Manager and Parameter Store appropriately
- [ ] Request and manage SSL/TLS certificates with ACM
- [ ] Apply data classification principles
- [ ] Encrypt RDS databases and snapshots

## 🎯 Success Criteria

You're ready for Phase 5 when you can:
1. Design a comprehensive data encryption strategy
2. Secure an S3 bucket from scratch
3. Implement automated secrets rotation
4. Troubleshoot encryption-related issues
5. Explain envelope encryption to a technical peer

> **💡 Pro Tip:** Encryption should be the default, not the exception. Always ask "Why isn't this encrypted?" rather than "Should this be encrypted?"

## 🔜 What's Next?

Fantastic work securing your data! Now let's learn to detect threats and monitor your environment.

**Next:** [Phase 5: Monitoring & Threat Detection](phase-5-monitoring-threat-detection.md) - Set up comprehensive security monitoring with CloudTrail, GuardDuty, and more

---

**Remember:** "Data breaches don't happen to encrypted data, they happen to unencrypted data!" 🔒
