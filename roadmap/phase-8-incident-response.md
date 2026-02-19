# 🚨 Phase 8: Incident Response

> **Prepare for and respond to security incidents effectively**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Security incidents are inevitable. The key is being prepared to detect, respond, and recover quickly. This phase covers incident response frameworks, AWS-specific IR procedures, automation, and forensics.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Understand the incident response lifecycle
- ✅ Implement AWS incident response procedures
- ✅ Automate responses with EventBridge and Lambda
- ✅ Perform forensic investigations in AWS
- ✅ Develop incident response playbooks
- ✅ Test your IR capabilities
- ✅ Learn from incidents through post-mortems

## 📚 Topics & Progress Tracker

### Week 1: Incident Response Fundamentals

#### The IR Lifecycle (NIST Framework)
- [ ] **1. Preparation** - Tools, training, processes
- [ ] **2. Detection & Analysis** - Identify and investigate
- [ ] **3. Containment** - Isolate and limit damage
- [ ] **4. Eradication** - Remove threat
- [ ] **5. Recovery** - Restore systems
- [ ] **6. Post-Incident Activity** - Lessons learned

**IR Lifecycle Diagram:**
```
┌──────────────────────────────────────────────────┐
│      INCIDENT RESPONSE LIFECYCLE                 │
├──────────────────────────────────────────────────┤
│                                                  │
│  1. PREPARATION                                  │
│     ├─ Incident response plan                   │
│     ├─ Security tools configured                │
│     ├─ Team trained                             │
│     └─ Runbooks prepared                        │
│                    ↓                             │
│  2. DETECTION & ANALYSIS                         │
│     ├─ GuardDuty finding                        │
│     ├─ Security Hub alert                       │
│     ├─ CloudWatch alarm                         │
│     └─ Investigation begins                     │
│                    ↓                             │
│  3. CONTAINMENT                                  │
│     ├─ Isolate affected resources               │
│     ├─ Revoke credentials                       │
│     ├─ Apply temporary fixes                    │
│     └─ Prevent spread                           │
│                    ↓                             │
│  4. ERADICATION                                  │
│     ├─ Remove malware                           │
│     ├─ Close vulnerabilities                    │
│     ├─ Update configurations                    │
│     └─ Verify threat removed                    │
│                    ↓                             │
│  5. RECOVERY                                     │
│     ├─ Restore from backups                     │
│     ├─ Rebuild systems                          │
│     ├─ Monitor for re-infection                 │
│     └─ Resume normal operations                 │
│                    ↓                             │
│  6. POST-INCIDENT ACTIVITY                       │
│     ├─ Document incident                        │
│     ├─ Conduct post-mortem                      │
│     ├─ Update procedures                        │
│     └─ Share lessons learned                    │
│                                                  │
└──────────────────────────────────────────────────┘
```

#### AWS Incident Response Tools
- [ ] Amazon Detective - Investigation
- [ ] AWS CloudTrail - Audit logs
- [ ] VPC Flow Logs - Network traffic
- [ ] AWS Config - Configuration history
- [ ] Amazon GuardDuty - Threat detection
- [ ] AWS Systems Manager - Remote management
- [ ] AWS Backup - Recovery

> **🔑 Key Takeaway:** The time to prepare for an incident is before it happens. Have your tools, processes, and team ready.

#### Incident Severity Levels
- [ ] **SEV1 - Critical**: Complete service outage, data breach
- [ ] **SEV2 - High**: Significant functionality impaired
- [ ] **SEV3 - Medium**: Minor functionality impaired
- [ ] **SEV4 - Low**: Informational, no impact

**Incident Classification:**
| Severity | Impact | Response Time | Examples |
|----------|--------|---------------|----------|
| **SEV1** | Critical | Immediate | Data breach, ransomware, complete outage |
| **SEV2** | High | < 1 hour | Compromised credentials, service degradation |
| **SEV3** | Medium | < 4 hours | Suspicious activity, minor vulnerabilities |
| **SEV4** | Low | < 24 hours | Policy violations, low-risk findings |

### Week 2: AWS-Specific IR Procedures

#### Compromised EC2 Instance Response
- [ ] Isolate the instance (security group)
- [ ] Create forensic snapshot
- [ ] Capture memory dump
- [ ] Terminate or quarantine
- [ ] Analyze logs and artifacts

**EC2 Compromise Response Script:**
```python
#!/usr/bin/env python3
"""
Automated response to compromised EC2 instance
"""
import boto3
import datetime

def quarantine_instance(instance_id, reason):
    """Isolate compromised instance"""
    ec2 = boto3.client('ec2')
    
    print(f"🚨 Quarantining instance: {instance_id}")
    print(f"   Reason: {reason}")
    
    # Get instance details
    instance = ec2.describe_instances(InstanceIds=[instance_id])
    vpc_id = instance['Reservations'][0]['Instances'][0]['VpcId']
    
    # Create forensic security group (deny all)
    timestamp = datetime.datetime.now().strftime('%Y%m%d-%H%M%S')
    sg_name = f'forensic-quarantine-{timestamp}'
    
    forensic_sg = ec2.create_security_group(
        GroupName=sg_name,
        Description=f'Quarantine SG for {instance_id}',
        VpcId=vpc_id
    )
    sg_id = forensic_sg['GroupId']
    
    # Tag security group
    ec2.create_tags(
        Resources=[sg_id],
        Tags=[
            {'Key': 'Purpose', 'Value': 'Forensics'},
            {'Key': 'InstanceId', 'Value': instance_id},
            {'Key': 'CreatedBy', 'Value': 'IRAutomation'}
        ]
    )
    
    # Remove all egress rules (default allows all)
    ec2.revoke_security_group_egress(
        GroupId=sg_id,
        IpPermissions=[{
            'IpProtocol': '-1',
            'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
        }]
    )
    
    # Apply forensic security group
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        Groups=[sg_id]
    )
    
    print(f"✅ Instance isolated with SG: {sg_id}")
    
    # Create snapshots of all volumes
    print("📸 Creating forensic snapshots...")
    volumes = ec2.describe_volumes(
        Filters=[{
            'Name': 'attachment.instance-id',
            'Values': [instance_id]
        }]
    )
    
    snapshot_ids = []
    for volume in volumes['Volumes']:
        volume_id = volume['VolumeId']
        snapshot = ec2.create_snapshot(
            VolumeId=volume_id,
            Description=f'Forensic snapshot for incident - {instance_id}',
            TagSpecifications=[{
                'ResourceType': 'snapshot',
                'Tags': [
                    {'Key': 'Purpose', 'Value': 'Forensics'},
                    {'Key': 'InstanceId', 'Value': instance_id},
                    {'Key': 'Timestamp', 'Value': timestamp}
                ]
            }]
        )
        snapshot_ids.append(snapshot['SnapshotId'])
        print(f"   Created snapshot: {snapshot['SnapshotId']}")
    
    # Add termination protection
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        DisableApiTermination={'Value': True}
    )
    
    # Send notification
    sns = boto3.client('sns')
    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:SecurityIncidents',
        Subject=f'🚨 EC2 Instance Quarantined: {instance_id}',
        Message=f"""
Instance {instance_id} has been quarantined.

Reason: {reason}
Forensic SG: {sg_id}
Snapshots: {', '.join(snapshot_ids)}
Timestamp: {timestamp}

Action required: Investigate and remediate.
"""
    )
    
    return {
        'instance_id': instance_id,
        'security_group': sg_id,
        'snapshots': snapshot_ids
    }

if __name__ == "__main__":
    # Example usage
    quarantine_instance(
        'i-1234567890abcdef0',
        'GuardDuty finding: Cryptocurrency mining'
    )
```

#### Compromised IAM Credentials Response
- [ ] Rotate access keys immediately
- [ ] Revoke active sessions
- [ ] Review CloudTrail for unauthorized activity
- [ ] Check for resource modifications
- [ ] Enable MFA if not already enabled

**IAM Credential Compromise Response:**
```python
#!/usr/bin/env python3
"""
Response to compromised IAM credentials
"""
import boto3
from datetime import datetime, timedelta

def respond_to_compromised_credentials(username):
    """Respond to compromised IAM user credentials"""
    iam = boto3.client('iam')
    cloudtrail = boto3.client('cloudtrail')
    
    print(f"🚨 Responding to compromised credentials for: {username}")
    
    # 1. Disable all access keys
    print("🔒 Disabling access keys...")
    access_keys = iam.list_access_keys(UserName=username)
    for key in access_keys['AccessKeyMetadata']:
        key_id = key['AccessKeyId']
        iam.update_access_key(
            UserName=username,
            AccessKeyId=key_id,
            Status='Inactive'
        )
        print(f"   Disabled key: {key_id}")
    
    # 2. Reset console password
    print("🔑 Resetting console password...")
    try:
        iam.update_login_profile(
            UserName=username,
            Password='TemporaryComplexPassword123!',
            PasswordResetRequired=True
        )
        print("   Console password reset")
    except iam.exceptions.NoSuchEntityException:
        print("   No console password to reset")
    
    # 3. Revoke all active sessions
    print("⏹️  Revoking active sessions...")
    try:
        iam.put_user_policy(
            UserName=username,
            PolicyName='RevokeOldSessions',
            PolicyDocument=f'''{{
                "Version": "2012-10-17",
                "Statement": [{{
                    "Effect": "Deny",
                    "Action": "*",
                    "Resource": "*",
                    "Condition": {{
                        "DateLessThan": {{
                            "aws:TokenIssueTime": "{datetime.utcnow().isoformat()}Z"
                        }}
                    }}
                }}]
            }}'''
        )
        print("   Active sessions revoked")
    except Exception as e:
        print(f"   Error revoking sessions: {e}")
    
    # 4. Review recent activity
    print("🔍 Analyzing recent activity...")
    end_time = datetime.now()
    start_time = end_time - timedelta(days=7)
    
    events = cloudtrail.lookup_events(
        LookupAttributes=[{
            'AttributeKey': 'Username',
            'AttributeValue': username
        }],
        StartTime=start_time,
        EndTime=end_time,
        MaxResults=50
    )
    
    suspicious_events = []
    for event in events['Events']:
        event_name = event['EventName']
        # Flag potentially malicious actions
        if any(action in event_name for action in [
            'Create', 'Delete', 'Modify', 'Put', 'Update'
        ]):
            suspicious_events.append({
                'time': event['EventTime'],
                'event': event_name,
                'resources': event.get('Resources', [])
            })
    
    print(f"   Found {len(suspicious_events)} potentially suspicious events")
    
    # 5. Send detailed report
    sns = boto3.client('sns')
    message = f"""
IAM Credential Compromise Response for: {username}

Actions Taken:
- All access keys disabled
- Console password reset
- Active sessions revoked

Recent Activity:
- Total events: {len(events['Events'])}
- Suspicious events: {len(suspicious_events)}

Review CloudTrail for detailed activity.
"""
    
    sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:SecurityIncidents',
        Subject=f'🚨 IAM Credentials Compromised: {username}',
        Message=message
    )
    
    print("✅ Response complete. Review CloudTrail for detailed investigation.")

if __name__ == "__main__":
    respond_to_compromised_credentials('compromised-user')
```

#### Data Exfiltration Response
- [ ] Identify affected S3 buckets
- [ ] Enable S3 access logging if not enabled
- [ ] Review CloudTrail for suspicious GetObject calls
- [ ] Block unauthorized IP addresses
- [ ] Rotate KMS keys if data was encrypted

### Week 3: Automated Incident Response

#### EventBridge + Lambda Automation
- [ ] Trigger Lambda from GuardDuty findings
- [ ] Trigger Lambda from Security Hub
- [ ] Trigger Lambda from CloudWatch alarms
- [ ] Implement auto-remediation workflows

**EventBridge Rule for GuardDuty:**
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [7, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8, 7.9, 8, 8.0, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9]
  }
}
```

**Lambda IR Orchestrator:**
```python
#!/usr/bin/env python3
"""
Incident response orchestrator
"""
import boto3
import json

def lambda_handler(event, context):
    """
    Route findings to appropriate response functions
    """
    source = event.get('source')
    detail = event.get('detail', {})
    
    if source == 'aws.guardduty':
        handle_guardduty_finding(detail)
    elif source == 'aws.securityhub':
        handle_securityhub_finding(detail)
    elif source == 'aws.config':
        handle_config_change(detail)
    
    return {'statusCode': 200}

def handle_guardduty_finding(finding):
    """Handle GuardDuty finding"""
    finding_type = finding.get('type', '')
    severity = finding.get('severity', 0)
    
    print(f"GuardDuty Finding: {finding_type} (Severity: {severity})")
    
    # Route to specific handlers
    if 'UnauthorizedAccess:EC2' in finding_type:
        handle_ec2_unauthorized_access(finding)
    elif 'UnauthorizedAccess:IAMUser' in finding_type:
        handle_iam_unauthorized_access(finding)
    elif 'CryptoCurrency' in finding_type:
        handle_crypto_mining(finding)
    elif 'Trojan' in finding_type or 'Backdoor' in finding_type:
        handle_malware(finding)

def handle_ec2_unauthorized_access(finding):
    """Handle unauthorized EC2 access"""
    instance_id = finding['resource']['instanceDetails']['instanceId']
    
    # Quarantine instance
    quarantine_instance(instance_id, f"GuardDuty: {finding['type']}")

def handle_iam_unauthorized_access(finding):
    """Handle unauthorized IAM access"""
    principal = finding['resource']['accessKeyDetails']['principalId']
    
    if ':' in principal:
        username = principal.split(':')[1]
        respond_to_compromised_credentials(username)

def handle_crypto_mining(finding):
    """Handle cryptocurrency mining"""
    instance_id = finding['resource']['instanceDetails']['instanceId']
    
    # Immediate quarantine for crypto mining
    quarantine_instance(instance_id, "Cryptocurrency mining detected")

def handle_malware(finding):
    """Handle malware detection"""
    instance_id = finding['resource']['instanceDetails']['instanceId']
    
    # Quarantine and prepare for forensics
    quarantine_instance(instance_id, f"Malware detected: {finding['type']}")
```

#### Step Functions for Complex Workflows
- [ ] Design multi-step IR workflows
- [ ] Approval steps for destructive actions
- [ ] Parallel execution of tasks
- [ ] Error handling and rollback

**IR State Machine Example:**
```json
{
  "Comment": "Incident Response State Machine",
  "StartAt": "DetectIncident",
  "States": {
    "DetectIncident": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:DetectIncident",
      "Next": "ClassifyIncident"
    },
    "ClassifyIncident": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ClassifyIncident",
      "Next": "IsHighSeverity"
    },
    "IsHighSeverity": {
      "Type": "Choice",
      "Choices": [{
        "Variable": "$.severity",
        "NumericGreaterThanEquals": 7,
        "Next": "ImmediateContainment"
      }],
      "Default": "StandardResponse"
    },
    "ImmediateContainment": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "IsolateResource",
          "States": {
            "IsolateResource": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:us-east-1:123456789012:function:IsolateResource",
              "End": true
            }
          }
        },
        {
          "StartAt": "NotifyTeam",
          "States": {
            "NotifyTeam": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:us-east-1:123456789012:function:NotifyTeam",
              "End": true
            }
          }
        }
      ],
      "Next": "CreateForensicSnapshot"
    },
    "CreateForensicSnapshot": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:CreateSnapshot",
      "End": true
    },
    "StandardResponse": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:StandardResponse",
      "End": true
    }
  }
}
```

### Week 4: Forensics & Post-Incident Activities

#### AWS Forensics Best Practices
- [ ] Never investigate on production systems
- [ ] Create forensic copies (snapshots)
- [ ] Preserve evidence chain of custody
- [ ] Document everything
- [ ] Use forensic workstation in isolated VPC

**Forensic Analysis Workflow:**
```
1. Create snapshots of affected volumes
   ↓
2. Create forensic VPC (isolated, no internet)
   ↓
3. Launch forensic workstation in isolated VPC
   ↓
4. Attach snapshot as volume to forensic workstation
   ↓
5. Perform analysis (logs, memory, disk)
   ↓
6. Document findings
   ↓
7. Preserve evidence for legal/compliance
```

#### Memory and Disk Forensics
- [ ] Capture memory dumps (Linux: LiME, Windows: winpmem)
- [ ] Analyze with volatility framework
- [ ] Extract artifacts (process list, network connections)
- [ ] Timeline analysis

**Memory Dump Script:**
```bash
#!/bin/bash
# Capture memory dump on compromised Linux instance

# Install LiME (Linux Memory Extractor)
cd /tmp
git clone https://github.com/504ensicsLabs/LiME
cd LiME/src
make

# Capture memory
insmod lime-$(uname -r).ko "path=/tmp/memory.lime format=lime"

# Upload to S3 for analysis
aws s3 cp /tmp/memory.lime s3://forensics-bucket/incidents/$(date +%Y%m%d)/memory.lime

echo "Memory dump captured and uploaded"
```

#### Post-Incident Review
- [ ] Conduct blameless post-mortem
- [ ] Document timeline of events
- [ ] Identify root cause
- [ ] Determine what worked/didn't work
- [ ] Create action items for improvement

**Post-Mortem Template:**
```markdown
# Incident Post-Mortem

## Incident Summary
- **Date:** 2024-01-15
- **Duration:** 2 hours 30 minutes
- **Severity:** SEV2
- **Incident Commander:** John Doe

## Impact
- 50 EC2 instances affected
- No data loss
- Service degraded for 30 minutes

## Timeline
- 10:00 AM: GuardDuty alert received
- 10:05 AM: Incident declared
- 10:10 AM: Affected instances identified
- 10:15 AM: Instances quarantined
- 10:30 AM: Root cause identified (compromised credentials)
- 11:00 AM: Credentials rotated
- 12:30 PM: All services restored

## Root Cause
Access keys for service account were leaked in public GitHub repository.

## What Went Well
- GuardDuty detected anomaly within 5 minutes
- Automated quarantine worked as expected
- Team responded quickly

## What Didn't Go Well
- No secrets scanning on GitHub commits
- Service account had overly broad permissions
- Runbook was outdated

## Action Items
1. [ ] Implement git-secrets for all repos (Owner: Security Team, Due: Feb 1)
2. [ ] Review and scope down service account permissions (Owner: DevOps, Due: Jan 20)
3. [ ] Update IR runbooks quarterly (Owner: Security Team, Due: Jan 30)
4. [ ] Conduct IR tabletop exercise (Owner: Security Team, Due: Feb 15)
```

#### Incident Response Playbooks
- [ ] Create playbooks for common scenarios
- [ ] EC2 compromise playbook
- [ ] Data exfiltration playbook
- [ ] DDoS response playbook
- [ ] Ransomware response playbook

## 🛠️ Hands-On Labs

### Lab 1: Simulate and Respond to GuardDuty Finding
```bash
#!/bin/bash
# Simulate GuardDuty finding and test automated response

# Generate test finding (requires actual instance and activity)
# For testing, create EventBridge test event:

cat > test-guardduty-event.json << 'EOF'
{
  "version": "0",
  "id": "test-event-12345",
  "detail-type": "GuardDuty Finding",
  "source": "aws.guardduty",
  "time": "2024-01-15T10:00:00Z",
  "region": "us-east-1",
  "detail": {
    "schemaVersion": "2.0",
    "accountId": "123456789012",
    "region": "us-east-1",
    "partition": "aws",
    "id": "test-finding",
    "arn": "arn:aws:guardduty:us-east-1:123456789012:detector/test/finding/test",
    "type": "CryptoCurrency:EC2/BitcoinTool.B!DNS",
    "severity": 8.0,
    "resource": {
      "resourceType": "Instance",
      "instanceDetails": {
        "instanceId": "i-1234567890abcdef0"
      }
    }
  }
}
EOF

# Test event with Lambda
aws lambda invoke \
  --function-name IROrchestrator \
  --payload file://test-guardduty-event.json \
  response.json

cat response.json
```

### Lab 2: IR Tabletop Exercise
```markdown
# Scenario: Data Exfiltration

## Initial Alert
"GuardDuty has detected unusual S3 API activity. Large volumes of data 
being downloaded from production S3 bucket to unknown IP address."

## Your Tasks:
1. What is your first action?
2. How do you identify the scope?
3. What containment actions do you take?
4. How do you investigate the root cause?
5. What recovery steps are needed?
6. How do you prevent recurrence?

## Discussion Points:
- Who needs to be notified?
- What compliance requirements apply?
- Do we need to notify customers?
- What forensic data do we need to preserve?
```

## ✅ Phase 8 Checklist

Before moving to Phase 9, ensure you can:
- [ ] Explain the incident response lifecycle
- [ ] Respond to compromised EC2 instances
- [ ] Respond to compromised IAM credentials
- [ ] Automate incident response with Lambda
- [ ] Perform basic AWS forensics
- [ ] Conduct post-incident reviews
- [ ] Create incident response playbooks
- [ ] Run IR tabletop exercises

## 🎯 Success Criteria

You're ready for Phase 9 when you can:
1. Respond to a security incident from detection to resolution
2. Automate common incident response actions
3. Perform forensic analysis on AWS resources
4. Lead a post-incident review
5. Create and maintain IR playbooks

> **💡 Pro Tip:** Practice makes perfect. Run IR tabletop exercises quarterly to keep your team sharp.

## 🔜 What's Next?

Excellent incident response skills! Now let's secure your infrastructure code.

**Next:** [Phase 9: Infrastructure as Code Security](phase-9-iac-security.md) - Secure your Terraform and CloudFormation

---

**Remember:** "Hope is not a strategy. Prepare, practice, and be ready!" 🚨🛡️
