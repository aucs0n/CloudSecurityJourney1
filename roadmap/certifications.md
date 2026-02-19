# 🏆 AWS Certifications Roadmap

> **Your path to AWS security certification excellence**

## 📋 Overview

Certifications validate your cloud security expertise and significantly boost your career prospects. This guide provides a strategic roadmap for AWS certifications, with a focus on the **AWS Certified Security - Specialty** as the primary goal for Cloud Security Engineers.

## 🎯 Recommended Certification Path

```
┌──────────────────────────────────────────────────────────┐
│          CERTIFICATION PROGRESSION                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  START HERE (Optional but Recommended)                   │
│  ┌────────────────────────────────────────────┐        │
│  │ AWS Certified Cloud Practitioner           │        │
│  │ • Foundational knowledge                   │        │
│  │ • 90 minutes, 65 questions                 │        │
│  │ • $100 exam fee                            │        │
│  └────────────────┬───────────────────────────┘        │
│                   │                                      │
│                   ▼                                      │
│  LEVEL 2 (Highly Recommended)                          │
│  ┌────────────────────────────────────────────┐        │
│  │ AWS Certified Solutions Architect          │        │
│  │ - Associate                                │        │
│  │ • Broad AWS knowledge                      │        │
│  │ • 130 minutes, 65 questions                │        │
│  │ • $150 exam fee                            │        │
│  └────────────────┬───────────────────────────┘        │
│                   │                                      │
│                   ▼                                      │
│  ⭐ PRIMARY GOAL ⭐                                     │
│  ┌────────────────────────────────────────────┐        │
│  │ AWS Certified Security - Specialty         │        │
│  │ • Cloud security expertise                 │        │
│  │ • 170 minutes, 65 questions                │        │
│  │ • $300 exam fee                            │        │
│  │ • Valid for 3 years                        │        │
│  └────────────────┬───────────────────────────┘        │
│                   │                                      │
│                   ▼                                      │
│  ADVANCED (Optional)                                    │
│  ┌────────────────────────────────────────────┐        │
│  │ AWS Certified Solutions Architect          │        │
│  │ - Professional                             │        │
│  │ • Advanced architecture skills             │        │
│  │ • 180 minutes, 75 questions                │        │
│  │ • $300 exam fee                            │        │
│  └────────────────────────────────────────────┘        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## 1️⃣ AWS Certified Cloud Practitioner (CLF-C02)

### Overview
- **Level:** Foundational
- **Duration:** 90 minutes
- **Questions:** 65 (multiple choice and multiple response)
- **Passing Score:** 700/1000
- **Cost:** $100
- **Validity:** 3 years

### Is This For You?
✅ New to AWS  
✅ Want to validate basic cloud knowledge  
✅ Non-technical roles transitioning to cloud  
⚠️ Skip if you have 6+ months AWS experience

### Exam Domains
1. **Cloud Concepts (24%)**
   - Value proposition of AWS Cloud
   - Cloud economics
   - Cloud architecture design principles

2. **Security and Compliance (30%)**
   - AWS shared responsibility model
   - Security and compliance concepts
   - Access management capabilities

3. **Cloud Technology and Services (34%)**
   - Methods of deploying and operating in AWS
   - AWS global infrastructure
   - Core AWS services

4. **Billing, Pricing, and Support (12%)**
   - AWS pricing models
   - Account structures
   - Support resources

### Study Strategy (2-4 weeks)
- [ ] Complete AWS Cloud Practitioner Essentials (free)
- [ ] Read AWS whitepapers:
  - Overview of Amazon Web Services
  - AWS Well-Architected Framework
- [ ] Take practice exams (Tutorials Dojo recommended)
- [ ] Review AWS service documentation

### Resources
- **Free:** [AWS Skill Builder - Cloud Practitioner](https://skillbuilder.aws/)
- **Paid:** Udemy - Stephane Maarek's course ($15)
- **Practice Exams:** Tutorials Dojo ($15)

---

## 2️⃣ AWS Certified Solutions Architect - Associate (SAA-C03)

### Overview
- **Level:** Associate
- **Duration:** 130 minutes
- **Questions:** 65
- **Passing Score:** 720/1000
- **Cost:** $150
- **Validity:** 3 years

### Why This Matters for Security Engineers
Understanding AWS architecture is essential for security. This cert covers:
- VPC design and networking
- IAM and access control
- Data protection strategies
- High availability and disaster recovery
- Cost optimization (security budget management)

### Exam Domains
1. **Design Secure Architectures (30%)**
   - Secure access to AWS resources
   - Secure workloads and applications
   - Appropriate data security controls

2. **Design Resilient Architectures (26%)**
   - Scalable and loosely coupled architectures
   - Highly available and fault-tolerant architectures

3. **Design High-Performing Architectures (24%)**
   - Elastic and scalable solutions
   - High-performing storage and compute
   - High-performing database solutions

4. **Design Cost-Optimized Architectures (20%)**
   - Cost-effective storage
   - Cost-effective compute resources

### Study Strategy (6-8 weeks, 10-15 hrs/week)
- [ ] **Week 1-2:** IAM, VPC, EC2, S3 deep dive
- [ ] **Week 3-4:** Databases (RDS, DynamoDB), Load Balancers, Auto Scaling
- [ ] **Week 5-6:** CloudFront, Route 53, Lambda, ECS/EKS
- [ ] **Week 7:** Review weak areas, whitepapers
- [ ] **Week 8:** Practice exams (aim for 80%+ consistently)

### Hands-On Labs (Critical!)
- [ ] Build 3-tier web application with VPC
- [ ] Configure multi-region RDS with failover
- [ ] Set up CloudFront with S3 origin
- [ ] Implement Auto Scaling with ALB
- [ ] Deploy serverless application with Lambda + API Gateway

### Resources
- **Course:** A Cloud Guru or Udemy - Stephane Maarek ($30-40)
- **Practice Exams:** Tutorials Dojo ($20)
- **Hands-On:** AWS Free Tier labs
- **Whitepapers:**
  - AWS Well-Architected Framework
  - AWS Security Best Practices

### Pro Tips
> **💡 Focus on networking!** 30-40% of questions involve VPCs, subnets, route tables, security groups.

> **💡 Understand the "why"** behind architectural decisions, not just "what" services to use.

---

## 3️⃣ ⭐ AWS Certified Security - Specialty (SCS-C02)

### Overview
- **Level:** Specialty
- **Duration:** 170 minutes
- **Questions:** 65
- **Passing Score:** 750/1000
- **Cost:** $300
- **Validity:** 3 years
- **Prerequisites:** None official, but SAA-C03 recommended

### This Is THE Cloud Security Certification
The AWS Security Specialty is the gold standard for cloud security professionals. Passing this exam demonstrates:
- Deep knowledge of AWS security services
- Ability to design and implement secure architectures
- Understanding of compliance and data protection
- Expertise in incident response and logging

### Exam Domains

#### 1. Threat Detection and Incident Response (14%)
- **GuardDuty**
  - Finding types and severity
  - Automated response to findings
  - Integration with Security Hub
  
- **Security Hub**
  - Security standards (CIS, PCI-DSS, AWS Foundational)
  - Custom insights
  - Cross-account aggregation
  
- **CloudWatch & CloudTrail**
  - Log analysis
  - Metric filters and alarms
  - Event patterns for automation
  
- **Amazon Detective**
  - Investigation workflows
  - Visualizations and entity analysis
  
- **Systems Manager**
  - Incident Manager
  - Session logging and auditing

**Key Topics:**
- [ ] GuardDuty finding types and automated remediation
- [ ] CloudTrail log analysis and integrity validation
- [ ] EventBridge rules for security automation
- [ ] Forensic investigation procedures
- [ ] VPC Flow Logs analysis

#### 2. Security Logging and Monitoring (18%)
- **CloudTrail**
  - Organization trails
  - Data events vs management events
  - Log file validation
  - S3 log storage with encryption
  
- **VPC Flow Logs**
  - Flow log format and analysis
  - Athena queries for threat detection
  
- **AWS Config**
  - Configuration recording
  - Compliance rules
  - Remediation actions
  
- **Logging Best Practices**
  - Centralized logging architectures
  - Log retention policies
  - SIEM integration

**Key Topics:**
- [ ] Centralized logging with multi-account Organizations
- [ ] AWS Config rules and conformance packs
- [ ] Log aggregation strategies
- [ ] CloudWatch Logs Insights queries
- [ ] Macie for sensitive data discovery

#### 3. Infrastructure Security (20%)
- **Network Security**
  - VPC design (public/private subnets)
  - Security Groups vs NACLs
  - AWS WAF and Shield
  - Network Firewall
  - VPC endpoints (gateway and interface)
  
- **Compute Security**
  - EC2 security (IMDSv2, instance profiles)
  - Systems Manager Session Manager
  - Lambda security (VPC, IAM, encryption)
  - Container security (ECS, EKS, Fargate)
  
- **Edge Security**
  - CloudFront security
  - Route 53 DNSSEC
  - AWS Global Accelerator

**Key Topics:**
- [ ] VPC design patterns and network segmentation
- [ ] WAF rule creation and managed rule groups
- [ ] PrivateLink architecture
- [ ] Transit Gateway security
- [ ] IMDSv2 enforcement

#### 4. Identity and Access Management (16%)
- **IAM Deep Dive**
  - Policy evaluation logic
  - Permission boundaries
  - Service Control Policies (SCPs)
  - IAM Access Analyzer
  - Resource-based policies
  
- **AWS Organizations**
  - Multi-account strategies
  - Cross-account access patterns
  
- **Directory Services**
  - AWS Directory Service
  - AWS SSO (IAM Identity Center)
  - SAML 2.0 federation
  
- **Cognito**
  - User pools vs identity pools
  - Social identity providers
  - MFA enforcement

**Key Topics:**
- [ ] IAM policy evaluation order (explicit deny wins)
- [ ] Cross-account role assumption with external ID
- [ ] SCPs for organization-wide controls
- [ ] Permission boundaries use cases
- [ ] Temporary credentials with STS

#### 5. Data Protection (18%)
- **Encryption**
  - KMS key types and policies
  - CloudHSM for FIPS 140-2 Level 3
  - Envelope encryption
  - Key rotation strategies
  
- **S3 Security**
  - Encryption options (SSE-S3, SSE-KMS, SSE-C)
  - Bucket policies vs ACLs
  - S3 Access Points
  - Object Lock and Glacier Vault Lock
  - S3 Block Public Access
  
- **Secrets Management**
  - Secrets Manager vs Parameter Store
  - Automatic rotation
  - Cross-account access
  
- **RDS/Aurora Security**
  - Encryption at rest
  - IAM database authentication
  - SSL/TLS connections

**Key Topics:**
- [ ] KMS key policies and grants
- [ ] S3 bucket policy conditions (encryption, secure transport)
- [ ] Secrets rotation with Lambda
- [ ] Certificate Manager (ACM)
- [ ] DynamoDB encryption

#### 6. Management and Security Governance (14%)
- **Compliance**
  - AWS Artifact (compliance reports)
  - Audit Manager
  - Compliance frameworks (CIS, PCI-DSS, HIPAA)
  
- **AWS Control Tower**
  - Landing Zone
  - Guardrails (preventive and detective)
  - Account Factory
  
- **Service Catalog**
  - Compliant resource provisioning
  - Portfolio management
  
- **Trusted Advisor**
  - Security recommendations
  - Cost optimization

**Key Topics:**
- [ ] Control Tower guardrails
- [ ] AWS Config conformance packs
- [ ] Service Catalog constraints
- [ ] Resource tagging strategies
- [ ] Cost allocation and budgets

### Study Strategy (8-12 weeks, 15-20 hrs/week)

#### Weeks 1-3: Foundations
- [ ] Review IAM deeply (policies, roles, federation)
- [ ] Master VPC networking
- [ ] Understand encryption (KMS, CloudHSM, Certificate Manager)
- [ ] Hands-on: Build secure 3-tier VPC architecture

#### Weeks 4-6: Security Services
- [ ] CloudTrail, Config, GuardDuty, Security Hub
- [ ] Macie, Inspector, Detective
- [ ] Systems Manager (Session Manager, Patch Manager)
- [ ] Hands-on: Set up comprehensive security monitoring

#### Weeks 7-9: Advanced Topics
- [ ] Multi-account security with Organizations
- [ ] Control Tower and Landing Zones
- [ ] Incident response automation
- [ ] Container security (ECS, EKS)
- [ ] Hands-on: Deploy Control Tower, build IR automation

#### Weeks 10-11: Practice & Weak Areas
- [ ] Take practice exams (aim for 85%+)
- [ ] Review incorrect answers thoroughly
- [ ] Create flashcards for weak topics
- [ ] Read exam-specific whitepapers

#### Week 12: Final Prep
- [ ] Take 2-3 full practice exams
- [ ] Review all flagged questions
- [ ] Sleep well, schedule exam

### Essential Hands-On Labs
1. **Multi-Account Security Setup**
   - Create AWS Organization
   - Deploy SCPs
   - Centralize CloudTrail and Config
   - Set up GuardDuty with delegated admin

2. **Automated Incident Response**
   - GuardDuty finding triggers Lambda
   - Quarantine compromised EC2 instance
   - Rotate compromised IAM credentials
   - Send SNS notifications

3. **S3 Security Hardening**
   - Block public access
   - Enable versioning and MFA Delete
   - Enforce encryption
   - Configure lifecycle policies

4. **VPC Security Architecture**
   - Public/private subnets
   - NAT Gateway and Internet Gateway
   - VPC endpoints for S3 and DynamoDB
   - Security Groups and NACLs
   - VPC Flow Logs to S3 and CloudWatch

5. **KMS Encryption Implementation**
   - Create customer managed key
   - Configure key policy
   - Enable automatic rotation
   - Encrypt S3 bucket, EBS volume, RDS database

### Must-Read Whitepapers
- [ ] AWS Security Best Practices
- [ ] AWS Well-Architected Framework - Security Pillar
- [ ] AWS Key Management Service Best Practices
- [ ] Security at Scale: Logging in AWS
- [ ] AWS Security Incident Response Guide
- [ ] Organizing Your AWS Environment Using Multiple Accounts

### Resources

**Courses:**
- **A Cloud Guru** - AWS Certified Security Specialty ($39/month)
- **Linux Academy/Pluralsight** - Comprehensive course
- **Udemy - Stephane Maarek** - Highly rated ($20-30)

**Practice Exams:**
- **Tutorials Dojo** - 4 practice exams, 260 questions ($20) ⭐ **HIGHLY RECOMMENDED**
- **Whizlabs** - Multiple practice tests ($15)

**Books:**
- **AWS Certified Security Specialty Official Study Guide** (Wiley, 2020)

**Free Resources:**
- **AWS Skill Builder** - Security learning paths
- **AWS Security Blog** - Latest updates
- **AWS re:Inforce** - Annual security conference videos

### Exam Day Tips

**Before the Exam:**
- [ ] Review flagged topics one final time
- [ ] Get 8 hours of sleep
- [ ] Eat a good breakfast
- [ ] Arrive 15 minutes early (or start online exam early)

**During the Exam:**
- [ ] Read questions carefully (watch for "LEAST secure" or "NOT")
- [ ] Eliminate obviously wrong answers first
- [ ] Flag questions you're unsure about
- [ ] Manage your time (170 min for 65 Q = ~2.5 min/Q)
- [ ] Review flagged questions if time permits

**Question Patterns to Watch:**
- Scenario-based questions with multiple correct answers (choose MOST appropriate)
- IAM policy evaluation questions
- "Which option provides the MOST secure solution?"
- Cost vs security trade-offs
- Compliance requirements

### After Passing

**Update Your Profiles:**
- [ ] LinkedIn - Add certification with badge
- [ ] Resume - Highlight certification
- [ ] AWS Certification Portal - Download digital badge

**Next Steps:**
- [ ] AWS Certified Solutions Architect - Professional
- [ ] AWS Certified DevOps Engineer - Professional
- [ ] Industry certifications (CISSP, CCSP)

---

## 💡 General Certification Tips

### Study Techniques That Work
1. **Active Learning:** Don't just watch videos - build things!
2. **Spaced Repetition:** Review material multiple times over weeks
3. **Practice Exams:** Take at least 3-4 full practice exams
4. **Flashcards:** Create cards for services, use cases, limits
5. **Teaching:** Explain concepts to others (or rubber duck)

### Cost Savings
- **AWS Free Tier:** Use for labs (set billing alerts!)
- **Course Sales:** Wait for Udemy sales ($10-15 instead of $100+)
- **Employer Reimbursement:** Many companies cover exam fees
- **50% Discount:** Pass one exam, get 50% off next exam

### Maintaining Certifications
- **3-year validity:** Certifications expire after 3 years
- **Recertification:** Take same exam again or higher-level exam
- **Continuous Learning:** AWS updates services frequently

---

## 🎯 Your Certification Timeline

### Recommended Timeline for Cloud Security Engineer

**Months 1-2: Foundations**
- Complete Phase 1-2 of this roadmap
- Optionally take Cloud Practitioner

**Months 3-6: Solutions Architect Associate**
- Study for SAA-C03
- Build hands-on labs
- Take exam by Month 6

**Months 7-11: Security Specialty Prep**
- Complete Phases 3-10 of this roadmap
- Deep dive into security services
- Build security labs

**Month 12: Security Specialty Exam**
- Final review and practice exams
- Schedule and pass AWS Security Specialty
- Celebrate! 🎉

---

## 📊 Certification Value

### Job Market Impact
- **Salary Increase:** 10-15% on average with certifications
- **Job Opportunities:** Certifications required for many security roles
- **Credibility:** Validates your expertise to employers

### Career Progression
```
No Cert → Entry-level Security Analyst ($60-80k)
   ↓
AWS SAA → Junior Cloud Security Engineer ($80-100k)
   ↓
AWS Security Specialty → Cloud Security Engineer ($110-140k)
   ↓
Multiple Certs + Experience → Senior Cloud Security Engineer ($140-180k+)
```

---

## 🚀 Next Steps

1. **Choose your certification path** based on experience
2. **Set a target exam date** (creates accountability)
3. **Block study time** on your calendar
4. **Join study groups** (Reddit: r/AWSCertifications, Discord servers)
5. **Start building!** Hands-on experience is invaluable

**Good luck on your certification journey! 🏆**

---

**Back to:** [Main Roadmap](../ROADMAP.md) | **See Also:** [Resources](resources.md)
