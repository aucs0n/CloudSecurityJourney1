# 👁️ Phase 5: Monitoring & Threat Detection

> **Build your security operations foundation**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Security monitoring and threat detection are critical for maintaining a secure AWS environment. This phase covers the essential AWS services that provide visibility, detect threats, and enable rapid response to security incidents.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Configure comprehensive logging with AWS CloudTrail
- ✅ Set up security monitoring with CloudWatch
- ✅ Deploy Amazon GuardDuty for threat detection
- ✅ Centralize findings with AWS Security Hub
- ✅ Discover sensitive data with Amazon Macie
- ✅ Investigate security events with Amazon Detective
- ✅ Ensure compliance with AWS Config
- ✅ Build automated alerting and response workflows

## 📚 Topics & Progress Tracker

### Week 1: CloudTrail & CloudWatch

#### AWS CloudTrail
- [ ] Understand CloudTrail event types
- [ ] Management events vs Data events
- [ ] Create organization trail
- [ ] Enable log file validation
- [ ] Integrate with CloudWatch Logs
- [ ] Store logs in S3 with encryption

**CloudTrail Architecture:**
```
┌─────────────────────────────────────────────────┐
│         AWS CLOUDTRAIL LOGGING                  │
├─────────────────────────────────────────────────┤
│                                                 │
│  API Calls → CloudTrail → S3 + CloudWatch Logs │
│                                                 │
│  Event Types:                                   │
│  ├─ Management Events (Control Plane)          │
│  │  └─ CreateBucket, TerminateInstance, etc.   │
│  │                                              │
│  ├─ Data Events (Data Plane)                   │
│  │  └─ GetObject, PutObject, Lambda Invoke     │
│  │                                              │
│  └─ Insight Events (Anomalous activity)        │
│     └─ Unusual API call volumes                │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Creating CloudTrail:**
```bash
# Create S3 bucket for logs
aws s3api create-bucket \
  --bucket my-cloudtrail-logs \
  --region us-east-1

# Apply bucket policy
cat > cloudtrail-bucket-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::my-cloudtrail-logs"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-cloudtrail-logs/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket my-cloudtrail-logs \
  --policy file://cloudtrail-bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name my-organization-trail \
  --s3-bucket-name my-cloudtrail-logs \
  --is-multi-region-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging \
  --name my-organization-trail
```

> **🔑 Key Takeaway:** CloudTrail should be enabled on day one. It's your security camera for AWS - you can't investigate what you didn't log!

#### CloudTrail Best Practices
- [ ] Enable in all regions
- [ ] Enable log file integrity validation
- [ ] Encrypt logs with KMS
- [ ] Set up S3 lifecycle policies for log retention
- [ ] Monitor for trail modifications
- [ ] Use organization trail for multi-account

**CloudTrail Insights:**
```bash
# Enable CloudTrail Insights (detects unusual activity)
aws cloudtrail put-insight-selectors \
  --trail-name my-organization-trail \
  --insight-selectors '[{"InsightType": "ApiCallRateInsight"}]'
```

#### Amazon CloudWatch for Security
- [ ] Create log groups for security logs
- [ ] Set up metric filters
- [ ] Configure alarms for security events
- [ ] Build CloudWatch dashboards
- [ ] Use CloudWatch Logs Insights for queries

**Security Metric Filters:**
```bash
# Create log group
aws logs create-log-group \
  --log-group-name /aws/security/cloudtrail

# Metric filter for root account usage
aws logs put-metric-filter \
  --log-group-name /aws/security/cloudtrail \
  --filter-name RootAccountUsage \
  --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
  --metric-transformations \
    metricName=RootAccountUsageCount,\
    metricNamespace=CloudTrailMetrics,\
    metricValue=1

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name RootAccountUsageAlarm \
  --alarm-description "Triggers when root account is used" \
  --metric-name RootAccountUsageCount \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:SecurityAlerts
```

**Critical Security Metrics to Monitor:**
- [ ] Root account usage
- [ ] Unauthorized API calls
- [ ] IAM policy changes
- [ ] Security group changes
- [ ] Network ACL changes
- [ ] S3 bucket policy changes
- [ ] Failed console sign-ins
- [ ] CloudTrail logging changes

**CloudWatch Logs Insights Query Examples:**
```
# Failed console sign-ins
fields @timestamp, userIdentity.principalId, errorMessage
| filter eventName = "ConsoleLogin" and errorMessage = "Failed authentication"
| sort @timestamp desc
| limit 100

# IAM policy changes
fields @timestamp, userIdentity.arn, eventName, requestParameters
| filter eventSource = "iam.amazonaws.com" and eventName like /Policy/
| sort @timestamp desc

# S3 bucket deletions
fields @timestamp, userIdentity.arn, requestParameters.bucketName
| filter eventName = "DeleteBucket"
| sort @timestamp desc
```

### Week 2: Amazon GuardDuty

#### GuardDuty Overview
- [ ] Understand threat intelligence sources
- [ ] Enable GuardDuty
- [ ] Configure trusted IP lists and threat lists
- [ ] Integrate with Security Hub
- [ ] Set up automated responses
- [ ] Export findings to S3

**GuardDuty Architecture:**
```
┌──────────────────────────────────────────────────┐
│         AMAZON GUARDDUTY                         │
├──────────────────────────────────────────────────┤
│                                                  │
│  Data Sources:                                   │
│  ├─ VPC Flow Logs                               │
│  ├─ CloudTrail Management Events                │
│  ├─ CloudTrail S3 Data Events                   │
│  └─ DNS Logs                                     │
│                                                  │
│  Threat Intelligence:                            │
│  ├─ AWS Security Research                       │
│  ├─ CrowdStrike                                  │
│  ├─ Proofpoint                                   │
│  └─ Custom threat lists                         │
│                                                  │
│  Finding Types:                                  │
│  ├─ Reconnaissance                               │
│  ├─ Instance Compromise                          │
│  ├─ Account Compromise                           │
│  ├─ Bucket Compromise                            │
│  ├─ Malware                                      │
│  └─ Cryptocurrency Mining                        │
│                                                  │
└──────────────────────────────────────────────────┘
```

**Enable GuardDuty:**
```bash
# Enable GuardDuty
aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES

# Get detector ID
DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)

# Upload trusted IP list
aws guardduty create-ip-set \
  --detector-id $DETECTOR_ID \
  --name TrustedIPs \
  --format TXT \
  --location s3://my-bucket/trusted-ips.txt \
  --activate

# Upload threat list
aws guardduty create-threat-intel-set \
  --detector-id $DETECTOR_ID \
  --name KnownMaliciousIPs \
  --format TXT \
  --location s3://my-bucket/threat-ips.txt \
  --activate
```

#### Common GuardDuty Finding Types
- [ ] **UnauthorizedAccess:EC2/SSHBruteForce** - SSH brute force attack
- [ ] **CryptoCurrency:EC2/BitcoinTool.B!DNS** - Bitcoin mining activity
- [ ] **Trojan:EC2/BlackholeTraffic** - EC2 attempting to communicate with malicious IPs
- [ ] **Recon:EC2/PortProbeUnprotectedPort** - Port scanning activity
- [ ] **UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration** - Credentials used outside EC2
- [ ] **Policy:IAMUser/RootCredentialUsage** - Root credentials usage

**GuardDuty Severity Levels:**
| Level | Score | Action Required |
|-------|-------|----------------|
| **Low** | 0.1-3.9 | Investigate when possible |
| **Medium** | 4.0-6.9 | Investigate within hours |
| **High** | 7.0-8.9 | Investigate immediately |

> **💡 Pro Tip:** Set up automated remediation for common findings. For example, automatically isolate instances flagged for cryptocurrency mining.

#### Automated GuardDuty Response
```python
#!/usr/bin/env python3
"""
Lambda function to auto-isolate compromised EC2 instances
"""
import boto3
import json

def lambda_handler(event, context):
    """
    Triggered by GuardDuty finding via EventBridge
    """
    ec2 = boto3.client('ec2')
    
    # Parse GuardDuty finding
    finding = event['detail']
    finding_type = finding['type']
    severity = finding['severity']
    
    # Only act on high severity instance compromise
    if severity >= 7.0 and 'EC2' in finding_type:
        instance_id = finding['resource']['instanceDetails']['instanceId']
        
        print(f"🚨 High severity finding for instance: {instance_id}")
        print(f"   Finding type: {finding_type}")
        
        # Create forensic security group (deny all)
        forensic_sg = ec2.create_security_group(
            GroupName=f'forensic-{instance_id}',
            Description='Quarantine security group',
            VpcId=get_instance_vpc(ec2, instance_id)
        )
        
        # Remove all ingress/egress rules (isolate instance)
        sg_id = forensic_sg['GroupId']
        
        # Modify instance security group
        ec2.modify_instance_attribute(
            InstanceId=instance_id,
            Groups=[sg_id]
        )
        
        # Create snapshot for forensics
        volumes = ec2.describe_volumes(
            Filters=[{'Name': 'attachment.instance-id', 'Values': [instance_id]}]
        )
        
        for volume in volumes['Volumes']:
            ec2.create_snapshot(
                VolumeId=volume['VolumeId'],
                Description=f'Forensic snapshot - GuardDuty finding'
            )
        
        print(f"✅ Instance {instance_id} isolated and snapshot created")
        
        # Send SNS notification
        sns = boto3.client('sns')
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:123456789012:SecurityAlerts',
            Subject=f'GuardDuty: Instance {instance_id} Quarantined',
            Message=json.dumps(finding, indent=2)
        )
    
    return {'statusCode': 200}

def get_instance_vpc(ec2, instance_id):
    """Get VPC ID for instance"""
    response = ec2.describe_instances(InstanceIds=[instance_id])
    return response['Reservations'][0]['Instances'][0]['VpcId']
```

### Week 3: Security Hub, Detective & Macie

#### AWS Security Hub
- [ ] Enable Security Hub
- [ ] Enable security standards (CIS, PCI-DSS, AWS Foundational)
- [ ] Integrate findings from multiple services
- [ ] Use Security Hub Insights
- [ ] Export findings to SIEM
- [ ] Custom actions for remediation

**Security Hub Architecture:**
```
┌────────────────────────────────────────────────┐
│         AWS SECURITY HUB                       │
│         (Central Dashboard)                    │
├────────────────────────────────────────────────┤
│                                                │
│  Findings from:                                │
│  ├─ GuardDuty                                  │
│  ├─ Inspector                                  │
│  ├─ Macie                                      │
│  ├─ Firewall Manager                           │
│  ├─ IAM Access Analyzer                        │
│  ├─ Systems Manager                            │
│  └─ Third-party tools (50+)                    │
│                                                │
│  Security Standards:                           │
│  ├─ AWS Foundational Security Best Practices  │
│  ├─ CIS AWS Foundations Benchmark             │
│  ├─ PCI-DSS                                    │
│  └─ NIST 800-53                                │
│                                                │
└────────────────────────────────────────────────┘
```

**Enable Security Hub:**
```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Enable security standards
aws securityhub batch-enable-standards \
  --standards-subscription-requests \
    StandardsArn=arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0 \
    StandardsArn=arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.2.0

# Get findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}'
```

#### Amazon Detective
- [ ] Enable Detective
- [ ] Investigate security findings
- [ ] Visualize activity patterns
- [ ] Analyze VPC flow logs
- [ ] Trace suspicious activity
- [ ] Generate investigation reports

**Detective Use Cases:**
- 🔍 Investigate GuardDuty findings
- 🔍 Analyze unusual API activity
- 🔍 Trace credential usage
- 🔍 Identify compromised resources
- 🔍 Understand blast radius of incidents

```bash
# Enable Detective
aws detective create-graph

# Get graph ARN
GRAPH_ARN=$(aws detective list-graphs --query 'GraphList[0].Arn' --output text)

# Invite members
aws detective create-members \
  --graph-arn $GRAPH_ARN \
  --accounts AccountId=123456789012,EmailAddress=security@example.com
```

#### Amazon Macie
- [ ] Enable Macie
- [ ] Create data discovery jobs
- [ ] Identify sensitive data (PII, credentials)
- [ ] Configure sensitive data types
- [ ] Review findings
- [ ] Remediate exposed data

**Macie Capabilities:**
```
Amazon Macie detects:
├─ Personal Identifiable Information (PII)
│  ├─ Names, addresses, phone numbers
│  ├─ Social Security Numbers
│  ├─ Credit card numbers
│  └─ Passport numbers
│
├─ Financial Information
│  ├─ Bank account numbers
│  └─ Tax IDs
│
├─ Healthcare Information (PHI)
│  └─ Medical record numbers
│
└─ Credentials
   ├─ AWS access keys
   ├─ Private keys
   └─ API keys
```

**Enable Macie:**
```bash
# Enable Macie
aws macie2 enable-macie

# Create discovery job
aws macie2 create-classification-job \
  --job-type ONE_TIME \
  --s3-job-definition '{
    "bucketDefinitions": [{
      "accountId": "123456789012",
      "buckets": ["my-data-bucket"]
    }]
  }' \
  --name "SensitiveDataScan" \
  --description "Scan for PII in production buckets"
```

### Week 4: AWS Config & Compliance Monitoring

#### AWS Config
- [ ] Enable AWS Config
- [ ] Configure recording of resources
- [ ] Create Config Rules
- [ ] Use conformance packs
- [ ] Aggregate data across accounts
- [ ] Remediate non-compliant resources

**AWS Config Architecture:**
```
┌────────────────────────────────────────────────┐
│         AWS CONFIG                             │
├────────────────────────────────────────────────┤
│                                                │
│  Configuration Recorder                        │
│  └─ Tracks resource configuration changes     │
│                                                │
│  Config Rules                                  │
│  ├─ AWS Managed Rules (200+)                  │
│  └─ Custom Rules (Lambda-based)               │
│                                                │
│  Conformance Packs                             │
│  └─ Collection of Config Rules                │
│                                                │
│  Remediation Actions                           │
│  └─ Automatic or manual fixes                 │
│                                                │
└────────────────────────────────────────────────┘
```

**Enable AWS Config:**
```bash
# Create S3 bucket for Config
aws s3api create-bucket \
  --bucket my-aws-config-bucket \
  --region us-east-1

# Create IAM role for Config
# (Role creation steps omitted for brevity)

# Enable Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/config-role \
  --recording-group allSupported=true,includeGlobalResourceTypes=true

aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=my-aws-config-bucket

# Start recording
aws configservice start-configuration-recorder \
  --configuration-recorder-name default
```

**Essential Config Rules:**
- [ ] **encrypted-volumes** - EBS volumes must be encrypted
- [ ] **s3-bucket-public-read-prohibited** - S3 buckets shouldn't allow public read
- [ ] **s3-bucket-public-write-prohibited** - S3 buckets shouldn't allow public write
- [ ] **iam-password-policy** - IAM password policy meets requirements
- [ ] **root-account-mfa-enabled** - Root account has MFA
- [ ] **cloudtrail-enabled** - CloudTrail is enabled
- [ ] **rds-encryption-enabled** - RDS instances are encrypted

```bash
# Deploy Config rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "encrypted-volumes",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "ENCRYPTED_VOLUMES"
    }
  }'
```

**Auto-Remediation Example:**
```bash
# Configure auto-remediation for public S3 buckets
aws configservice put-remediation-configurations \
  --remediation-configurations '[{
    "ConfigRuleName": "s3-bucket-public-read-prohibited",
    "TargetType": "SSM_DOCUMENT",
    "TargetIdentifier": "AWS-PublishSNSNotification",
    "TargetVersion": "1",
    "Parameters": {
      "AutomationAssumeRole": {
        "StaticValue": {
          "Values": ["arn:aws:iam::123456789012:role/ConfigRemediationRole"]
        }
      },
      "TopicArn": {
        "StaticValue": {
          "Values": ["arn:aws:sns:us-east-1:123456789012:ConfigAlerts"]
        }
      }
    },
    "Automatic": true
  }]'
```

## 🛠️ Hands-On Labs

### Lab 1: Complete Logging Setup
```bash
#!/bin/bash
# Set up comprehensive security logging

# Variables
ACCOUNT_ID="123456789012"
REGION="us-east-1"

# 1. Enable CloudTrail
echo "Setting up CloudTrail..."
aws cloudtrail create-trail \
  --name security-trail \
  --s3-bucket-name cloudtrail-logs-${ACCOUNT_ID} \
  --is-multi-region-trail \
  --enable-log-file-validation

aws cloudtrail start-logging --name security-trail

# 2. Enable GuardDuty
echo "Enabling GuardDuty..."
aws guardduty create-detector --enable

# 3. Enable Security Hub
echo "Enabling Security Hub..."
aws securityhub enable-security-hub

# 4. Enable Config
echo "Enabling AWS Config..."
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::${ACCOUNT_ID}:role/config-role

aws configservice start-configuration-recorder --configuration-recorder-name default

# 5. Enable Macie
echo "Enabling Macie..."
aws macie2 enable-macie

echo "✅ All security services enabled!"
```

### Lab 2: Security Monitoring Dashboard
```python
#!/usr/bin/env python3
"""
Create CloudWatch dashboard for security metrics
"""
import boto3
import json

def create_security_dashboard():
    cloudwatch = boto3.client('cloudwatch')
    
    dashboard_body = {
        "widgets": [
            {
                "type": "metric",
                "properties": {
                    "metrics": [
                        ["CloudTrailMetrics", "RootAccountUsageCount"],
                        [".", "UnauthorizedAPICalls"],
                        [".", "ConsoleSignInFailures"]
                    ],
                    "period": 300,
                    "stat": "Sum",
                    "region": "us-east-1",
                    "title": "Security Events"
                }
            },
            {
                "type": "metric",
                "properties": {
                    "metrics": [
                        ["AWS/GuardDuty", "FindingCount", {"stat": "Sum"}]
                    ],
                    "period": 300,
                    "region": "us-east-1",
                    "title": "GuardDuty Findings"
                }
            }
        ]
    }
    
    cloudwatch.put_dashboard(
        DashboardName='SecurityMonitoring',
        DashboardBody=json.dumps(dashboard_body)
    )
    
    print("✅ Security dashboard created!")

if __name__ == "__main__":
    create_security_dashboard()
```

## ✅ Phase 5 Checklist

Before moving to Phase 6, ensure you can:
- [ ] Enable and configure CloudTrail for comprehensive logging
- [ ] Create CloudWatch metric filters and alarms for security events
- [ ] Deploy GuardDuty and understand finding types
- [ ] Use Security Hub as a central security dashboard
- [ ] Enable Macie to discover sensitive data
- [ ] Configure AWS Config rules for compliance
- [ ] Investigate security incidents with Detective
- [ ] Set up automated responses to security findings

## 🎯 Success Criteria

You're ready for Phase 6 when you can:
1. Design a complete security monitoring strategy
2. Investigate a GuardDuty finding from start to finish
3. Create custom CloudWatch alarms for security events
4. Respond automatically to common security threats
5. Use Security Hub to track your security posture

> **💡 Pro Tip:** "You can't protect what you can't see." Comprehensive logging and monitoring are non-negotiable for cloud security.

## 🔜 What's Next?

Excellent work building your security monitoring foundation! Now let's ensure compliance.

**Next:** [Phase 6: Compliance & Vulnerability Management](phase-6-compliance-vulnerability.md) - Master Inspector, Config Rules, and compliance frameworks

---

**Remember:** "Detect fast, respond faster!" 🚨🔍
