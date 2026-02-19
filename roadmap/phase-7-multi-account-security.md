# 🏢 Phase 7: Multi-Account Security

> **Scale security across your AWS Organization**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

As organizations grow, managing security across multiple AWS accounts becomes critical. This phase covers AWS Organizations, Service Control Policies, AWS Control Tower, and Landing Zone architecture for enterprise-scale security.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Design multi-account architectures
- ✅ Implement AWS Organizations
- ✅ Create and apply Service Control Policies (SCPs)
- ✅ Deploy AWS Control Tower
- ✅ Build secure Landing Zones
- ✅ Configure delegated administration
- ✅ Manage cross-account access patterns
- ✅ Centralize security services

## 📚 Topics & Progress Tracker

### Week 1: AWS Organizations

#### AWS Organizations Fundamentals
- [ ] Understand Organization structure
- [ ] Root, OUs (Organizational Units), and accounts
- [ ] Consolidated billing benefits
- [ ] Organization-wide features
- [ ] Account creation strategies

**AWS Organizations Hierarchy:**
```
┌─────────────────────────────────────────────────┐
│         AWS ORGANIZATION STRUCTURE              │
├─────────────────────────────────────────────────┤
│                                                 │
│             Root (Management Account)           │
│                      │                          │
│         ┌────────────┴────────────┐            │
│         │                         │            │
│   Security OU              Workloads OU        │
│    ├─ Audit Account         ├─ Production OU   │
│    ├─ Log Archive           │   ├─ Prod-App1   │
│    └─ Security Tools        │   └─ Prod-App2   │
│                             │                   │
│                             ├─ Development OU   │
│                             │   ├─ Dev-App1     │
│                             │   └─ Dev-App2     │
│                             │                   │
│                             └─ Staging OU       │
│                                 ├─ Stage-App1   │
│                                 └─ Stage-App2   │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Create Organization:**
```bash
# Create organization
aws organizations create-organization \
  --feature-set ALL

# Get organization details
aws organizations describe-organization

# Create Organizational Unit
aws organizations create-organizational-unit \
  --parent-id r-xxxx \
  --name "Production"

# Create new account
aws organizations create-account \
  --email prod-account@example.com \
  --account-name "Production Account"
```

#### Organizational Unit Design
- [ ] Design OU structure for security and compliance
- [ ] Separate production, development, staging
- [ ] Security OU for audit and logging
- [ ] Sandbox OU for experimentation
- [ ] Business unit-based OUs

**Recommended OU Structure:**
```
Root
├── Security OU
│   ├── Security Tooling Account (GuardDuty, Security Hub)
│   ├── Log Archive Account (CloudTrail, VPC Flow Logs)
│   └── Audit Account (Read-only access)
│
├── Infrastructure OU
│   ├── Network Account (Transit Gateway, VPC)
│   └── Shared Services Account (Active Directory, DNS)
│
├── Workloads OU
│   ├── Production OU
│   ├── Staging OU
│   └── Development OU
│
├── Sandbox OU
│   └── Individual developer accounts
│
└── Suspended OU
    └── Decommissioned accounts
```

> **🔑 Key Takeaway:** Management account should only manage the organization. Workloads should run in member accounts.

#### Consolidated Billing Benefits
- [ ] Single payment method
- [ ] Volume discounts
- [ ] Reserved Instance sharing
- [ ] Savings Plans sharing
- [ ] Cost allocation tags

### Week 2: Service Control Policies (SCPs)

#### SCP Fundamentals
- [ ] Understand SCP vs IAM policies
- [ ] SCP inheritance model
- [ ] Allow lists vs Deny lists
- [ ] SCP best practices
- [ ] Testing SCPs safely

**SCP Inheritance:**
```
┌─────────────────────────────────────────────┐
│      SCP INHERITANCE MODEL                  │
├─────────────────────────────────────────────┤
│                                             │
│  Root (SCP: FullAWSAccess)                  │
│   │                                         │
│   ├─ Production OU (SCP: DenyRegions)       │
│   │   │                                     │
│   │   └─ Account A                          │
│   │       Effective Policy =                │
│   │       FullAWSAccess ∩ DenyRegions       │
│   │                                         │
│   └─ Sandbox OU (SCP: AllowExperimentation) │
│       │                                     │
│       └─ Account B                          │
│           Effective Policy =                │
│           FullAWSAccess ∩ AllowExperimentation│
│                                             │
└─────────────────────────────────────────────┘
```

**Essential SCPs:**

**1. Require Encryption:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedS3",
      "Effect": "Deny",
      "Action": "s3:PutObject",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": ["AES256", "aws:kms"]
        }
      }
    },
    {
      "Sid": "DenyUnencryptedEBS",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:volume/*",
      "Condition": {
        "Bool": {
          "ec2:Encrypted": "false"
        }
      }
    }
  ]
}
```

**2. Restrict Regions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllOutsideApprovedRegions",
      "Effect": "Deny",
      "NotAction": [
        "cloudfront:*",
        "iam:*",
        "route53:*",
        "support:*",
        "organizations:*"
      ],
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

**3. Protect CloudTrail:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ProtectCloudTrail",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "cloudtrail:PutEventSelectors"
      ],
      "Resource": "*"
    }
  ]
}
```

**4. Require MFA for Sensitive Actions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireMFAForSensitiveActions",
      "Effect": "Deny",
      "Action": [
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "rds:DeleteDBInstance",
        "s3:DeleteBucket"
      ],
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

**5. Prevent Leaving Organization:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventLeavingOrganization",
      "Effect": "Deny",
      "Action": [
        "organizations:LeaveOrganization"
      ],
      "Resource": "*"
    }
  ]
}
```

**Apply SCP:**
```bash
# Create SCP
aws organizations create-policy \
  --content file://require-encryption-scp.json \
  --description "Require encryption for S3 and EBS" \
  --name RequireEncryption \
  --type SERVICE_CONTROL_POLICY

# Attach SCP to OU
aws organizations attach-policy \
  --policy-id p-xxxx \
  --target-id ou-xxxx
```

### Week 3: AWS Control Tower

#### Control Tower Overview
- [ ] Understand Control Tower architecture
- [ ] Landing Zone components
- [ ] Guardrails (preventive and detective)
- [ ] Account Factory
- [ ] Account vending process

**Control Tower Architecture:**
```
┌──────────────────────────────────────────────────┐
│         AWS CONTROL TOWER                        │
├──────────────────────────────────────────────────┤
│                                                  │
│  Management Account                              │
│  └─ Control Tower orchestration                  │
│                                                  │
│  Audit Account                                   │
│  ├─ SNS notifications                            │
│  ├─ AWS Config aggregator                       │
│  └─ CloudWatch logs for compliance              │
│                                                  │
│  Log Archive Account                             │
│  ├─ Centralized CloudTrail logs                 │
│  ├─ Config logs                                  │
│  └─ VPC Flow Logs                                │
│                                                  │
│  Guardrails                                      │
│  ├─ Mandatory (Always enforced)                 │
│  ├─ Strongly recommended                         │
│  └─ Elective (Optional)                          │
│                                                  │
│  Account Factory                                 │
│  └─ Automated account provisioning               │
│                                                  │
└──────────────────────────────────────────────────┘
```

#### Control Tower Guardrails
- [ ] **Mandatory Guardrails** (Cannot be disabled)
  - Disallow policy changes to log archive
  - Disallow public read access to log archive
  - Enable CloudTrail in all accounts
- [ ] **Strongly Recommended Guardrails**
  - Enable MFA for root user
  - Disallow public read access to S3 buckets
  - Disallow public write access to S3 buckets
- [ ] **Elective Guardrails**
  - Deny root user access keys
  - Disallow specific EC2 instance types
  - Restrict Amazon EBS volume types

**Guardrail Types:**
| Type | Implementation | Detection Time |
|------|----------------|----------------|
| **Preventive** | SCPs | Real-time |
| **Detective** | AWS Config Rules | Minutes |

#### Account Factory
- [ ] Provision accounts via Service Catalog
- [ ] Standardized account configuration
- [ ] Automated baseline deployment
- [ ] Network configuration
- [ ] Security controls

**Account Vending Process:**
```
1. User requests account via Service Catalog
   ↓
2. Control Tower provisions account
   ↓
3. Automatically applies:
   - Baseline security controls
   - GuardRails
   - CloudTrail logging
   - Config recording
   - Default VPC configuration
   ↓
4. Account ready for use
```

### Week 4: Landing Zone & Centralization

#### Landing Zone Design
- [ ] Multi-account strategy
- [ ] Network design (hub-and-spoke)
- [ ] Centralized logging
- [ ] Centralized security services
- [ ] Identity federation (AWS SSO)

**Secure Landing Zone Architecture:**
```
┌────────────────────────────────────────────────────────────┐
│              SECURE LANDING ZONE                           │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────────┐         ┌──────────────────┐       │
│  │ Management       │         │  Audit Account   │       │
│  │ Account          │◄───────►│  (Read-only)     │       │
│  │ - Organizations  │         │  - Config Agg.   │       │
│  │ - Control Tower  │         │  - Security Hub  │       │
│  └──────────────────┘         └──────────────────┘       │
│           │                             │                 │
│           │                             │                 │
│  ┌────────▼─────────────────────────────▼─────────┐     │
│  │        Log Archive Account                     │     │
│  │        - Centralized CloudTrail                │     │
│  │        - VPC Flow Logs                         │     │
│  │        - S3 Access Logs                        │     │
│  └────────────────────────────────────────────────┘     │
│                                                          │
│  ┌───────────────────────────────────────────────┐     │
│  │     Security Tooling Account                  │     │
│  │     - GuardDuty (delegated admin)             │     │
│  │     - Security Hub (delegated admin)          │     │
│  │     - Macie (delegated admin)                 │     │
│  │     - Inspector (delegated admin)             │     │
│  └───────────────────────────────────────────────┘     │
│                                                          │
│  ┌───────────────────────────────────────────────┐     │
│  │     Network Account                           │     │
│  │     - Transit Gateway                         │     │
│  │     - Route 53 Resolver                       │     │
│  │     - Network Firewall                        │     │
│  └───────────────────────────────────────────────┘     │
│                                                          │
│         ┌──────────────┬──────────────┐                │
│         │              │              │                │
│    ┌────▼────┐   ┌────▼────┐   ┌────▼────┐           │
│    │Prod OU  │   │Stage OU │   │ Dev OU  │           │
│    └─────────┘   └─────────┘   └─────────┘           │
│                                                          │
└────────────────────────────────────────────────────────────┘
```

#### Delegated Administration
- [ ] GuardDuty delegated admin
- [ ] Security Hub delegated admin
- [ ] Macie delegated admin
- [ ] Inspector delegated admin
- [ ] Firewall Manager delegated admin

**Enable Delegated Admin:**
```bash
# Enable GuardDuty delegated admin
aws guardduty enable-organization-admin-account \
  --admin-account-id 111111111111

# Enable Security Hub delegated admin
aws securityhub enable-organization-admin-account \
  --admin-account-id 111111111111

# Enable Macie delegated admin
aws macie2 enable-organization-admin-account \
  --admin-account-id 111111111111
```

> **💡 Pro Tip:** Use a dedicated security tooling account as delegated admin rather than the management account.

#### AWS SSO (IAM Identity Center)
- [ ] Centralized access management
- [ ] Integration with identity providers
- [ ] Permission sets
- [ ] Multi-account access
- [ ] Temporary credentials

**Permission Set Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "s3:ListAllMyBuckets",
        "cloudwatch:Describe*",
        "cloudwatch:Get*",
        "cloudwatch:List*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Cross-Account Access Patterns
- [ ] IAM roles for cross-account access
- [ ] Resource-based policies
- [ ] AWS Resource Access Manager (RAM)
- [ ] VPC peering and Transit Gateway
- [ ] Cross-account S3 access

**Cross-Account Role Assumption:**
```bash
# In target account: Create role
aws iam create-role \
  --role-name CrossAccountReadOnly \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::222222222222:root"},
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {"sts:ExternalId": "unique-id-12345"}
      }
    }]
  }'

# Attach policy
aws iam attach-role-policy \
  --role-name CrossAccountReadOnly \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# In source account: Assume role
aws sts assume-role \
  --role-arn arn:aws:iam::111111111111:role/CrossAccountReadOnly \
  --role-session-name my-session \
  --external-id unique-id-12345
```

## 🛠️ Hands-On Labs

### Lab 1: Create Multi-Account Structure
```bash
#!/bin/bash
# Set up basic multi-account organization

# Create organization
aws organizations create-organization --feature-set ALL

ORG_ID=$(aws organizations describe-organization --query 'Organization.Id' --output text)
ROOT_ID=$(aws organizations list-roots --query 'Roots[0].Id' --output text)

# Create Security OU
SECURITY_OU=$(aws organizations create-organizational-unit \
  --parent-id $ROOT_ID \
  --name Security \
  --query 'OrganizationalUnit.Id' \
  --output text)

# Create Workloads OU
WORKLOADS_OU=$(aws organizations create-organizational-unit \
  --parent-id $ROOT_ID \
  --name Workloads \
  --query 'OrganizationalUnit.Id' \
  --output text)

# Create Production sub-OU
PROD_OU=$(aws organizations create-organizational-unit \
  --parent-id $WORKLOADS_OU \
  --name Production \
  --query 'OrganizationalUnit.Id' \
  --output text)

echo "✅ Organization structure created!"
echo "Security OU: $SECURITY_OU"
echo "Workloads OU: $WORKLOADS_OU"
echo "Production OU: $PROD_OU"
```

### Lab 2: Deploy Organization-Wide SCPs
```bash
#!/bin/bash
# Deploy essential SCPs

# Create "Require Encryption" SCP
cat > require-encryption.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedS3",
      "Effect": "Deny",
      "Action": "s3:PutObject",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": ["AES256", "aws:kms"]
        }
      }
    }
  ]
}
EOF

POLICY_ID=$(aws organizations create-policy \
  --content file://require-encryption.json \
  --description "Require encryption for S3 uploads" \
  --name RequireEncryption \
  --type SERVICE_CONTROL_POLICY \
  --query 'Policy.PolicySummary.Id' \
  --output text)

# Attach to root (applies to all accounts)
aws organizations attach-policy \
  --policy-id $POLICY_ID \
  --target-id $ROOT_ID

echo "✅ SCP deployed organization-wide!"
```

### Lab 3: Centralize Security Services
```python
#!/usr/bin/env python3
"""
Enable centralized security services across organization
"""
import boto3

def enable_guardduty_org(admin_account_id):
    """Enable GuardDuty for organization"""
    guardduty = boto3.client('guardduty')
    
    # Create detector in admin account
    detector = guardduty.create_detector(Enable=True)
    detector_id = detector['DetectorId']
    
    # Enable organization admin
    guardduty.enable_organization_admin_account(
        AdminAccountId=admin_account_id
    )
    
    print(f"✅ GuardDuty enabled with detector: {detector_id}")
    print(f"   Admin account: {admin_account_id}")

def enable_securityhub_org(admin_account_id):
    """Enable Security Hub for organization"""
    securityhub = boto3.client('securityhub')
    
    # Enable Security Hub
    securityhub.enable_security_hub()
    
    # Enable organization admin
    securityhub.enable_organization_admin_account(
        AdminAccountId=admin_account_id
    )
    
    # Enable standards
    securityhub.batch_enable_standards(
        StandardsSubscriptionRequests=[
            {
                'StandardsArn': 'arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0'
            }
        ]
    )
    
    print(f"✅ Security Hub enabled")
    print(f"   Admin account: {admin_account_id}")

def main():
    admin_account_id = '111111111111'  # Security tooling account
    
    print("Enabling centralized security services...")
    enable_guardduty_org(admin_account_id)
    enable_securityhub_org(admin_account_id)
    print("\n✅ All security services centralized!")

if __name__ == "__main__":
    main()
```

## ✅ Phase 7 Checklist

Before moving to Phase 8, ensure you can:
- [ ] Design a multi-account organizational structure
- [ ] Create and apply Service Control Policies
- [ ] Understand AWS Control Tower components
- [ ] Configure delegated administration for security services
- [ ] Implement cross-account access patterns
- [ ] Set up centralized logging
- [ ] Deploy a secure Landing Zone
- [ ] Use AWS SSO for centralized access management

## 🎯 Success Criteria

You're ready for Phase 8 when you can:
1. Design a complete multi-account strategy
2. Implement organization-wide security controls with SCPs
3. Centralize security services across accounts
4. Set up secure cross-account access
5. Explain the benefits of Landing Zones

> **💡 Pro Tip:** Start with Control Tower if you're building from scratch. It provides best-practice guardrails out of the box.

## 🔜 What's Next?

Excellent work on multi-account security! Now let's prepare for the inevitable: security incidents.

**Next:** [Phase 8: Incident Response](phase-8-incident-response.md) - Master incident response procedures and automation

---

**Remember:** "Security at scale requires automation, centralization, and clear boundaries!" 🏢🔒
