# Phase 4: Monitoring, Logging & Threat Detection

## Overview
Effective security monitoring and logging are essential for detecting threats, investigating incidents, and maintaining compliance. This phase covers AWS security monitoring services and log analysis techniques.

## Learning Objectives
By the end of this phase, you will be able to:
- Configure CloudTrail for comprehensive API logging
- Use CloudWatch for metrics, logs, and alarms
- Enable and interpret GuardDuty findings
- Centralize security findings with Security Hub
- Investigate threats with Amazon Detective
- Discover sensitive data with Amazon Macie
- Analyze VPC Flow Logs
- Implement automated responses with EventBridge

---

## Core Services Overview

| Service | Purpose | Key Features |
|---------|---------|--------------|
| **CloudTrail** | API activity logging | Who did what, when, compliance audits |
| **CloudWatch** | Monitoring & alerting | Metrics, logs, dashboards, alarms |
| **AWS Config** | Resource configuration tracking | Compliance, change history, remediation |
| **GuardDuty** | Intelligent threat detection | Machine learning, anomaly detection |
| **Security Hub** | Centralized security posture | Aggregate findings, compliance standards |
| **Detective** | Security investigation | Visual analysis, root cause investigation |
| **Macie** | Sensitive data discovery | PII/PHI discovery in S3 |

---

## AWS CloudTrail

### CloudTrail Event Types

1. **Management Events**: Control plane operations (default)
   - Creating EC2 instances
   - Modifying security groups
   - Creating IAM users

2. **Data Events**: Data plane operations (opt-in)
   - S3 object-level operations (GetObject, PutObject)
   - Lambda function executions

3. **Insights Events**: Unusual API activity (opt-in)
   - Detected anomalies in API usage

### Enable Organization-Wide CloudTrail

```bash
# Create S3 bucket for logs
aws s3api create-bucket \
  --bucket cloudtrail-logs-123456789012 \
  --region us-east-1

# Apply bucket policy
cat > bucket-policy.json << 'POLICY'
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
      "Resource": "arn:aws:s3:::cloudtrail-logs-123456789012"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::cloudtrail-logs-123456789012/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
POLICY

aws s3api put-bucket-policy \
  --bucket cloudtrail-logs-123456789012 \
  --policy file://bucket-policy.json

# Create trail
aws cloudtrail create-trail \
  --name organization-trail \
  --s3-bucket-name cloudtrail-logs-123456789012 \
  --is-multi-region-trail \
  --is-organization-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging \
  --name organization-trail
```

### Query CloudTrail Logs

```bash
# Recent events
aws cloudtrail lookup-events \
  --max-results 10

# Filter by user
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=alice

# Filter by event name
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=DeleteBucket
```

---

## Amazon GuardDuty

### GuardDuty Finding Types

**EC2 Findings:**
- `Backdoor:EC2/C&CActivity.B`: Instance communicating with C&C server
- `CryptoCurrency:EC2/BitcoinTool.B`: Bitcoin mining detected
- `Trojan:EC2/DNSDataExfiltration`: DNS exfiltration detected
- `UnauthorizedAccess:EC2/SSHBruteForce`: SSH brute force attack

**IAM Findings:**
- `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`: Credentials used outside AWS
- `Stealth:IAMUser/PasswordPolicyChange`: Password policy weakened
- `PenTest:IAMUser/KaliLinux`: API calls from Kali Linux

**S3 Findings:**
- `Policy:S3/BucketAnonymousAccessGranted`: Bucket made public
- `Exfiltration:S3/ObjectRead.Unusual`: Unusual S3 object read

### Enable GuardDuty

```bash
# Enable GuardDuty
aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES

# List findings
DETECTOR_ID=$(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)

aws guardduty list-findings \
  --detector-id $DETECTOR_ID \
  --max-results 50

# Get finding details
aws guardduty get-findings \
  --detector-id $DETECTOR_ID \
  --finding-ids <finding-id>
```

### Generate Test Findings

```bash
# Generate sample findings for testing
aws guardduty create-sample-findings \
  --detector-id $DETECTOR_ID \
  --finding-types "UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration" \
    "Backdoor:EC2/C&CActivity.B"
```

---

## AWS Security Hub

### Enable Security Hub

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Enable security standards
aws securityhub batch-enable-standards \
  --standards-subscription-requests '[
    {"StandardsArn": "arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0"},
    {"StandardsArn": "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.2.0"}
  ]'

# Get findings
aws securityhub get-findings \
  --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}'
```

### Security Standards

1. **AWS Foundational Security Best Practices**
   - 50+ automated checks
   - S3, IAM, EC2, RDS, etc.

2. **CIS AWS Foundations Benchmark**
   - Industry best practices
   - Compliance-focused

3. **PCI-DSS**
   - Payment Card Industry standards
   - For payment processing workloads

---

## Amazon CloudWatch

### CloudWatch Logs Insights Queries

**Find failed login attempts:**
```
fields @timestamp, userIdentity.principalId, errorCode, errorMessage
| filter eventName == "ConsoleLogin" and errorMessage exists
| sort @timestamp desc
| limit 20
```

**Find S3 bucket deletions:**
```
fields @timestamp, userIdentity.principalId, requestParameters.bucketName
| filter eventName == "DeleteBucket"
| sort @timestamp desc
```

**Find IAM policy changes:**
```
fields @timestamp, userIdentity.principalId, eventName, requestParameters
| filter eventName like /Policy/
| sort @timestamp desc
```

### CloudWatch Alarms for Security

```bash
# Alarm for root account usage
aws cloudwatch put-metric-alarm \
  --alarm-name root-account-usage \
  --alarm-description "Alert on root account usage" \
  --metric-name RootAccountUsage \
  --namespace AWS/CloudTrail \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:security-alerts
```

---

## Automated Response with EventBridge

### Pattern: Auto-Isolate Compromised EC2 Instance

**EventBridge Rule:**
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "type": ["Backdoor:EC2/C&CActivity.B"]
  }
}
```

**Lambda Function (Python):**
```python
import boto3

ec2 = boto3.client('ec2')
sns = boto3.client('sns')

def lambda_handler(event, context):
    finding = event['detail']
    instance_id = finding['resource']['instanceDetails']['instanceId']
    
    # Get current security groups
    response = ec2.describe_instances(InstanceIds=[instance_id])
    vpc_id = response['Reservations'][0]['Instances'][0]['VpcId']
    
    # Create isolation security group (deny all)
    sg_response = ec2.create_security_group(
        GroupName=f'isolation-{instance_id}',
        Description='Isolation security group for compromised instance',
        VpcId=vpc_id
    )
    
    isolation_sg_id = sg_response['GroupId']
    
    # Apply isolation security group
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        Groups=[isolation_sg_id]
    )
    
    # Send notification
    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:security-alerts',
        Subject=f'EC2 Instance {instance_id} Isolated',
        Message=f'Instance {instance_id} has been isolated due to GuardDuty finding: {finding["type"]}'
    )
    
    return {
        'statusCode': 200,
        'body': f'Instance {instance_id} isolated successfully'
    }
```

---

## Best Practices

### CloudTrail
- [ ] Enable in all regions
- [ ] Enable log file validation
- [ ] Centralize logs in dedicated security account
- [ ] Enable data events for sensitive S3 buckets
- [ ] Set up CloudWatch Logs integration
- [ ] Encrypt logs with KMS

### GuardDuty
- [ ] Enable in all accounts and regions
- [ ] Configure centralized findings in Security Hub
- [ ] Automate response with EventBridge + Lambda
- [ ] Review findings daily
- [ ] Tune findings (suppress false positives)

### Security Monitoring
- [ ] Set up CloudWatch dashboards for security metrics
- [ ] Create alarms for critical security events
- [ ] Regular log analysis (weekly/monthly)
- [ ] Automate common response actions
- [ ] Document incident response procedures

---

[← Previous: Data Security](./phase-3-data-security.md) | [Back to Roadmap](../ROADMAP.md) | [Next: Compliance →](./phase-5-compliance.md)
