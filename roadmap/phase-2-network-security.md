# Phase 2: Network Security

## Overview
Network security in AWS is critical for protecting your resources and data. This phase covers VPC architecture, network segmentation, traffic filtering, and AWS network security services.

## Learning Objectives
By the end of this phase, you will be able to:
- Design secure VPC architectures
- Implement network segmentation
- Configure Security Groups and NACLs
- Deploy AWS WAF and Shield
- Analyze network traffic with VPC Flow Logs
- Use AWS Network Firewall and PrivateLink

---

## VPC Fundamentals

### VPC Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│  Region: us-east-1                                                 │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  VPC (10.0.0.0/16)                                           │  │
│  │  ┌──────────────────────┐  ┌──────────────────────────────┐ │  │
│  │  │  AZ-1A               │  │  AZ-1B                       │ │  │
│  │  │  ┌────────────────┐  │  │  ┌────────────────────────┐ │ │  │
│  │  │  │ Public Subnet  │  │  │  │ Public Subnet          │ │ │  │
│  │  │  │ (10.0.1.0/24)  │  │  │  │ (10.0.2.0/24)          │ │ │  │
│  │  │  │  - Web Tier    │  │  │  │  - Web Tier            │ │ │  │
│  │  │  │  - NAT Gateway │  │  │  │  - NAT Gateway         │ │ │  │
│  │  │  └────────┬───────┘  │  │  └────────┬───────────────┘ │ │  │
│  │  │           ↓          │  │           ↓                 │ │  │
│  │  │  ┌────────────────┐  │  │  ┌────────────────────────┐ │ │  │
│  │  │  │ Private Subnet │  │  │  │ Private Subnet         │ │ │  │
│  │  │  │ (10.0.11.0/24) │  │  │  │ (10.0.12.0/24)         │ │ │  │
│  │  │  │  - App Tier    │  │  │  │  - App Tier            │ │ │  │
│  │  │  └────────┬───────┘  │  │  └────────┬───────────────┘ │ │  │
│  │  │           ↓          │  │           ↓                 │ │  │
│  │  │  ┌────────────────┐  │  │  ┌────────────────────────┐ │ │  │
│  │  │  │ Private Subnet │  │  │  │ Private Subnet         │ │ │  │
│  │  │  │ (10.0.21.0/24) │  │  │  │ (10.0.22.0/24)         │ │ │  │
│  │  │  │  - DB Tier     │  │  │  │  - DB Tier             │ │ │  │
│  │  │  └────────────────┘  │  │  └────────────────────────┘ │ │  │
│  │  └──────────────────────┘  └──────────────────────────────┘ │  │
│  │                                                               │  │
│  │  Internet Gateway (IGW) ←→ Internet                          │  │
│  └──────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
```

### VPC Components

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **VPC** | Isolated virtual network | CIDR block, DNS, DHCP options |
| **Subnet** | Network segment within VPC | Public or private, per AZ |
| **Internet Gateway (IGW)** | Internet connectivity | Attached to VPC |
| **NAT Gateway** | Outbound internet for private subnets | Per AZ, elastic IP |
| **Route Table** | Direct network traffic | Associated with subnets |
| **Security Group** | Instance-level firewall (stateful) | Allow rules only |
| **NACL** | Subnet-level firewall (stateless) | Allow and deny rules |

---

## Security Groups vs NACLs

### Security Groups (Stateful)

**Characteristics:**
- Operates at instance level (ENI)
- Stateful: Return traffic automatically allowed
- Allow rules only (no deny rules)
- Evaluate all rules before deciding
- Can reference other security groups

**Example: Web Server Security Group**
```bash
# Allow HTTPS from anywhere
Inbound:
  Type: HTTPS
  Protocol: TCP
  Port: 443
  Source: 0.0.0.0/0

# Allow SSH from bastion security group
Inbound:
  Type: SSH
  Protocol: TCP
  Port: 22
  Source: sg-bastion-12345

# Outbound: All traffic allowed by default
Outbound:
  Type: All traffic
  Protocol: All
  Port: All
  Destination: 0.0.0.0/0
```

### Network ACLs (Stateless)

**Characteristics:**
- Operates at subnet level
- Stateless: Must allow both inbound and outbound
- Both allow and deny rules
- Rules processed in numerical order
- Default NACL allows all traffic

**Example: Web Tier NACL**
```bash
Inbound Rules:
  100: ALLOW HTTP from 0.0.0.0/0
  110: ALLOW HTTPS from 0.0.0.0/0
  120: ALLOW Ephemeral ports (1024-65535) from 0.0.0.0/0
  *  : DENY all traffic

Outbound Rules:
  100: ALLOW HTTP to 0.0.0.0/0
  110: ALLOW HTTPS to 0.0.0.0/0
  120: ALLOW Ephemeral ports (1024-65535) to 0.0.0.0/0
  *  : DENY all traffic
```

### Comparison Table

| Feature | Security Group | NACL |
|---------|----------------|------|
| **Level** | Instance (ENI) | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow and Deny |
| **Processing** | All rules evaluated | Processed in order |
| **Default** | Deny all inbound | Allow all |
| **Association** | Can apply many to instance | One per subnet |

---

## Hands-On Labs

### Lab 1: Build Secure 3-Tier VPC

**Objective**: Create a production-ready VPC with public and private subnets.

**Architecture:**
- 1 VPC
- 2 Availability Zones
- 6 Subnets (3 per AZ: public, private-app, private-db)
- Internet Gateway
- 2 NAT Gateways (one per AZ)
- Route tables
- Security groups

**Steps:**

1. **Create VPC:**
```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=production-vpc}]'
```

2. **Create Subnets:**
```bash
# Public subnet AZ1
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-1a}]'

# Private app subnet AZ1
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.11.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-app-1a}]'

# Private DB subnet AZ1
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.21.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-db-1a}]'

# Repeat for AZ2 (us-east-1b) with .2.0/24, .12.0/24, .22.0/24
```

3. **Create and attach Internet Gateway:**
```bash
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=production-igw}]'

aws ec2 attach-internet-gateway \
  --vpc-id vpc-xxxxx \
  --internet-gateway-id igw-xxxxx
```

4. **Create NAT Gateways:**
```bash
# Allocate Elastic IPs
aws ec2 allocate-address --domain vpc

# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
  --subnet-id subnet-public-1a \
  --allocation-id eipalloc-xxxxx \
  --tag-specifications 'ResourceType=nat-gateway,Tags=[{Key=Name,Value=nat-1a}]'
```

5. **Configure Route Tables:**
```bash
# Public route table
aws ec2 create-route-table \
  --vpc-id vpc-xxxxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]'

aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxxxx

# Private route table (for app tier)
aws ec2 create-route-table \
  --vpc-id vpc-xxxxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=private-app-rt}]'

aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-xxxxx
```

### Lab 2: Configure Security Groups

**Objective**: Create layered security with security groups.

**Security Groups to Create:**
1. ALB Security Group
2. Web Server Security Group
3. App Server Security Group
4. Database Security Group

```bash
# 1. ALB Security Group
aws ec2 create-security-group \
  --group-name alb-sg \
  --description "ALB security group" \
  --vpc-id vpc-xxxxx

aws ec2 authorize-security-group-ingress \
  --group-id sg-alb \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# 2. Web Server Security Group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id vpc-xxxxx

# Allow HTTP/HTTPS from ALB only
aws ec2 authorize-security-group-ingress \
  --group-id sg-web \
  --protocol tcp \
  --port 443 \
  --source-group sg-alb

# 3. App Server Security Group
aws ec2 create-security-group \
  --group-name app-sg \
  --description "App server security group" \
  --vpc-id vpc-xxxxx

# Allow port 8080 from web tier only
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 8080 \
  --source-group sg-web

# 4. Database Security Group
aws ec2 create-security-group \
  --group-name db-sg \
  --description "Database security group" \
  --vpc-id vpc-xxxxx

# Allow PostgreSQL from app tier only
aws ec2 authorize-security-group-ingress \
  --group-id sg-db \
  --protocol tcp \
  --port 5432 \
  --source-group sg-app
```

### Lab 3: Enable VPC Flow Logs

**Objective**: Capture and analyze network traffic.

**Steps:**

1. **Create CloudWatch Log Group:**
```bash
aws logs create-log-group \
  --log-group-name /aws/vpc/flowlogs
```

2. **Create IAM Role for Flow Logs:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

3. **Create Flow Logs:**
```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole
```

4. **Analyze Flow Logs:**
```bash
# Query rejected connections
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern '[version, account, eni, source, destination, srcport, destport, protocol, packets, bytes, windowstart, windowend, action="REJECT", flowlogstatus]'
```

**Flow Log Format:**
```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status

Example:
2 123456789012 eni-abc12345 10.0.1.5 203.0.113.5 49152 443 6 10 840 1620000000 1620000060 ACCEPT OK
```

### Lab 4: Deploy AWS WAF

**Objective**: Protect web applications with AWS WAF.

**Steps:**

1. **Create Web ACL:**
```bash
aws wafv2 create-web-acl \
  --name production-web-acl \
  --scope REGIONAL \
  --default-action Allow={} \
  --rules file://waf-rules.json \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=ProductionWebACL
```

2. **Create WAF Rules:**

**Rate Limiting Rule:**
```json
{
  "Name": "RateLimitRule",
  "Priority": 1,
  "Statement": {
    "RateBasedStatement": {
      "Limit": 2000,
      "AggregateKeyType": "IP"
    }
  },
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "RateLimitRule"
  }
}
```

**Geo Blocking Rule:**
```json
{
  "Name": "GeoBlockRule",
  "Priority": 2,
  "Statement": {
    "GeoMatchStatement": {
      "CountryCodes": ["CN", "RU"]
    }
  },
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "GeoBlockRule"
  }
}
```

**SQL Injection Rule:**
```json
{
  "Name": "SQLInjectionRule",
  "Priority": 3,
  "Statement": {
    "ManagedRuleGroupStatement": {
      "VendorName": "AWS",
      "Name": "AWSManagedRulesSQLiRuleSet"
    }
  },
  "OverrideAction": {
    "None": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "SQLInjectionRule"
  }
}
```

3. **Associate WAF with ALB:**
```bash
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:us-east-1:123456789012:regional/webacl/production-web-acl/xxxxx \
  --resource-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/production-alb/xxxxx
```

### Lab 5: AWS Network Firewall

**Objective**: Deploy Network Firewall for advanced traffic filtering.

**Use Cases:**
- Block specific domains (e.g., malware C2 servers)
- Deep packet inspection
- IDS/IPS capabilities
- Protocol detection

**Steps:**

1. **Create Firewall Policy:**
```bash
aws network-firewall create-firewall-policy \
  --firewall-policy-name production-fw-policy \
  --firewall-policy file://firewall-policy.json
```

2. **Create Rule Group (block malicious domains):**
```json
{
  "RulesSource": {
    "RulesSourceList": {
      "TargetTypes": ["HTTP_HOST", "TLS_SNI"],
      "Targets": [
        ".malicious-domain.com",
        "badsite.net"
      ],
      "GeneratedRulesType": "DENYLIST"
    }
  }
}
```

3. **Deploy Firewall:**
```bash
aws network-firewall create-firewall \
  --firewall-name production-firewall \
  --firewall-policy-arn arn:aws:network-firewall:us-east-1:123456789012:firewall-policy/production-fw-policy \
  --vpc-id vpc-xxxxx \
  --subnet-mappings SubnetId=subnet-xxxxx
```

---

## AWS Network Security Services

### 1. AWS Shield

**Standard (Free):**
- Automatic DDoS protection
- Layer 3/4 protection
- Always-on detection

**Advanced ($3,000/month):**
- Enhanced DDoS protection
- 24/7 DDoS Response Team (DRT)
- Cost protection
- Layer 7 attack mitigation

### 2. AWS WAF

**Pricing**: $5/month per Web ACL + $1/rule + $0.60 per million requests

**Key Features:**
- Rate limiting
- Geo-blocking
- IP filtering
- Managed rule groups (OWASP Top 10)
- Bot control

### 3. AWS Network Firewall

**Pricing**: ~$0.395/hour per endpoint + data processing fees

**Use Cases:**
- Domain filtering
- IDS/IPS
- Protocol detection
- Deep packet inspection

### 4. AWS PrivateLink

**Purpose**: Private connectivity to AWS services without internet exposure

**Benefits:**
- No IGW or NAT required
- Traffic stays on AWS network
- Fine-grained access control

**Example: S3 VPC Endpoint:**
```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxxxx
```

---

## Security Best Practices

### Network Segmentation
- [ ] Use multiple subnets (public, private, data)
- [ ] Separate environments (dev, staging, prod)
- [ ] Implement least privilege network access
- [ ] Use separate VPCs for different workloads

### Defense in Depth
- [ ] Use both Security Groups and NACLs
- [ ] Deploy WAF for web applications
- [ ] Enable VPC Flow Logs
- [ ] Monitor with GuardDuty
- [ ] Use Network Firewall for advanced filtering

### High Availability
- [ ] Deploy across multiple AZs
- [ ] Use NAT Gateways (not NAT instances)
- [ ] Implement redundant connections
- [ ] Test failover scenarios

---

## Troubleshooting Network Issues

### Issue: Can't SSH to EC2 instance

**Checklist:**
1. Security Group allows port 22 from your IP
2. NACL allows port 22 inbound and ephemeral ports outbound
3. Instance has public IP (if in public subnet)
4. Route table has route to IGW (if in public subnet)
5. Instance is running
6. Check VPC Flow Logs for rejected connections

### Issue: Private subnet can't access internet

**Checklist:**
1. NAT Gateway exists in public subnet
2. NAT Gateway has Elastic IP
3. Private subnet route table points 0.0.0.0/0 to NAT Gateway
4. Security Group allows outbound traffic
5. NACL allows outbound traffic and return ephemeral ports

---

## Exam Tips

1. **Security Groups are stateful**: Return traffic automatically allowed
2. **NACLs are stateless**: Must explicitly allow return traffic
3. **Security Groups**: Allow rules only, NACLs: Allow and Deny
4. **Default NACL**: Allows all, Custom NACL: Denies all by default
5. **VPC Peering**: Non-transitive, must be explicitly configured
6. **VPC Flow Logs**: Capture metadata, not packet contents
7. **PrivateLink**: Private connectivity without internet exposure

---

## Additional Resources

- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
- [AWS WAF Developer Guide](https://docs.aws.amazon.com/waf/latest/developerguide/)
- [VPC Flow Logs Analysis](https://aws.amazon.com/blogs/networking-and-content-delivery/)

---

[← Previous Phase: IAM](./phase-1-iam.md) | [Back to Main Roadmap](../ROADMAP.md) | [Next Phase: Data Security →](./phase-3-data-security.md)
