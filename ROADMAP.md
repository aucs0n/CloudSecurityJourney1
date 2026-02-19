# 🛣️ Cloud Security Engineering Roadmap

> A comprehensive, progressive learning guide to become an AWS Cloud Security Engineer

---

## 📋 Table of Contents

- [Progress Tracker](#-progress-tracker)
- [Prerequisites & Foundations](#-prerequisites--foundations)
- [Core AWS Security Domains](#-core-aws-security-domains)
- [Certifications Roadmap](#-certifications-roadmap)
- [Hands-On Practice Resources](#-hands-on-practice-resources)
- [Suggested Timeline](#-suggested-timeline)
- [Day-to-Day Responsibilities](#-day-to-day-responsibilities-of-a-cloud-security-engineer)
- [Quick Start Guide](#-quick-start-guide)

---

## 📊 Progress Tracker

Track your journey through the Cloud Security Engineering learning path:

### Foundations
- [ ] Networking fundamentals completed
- [ ] Security fundamentals completed
- [ ] Linux basics and scripting completed
- [ ] Cloud computing concepts completed

### Core Security Phases
- [ ] Phase 1: Identity & Access Management (IAM)
- [ ] Phase 2: Network Security
- [ ] Phase 3: Data Security & Encryption
- [ ] Phase 4: Monitoring, Logging & Threat Detection
- [ ] Phase 5: Vulnerability & Compliance Management
- [ ] Phase 6: Secrets & Credentials Management
- [ ] Phase 7: Compute Security
- [ ] Phase 8: Multi-Account & Organizational Security
- [ ] Phase 9: Incident Response
- [ ] Phase 10: Infrastructure as Code (IaC) Security

### Certifications
- [ ] AWS Certified Cloud Practitioner (CLF-C02)
- [ ] AWS Certified Solutions Architect Associate (SAA-C03)
- [ ] AWS Certified Security – Specialty (SCS-C02) ⭐

### Practical Experience
- [ ] Completed at least 3 hands-on labs
- [ ] Built personal cloud security project
- [ ] Participated in CTF challenge

---

## 🎯 Prerequisites & Foundations

Before diving into AWS security, ensure you have a solid understanding of these fundamental concepts:

### 1. Networking Fundamentals 🌐

**Core Concepts:**
- **TCP/IP Model**: Understanding layers and protocols
- **DNS**: Domain Name System, DNS records (A, AAAA, CNAME, MX, TXT)
- **HTTP/HTTPS**: Request/response cycles, status codes, headers
- **TLS/SSL**: Encryption protocols, certificates, handshakes
- **Firewalls**: Stateful vs stateless, rules, zones
- **VPNs**: Site-to-site, client-to-site, IPsec, OpenVPN
- **Subnets & CIDR**: IP addressing, subnet masks, CIDR notation (e.g., 10.0.0.0/24)

**ASCII Diagram - TCP/IP Stack:**
```
┌──────────────────────────────┐
│   Application Layer (HTTP)   │
├──────────────────────────────┤
│   Transport Layer (TCP/UDP)  │
├──────────────────────────────┤
│   Network Layer (IP)         │
├──────────────────────────────┤
│   Data Link Layer (Ethernet) │
└──────────────────────────────┘
```

**Learning Resources:**
- CompTIA Network+ materials
- NetworkChuck YouTube channel
- Cisco Networking Basics

### 2. Security Fundamentals 🔒

**Core Principles:**
- **CIA Triad**:
  - **Confidentiality**: Data is protected from unauthorized access
  - **Integrity**: Data remains accurate and unaltered
  - **Availability**: Systems and data are accessible when needed

- **AAA Framework**:
  - **Authentication**: Verifying identity (who you are)
  - **Authorization**: Granting permissions (what you can do)
  - **Accounting**: Tracking actions (what you did)

- **Zero Trust Architecture**: Never trust, always verify
- **Least Privilege**: Minimum necessary permissions
- **Defense in Depth**: Multiple layers of security controls
- **Security by Design**: Building security from the ground up

**ASCII Diagram - Zero Trust Model:**
```
┌─────────────────────────────────────────┐
│        Verify Explicitly                │
│  ┌─────────────────────────────────┐   │
│  │  Use Least Privilege Access     │   │
│  │  ┌───────────────────────────┐  │   │
│  │  │  Assume Breach            │  │   │
│  │  │  (Continuous Verification)│  │   │
│  │  └───────────────────────────┘  │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### 3. Linux Basics & Scripting 🐧

**Essential Skills:**
- **Command Line**: Navigation, file operations, permissions
- **Bash Scripting**: Loops, conditionals, functions, cron jobs
- **Python**: Boto3 (AWS SDK), automation scripts, API interactions
- **Package Management**: apt, yum, pip
- **Process Management**: ps, top, systemctl
- **Log Analysis**: grep, awk, sed, tail

**Sample Commands:**
```bash
# Check running processes
ps aux | grep python

# Monitor system logs
tail -f /var/log/syslog

# AWS CLI example
aws s3 ls --profile production
```

### 4. Cloud Computing Concepts ☁️

**Service Models:**
- **IaaS** (Infrastructure as a Service): EC2, VPC, EBS
- **PaaS** (Platform as a Service): Elastic Beanstalk, RDS
- **SaaS** (Software as a Service): AWS WorkSpaces, Amazon Chime

**Shared Responsibility Model:**
```
┌────────────────────────────────────────────────┐
│           Customer Responsibility              │
│  (Security IN the Cloud)                       │
│  ├─ Application Data                           │
│  ├─ IAM Users, Roles, Policies                 │
│  ├─ Operating System Configuration             │
│  └─ Network & Firewall Configuration           │
├────────────────────────────────────────────────┤
│           AWS Responsibility                   │
│  (Security OF the Cloud)                       │
│  ├─ Physical Infrastructure                    │
│  ├─ Hardware & Networking                      │
│  ├─ Hypervisor & Global Infrastructure         │
│  └─ Managed Service Operations                 │
└────────────────────────────────────────────────┘
```

---

## 🏗️ Core AWS Security Domains

### Phase 1 – Identity & Access Management (IAM) 🔑

**What You'll Learn:**
- IAM Users, Groups, Roles, and Policies
- Policy types: Managed, Inline, Resource-based, SCPs
- Multi-Factor Authentication (MFA)
- Permission Boundaries
- Service Control Policies (SCPs)
- IAM Access Analyzer
- IAM Policy Simulator
- Cross-account access patterns

**ASCII Diagram - IAM Hierarchy:**
```
┌─────────────────────────────────────────────┐
│          AWS Organization (Root)            │
│  ┌───────────────────────────────────────┐  │
│  │  Organizational Unit (OU)             │  │
│  │  ┌─────────────────────────────────┐  │  │
│  │  │  AWS Account                    │  │  │
│  │  │  ┌───────────────────────────┐  │  │  │
│  │  │  │  IAM Users / Groups       │  │  │  │
│  │  │  │  ┌─────────────────────┐  │  │  │  │
│  │  │  │  │  IAM Roles         │  │  │  │  │
│  │  │  │  │  └─> Policies      │  │  │  │  │
│  │  │  │  └─────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────┘  │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

**Key Best Practices:**
- Never use root account for daily tasks
- Enable MFA for all users, especially privileged accounts
- Use IAM roles instead of long-term credentials
- Apply least privilege principle
- Regularly audit and rotate credentials
- Use IAM Access Analyzer to identify public/cross-account access

**Hands-On Labs:**
- Create IAM users with specific permissions
- Implement MFA for root and IAM users
- Create cross-account roles
- Use IAM Policy Simulator to test policies

**[📖 Detailed Phase 1 Guide →](./roadmap/phase-1-iam.md)**

---

### Phase 2 – Network Security 🌐

**What You'll Learn:**
- VPC architecture and components
- Public vs private subnets
- Internet Gateway (IGW), NAT Gateway/Instance
- Security Groups (stateful) vs NACLs (stateless)
- AWS WAF (Web Application Firewall)
- AWS Shield (DDoS protection)
- AWS Network Firewall
- VPC Flow Logs
- AWS PrivateLink
- Transit Gateway
- Route 53 Resolver DNS Firewall

**ASCII Diagram - VPC Architecture:**
```
┌────────────────────────────────────────────────────────────┐
│  VPC (10.0.0.0/16)                                         │
│  ┌──────────────────────────┐  ┌──────────────────────┐   │
│  │  Public Subnet           │  │  Private Subnet      │   │
│  │  (10.0.1.0/24)           │  │  (10.0.2.0/24)       │   │
│  │  ┌────────────────────┐  │  │  ┌────────────────┐ │   │
│  │  │  EC2 (Web Server)  │  │  │  │  EC2 (App)     │ │   │
│  │  │  + Security Group  │  │  │  │  + Security Grp│ │   │
│  │  └────────────────────┘  │  │  └────────────────┘ │   │
│  │          ↓               │  │          ↓          │   │
│  │  Internet Gateway (IGW)  │  │  NAT Gateway       │   │
│  └──────────┬───────────────┘  └──────────┬─────────┘   │
│             ↓                              ↓             │
│         Internet ←────────────────────────┘              │
└────────────────────────────────────────────────────────────┘
```

**Security Best Practices:**
- Use private subnets for application and database tiers
- Implement network segmentation
- Apply principle of least privilege to Security Groups
- Use NACLs as an additional layer of defense
- Enable VPC Flow Logs for traffic analysis
- Use AWS PrivateLink to avoid internet exposure
- Implement AWS WAF rules to block common attacks

**Hands-On Labs:**
- Design and deploy a multi-tier VPC
- Configure Security Groups and NACLs
- Set up VPC Flow Logs and analyze traffic
- Implement AWS WAF rules

**[📖 Detailed Phase 2 Guide →](./roadmap/phase-2-network-security.md)**

---

### Phase 3 – Data Security & Encryption 🔐

**What You'll Learn:**
- AWS Key Management Service (KMS)
- AWS Managed Keys vs Customer Managed Keys (CMKs)
- Key rotation policies
- CloudHSM for dedicated hardware security modules
- S3 security best practices
- S3 bucket policies and ACLs
- S3 Block Public Access
- S3 Object Versioning
- S3 Object Lock (WORM)
- Encryption at rest (S3, EBS, RDS, DynamoDB)
- Encryption in transit (TLS/SSL)
- AWS Certificate Manager (ACM)

**ASCII Diagram - Encryption Flow:**
```
┌────────────────────────────────────────────────┐
│  Application / User                            │
└────────────┬───────────────────────────────────┘
             ↓ (Request with data)
┌────────────────────────────────────────────────┐
│  AWS KMS - Customer Master Key (CMK)           │
│  ├─ Key Policy                                 │
│  ├─ Automatic Key Rotation (Yearly)            │
│  └─ CloudTrail Audit Logs                      │
└────────────┬───────────────────────────────────┘
             ↓ (Data Encryption Key - DEK)
┌────────────────────────────────────────────────┐
│  Encrypted Storage (S3, EBS, RDS)              │
│  ├─ Data encrypted at rest                     │
│  └─ Encrypted DEK stored with data             │
└────────────────────────────────────────────────┘
```

**S3 Security Layers:**
1. **Bucket Policies**: Resource-based policies for bucket-level access
2. **IAM Policies**: User/role-based permissions
3. **S3 ACLs**: Legacy access control (avoid when possible)
4. **Block Public Access**: Safeguard against accidental public exposure
5. **Encryption**: SSE-S3, SSE-KMS, SSE-C
6. **Versioning**: Protect against accidental deletion
7. **MFA Delete**: Require MFA for object deletion

**Best Practices:**
- Always encrypt sensitive data at rest
- Use AWS managed keys for simplicity, CMKs for control
- Enable automatic key rotation
- Use TLS 1.2+ for data in transit
- Implement S3 Block Public Access by default
- Enable S3 versioning for critical data
- Use VPC endpoints to avoid internet traffic

**Hands-On Labs:**
- Create and manage KMS keys
- Configure S3 bucket encryption
- Implement bucket policies for least privilege
- Set up S3 access logging

**[📖 Detailed Phase 3 Guide →](./roadmap/phase-3-data-security.md)**

---

### Phase 4 – Monitoring, Logging & Threat Detection 📊

**What You'll Learn:**
- AWS CloudTrail (API call logging)
- Amazon CloudWatch (Metrics, Logs, Alarms)
- AWS Config (Resource configuration tracking)
- Amazon GuardDuty (Threat detection)
- AWS Security Hub (Centralized security view)
- Amazon Detective (Security investigation)
- Amazon Macie (Sensitive data discovery)
- VPC Flow Logs analysis
- EventBridge for security automation

**ASCII Diagram - Security Monitoring Stack:**
```
┌────────────────────────────────────────────────────────┐
│  Security Hub (Central Dashboard)                      │
│  ├─ Aggregate findings from multiple services          │
│  ├─ Security standards (CIS, PCI-DSS)                  │
│  └─ Automated remediation workflows                    │
└─────────────────┬──────────────────────────────────────┘
                  ↓ (Findings from)
┌─────────────────────────────────────────────────────────┐
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────┐   │
│  │ GuardDuty    │  │ Macie       │  │ Inspector    │   │
│  │ (Threats)    │  │ (PII/PHI)   │  │ (Vulns)      │   │
│  └──────┬───────┘  └──────┬──────┘  └──────┬───────┘   │
│         ↓                 ↓                  ↓           │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Data Sources                                    │   │
│  │  ├─ CloudTrail (API Logs)                        │   │
│  │  ├─ VPC Flow Logs (Network Traffic)              │   │
│  │  ├─ DNS Logs                                     │   │
│  │  └─ S3 Access Logs                               │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Key Services Explained:**

| Service | Purpose | Use Cases |
|---------|---------|-----------|
| **CloudTrail** | API activity logging | Who did what, when? Compliance audits |
| **CloudWatch** | Metrics & logs | Performance monitoring, log aggregation |
| **AWS Config** | Configuration tracking | Resource compliance, change history |
| **GuardDuty** | Threat detection | Malicious activity, compromised instances |
| **Security Hub** | Security posture | Centralized findings, compliance checks |
| **Macie** | Data classification | Find PII/PHI in S3 buckets |
| **Detective** | Investigation | Root cause analysis, visual timelines |

**Best Practices:**
- Enable CloudTrail in all regions and all accounts
- Centralize logs to a dedicated security account
- Set up CloudWatch alarms for critical events
- Enable GuardDuty in all accounts
- Use Security Hub for aggregated security view
- Automate responses with EventBridge + Lambda

**Hands-On Labs:**
- Enable and configure CloudTrail
- Set up CloudWatch alarms for security events
- Enable GuardDuty and analyze findings
- Configure Security Hub with security standards

**[📖 Detailed Phase 4 Guide →](./roadmap/phase-4-monitoring-logging.md)**

---

### Phase 5 – Vulnerability & Compliance Management ✅

**What You'll Learn:**
- Amazon Inspector (Vulnerability scanning)
- AWS Config Rules (Compliance automation)
- AWS Audit Manager (Audit evidence collection)
- AWS Artifact (Compliance reports)
- Systems Manager Patch Manager
- Compliance frameworks:
  - CIS AWS Foundations Benchmark
  - PCI-DSS (Payment Card Industry)
  - HIPAA (Healthcare)
  - NIST 800-53, NIST CSF
  - SOC 2
  - ISO 27001
  - GDPR considerations

**Compliance Framework Overview:**

| Framework | Focus Area | AWS Services |
|-----------|------------|--------------|
| **CIS Benchmark** | Security best practices | Config Rules, Security Hub |
| **PCI-DSS** | Payment data security | WAF, KMS, CloudTrail, VPC |
| **HIPAA** | Healthcare data | KMS, CloudTrail, VPC, S3 encryption |
| **NIST 800-53** | Federal security controls | Full suite of AWS security services |
| **SOC 2** | Service organization controls | CloudTrail, Config, GuardDuty |
| **ISO 27001** | Information security | Security Hub, Audit Manager |

**AWS Config Rules Examples:**
- `s3-bucket-public-read-prohibited`
- `encrypted-volumes`
- `iam-password-policy`
- `cloudtrail-enabled`
- `root-account-mfa-enabled`

**Best Practices:**
- Use AWS Config Rules for continuous compliance
- Enable Amazon Inspector for EC2 and ECR vulnerability scans
- Implement automated remediation for non-compliant resources
- Regularly review compliance reports in AWS Artifact
- Use Audit Manager for evidence collection
- Maintain compliance documentation

**Hands-On Labs:**
- Enable AWS Config and create custom rules
- Run Amazon Inspector scans
- Set up Audit Manager for SOC 2 or PCI-DSS
- Implement automated remediation with Lambda

**[📖 Detailed Phase 5 Guide →](./roadmap/phase-5-compliance.md)**

---

### Phase 6 – Secrets & Credentials Management 🔑

**What You'll Learn:**
- AWS Secrets Manager
- AWS Systems Manager Parameter Store
- When to use Secrets Manager vs Parameter Store
- Secret rotation automation
- KMS integration with secrets
- Environment variable security
- Avoiding hardcoded credentials
- IAM roles for applications

**Comparison: Secrets Manager vs Parameter Store:**

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| **Cost** | $0.40/secret/month + API calls | Free (Standard), $0.05/param/month (Advanced) |
| **Secret Rotation** | Built-in, automatic | Manual (requires custom Lambda) |
| **Cross-Region** | Cross-region replication | Not supported |
| **KMS Encryption** | Mandatory | Optional |
| **Use Case** | Database credentials, API keys | Application config, non-sensitive params |
| **Max Size** | 64 KB | 4 KB (Standard), 8 KB (Advanced) |

**ASCII Diagram - Secrets Management Flow:**
```
┌─────────────────────────────────────────────┐
│  Application (Lambda, EC2, ECS)             │
└────────────┬────────────────────────────────┘
             ↓ (Request secret via IAM role)
┌─────────────────────────────────────────────┐
│  AWS Secrets Manager                        │
│  ├─ Encrypted secret (KMS)                  │
│  ├─ Automatic rotation (Lambda)             │
│  └─ Version management                      │
└────────────┬────────────────────────────────┘
             ↓ (Decrypt with KMS)
┌─────────────────────────────────────────────┐
│  AWS KMS (Customer Master Key)              │
│  └─ Decrypt secret and return to app        │
└─────────────────────────────────────────────┘
```

**Best Practices:**
- **Never** hardcode credentials in source code or config files
- Use IAM roles for EC2, Lambda, ECS tasks
- Implement automatic secret rotation (30-90 days)
- Use fine-grained IAM policies for secret access
- Enable CloudTrail logging for secret access
- Use VPC endpoints to avoid internet traffic
- Tag secrets for better management

**Common Patterns:**
```python
# Bad - Hardcoded credentials ❌
username = "admin"
password = "SuperSecret123"

# Good - Using Secrets Manager ✅
import boto3
import json

client = boto3.client('secretsmanager')
response = client.get_secret_value(SecretId='prod/db/credentials')
secret = json.loads(response['SecretString'])
username = secret['username']
password = secret['password']
```

**Hands-On Labs:**
- Store secrets in Secrets Manager
- Configure automatic rotation for RDS credentials
- Use Parameter Store for application configuration
- Implement secret access from Lambda function

**[📖 Detailed Phase 6 Guide →](./roadmap/phase-6-secrets-management.md)**

---

### Phase 7 – Compute Security 💻

**What You'll Learn:**
- EC2 security best practices
- IMDSv2 (Instance Metadata Service v2)
- Systems Manager Session Manager (SSM)
- AWS Systems Manager Patch Manager
- Golden AMIs and hardening
- Lambda security considerations
- Container security (ECS, EKS, Fargate)
- ECR image scanning
- IAM Roles for Service Accounts (IRSA) in EKS

**EC2 Security Best Practices:**

1. **Disable IMDSv1, use IMDSv2 only**
   - Prevents SSRF attacks from accessing metadata
   
2. **Use SSM Session Manager instead of SSH**
   - No need for SSH keys or bastion hosts
   - All sessions logged in CloudTrail
   
3. **Implement automated patching**
   - Use Systems Manager Patch Manager
   - Define maintenance windows
   
4. **Create hardened Golden AMIs**
   - Remove unnecessary software
   - Apply security configurations
   - Automate with Packer

5. **Enable detailed monitoring and logging**

**Lambda Security Checklist:**
- [ ] Use least privilege IAM execution roles
- [ ] Store secrets in Secrets Manager, not environment variables
- [ ] Enable VPC configuration only when needed
- [ ] Set appropriate timeout and memory limits
- [ ] Enable X-Ray tracing for debugging
- [ ] Use Lambda layers for shared dependencies
- [ ] Implement input validation
- [ ] Use AWS SAM or Terraform for IaC

**Container Security (ECS/EKS):**

**ASCII Diagram - EKS Security Architecture:**
```
┌──────────────────────────────────────────────┐
│  Amazon EKS Cluster                          │
│  ┌────────────────────────────────────────┐  │
│  │  Namespace: Production                 │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │  Pod (Container)                 │  │  │
│  │  │  ├─ IRSA (IAM Role)              │  │  │
│  │  │  ├─ Network Policy               │  │  │
│  │  │  ├─ Pod Security Policy          │  │  │
│  │  │  └─ Resource Limits              │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────┐  │
│  │  ECR (Container Registry)              │  │
│  │  ├─ Image Scanning (Vulnerabilities)   │  │
│  │  ├─ Image Signing                      │  │
│  │  └─ Lifecycle Policies                 │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

**Container Security Best Practices:**
- Scan images with Amazon ECR or third-party tools
- Use distroless or minimal base images
- Implement least privilege for pod IAM roles (IRSA)
- Use network policies to restrict pod-to-pod communication
- Enable audit logging for Kubernetes API
- Regularly update container images
- Use Pod Security Standards/Policies

**Hands-On Labs:**
- Configure IMDSv2 on EC2 instances
- Use SSM Session Manager to connect to instances
- Deploy containerized app with ECR scanning
- Implement IRSA for EKS pods

**[📖 Detailed Phase 7 Guide →](./roadmap/phase-7-compute-security.md)**

---

### Phase 8 – Multi-Account & Organizational Security 🏢

**What You'll Learn:**
- AWS Organizations fundamentals
- Organizational Units (OUs) structure
- Service Control Policies (SCPs)
- AWS Control Tower
- Landing Zone architecture
- Delegated administrator for Security Hub & GuardDuty
- Centralized logging architecture
- Cross-account IAM roles
- AWS Resource Access Manager (RAM)

**ASCII Diagram - Multi-Account Organization Structure:**
```
┌─────────────────────────────────────────────────────────┐
│  Management Account (Root)                              │
│  └─ Billing & Organization Management Only              │
└────────────┬────────────────────────────────────────────┘
             ↓
┌────────────────────────────────────────────────────────┐
│  Security OU                                           │
│  ├─ Security Tooling Account (GuardDuty, Security Hub)│
│  ├─ Log Archive Account (CloudTrail, Config, VPC Logs)│
│  └─ Audit Account (Read-only access for auditors)     │
└────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────┐
│  Workloads OU                                          │
│  ├─ Production Account                                 │
│  ├─ Staging Account                                    │
│  ├─ Development Account                                │
│  └─ Sandbox Accounts                                   │
└────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────┐
│  Infrastructure OU                                     │
│  ├─ Network Account (Transit Gateway, VPC)            │
│  └─ Shared Services Account (AD, DNS)                 │
└────────────────────────────────────────────────────────┘
```

**Service Control Policies (SCPs) - Key Use Cases:**

1. **Prevent disabling security services:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "guardduty:DeleteDetector",
        "securityhub:DisableSecurityHub",
        "config:DeleteConfigurationRecorder"
      ],
      "Resource": "*"
    }
  ]
}
```

2. **Restrict regions:**
```json
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
```

**AWS Control Tower Benefits:**
- Pre-configured multi-account environment
- Automated account provisioning (Account Factory)
- Guardrails (detective and preventive)
- Dashboard for compliance overview
- Integrated with AWS Organizations

**Landing Zone Design Principles:**
- Separate accounts by environment and workload
- Centralize security logging
- Use delegated administrators
- Implement network segmentation
- Automate account creation
- Apply SCPs at OU level

**Best Practices:**
- Never use the management account for workloads
- Implement cross-account roles with MFA
- Use separate accounts for production and development
- Centralize logs in a dedicated account
- Enable all security services at organization level
- Use AWS Control Tower for governance

**Hands-On Labs:**
- Set up AWS Organizations
- Create OUs and apply SCPs
- Deploy AWS Control Tower
- Configure cross-account access

**[📖 Detailed Phase 8 Guide →](./roadmap/phase-8-multi-account.md)**

---

### Phase 9 – Incident Response 🚨

**What You'll Learn:**
- Incident Response (IR) lifecycle
- AWS incident response tools
- Automated response with EventBridge + Lambda
- Forensics with EBS snapshots and memory capture
- Security playbooks and runbooks
- Communication and escalation procedures
- Post-incident reviews and lessons learned

**Incident Response Lifecycle:**
```
┌───────────────────────────────────────────────────────┐
│  1. PREPARE                                           │
│     - Establish IR team and roles                     │
│     - Create playbooks and runbooks                   │
│     - Deploy security tools (GuardDuty, etc.)         │
│     - Set up communication channels                   │
└────────────┬──────────────────────────────────────────┘
             ↓
┌───────────────────────────────────────────────────────┐
│  2. DETECT                                            │
│     - Monitor for security events (GuardDuty, etc.)   │
│     - Alert on anomalies                              │
│     - Triage and validate alerts                      │
└────────────┬──────────────────────────────────────────┘
             ↓
┌───────────────────────────────────────────────────────┐
│  3. CONTAIN                                           │
│     - Isolate affected resources                      │
│     - Block malicious IPs                             │
│     - Revoke compromised credentials                  │
│     - Preserve evidence (snapshots, logs)             │
└────────────┬──────────────────────────────────────────┘
             ↓
┌───────────────────────────────────────────────────────┐
│  4. ERADICATE                                         │
│     - Remove malware and backdoors                    │
│     - Patch vulnerabilities                           │
│     - Update security controls                        │
└────────────┬──────────────────────────────────────────┘
             ↓
┌───────────────────────────────────────────────────────┐
│  5. RECOVER                                           │
│     - Restore systems from clean backups              │
│     - Monitor for signs of reinfection                │
│     - Return to normal operations                     │
└────────────┬──────────────────────────────────────────┘
             ↓
┌───────────────────────────────────────────────────────┐
│  6. LESSONS LEARNED                                   │
│     - Conduct post-incident review                    │
│     - Update playbooks and procedures                 │
│     - Implement improvements                          │
└───────────────────────────────────────────────────────┘
```

**Common Incident Scenarios & Response:**

| Incident Type | Detection | Containment | Tools |
|---------------|-----------|-------------|-------|
| **Compromised IAM credentials** | GuardDuty finding | Disable user, rotate keys | IAM, CloudTrail |
| **Malicious EC2 instance** | GuardDuty, VPC Flow Logs | Isolate with Security Group | EC2, GuardDuty |
| **S3 data exfiltration** | CloudTrail, Macie | Block access, revoke permissions | S3, IAM, CloudTrail |
| **DDoS attack** | CloudWatch, Shield | Enable Shield Advanced, WAF rules | Shield, WAF |
| **Cryptomining** | GuardDuty, CloudWatch | Terminate instance, block IPs | EC2, GuardDuty |

**Automated Response with EventBridge + Lambda:**
```
┌────────────────────────────────────────────────┐
│  GuardDuty Finding                             │
│  (e.g., UnauthorizedAccess:IAMUser)            │
└────────────┬───────────────────────────────────┘
             ↓
┌────────────────────────────────────────────────┐
│  Amazon EventBridge Rule                       │
│  (Filter specific finding types)               │
└────────────┬───────────────────────────────────┘
             ↓
┌────────────────────────────────────────────────┐
│  Lambda Function (Automated Response)          │
│  - Disable IAM user                            │
│  - Revoke active sessions                      │
│  - Send SNS notification                       │
│  - Create incident ticket                      │
└────────────────────────────────────────────────┘
```

**Forensics Best Practices:**
1. **Preserve evidence immediately:**
   - Create EBS snapshots before terminating instances
   - Export CloudTrail logs to S3
   - Capture memory dumps if needed

2. **Maintain chain of custody:**
   - Document all actions
   - Use separate forensics account
   - Restrict access to evidence

3. **Analyze in isolated environment:**
   - Use dedicated forensics VPC
   - No internet access
   - Use forensics tools (Volatility, Sleuth Kit)

**IR Playbook Example - Compromised EC2 Instance:**
1. **Detect**: GuardDuty finding "Backdoor:EC2/C&CActivity.B"
2. **Assess**: Review instance details, VPC Flow Logs, CloudTrail
3. **Contain**:
   - Create EBS snapshot for forensics
   - Isolate instance (modify Security Group to deny all)
   - Tag instance with "Incident-" + ticket number
4. **Eradicate**:
   - Identify and patch vulnerability
   - Update AMI with security fixes
5. **Recover**:
   - Launch new instance from patched AMI
   - Update DNS/load balancer
   - Terminate compromised instance
6. **Post-Incident**:
   - Conduct forensics on snapshot
   - Update playbooks
   - Report to stakeholders

**Hands-On Labs:**
- Create IR playbooks for common scenarios
- Build automated response with EventBridge + Lambda
- Practice forensics with EBS snapshots
- Simulate incident response drill

**[📖 Detailed Phase 9 Guide →](./roadmap/phase-9-incident-response.md)**

---

### Phase 10 – Infrastructure as Code (IaC) Security 📝

**What You'll Learn:**
- IaC fundamentals (Terraform, CloudFormation)
- Security scanning tools
- Policy as Code
- CI/CD pipeline security
- Secrets management in IaC
- State file security
- Drift detection

**Popular IaC Security Scanning Tools:**

| Tool | Language | Features | Cost |
|------|----------|----------|------|
| **Checkov** | Python | Multi-cloud, 1000+ policies | Free (Open Source) |
| **tfsec** | Go | Terraform-specific, fast | Free (Open Source) |
| **cfn-nag** | Ruby | CloudFormation scanning | Free (Open Source) |
| **Semgrep** | Python | Custom rules, code patterns | Free + Paid |
| **Terrascan** | Go | Multi-IaC support | Free (Open Source) |
| **Snyk IaC** | - | Commercial, extensive rules | Paid |
| **Bridgecrew** | - | Platform, remediation | Paid |

**ASCII Diagram - Secure IaC Pipeline:**
```
┌────────────────────────────────────────────────────┐
│  Developer                                         │
│  └─> git push (Terraform code)                    │
└────────────┬───────────────────────────────────────┘
             ↓
┌────────────────────────────────────────────────────┐
│  CI/CD Pipeline (GitHub Actions, GitLab, Jenkins)  │
│  ┌──────────────────────────────────────────────┐  │
│  │  1. Code Checkout                            │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │  2. Static Analysis                          │  │
│  │     - tfsec (Terraform security)             │  │
│  │     - Checkov (Compliance checks)            │  │
│  │     - Semgrep (Custom rules)                 │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │  3. Terraform Plan                           │  │
│  │     - Review proposed changes                │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │  4. Manual Approval (Production)             │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │  5. Terraform Apply                          │  │
│  │     - Deploy to AWS                          │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │  6. Post-Deployment Validation               │  │
│  │     - AWS Config compliance check            │  │
│  └──────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────┘
```

**Common IaC Security Issues:**

1. **Hardcoded Secrets** ❌
```hcl
# Bad
resource "aws_db_instance" "database" {
  username = "admin"
  password = "SuperSecret123"  # Never do this!
}
```

```hcl
# Good ✅
data "aws_secretsmanager_secret_version" "db_creds" {
  secret_id = "prod/db/credentials"
}

resource "aws_db_instance" "database" {
  username = jsondecode(data.aws_secretsmanager_secret_version.db_creds.secret_string)["username"]
  password = jsondecode(data.aws_secretsmanager_secret_version.db_creds.secret_string)["password"]
}
```

2. **Public S3 Buckets** ❌
```hcl
# Bad - Public bucket
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
  acl    = "public-read"  # Dangerous!
}
```

```hcl
# Good - Private bucket with Block Public Access ✅
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

3. **Overly Permissive Security Groups** ❌
```hcl
# Bad - Open to the world
resource "aws_security_group" "web" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Too permissive!
  }
}
```

```hcl
# Good - Specific rules ✅
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

**State File Security:**
- Store Terraform state in S3 with encryption
- Enable versioning on state bucket
- Use DynamoDB for state locking
- Restrict access to state files (contain sensitive data)
- Never commit state files to version control

**Example: Secure Terraform Backend**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:123456789:key/..."
    dynamodb_table = "terraform-locks"
  }
}
```

**Policy as Code with Sentinel (Terraform Enterprise):**
```hcl
# Enforce encryption on all S3 buckets
import "tfplan/v2" as tfplan

main = rule {
  all tfplan.resource_changes as _, rc {
    rc.type is "aws_s3_bucket" implies
    rc.change.after.server_side_encryption_configuration is not null
  }
}
```

**Best Practices:**
- Scan IaC before deployment (shift-left security)
- Use modules with security built-in
- Implement peer review for IaC changes
- Store secrets in Secrets Manager/Parameter Store
- Enable drift detection
- Maintain separate environments (dev, staging, prod)
- Use CI/CD for consistent deployments

**Hands-On Labs:**
- Write Terraform code for secure VPC
- Run Checkov and tfsec scans
- Build CI/CD pipeline with security gates
- Implement policy as code

**[📖 Detailed Phase 10 Guide →](./roadmap/phase-10-iac-security.md)**

---

## 🎓 Certifications Roadmap

A strategic certification path to validate your cloud security expertise:

### Certification Tiers

```
┌─────────────────────────────────────────────────────┐
│  EXPERT LEVEL                                       │
│  ┌───────────────────────────────────────────────┐  │
│  │  CISSP (Certified Information Systems        │  │
│  │         Security Professional)                │  │
│  │  CCSP (Certified Cloud Security Professional)│  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  SPECIALTY LEVEL                                    │
│  ┌───────────────────────────────────────────────┐  │
│  │  ⭐ AWS Certified Security – Specialty        │  │
│  │  (SCS-C02) - PRIMARY TARGET                   │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  ASSOCIATE LEVEL                                    │
│  ┌───────────────────────────────────────────────┐  │
│  │  AWS Certified Solutions Architect Associate  │  │
│  │  (SAA-C03)                                    │  │
│  │  AWS Certified SysOps Administrator Associate │  │
│  │  (SOA-C02)                                    │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│  FOUNDATION LEVEL                                   │
│  ┌───────────────────────────────────────────────┐  │
│  │  AWS Certified Cloud Practitioner (CLF-C02)   │  │
│  │  CompTIA Security+ (Optional)                 │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Detailed Certification Guide

#### 1. AWS Certified Cloud Practitioner (CLF-C02)
**Level**: Foundation  
**Duration**: 90 minutes  
**Cost**: $100  
**Prerequisites**: None

**What It Covers:**
- AWS Cloud concepts
- Security and compliance basics
- Core AWS services
- Billing and pricing

**Recommended Study Time**: 2-4 weeks  
**Best For**: Complete beginners to AWS

**Study Resources:**
- AWS Cloud Practitioner Essentials (free on AWS Skill Builder)
- AWS Certified Cloud Practitioner Official Study Guide
- Practice exams on Udemy

---

#### 2. AWS Certified Solutions Architect Associate (SAA-C03)
**Level**: Associate  
**Duration**: 130 minutes  
**Cost**: $150  
**Prerequisites**: Recommended 1 year AWS experience

**What It Covers:**
- Design resilient architectures
- High-performing architectures
- Secure applications and architectures
- Cost-optimized architectures

**Key Security Topics:**
- IAM policies and roles
- VPC design and security
- Data encryption (KMS)
- Security best practices

**Recommended Study Time**: 1-2 months  
**Best For**: Understanding AWS architecture before diving into security

**Study Resources:**
- Stephane Maarek's SAA-C03 course (Udemy)
- AWS Solutions Architect Associate Official Study Guide
- Adrian Cantrill's course
- Tutorials Dojo practice exams

---

#### 3. ⭐ AWS Certified Security – Specialty (SCS-C02)
**Level**: Specialty  
**Duration**: 170 minutes  
**Cost**: $300  
**Prerequisites**: Recommended SAA or SysOps + 2 years AWS security experience

**Exam Domains:**

| Domain | Weight |
|--------|--------|
| 1. Threat Detection and Incident Response | 14% |
| 2. Security Logging and Monitoring | 18% |
| 3. Infrastructure Security | 20% |
| 4. Identity and Access Management | 16% |
| 5. Data Protection | 18% |
| 6. Management and Security Governance | 14% |

**What It Covers:**
- All topics in Phase 1-10 of this roadmap
- In-depth security services (GuardDuty, Security Hub, Macie, etc.)
- Incident response procedures
- Compliance and governance
- Encryption and key management
- Network security (WAF, Shield, Firewall)

**Recommended Study Time**: 2-3 months (with hands-on labs)

**Study Resources:**
- Zeal Vora's AWS Security Specialty course (Udemy)
- Adrian Cantrill's Security Specialty course
- AWS Security Specialty Official Study Guide
- Tutorials Dojo practice exams
- A Cloud Guru / Pluralsight courses
- **Practice with CloudGoat** (hands-on vulnerable AWS environment)

**Hands-On Practice is Critical:**
- Build security architectures in your own AWS account
- Complete labs on AWS Skill Builder
- Practice with CloudGoat scenarios
- Review real-world security incidents

---

#### 4. AWS Certified SysOps Administrator Associate (SOA-C02)
**Level**: Associate  
**Duration**: 180 minutes (includes lab portion)  
**Cost**: $150  
**Prerequisites**: Recommended 1 year AWS operations experience

**What It Covers:**
- Monitoring, logging, and remediation
- Security and compliance
- Deployment, provisioning, and automation
- Networking and content delivery

**Why It's Useful for Security:**
- Deep understanding of CloudWatch, CloudTrail
- Systems Manager (Patch Manager, Session Manager)
- Operational best practices
- Includes hands-on labs (unique to this cert)

**Recommended Study Time**: 1-2 months

---

#### 5. Certified Cloud Security Professional (CCSP)
**Level**: Expert  
**Issuing Organization**: (ISC)²  
**Cost**: $599  
**Prerequisites**: 5 years IT experience (3 in security, 1 in cloud)

**What It Covers:**
- Cloud concepts and architecture
- Cloud data security
- Cloud platform and infrastructure security
- Cloud application security
- Cloud security operations
- Legal, risk, and compliance

**Best For**: Senior security professionals, multi-cloud environments

---

#### 6. Certified Information Systems Security Professional (CISSP)
**Level**: Expert  
**Issuing Organization**: (ISC)²  
**Cost**: $749  
**Prerequisites**: 5 years security experience

**What It Covers:**
- 8 domains of security (not cloud-specific)
- Security and risk management
- Asset security
- Security architecture
- Communications and network security

**Best For**: Security leadership roles, management positions

---

#### 7. CompTIA Security+
**Level**: Foundation  
**Cost**: $392  
**Prerequisites**: None (but Network+ recommended)

**What It Covers:**
- General security concepts
- Threats, attacks, and vulnerabilities
- Architecture and design
- Implementation
- Operations and incident response

**Best For**: Those new to security, DoD 8570 compliance

---

### Recommended Certification Path

**For Career Changers (0-1 year experience):**
1. CompTIA Security+ (optional, foundation)
2. AWS Cloud Practitioner (CLF-C02)
3. AWS Solutions Architect Associate (SAA-C03)
4. AWS Certified Security – Specialty (SCS-C02) ⭐

**For Cloud Engineers Transitioning to Security:**
1. AWS Solutions Architect Associate (SAA-C03) - if not already certified
2. AWS Certified Security – Specialty (SCS-C02) ⭐
3. CCSP or CISSP (for senior roles)

**For Security Professionals Learning Cloud:**
1. AWS Cloud Practitioner (CLF-C02)
2. AWS Certified Security – Specialty (SCS-C02) ⭐
3. AWS Solutions Architect Associate (SAA-C03) - for broader AWS knowledge

---

## 🎮 Hands-On Practice Resources

Theory alone won't make you a Cloud Security Engineer. Hands-on practice is essential!

### 1. AWS Free Tier
**Cost**: Free (12 months)  
**What You Get:**
- 750 hours/month EC2 t2.micro instances
- 5GB S3 storage
- Free tier for most security services (GuardDuty, Inspector, etc.)

**Practice Ideas:**
- Build a secure 3-tier VPC
- Configure GuardDuty and generate test findings
- Implement automated remediation with Lambda
- Set up CloudTrail and analyze logs

**⚠️ Cost Warning**: Always set up billing alerts!

---

### 2. AWS Skill Builder
**Cost**: Free tier available, Premium $29/month  
**URL**: [https://skillbuilder.aws](https://skillbuilder.aws)

**What's Available:**
- 500+ free digital courses
- Hands-on labs with temporary AWS accounts
- Learning plans for certifications
- Game-based learning (Cloud Quest, Jam)

**Recommended Labs:**
- Introduction to AWS Identity and Access Management (IAM)
- Introduction to Amazon GuardDuty
- Securing VPC with Security Groups and NACLs
- AWS WAF Rules Configuration

---

### 3. CloudGoat (Rhino Security Labs)
**Cost**: Free (Open Source)  
**URL**: [https://github.com/RhinoSecurityLabs/cloudgoat](https://github.com/RhinoSecurityLabs/cloudgoat)

**What It Is:**
- Vulnerable-by-design AWS infrastructure
- Practice finding and exploiting cloud misconfigurations
- Learn attacker techniques to better defend

**Scenarios:**
- IAM privilege escalation
- Lambda function exploitation
- EC2 SSRF attacks
- S3 misconfigurations

**Perfect For**: Hands-on offensive security practice

---

### 4. Flaws.cloud & Flaws2.cloud
**Cost**: Free  
**URL**: [http://flaws.cloud](http://flaws.cloud), [http://flaws2.cloud](http://flaws2.cloud)

**What It Is:**
- CTF-style challenges focusing on AWS security
- Learn by exploiting misconfigurations
- Teaches common AWS security pitfalls

**Topics Covered:**
- S3 bucket misconfigurations
- IAM permissions issues
- EC2 metadata service exploitation
- Snapshot and AMI exposure

---

### 5. TryHackMe
**Cost**: Free tier, Premium $10.99/month  
**URL**: [https://tryhackme.com](https://tryhackme.com)

**AWS-Related Rooms:**
- Advent of Cyber (includes cloud challenges)
- S3 Vulnerabilities
- Cloud Security 101
- Penetration Testing Fundamentals

---

### 6. A Cloud Guru / Pluralsight
**Cost**: $35-47/month  

**What's Available:**
- Guided learning paths
- Hands-on labs with cloud playgrounds
- Practice exams
- Comprehensive video courses

**Recommended Courses:**
- AWS Certified Security Specialty
- AWS Security Essentials
- Practical Event Driven Security

---

### 7. AWS Well-Architected Labs
**Cost**: Free (pay only for AWS resources used)  
**URL**: [https://wellarchitectedlabs.com](https://wellarchitectedlabs.com)

**Security Labs:**
- Automated Deployment of Detective Controls
- Automated Deployment of IAM Groups and Roles
- Automated Deployment of VPC
- CloudFront with S3 Bucket Origin

---

### 8. HackTheBox (Cloud Challenges)
**Cost**: Free tier, VIP $14/month  
**URL**: [https://www.hackthebox.com](https://www.hackthebox.com)

**What's Available:**
- Cloud-focused pentesting challenges
- Retired boxes for practice
- Certifications: CPTS includes cloud

---

### 9. PentesterLab
**Cost**: Free tier, Pro $20/month  
**URL**: [https://pentesterlab.com](https://pentesterlab.com)

**AWS-Related Badges:**
- AWS Security badges
- Cloud exploitation techniques

---

### 10. Sadcloud (NCCGROUP)
**Cost**: Free (Open Source)  
**URL**: [https://github.com/nccgroup/sadcloud](https://github.com/nccgroup/sadcloud)

**What It Is:**
- Tool for spinning up insecure AWS infrastructure
- Practice finding misconfigurations
- Learn security scanning tools

---

### Practice Projects to Build Your Portfolio

1. **Secure Landing Zone**
   - Multi-account setup with AWS Organizations
   - Centralized logging and monitoring
   - Automated security checks

2. **Automated Incident Response**
   - EventBridge + Lambda for automated remediation
   - GuardDuty integration
   - SNS notifications

3. **Secure CI/CD Pipeline**
   - GitHub Actions or GitLab CI
   - IaC security scanning (Checkov, tfsec)
   - Automated deployments with security gates

4. **Security Monitoring Dashboard**
   - CloudWatch dashboards
   - Security Hub integration
   - Custom metrics and alarms

5. **Compliance Automation**
   - AWS Config custom rules
   - Automated remediation
   - Compliance reporting

---

## 📅 Suggested Timeline

A realistic 12-month journey from beginner to Cloud Security Engineer:

| Month | Focus Area | Key Activities | Deliverables |
|-------|------------|----------------|--------------|
| **1-2** | **Foundations** | - Complete networking fundamentals course<br>- Learn Linux basics<br>- Start Python for beginners<br>- AWS Cloud Practitioner study | - Set up AWS free tier account<br>- Complete 10 Linux labs<br>- Write basic Python scripts |
| **3** | **AWS Basics + IAM** | - Study for AWS Cloud Practitioner<br>- Deep dive into IAM<br>- Practice with AWS CLI | - **Pass CLF-C02 exam**<br>- Build multi-user IAM setup<br>- Document IAM best practices |
| **4** | **Network Security** | - VPC architecture deep dive<br>- Security Groups vs NACLs<br>- AWS WAF basics | - Build secure 3-tier VPC<br>- Configure VPC Flow Logs<br>- Analyze network traffic |
| **5** | **Data Security** | - KMS and encryption<br>- S3 security best practices<br>- Start SAA-C03 study | - Implement encrypted S3 buckets<br>- Configure KMS key rotation<br>- Secure data at rest and in transit |
| **6** | **Monitoring & Detection** | - CloudTrail, CloudWatch, Config<br>- GuardDuty, Security Hub<br>- Continue SAA-C03 study | - **Pass SAA-C03 exam**<br>- Set up centralized logging<br>- Configure GuardDuty findings |
| **7** | **Compliance & Compute** | - AWS Config Rules<br>- Amazon Inspector<br>- EC2 security (IMDSv2, SSM) | - Build compliance automation<br>- Run Inspector scans<br>- Harden EC2 instances |
| **8** | **Security Specialty Prep** | - Start SCS-C02 focused study<br>- Complete AWS Skill Builder labs<br>- Review all previous topics | - Complete 20 hands-on labs<br>- Take practice exams<br>- Build study notes |
| **9** | **Multi-Account & IaC** | - AWS Organizations, Control Tower<br>- Terraform for AWS<br>- Continue SCS-C02 prep | - Deploy multi-account structure<br>- Write Terraform code with security<br>- Scan IaC with Checkov |
| **10** | **Incident Response** | - IR procedures and playbooks<br>- Forensics basics<br>- Final SCS-C02 review | - Create IR runbooks<br>- Practice incident scenarios<br>- **Pass SCS-C02 exam** ⭐ |
| **11** | **Advanced Topics & CTFs** | - Complete CloudGoat scenarios<br>- Participate in AWS CTF<br>- Flaws.cloud challenges | - Complete 5 CloudGoat scenarios<br>- Document findings<br>- Build CTF write-ups |
| **12** | **Portfolio & Job Prep** | - Build personal projects<br>- Create GitHub portfolio<br>- Update resume and LinkedIn<br>- Apply for jobs | - 3 portfolio projects on GitHub<br>- Technical blog posts<br>- Start job applications |

### Weekly Time Commitment Recommendations

- **Months 1-6**: 10-15 hours/week
- **Months 7-10** (Cert prep): 15-20 hours/week
- **Months 11-12**: 10-15 hours/week

### Flexibility Notes

- This is a **suggested** timeline. Adjust based on your background and availability.
- If you already have AWS experience, you can skip or accelerate early months.
- If you have security experience, you might move faster through security concepts.
- Working full-time? Extend the timeline to 18-24 months.
- Career changers may need additional foundational time.

---

## 💼 Day-to-Day Responsibilities of a Cloud Security Engineer

Here's what you'll actually be doing on the job:

### Common Tasks & Tools

| Responsibility | Tasks | Tools & Services |
|----------------|-------|------------------|
| **Security Monitoring** | - Review GuardDuty, Security Hub findings<br>- Analyze CloudTrail logs for anomalies<br>- Monitor for policy violations<br>- Investigate security alerts | GuardDuty, Security Hub, CloudWatch, Detective, Splunk, SIEM tools |
| **IAM Management** | - Review and approve access requests<br>- Audit IAM policies and permissions<br>- Implement least privilege<br>- Conduct access reviews | IAM, IAM Access Analyzer, AWS SSO, Okta, Terraform |
| **Vulnerability Management** | - Run and review Inspector scans<br>- Patch management with Systems Manager<br>- Container image scanning<br>- Track remediation efforts | Inspector, Systems Manager, ECR scanning, Qualys, Tenable |
| **Compliance** | - Maintain compliance with frameworks<br>- Respond to audit requests<br>- Implement AWS Config Rules<br>- Generate compliance reports | AWS Config, Audit Manager, AWS Artifact, Security Hub standards |
| **Incident Response** | - Respond to security incidents<br>- Conduct forensic investigations<br>- Coordinate with teams during incidents<br>- Write post-incident reports | GuardDuty, CloudTrail, Detective, VPC Flow Logs, Forensics tools |
| **Architecture Reviews** | - Review infrastructure designs<br>- Provide security recommendations<br>- Threat modeling<br>- Security assessments | AWS Well-Architected Tool, Threat modeling frameworks |
| **Automation** | - Build security automation<br>- Create remediation workflows<br>- Develop security tools<br>- Infrastructure as Code | Lambda, EventBridge, Step Functions, Python, Terraform |
| **Training & Awareness** | - Conduct security training<br>- Create security documentation<br>- Mentor developers on secure coding<br>- Security champions program | Confluence, Slack, Internal wikis |

### Typical Day Schedule

**Morning:**
- 9:00 AM: Check overnight GuardDuty/Security Hub findings
- 9:30 AM: Daily stand-up with security team
- 10:00 AM: Review and triage security alerts
- 11:00 AM: Work on compliance automation project

**Afternoon:**
- 12:00 PM: Lunch
- 1:00 PM: Architecture review meeting with dev team
- 2:00 PM: Investigate suspicious CloudTrail activity
- 3:00 PM: Update IAM policies for new service
- 4:00 PM: Write security runbook documentation
- 5:00 PM: Review pull requests with security implications

**Variables:**
- Incident response can disrupt the schedule
- On-call rotations for security incidents
- Quarterly compliance audits require extra focus
- Project work varies (automation, tooling, etc.)

### Skills Matrix

| Skill Category | Beginner | Intermediate | Advanced |
|----------------|----------|--------------|----------|
| **AWS Services** | Basic understanding of IAM, VPC, S3 | Configure security services, troubleshoot issues | Design complex architectures, optimize costs |
| **Scripting** | Read and modify scripts | Write automation scripts | Develop tools and frameworks |
| **Incident Response** | Assist with investigations | Lead minor incidents | Lead major incidents, forensics |
| **Compliance** | Understand requirements | Implement controls | Design compliance programs |

---

## 🚀 Quick Start Guide

**Ready to start TODAY? Follow these steps:**

### Week 1: Foundation Setup

- [ ] Create AWS free tier account
- [ ] Set up billing alerts ($5, $10, $20 thresholds)
- [ ] Enable MFA on root account
- [ ] Create IAM admin user (not root)
- [ ] Install AWS CLI on your machine
- [ ] Set up `~/.aws/credentials` with IAM user
- [ ] Join AWS subreddit and Discord communities
- [ ] Bookmark AWS Security Blog

**Commands:**
```bash
# Install AWS CLI (macOS)
brew install awscli

# Install AWS CLI (Linux)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure
```

### Week 2: First Hands-On Lab

- [ ] Complete "Introduction to IAM" on AWS Skill Builder
- [ ] Create IAM users, groups, and policies
- [ ] Enable MFA for IAM users
- [ ] Try IAM Policy Simulator
- [ ] Document what you learned in a blog post or note

### Week 3: Networking Fundamentals

- [ ] Watch NetworkChuck TCP/IP videos
- [ ] Understand subnetting and CIDR notation
- [ ] Practice with subnet calculator
- [ ] Read AWS VPC documentation
- [ ] Complete "VPC Basics" on AWS Skill Builder

### Week 4: First AWS Project

- [ ] Build a simple VPC with public and private subnets
- [ ] Launch EC2 instance in public subnet
- [ ] Configure Security Groups
- [ ] Connect via SSH (then disable SSH and use SSM)
- [ ] Document your architecture with a diagram

### First Month Goal
**By end of Month 1**: You should be comfortable with AWS console, IAM basics, and VPC fundamentals. Consider scheduling your AWS Cloud Practitioner exam for end of Month 3.

---

## 📚 Additional Resources

### Books
- **"AWS Security" by Dylan Shield** - Comprehensive AWS security guide
- **"AWS Certified Security Study Guide" by Wally Rowe** - Official exam prep
- **"The Phoenix Project"** - DevOps and security culture
- **"NIST Cybersecurity Framework"** - Security fundamentals

### Blogs & Websites
- AWS Security Blog: [https://aws.amazon.com/blogs/security/](https://aws.amazon.com/blogs/security/)
- Rhino Security Labs Blog: [https://rhinosecuritylabs.com/blog/](https://rhinosecuritylabs.com/blog/)
- Chris Farris Blog: [https://www.chrisfarris.com/](https://www.chrisfarris.com/)
- AWS re:Inforce YouTube Channel

### Communities
- r/AWSCertifications (Reddit)
- AWS Security Subreddit
- TechStudySlack (Slack)
- AWS Community Builders Program

### YouTube Channels
- Stephane Maarek
- FreeCodeCamp AWS courses
- NetworkChuck
- Be A Better Dev

### Podcasts
- AWS Podcast (Security episodes)
- Cloud Security Podcast
- Darknet Diaries (Security stories)

---

## 🎯 Success Tips

1. **Hands-On Practice > Theory**: Always lab what you learn
2. **Document Everything**: Build your own knowledge base
3. **Build in Public**: Share your learning journey (blog, LinkedIn)
4. **Join Communities**: Learn from others, ask questions
5. **Real Projects Matter**: Build portfolio projects on GitHub
6. **Stay Current**: Follow AWS Security Blog for new services
7. **Think Like an Attacker**: Understand offensive techniques to defend better
8. **Automate When Possible**: Learn to code security solutions
9. **Be Patient**: This is a 12+ month journey, not a sprint
10. **Have Fun**: Cloud security is exciting and rewarding!

---

## 🌟 Your Journey Starts Now

Cloud security is one of the most in-demand skills in tech. With dedication, hands-on practice, and this roadmap, you'll be well on your way to becoming a Cloud Security Engineer.

**Remember:**
- Start small, build momentum
- Practice consistently (10-15 hours/week)
- Certifications validate knowledge, but hands-on skills get jobs
- Build a portfolio of real projects
- Stay curious and keep learning

**Good luck on your Cloud Security Journey! 🚀**

---

## 📖 Detailed Phase Guides

For in-depth guides, commands, and examples for each phase, see:

- [Phase 1: Identity & Access Management](./roadmap/phase-1-iam.md)
- [Phase 2: Network Security](./roadmap/phase-2-network-security.md)
- [Phase 3: Data Security & Encryption](./roadmap/phase-3-data-security.md)
- [Phase 4: Monitoring, Logging & Threat Detection](./roadmap/phase-4-monitoring-logging.md)
- [Phase 5: Vulnerability & Compliance Management](./roadmap/phase-5-compliance.md)
- [Phase 6: Secrets & Credentials Management](./roadmap/phase-6-secrets-management.md)
- [Phase 7: Compute Security](./roadmap/phase-7-compute-security.md)
- [Phase 8: Multi-Account & Organizational Security](./roadmap/phase-8-multi-account.md)
- [Phase 9: Incident Response](./roadmap/phase-9-incident-response.md)
- [Phase 10: Infrastructure as Code Security](./roadmap/phase-10-iac-security.md)

---

**Version**: 1.0  
**Last Updated**: February 2026  
**Maintained by**: CloudSecurityJourney1 Community

*This is a living document. Contributions and suggestions are welcome via GitHub Issues and Pull Requests.*
