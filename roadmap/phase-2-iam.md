# 🔐 Phase 2: AWS IAM Deep Dive

> **Master Identity and Access Management - The foundation of AWS security**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

AWS Identity and Access Management (IAM) is the cornerstone of AWS security. Before you can secure anything else in AWS, you must master IAM. This phase covers everything from basic concepts to advanced patterns that professional cloud security engineers use daily.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Understand IAM users, groups, and roles
- ✅ Write and evaluate IAM policies
- ✅ Implement principle of least privilege
- ✅ Use Service Control Policies (SCPs) effectively
- ✅ Configure permission boundaries
- ✅ Utilize IAM Access Analyzer
- ✅ Apply IAM best practices in real-world scenarios

## 📚 Topics & Progress Tracker

### Week 1: IAM Fundamentals

#### IAM Identities
- [ ] Understand IAM users vs root user
- [ ] Create and manage IAM users
- [ ] Work with IAM groups for organization
- [ ] Implement IAM roles for services
- [ ] Configure federated access

**IAM Identity Types:**
```
┌────────────────────────────────────────────────────┐
│              IAM IDENTITY TYPES                    │
├────────────────────────────────────────────────────┤
│ Root User                                          │
│  └─ Full access to everything (use with MFA only!)│
│                                                    │
│ IAM Users                                          │
│  └─ Long-term credentials for people               │
│                                                    │
│ IAM Groups                                         │
│  └─ Collection of users (cannot be nested)         │
│                                                    │
│ IAM Roles                                          │
│  └─ Temporary credentials for services/users       │
│                                                    │
│ Federated Users (SSO)                             │
│  └─ External identity providers (SAML, OIDC)       │
└────────────────────────────────────────────────────┘
```

#### Authentication Methods
- [ ] Access keys (programmatic access)
- [ ] Console password (web access)
- [ ] Multi-Factor Authentication (MFA)
- [ ] Security tokens (temporary credentials)
- [ ] SSH keys for AWS CodeCommit

> **🔑 Key Takeaway:** Never use access keys for root account. Always use IAM users with MFA for human access.

#### IAM Best Practices - Users
- [ ] Enable MFA for all users
- [ ] Rotate credentials regularly (90 days)
- [ ] Use IAM groups for permission management
- [ ] Grant least privilege
- [ ] Disable unused credentials

**Lab: Create a Secure IAM User**
```bash
# Create IAM user
aws iam create-user --user-name john-developer

# Add user to group
aws iam add-user-to-group \
  --user-name john-developer \
  --group-name developers

# Create login profile with password
aws iam create-login-profile \
  --user-name john-developer \
  --password 'TempPassword123!' \
  --password-reset-required

# Enable MFA (requires manual step in console or MFA device)
```

### Week 2: IAM Policies

#### Policy Types
- [ ] Identity-based policies (attached to users, groups, roles)
- [ ] Resource-based policies (attached to resources like S3 buckets)
- [ ] Permission boundaries
- [ ] Service Control Policies (SCPs)
- [ ] Access Control Lists (ACLs)
- [ ] Session policies

**Policy Types Hierarchy:**
```
┌─────────────────────────────────────────────────────┐
│         IAM POLICY EVALUATION ORDER                 │
├─────────────────────────────────────────────────────┤
│ 1. Explicit DENY (always wins)                      │
│ 2. Organization SCPs (boundaries at org level)      │
│ 3. Resource-based policies (allow cross-account)    │
│ 4. Identity-based policies (user/role permissions)  │
│ 5. IAM permission boundaries (max permissions)      │
│ 6. Session policies (temporary credential limits)   │
│ 7. Implicit DENY (default if nothing allows)        │
└─────────────────────────────────────────────────────┘

Decision Logic: Explicit DENY > ALLOW > Implicit DENY
```

#### IAM Policy Structure
- [ ] Understand policy JSON structure
- [ ] Effect: Allow vs Deny
- [ ] Principal: Who/what can access
- [ ] Action: What operations are allowed
- [ ] Resource: Which AWS resources
- [ ] Condition: When access is allowed

**IAM Policy Anatomy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DescriptiveStatementID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/username"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
```

#### Writing IAM Policies
- [ ] Use IAM policy generator
- [ ] Understand wildcards (* and ?)
- [ ] Apply conditions effectively
- [ ] Use policy variables (${aws:username})
- [ ] Test policies with policy simulator

**Example: S3 Read-Only Access**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-secure-bucket",
        "arn:aws:s3:::my-secure-bucket/*"
      ]
    }
  ]
}
```

**Example: EC2 with Conditions**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": ["t2.micro", "t2.small"]
        },
        "StringLike": {
          "ec2:Region": "us-east-*"
        }
      }
    }
  ]
}
```

**Example: Allow User to Manage Own Credentials**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:GetUser",
        "iam:ChangePassword",
        "iam:CreateAccessKey",
        "iam:DeleteAccessKey",
        "iam:ListAccessKeys",
        "iam:UpdateAccessKey"
      ],
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:ListUsers",
        "iam:ListVirtualMFADevices"
      ],
      "Resource": "*"
    }
  ]
}
```

> **💡 Pro Tip:** Always start with least privilege and add permissions as needed. It's easier to grant access than revoke it.

#### Policy Conditions
- [ ] String conditions (StringEquals, StringLike)
- [ ] Date conditions (DateGreaterThan, DateLessThan)
- [ ] IP address conditions (IpAddress, NotIpAddress)
- [ ] Boolean conditions
- [ ] ARN conditions

**Common Condition Examples:**

```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": ["203.0.113.0/24", "198.51.100.0/24"]
    }
  }
}
```

```json
{
  "Condition": {
    "DateGreaterThan": {
      "aws:CurrentTime": "2024-01-01T00:00:00Z"
    },
    "DateLessThan": {
      "aws:CurrentTime": "2024-12-31T23:59:59Z"
    }
  }
}
```

```json
{
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "true"
    }
  }
}
```

### Week 3: IAM Roles & Advanced Concepts

#### IAM Roles
- [ ] Understand when to use roles vs users
- [ ] Create service roles (for EC2, Lambda, etc.)
- [ ] Cross-account access with roles
- [ ] Role assumption process
- [ ] Role trust policies

**When to Use Roles:**
- ✅ AWS services (EC2, Lambda, ECS)
- ✅ Cross-account access
- ✅ Federated users (SSO)
- ✅ Temporary access for external users
- ❌ Long-term credentials for applications

**Creating an EC2 Instance Role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Cross-Account Access Role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id-12345"
        }
      }
    }
  ]
}
```

> **🔑 Key Takeaway:** Always use External ID for third-party cross-account access to prevent the "confused deputy" problem.

#### Assuming Roles
- [ ] Use AWS CLI to assume roles
- [ ] Understand STS (Security Token Service)
- [ ] Configure role chaining
- [ ] Set maximum session duration

```bash
# Assume a role
aws sts assume-role \
  --role-arn "arn:aws:iam::123456789012:role/MyRole" \
  --role-session-name "my-session" \
  --duration-seconds 3600

# Use temporary credentials
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."
```

#### Permission Boundaries
- [ ] Understand permission boundaries concept
- [ ] Set maximum permissions for users/roles
- [ ] Combine with identity-based policies
- [ ] Use cases for permission boundaries

**Permission Boundary Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "ec2:*",
        "rds:*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "iam:*",
        "organizations:*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Service Control Policies (SCPs)
- [ ] Understand SCPs vs IAM policies
- [ ] Apply SCPs to AWS Organizations
- [ ] Create deny lists and allow lists
- [ ] Test SCP impact

**SCP Example - Deny Specific Regions:**
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
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2",
            "eu-west-1"
          ]
        }
      }
    }
  ]
}
```

**SCP Example - Require MFA:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

**SCP Example - Prevent Leaving Organization:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "organizations:LeaveOrganization"
      ],
      "Resource": "*"
    }
  ]
}
```

### Week 4: IAM Tools & Best Practices

#### IAM Access Analyzer
- [ ] Enable IAM Access Analyzer
- [ ] Analyze external access to resources
- [ ] Review findings and remediate
- [ ] Generate policies from CloudTrail logs

**Enable IAM Access Analyzer:**
```bash
aws accessanalyzer create-analyzer \
  --analyzer-name my-account-analyzer \
  --type ACCOUNT

# List findings
aws accessanalyzer list-findings \
  --analyzer-arn arn:aws:access-analyzer:us-east-1:123456789012:analyzer/my-account-analyzer
```

#### IAM Credential Report
- [ ] Generate credential reports
- [ ] Audit password ages
- [ ] Review access key usage
- [ ] Identify inactive users

```bash
# Generate credential report
aws iam generate-credential-report

# Download credential report
aws iam get-credential-report --output text --query Content | base64 --decode > credentials.csv
```

#### IAM Policy Simulator
- [ ] Test policies before deployment
- [ ] Validate effective permissions
- [ ] Debug access issues
- [ ] Simulate different scenarios

#### IAM Best Practices Checklist
- [ ] Root account: MFA enabled, no access keys, rarely used
- [ ] Users: Individual accounts, MFA enforced, strong passwords
- [ ] Groups: Use for permission management, avoid user-level policies
- [ ] Roles: For services and cross-account access
- [ ] Policies: Least privilege, specific resources, use conditions
- [ ] Credentials: Rotate regularly, disable unused, no hardcoded secrets
- [ ] Monitoring: Enable CloudTrail, review IAM activity, set alerts
- [ ] Permissions: Regular audits, remove unnecessary access, use Access Analyzer

**The Golden Rules:**
```
1. NEVER use root account for daily tasks
2. ALWAYS enable MFA (hardware tokens for production)
3. GRANT least privilege (start restrictive, expand as needed)
4. ROTATE credentials regularly (90 days maximum)
5. USE roles for applications (not access keys)
6. AUDIT regularly (monthly reviews minimum)
7. MONITOR access (CloudTrail + CloudWatch)
```

## 🛠️ Hands-On Labs

### Lab 1: Create Developer IAM Group
```bash
# Create group
aws iam create-group --group-name developers

# Attach AWS managed policies
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Create inline policy for additional restrictions
cat > developer-restrictions.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "iam:*",
        "organizations:*",
        "account:*"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam put-group-policy \
  --group-name developers \
  --policy-name RestrictIAM \
  --policy-document file://developer-restrictions.json
```

### Lab 2: S3 Bucket Policy with Encryption Requirement
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
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    },
    {
      "Sid": "RequireSecureTransport",
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

### Lab 3: Cross-Account Access Setup

**In Account A (Resource Account):**
```bash
# Create role that Account B can assume
cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "SecureExternalId123"
        }
      }
    }
  ]
}
EOF

aws iam create-role \
  --role-name CrossAccountS3Access \
  --assume-role-policy-document file://trust-policy.json

# Attach permissions
aws iam attach-role-policy \
  --role-name CrossAccountS3Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

**In Account B (Accessing Account):**
```bash
# Assume the role
aws sts assume-role \
  --role-arn "arn:aws:iam::111111111111:role/CrossAccountS3Access" \
  --role-session-name "s3-access-session" \
  --external-id "SecureExternalId123"
```

### Lab 4: Policy from CloudTrail Access Analyzer
```bash
# Generate policy from last 90 days of CloudTrail
aws accessanalyzer create-access-preview \
  --analyzer-arn arn:aws:access-analyzer:us-east-1:123456789012:analyzer/my-analyzer \
  --configurations file://access-preview-config.json
```

## 🔍 Common IAM Troubleshooting

### Access Denied Errors
```bash
# Check effective permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/john \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt

# Review CloudTrail for detailed error
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=john \
  --max-results 10
```

### Debugging Policy Evaluation
```
Step-by-step checklist:
1. Is there an explicit DENY? (Check SCPs, permission boundaries)
2. Is there an explicit ALLOW? (Check identity and resource policies)
3. Check conditions (IP restrictions, MFA requirements, etc.)
4. Review resource-based policies
5. Verify role trust relationships
6. Check session policies (if using temporary credentials)
```

## 📊 IAM Security Comparison Table

| Feature | IAM Users | IAM Roles | IAM Groups |
|---------|-----------|-----------|------------|
| Long-term credentials | ✅ Yes | ❌ No | N/A |
| Temporary credentials | ❌ No | ✅ Yes | N/A |
| Can login to console | ✅ Yes | ❌ No | ❌ No |
| Can be assumed | ❌ No | ✅ Yes | ❌ No |
| Has policies | ✅ Yes | ✅ Yes | ✅ Yes |
| Best for | People | Services/Apps | Organizing users |
| Max per account | 5,000 | 1,000 | 300 |

## ✅ Phase 2 Checklist

Before moving to Phase 3, ensure you can:
- [ ] Explain the difference between users, groups, and roles
- [ ] Write an IAM policy from scratch
- [ ] Evaluate which policy would take effect in a given scenario
- [ ] Configure cross-account access with roles
- [ ] Use IAM Access Analyzer to find security issues
- [ ] Apply permission boundaries effectively
- [ ] Create and apply Service Control Policies
- [ ] Debug "Access Denied" errors systematically

## 🎯 Success Criteria

You're ready for Phase 3 when you can:
1. Design an IAM structure for a multi-team organization
2. Write policies that enforce security requirements
3. Troubleshoot IAM permission issues independently
4. Explain policy evaluation order
5. Set up secure cross-account access

> **💡 Pro Tip:** IAM is the most critical AWS service for security. Spend extra time here if needed. Everything else builds on IAM!

## 🔜 What's Next?

Great job mastering IAM! You now control who can access what in AWS.

**Next:** [Phase 3: Network Security](phase-3-network-security.md) - Secure your AWS network infrastructure with VPCs, Security Groups, and more

---

**Remember:** "Identity is the new perimeter." Master IAM, and you've mastered the foundation of AWS security! 🔐
