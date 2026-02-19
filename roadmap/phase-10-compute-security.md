# 💻 Phase 10: Compute Security

> **Secure EC2, Lambda, and container workloads**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Compute resources are where your applications run, making them prime targets for attackers. This phase covers security best practices for EC2 instances, serverless Lambda functions, and containerized workloads on ECS and EKS.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Harden EC2 instances with IMDSv2 and SSM
- ✅ Secure Lambda functions
- ✅ Implement container security for ECS/EKS
- ✅ Use ECR image scanning
- ✅ Configure IRSA (IAM Roles for Service Accounts)
- ✅ Manage patches with Systems Manager
- ✅ Build secure golden AMIs

## 📚 Topics & Progress Tracker

### Week 1: EC2 Security

#### EC2 Security Best Practices
- [ ] Use latest generation instance types
- [ ] Enable IMDSv2 (Instance Metadata Service v2)
- [ ] Use SSM Session Manager instead of SSH
- [ ] Implement proper IAM instance profiles
- [ ] Use golden AMIs
- [ ] Enable detailed monitoring

**EC2 Security Layers:**
```
┌─────────────────────────────────────────────────┐
│         EC2 SECURITY LAYERS                     │
├─────────────────────────────────────────────────┤
│                                                 │
│ 1. Network Security                             │
│    ├─ Security Groups (stateful)                │
│    ├─ NACLs (stateless)                         │
│    └─ Private subnets for sensitive workloads   │
│                                                 │
│ 2. Instance Security                            │
│    ├─ IMDSv2 enabled                            │
│    ├─ No public IP (use bastion/SSM)            │
│    ├─ Instance profile with minimal permissions │
│    └─ Encrypted EBS volumes                     │
│                                                 │
│ 3. OS Security                                  │
│    ├─ Hardened OS (CIS benchmark)               │
│    ├─ Automated patching                        │
│    ├─ No root login                             │
│    └─ Host-based firewall (iptables/firewalld)  │
│                                                 │
│ 4. Application Security                         │
│    ├─ Run as non-root user                      │
│    ├─ Secrets from Secrets Manager              │
│    └─ Application-level encryption              │
│                                                 │
└─────────────────────────────────────────────────┘
```

#### IMDSv2 (Instance Metadata Service v2)
- [ ] Understand IMDSv1 vs IMDSv2 security differences
- [ ] Enforce IMDSv2 using IAM policies
- [ ] Migrate existing instances to IMDSv2
- [ ] Test applications with IMDSv2

**Enable IMDSv2:**
```bash
# Launch instance with IMDSv2 required
aws ec2 run-instances \
  --image-id ami-xxxxx \
  --instance-type t3.micro \
  --metadata-options \
    HttpTokens=required,\
    HttpPutResponseHopLimit=1,\
    HttpEndpoint=enabled

# Modify existing instance to require IMDSv2
aws ec2 modify-instance-metadata-options \
  --instance-id i-xxxxx \
  --http-tokens required \
  --http-endpoint enabled
```

> **🔑 Key Takeaway:** IMDSv2 protects against SSRF attacks. Always use it for new instances.

**Test IMDSv2:**
```bash
# Get token (required for IMDSv2)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Use token to access metadata
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```

#### SSM Session Manager
- [ ] Configure SSM Session Manager
- [ ] Replace SSH with SSM sessions
- [ ] Enable session logging to S3/CloudWatch
- [ ] Configure session preferences
- [ ] Audit session activity

**SSM Session Manager Setup:**
```bash
# Create IAM role for SSM
cat > ssm-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "ec2.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
EOF

aws iam create-role \
  --role-name EC2-SSM-Role \
  --assume-role-policy-document file://ssm-trust-policy.json

# Attach SSM managed policy
aws iam attach-role-policy \
  --role-name EC2-SSM-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-SSM-Profile

aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-SSM-Profile \
  --role-name EC2-SSM-Role

# Connect to instance via SSM
aws ssm start-session --target i-xxxxx
```

**Configure Session Logging:**
```bash
# Create S3 bucket for session logs
aws s3api create-bucket \
  --bucket ssm-session-logs-123456789012 \
  --region us-east-1

# Configure session preferences
cat > session-preferences.json << 'EOF'
{
  "inputs": {
    "s3BucketName": "ssm-session-logs-123456789012",
    "s3KeyPrefix": "session-logs/",
    "s3EncryptionEnabled": true,
    "cloudWatchLogGroupName": "/aws/ssm/sessions",
    "cloudWatchEncryptionEnabled": true,
    "kmsKeyId": "alias/ssm-key",
    "runAsEnabled": true,
    "runAsDefaultUser": "ssm-user",
    "idleSessionTimeout": "20"
  }
}
EOF

aws ssm create-document \
  --name "SSM-SessionManagerRunShell" \
  --document-type "Session" \
  --content file://session-preferences.json
```

#### Golden AMIs
- [ ] Build hardened base images
- [ ] Implement CIS benchmarks
- [ ] Automate AMI creation with Packer
- [ ] Version and tag AMIs
- [ ] Regular AMI updates

**Packer Template for Golden AMI:**
```json
{
  "variables": {
    "aws_region": "us-east-1",
    "ami_name": "golden-ami-{{timestamp}}",
    "instance_type": "t3.micro"
  },
  "builders": [{
    "type": "amazon-ebs",
    "region": "{{user `aws_region`}}",
    "source_ami_filter": {
      "filters": {
        "virtualization-type": "hvm",
        "name": "amzn2-ami-hvm-*-x86_64-gp2",
        "root-device-type": "ebs"
      },
      "owners": ["amazon"],
      "most_recent": true
    },
    "instance_type": "{{user `instance_type`}}",
    "ssh_username": "ec2-user",
    "ami_name": "{{user `ami_name`}}",
    "encrypt_boot": true,
    "kms_key_id": "alias/ami-encryption-key",
    "tags": {
      "Name": "Golden AMI",
      "Environment": "Production",
      "Type": "Hardened",
      "CIS_Benchmark": "Level_2"
    }
  }],
  "provisioners": [
    {
      "type": "shell",
      "script": "scripts/hardening.sh"
    },
    {
      "type": "shell",
      "inline": [
        "sudo yum update -y",
        "sudo yum install -y amazon-ssm-agent",
        "sudo systemctl enable amazon-ssm-agent"
      ]
    }
  ]
}
```

**Hardening Script:**
```bash
#!/bin/bash
# hardening.sh - CIS benchmark hardening

# Disable root login
sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config

# Disable password authentication (use keys only)
sudo sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config

# Set strong password policy
sudo sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 90/' /etc/login.defs
sudo sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS 7/' /etc/login.defs
sudo sed -i 's/^PASS_MIN_LEN.*/PASS_MIN_LEN 14/' /etc/login.defs

# Enable firewall
sudo systemctl enable firewalld
sudo systemctl start firewalld

# Configure automatic security updates
sudo yum install -y yum-cron
sudo systemctl enable yum-cron
sudo systemctl start yum-cron

# Remove unnecessary packages
sudo yum remove -y \
  telnet \
  rsh \
  rsh-server \
  telnet-server

# Set file permissions
sudo chmod 600 /etc/ssh/sshd_config
sudo chmod 644 /etc/passwd
sudo chmod 000 /etc/shadow

echo "Hardening complete!"
```

#### AWS Systems Manager Patch Manager
- [ ] Create patch baselines
- [ ] Configure maintenance windows
- [ ] Automated patching schedules
- [ ] Patch compliance reporting

```bash
# Create patch baseline
aws ssm create-patch-baseline \
  --name "CriticalAndSecurityPatches" \
  --operating-system AMAZON_LINUX_2 \
  --approval-rules '{"PatchRules":[{"PatchFilterGroup":{"PatchFilters":[{"Key":"CLASSIFICATION","Values":["Security","Critical"]}]},"ApprovalAfterDays":7,"ComplianceLevel":"CRITICAL"}]}'

# Create maintenance window
aws ssm create-maintenance-window \
  --name "WeeklyPatching" \
  --schedule "cron(0 2 ? * SUN *)" \
  --duration 4 \
  --cutoff 1 \
  --allow-unassociated-targets

# Register targets
aws ssm register-target-with-maintenance-window \
  --window-id mw-xxxxx \
  --target-type "INSTANCE" \
  --targets "Key=tag:PatchGroup,Values=Production" \
  --resource-type INSTANCE
```

### Week 2: Lambda Security

#### Lambda Security Best Practices
- [ ] Principle of least privilege for IAM roles
- [ ] Use VPC for sensitive operations
- [ ] Enable X-Ray tracing
- [ ] Encrypt environment variables
- [ ] Use layers for common dependencies
- [ ] Implement function concurrency limits

**Secure Lambda Function (Python):**
```python
import json
import boto3
import os
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch AWS SDK for X-Ray tracing
patch_all()

# Initialize AWS clients outside handler (connection reuse)
secrets_client = boto3.client('secretsmanager')
s3_client = boto3.client('s3')

def get_secret(secret_name):
    """Retrieve secret from Secrets Manager"""
    try:
        response = secrets_client.get_secret_value(SecretId=secret_name)
        return json.loads(response['SecretString'])
    except Exception as e:
        print(f"Error retrieving secret: {e}")
        raise

@xray_recorder.capture('lambda_handler')
def lambda_handler(event, context):
    """
    Secure Lambda function with best practices
    """
    
    # Get configuration from environment
    bucket_name = os.environ['BUCKET_NAME']
    secret_name = os.environ['SECRET_NAME']
    
    # Input validation
    if 'key' not in event:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': 'Missing required parameter: key'})
        }
    
    key = event['key']
    
    # Sanitize input (prevent injection attacks)
    if not key.replace('/', '').replace('-', '').replace('_', '').isalnum():
        return {
            'statusCode': 400,
            'body': json.dumps({'error': 'Invalid key format'})
        }
    
    try:
        # Get credentials from Secrets Manager
        credentials = get_secret(secret_name)
        
        # Perform secure operation
        response = s3_client.get_object(
            Bucket=bucket_name,
            Key=key,
            ServerSideEncryption='aws:kms'
        )
        
        # Process data...
        
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Success'})
        }
        
    except Exception as e:
        print(f"Error processing request: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Internal server error'})
        }
```

**Lambda IAM Role (Minimal Permissions):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-specific-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:my-secret-*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt"
      ],
      "Resource": "arn:aws:kms:us-east-1:123456789012:key/xxxxx"
    }
  ]
}
```

#### Lambda in VPC
- [ ] When to use VPC with Lambda
- [ ] VPC endpoint configuration
- [ ] Security group and subnet selection
- [ ] NAT Gateway for internet access

```bash
# Deploy Lambda in VPC
aws lambda create-function \
  --function-name SecureLambda \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/LambdaRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --vpc-config \
    SubnetIds=subnet-xxxxx,subnet-yyyyy,\
    SecurityGroupIds=sg-xxxxx \
  --environment Variables='{
    "BUCKET_NAME":"my-bucket",
    "SECRET_NAME":"my-secret"
  }' \
  --kms-key-arn arn:aws:kms:us-east-1:123456789012:key/xxxxx \
  --tracing-config Mode=Active
```

### Week 3: Container Security (ECS/EKS)

#### Container Security Fundamentals
- [ ] Use official base images
- [ ] Scan images for vulnerabilities
- [ ] Run containers as non-root
- [ ] Use read-only root filesystems
- [ ] Limit container capabilities
- [ ] Network segmentation

**Secure Dockerfile:**
```dockerfile
# Use specific version, not 'latest'
FROM python:3.11-slim-bullseye AS builder

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Multi-stage build for smaller final image
FROM python:3.11-slim-bullseye

# Create non-root user in final image
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set working directory
WORKDIR /app

# Copy from builder
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /app /app

# Change ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Read-only root filesystem (write to /tmp only)
VOLUME ["/tmp"]

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import requests; requests.get('http://localhost:8080/health')" || exit 1

# Run application
CMD ["python", "app.py"]
```

#### ECR Image Scanning
- [ ] Enable automatic scanning on push
- [ ] Configure basic and enhanced scanning
- [ ] Review scan findings
- [ ] Implement image signing

```bash
# Create ECR repository with scanning enabled
aws ecr create-repository \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true \
  --encryption-configuration encryptionType=KMS,kmsKey=arn:aws:kms:us-east-1:123456789012:key/xxxxx

# Push image
docker tag my-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest

# Get scan findings
aws ecr describe-image-scan-findings \
  --repository-name my-app \
  --image-id imageTag=latest

# Enable enhanced scanning (Inspector)
aws ecr put-registry-scanning-configuration \
  --scan-type ENHANCED \
  --rules '[{"repositoryFilters":[{"filter":"*","filterType":"WILDCARD"}],"scanFrequency":"CONTINUOUS_SCAN"}]'
```

#### ECS Security
- [ ] Use Fargate for serverless containers
- [ ] Task IAM roles (not instance roles)
- [ ] Secrets management with Secrets Manager
- [ ] Network mode (awsvpc)
- [ ] Security groups per task

**Secure ECS Task Definition:**
```json
{
  "family": "secure-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ECSTaskRole",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ECSExecutionRole",
  "containerDefinitions": [{
    "name": "app",
    "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
    "essential": true,
    "portMappings": [{
      "containerPort": 8080,
      "protocol": "tcp"
    }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/secure-app",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    },
    "secrets": [{
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db-password"
    }],
    "readonlyRootFilesystem": true,
    "user": "1000:1000",
    "linuxParameters": {
      "capabilities": {
        "drop": ["ALL"]
      }
    }
  }]
}
```

#### EKS Security (Kubernetes)
- [ ] Enable cluster logging
- [ ] Use IRSA (IAM Roles for Service Accounts)
- [ ] Network policies
- [ ] Pod security standards
- [ ] RBAC configuration
- [ ] Secrets encryption with KMS

**IRSA Configuration:**
```bash
# Create OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve

# Create service account with IAM role
eksctl create iamserviceaccount \
  --name my-service-account \
  --namespace default \
  --cluster my-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

**Pod Security Policy:**
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
```

### Week 4: Advanced Compute Security

#### Runtime Security
- [ ] AWS GuardDuty EKS protection
- [ ] Falco for runtime monitoring
- [ ] Container anomaly detection
- [ ] File integrity monitoring

#### Compliance and Auditing
- [ ] CIS Docker Benchmark
- [ ] CIS Kubernetes Benchmark
- [ ] Automated compliance scanning
- [ ] Audit logs analysis

## ✅ Phase 10 Checklist

Before completing the roadmap, ensure you can:
- [ ] Configure EC2 instances with IMDSv2 and SSM
- [ ] Build and maintain golden AMIs
- [ ] Automate patch management with Systems Manager
- [ ] Write secure Lambda functions
- [ ] Implement container security best practices
- [ ] Scan container images for vulnerabilities
- [ ] Configure IRSA for EKS workloads
- [ ] Apply pod security policies

## 🎯 Success Criteria

You've mastered compute security when you can:
1. Harden EC2 instances following CIS benchmarks
2. Build secure serverless applications with Lambda
3. Deploy secure containerized applications
4. Implement runtime security monitoring
5. Automate security across compute resources

> **💡 Pro Tip:** Defense in depth! Layer multiple security controls at the network, instance, OS, and application levels.

## 🎊 Congratulations!

You've completed all 10 phases of the Cloud Security Engineering Roadmap! 

**Next Steps:**
1. 📚 Review [Certifications Guide](certifications.md) - Plan your AWS Security Specialty exam
2. 📖 Explore [Resources](resources.md) - Continue learning with curated materials
3. 🔄 Practice, practice, practice - Build labs, break things, learn
4. 🏆 Get certified - AWS Certified Security - Specialty
5. 💼 Apply for Cloud Security roles

---

**Remember:** "Security is a journey, not a destination. Keep learning!" 💻🔒🚀
