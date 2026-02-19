# Phase 9: Incident Response

## Overview
Effective incident response minimizes damage and recovery time from security incidents.

## Learning Objectives
- Understand the IR lifecycle
- Create incident response playbooks
- Automate response with EventBridge + Lambda
- Conduct forensic investigations
- Implement post-incident reviews

---

## Incident Response Lifecycle

```
1. PREPARE → 2. DETECT → 3. CONTAIN → 4. ERADICATE → 5. RECOVER → 6. LESSONS LEARNED
```

---

## Common Incident Scenarios

### Scenario 1: Compromised IAM Credentials

**Detection**: GuardDuty finding `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`

**Response:**
1. Disable IAM user/access key
2. Review CloudTrail for actions taken
3. Revoke active sessions
4. Rotate credentials
5. Investigate root cause

```bash
# Disable access key
aws iam update-access-key \
  --user-name compromised-user \
  --access-key-id AKIAIOSFODNN7EXAMPLE \
  --status Inactive

# Delete access key
aws iam delete-access-key \
  --user-name compromised-user \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

### Scenario 2: Malicious EC2 Instance

**Detection**: GuardDuty finding `Backdoor:EC2/C&CActivity.B`

**Response:**
1. Create EBS snapshot for forensics
2. Isolate instance with security group
3. Tag instance with incident ID
4. Investigate with Detective
5. Terminate or rebuild instance

```bash
# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxxxx \
  --description "Forensics snapshot - Incident-2024-001"

# Isolate instance
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxx \
  --groups sg-isolation-xxxxx

# Tag instance
aws ec2 create-tags \
  --resources i-xxxxx \
  --tags Key=Incident,Value=INC-2024-001 Key=Status,Value=Quarantined
```

---

## Automated Response

### EventBridge Rule for Auto-Response

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [7, 8, 9]
  }
}
```

### Lambda Response Function

```python
import boto3

def lambda_handler(event, context):
    finding = event['detail']
    finding_type = finding['type']
    severity = finding['severity']
    
    # Route to appropriate response function
    if 'EC2' in finding_type:
        handle_ec2_incident(finding)
    elif 'IAM' in finding_type:
        handle_iam_incident(finding)
    elif 'S3' in finding_type:
        handle_s3_incident(finding)
    
    # Create incident ticket
    create_incident_ticket(finding)
    
    # Send notification
    send_notification(finding)
```

---

## Forensics Best Practices

### Preserve Evidence
- [ ] Create EBS snapshots before termination
- [ ] Export CloudTrail logs to S3
- [ ] Capture memory dumps if needed
- [ ] Document all actions (chain of custody)

### Analyze in Isolation
- [ ] Use dedicated forensics VPC
- [ ] No internet access
- [ ] Use forensics tools (Volatility, Sleuth Kit)

---

## Incident Response Checklist

### Preparation Phase
- [ ] IR team and roles defined
- [ ] Playbooks created
- [ ] Security tools deployed
- [ ] Communication channels established

### Detection Phase
- [ ] Monitoring enabled
- [ ] Alerts configured
- [ ] Triage process defined

### Containment Phase
- [ ] Isolation procedures documented
- [ ] Evidence preservation processes
- [ ] Escalation paths defined

### Recovery Phase
- [ ] Rebuild procedures
- [ ] Monitoring for reinfection
- [ ] Return to normal operations

### Lessons Learned
- [ ] Post-incident review conducted
- [ ] Playbooks updated
- [ ] Improvements implemented

---

## Best Practices

- [ ] Document all incident response procedures
- [ ] Practice with tabletop exercises
- [ ] Automate common responses
- [ ] Maintain evidence chain of custody
- [ ] Conduct post-incident reviews
- [ ] Update playbooks regularly

---

[← Previous: Multi-Account](./phase-8-multi-account.md) | [Back to Roadmap](../ROADMAP.md) | [Next: IaC Security →](./phase-10-iac-security.md)
