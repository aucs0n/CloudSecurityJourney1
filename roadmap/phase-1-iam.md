# Phase 1: Identity & Access Management (IAM)

## Overview
IAM is the foundation of AWS security. Everything in AWS starts with proper identity and access management. This phase will teach you how to securely manage users, permissions, and access patterns.

## Learning Objectives
By the end of this phase, you will be able to:
- Create and manage IAM users, groups, and roles
- Write and understand IAM policies
- Implement multi-factor authentication (MFA)
- Use permission boundaries and SCPs
- Analyze permissions with IAM Access Analyzer
- Implement least privilege access

---

## Core Concepts

### IAM Components

```
┌─────────────────────────────────────────────┐
│  AWS Account                                │
│  ┌───────────────────────────────────────┐  │
│  │  IAM Users                            │  │
│  │  - Long-term credentials              │  │
│  │  - Individual people                  │  │
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │  IAM Groups                           │  │
│  │  - Collection of users                │  │
│  │  - Attach policies                    │  │
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │  IAM Roles                            │  │
│  │  - Temporary credentials              │  │
│  │  - For services and federated users  │  │
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │  IAM Policies                         │  │
│  │  - Define permissions (JSON)          │  │
│  │  - Attach to users/groups/roles       │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### IAM Policy Types

1. **Managed Policies**
   - AWS Managed: Created and maintained by AWS
   - Customer Managed: Created by you, reusable

2. **Inline Policies**
   - Embedded directly into user/group/role
   - 1:1 relationship

3. **Resource-Based Policies**
   - Attached to resources (S3 buckets, SQS queues)
   - Specify who can access the resource

4. **Service Control Policies (SCPs)**
   - Applied at organization/OU level
   - Maximum permissions boundary

---

## IAM Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Policy Elements:**
- **Version**: Policy language version (always "2012-10-17")
- **Statement**: Array of permissions
  - **Sid**: Statement ID (optional, descriptive)
  - **Effect**: Allow or Deny
  - **Action**: What actions are allowed/denied
  - **Resource**: Which resources the actions apply to
  - **Condition**: Optional conditions (IP, time, MFA, etc.)

---

## Best Practices

### 1. Root Account Security
- [ ] Enable MFA on root account
- [ ] Never use root account for daily tasks
- [ ] Delete root account access keys
- [ ] Use root account only for account-level tasks

### 2. IAM User Management
- [ ] Create individual IAM users (no shared credentials)
- [ ] Enable MFA for all users
- [ ] Use groups to assign permissions
- [ ] Enforce strong password policy
- [ ] Rotate credentials regularly (90 days)

### 3. Principle of Least Privilege
- [ ] Grant minimum permissions needed
- [ ] Start with no permissions, add as needed
- [ ] Review permissions regularly
- [ ] Remove unused users and permissions

### 4. Use IAM Roles Instead of Keys
- [ ] EC2 instances: Use instance profiles
- [ ] Lambda functions: Use execution roles
- [ ] External access: Use federated roles
- [ ] Cross-account access: Use assume role

---

## Hands-On Labs

### Lab 1: Create IAM Users and Groups

**Objective**: Set up a basic IAM structure with users and groups.

**Steps:**
1. Create IAM groups:
   ```bash
   # Developers group
   aws iam create-group --group-name Developers
   
   # Administrators group
   aws iam create-group --group-name Administrators
   ```

2. Attach policies to groups:
   ```bash
   # Attach PowerUserAccess to Developers
   aws iam attach-group-policy \
     --group-name Developers \
     --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
   
   # Attach AdministratorAccess to Administrators
   aws iam attach-group-policy \
     --group-name Administrators \
     --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
   ```

3. Create IAM users:
   ```bash
   aws iam create-user --user-name alice
   aws iam create-user --user-name bob
   ```

4. Add users to groups:
   ```bash
   aws iam add-user-to-group --user-name alice --group-name Developers
   aws iam add-user-to-group --user-name bob --group-name Administrators
   ```

5. Create login profiles:
   ```bash
   aws iam create-login-profile \
     --user-name alice \
     --password 'MySecurePassword123!' \
     --password-reset-required
   ```

### Lab 2: Implement MFA

**Objective**: Enable MFA for IAM users.

**Steps:**
1. Enable MFA via console:
   - Navigate to IAM > Users > [username]
   - Security credentials > Assign MFA device
   - Scan QR code with authenticator app
   - Enter two consecutive MFA codes

2. Verify MFA is enabled:
   ```bash
   aws iam list-mfa-devices --user-name alice
   ```

3. Create policy requiring MFA for actions:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "AllowAllActionsWithMFA",
         "Effect": "Allow",
         "Action": "*",
         "Resource": "*",
         "Condition": {
           "BoolIfExists": {
             "aws:MultiFactorAuthPresent": "true"
           }
         }
       }
     ]
   }
   ```

### Lab 3: Create Custom IAM Policy

**Objective**: Write a custom policy for S3 bucket access.

**Policy: Allow read-only access to specific S3 bucket**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-project-bucket"
    },
    {
      "Sid": "AllowGetObject",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-project-bucket/*"
    }
  ]
}
```

**Create policy via CLI:**
```bash
aws iam create-policy \
  --policy-name S3ReadOnlySpecificBucket \
  --policy-document file://s3-readonly-policy.json
```

### Lab 4: IAM Roles for EC2

**Objective**: Create an IAM role for EC2 instances.

**Steps:**
1. Create trust policy (allow EC2 to assume role):
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

2. Create the role:
   ```bash
   aws iam create-role \
     --role-name EC2-S3-ReadOnly-Role \
     --assume-role-policy-document file://trust-policy.json
   ```

3. Attach permissions policy:
   ```bash
   aws iam attach-role-policy \
     --role-name EC2-S3-ReadOnly-Role \
     --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
   ```

4. Create instance profile:
   ```bash
   aws iam create-instance-profile \
     --instance-profile-name EC2-S3-ReadOnly-Profile
   
   aws iam add-role-to-instance-profile \
     --instance-profile-name EC2-S3-ReadOnly-Profile \
     --role-name EC2-S3-ReadOnly-Role
   ```

5. Launch EC2 with instance profile:
   ```bash
   aws ec2 run-instances \
     --image-id ami-0c55b159cbfafe1f0 \
     --instance-type t2.micro \
     --iam-instance-profile Name=EC2-S3-ReadOnly-Profile
   ```

### Lab 5: Cross-Account Access

**Objective**: Set up cross-account IAM role.

**In Account B (resource account):**
1. Create role with trust policy for Account A:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::ACCOUNT-A-ID:root"
         },
         "Action": "sts:AssumeRole",
         "Condition": {
           "StringEquals": {
             "sts:ExternalId": "unique-external-id-123"
           }
         }
       }
     ]
   }
   ```

2. Create role:
   ```bash
   aws iam create-role \
     --role-name CrossAccountRole \
     --assume-role-policy-document file://trust-policy.json
   ```

**In Account A (user account):**
3. Create policy to assume role in Account B:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "sts:AssumeRole",
         "Resource": "arn:aws:iam::ACCOUNT-B-ID:role/CrossAccountRole"
       }
     ]
   }
   ```

4. Assume the role:
   ```bash
   aws sts assume-role \
     --role-arn arn:aws:iam::ACCOUNT-B-ID:role/CrossAccountRole \
     --role-session-name my-session \
     --external-id unique-external-id-123
   ```

### Lab 6: IAM Access Analyzer

**Objective**: Use IAM Access Analyzer to find external access.

**Steps:**
1. Enable IAM Access Analyzer:
   ```bash
   aws accessanalyzer create-analyzer \
     --analyzer-name my-account-analyzer \
     --type ACCOUNT
   ```

2. List findings:
   ```bash
   aws accessanalyzer list-findings \
     --analyzer-arn arn:aws:access-analyzer:us-east-1:123456789012:analyzer/my-account-analyzer
   ```

3. Review findings in console:
   - Navigate to IAM > Access Analyzer
   - Review findings for S3 buckets, IAM roles, KMS keys, etc.
   - Resolve findings or archive if expected

---

## IAM Policy Evaluation Logic

**How AWS determines if an action is allowed:**

```
┌─────────────────────────────────────────────┐
│  1. Decision starts at DENY                 │
└────────────┬────────────────────────────────┘
             ↓
┌─────────────────────────────────────────────┐
│  2. Evaluate all applicable policies:       │
│     - Identity-based policies               │
│     - Resource-based policies               │
│     - Permission boundaries                 │
│     - SCPs                                  │
│     - Session policies                      │
└────────────┬────────────────────────────────┘
             ↓
┌─────────────────────────────────────────────┐
│  3. Is there an explicit DENY?              │
│     YES → Action DENIED (stop)              │
│     NO → Continue                           │
└────────────┬────────────────────────────────┘
             ↓
┌─────────────────────────────────────────────┐
│  4. Is there an explicit ALLOW?             │
│     YES → Action ALLOWED                    │
│     NO → Action DENIED (implicit deny)      │
└─────────────────────────────────────────────┘
```

**Key principle**: Explicit DENY always wins!

---

## Common IAM Patterns

### Pattern 1: Read-Only Auditor Role
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudtrail:LookupEvents",
        "cloudwatch:GetMetricData",
        "config:Describe*",
        "iam:Get*",
        "iam:List*",
        "s3:GetBucketLocation",
        "s3:ListAllMyBuckets"
      ],
      "Resource": "*"
    }
  ]
}
```

### Pattern 2: Developer Self-Service MFA
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUsersToManageOwnMFA",
      "Effect": "Allow",
      "Action": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:ResyncMFADevice",
        "iam:DeleteVirtualMFADevice"
      ],
      "Resource": "arn:aws:iam::*:mfa/${aws:username}"
    }
  ]
}
```

### Pattern 3: Restrict by IP Address
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": [
            "203.0.113.0/24",
            "198.51.100.0/24"
          ]
        }
      }
    }
  ]
}
```

---

## Troubleshooting IAM

### Issue: Access Denied Error

**Steps to debug:**
1. Check CloudTrail for the failed API call
2. Use IAM Policy Simulator
3. Verify:
   - User/role has required permissions
   - No explicit deny in policies
   - Resource policy allows access
   - SCPs don't restrict access
   - Permission boundaries allow action

### IAM Policy Simulator

```bash
# Test if user can perform action
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt
```

---

## Security Checklist

- [ ] Root account has MFA enabled
- [ ] Root account access keys deleted
- [ ] Individual IAM users created (no shared accounts)
- [ ] All users have MFA enabled
- [ ] Strong password policy enforced
- [ ] Unused credentials removed
- [ ] IAM Access Analyzer enabled
- [ ] Regular access reviews conducted
- [ ] Least privilege principle applied
- [ ] Service roles used instead of access keys

---

## Exam Tips (for AWS Security Specialty)

1. **Understand policy evaluation logic**: Explicit deny > Explicit allow > Implicit deny
2. **Know when to use roles vs users**: Roles for services, users for people
3. **SCPs are permission boundaries**: They don't grant permissions, only limit them
4. **Resource-based policies**: Can grant cross-account access without assuming role
5. **Permission boundaries**: Limit maximum permissions for IAM entities
6. **Temporary credentials**: Always prefer over long-term access keys

---

## Additional Resources

- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS Security Blog - IAM](https://aws.amazon.com/blogs/security/tag/aws-identity-and-access-management/)

---

[← Back to Main Roadmap](../ROADMAP.md) | [Next Phase: Network Security →](./phase-2-network-security.md)
