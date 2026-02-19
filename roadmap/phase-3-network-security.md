# 🌐 Phase 3: Network Security

> **Secure your AWS network infrastructure like a pro**

**Estimated Time:** 1 month (4 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Network security is your first line of defense in AWS. Understanding how to properly architect and secure Virtual Private Clouds (VPCs), configure security groups and NACLs, and leverage AWS network security services is essential for protecting your cloud infrastructure.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Design secure VPC architectures
- ✅ Configure Security Groups and Network ACLs effectively
- ✅ Implement AWS WAF for application protection
- ✅ Use AWS Shield for DDoS protection
- ✅ Analyze VPC Flow Logs for security insights
- ✅ Leverage PrivateLink and Transit Gateway
- ✅ Apply network segmentation strategies

## 📚 Topics & Progress Tracker

### Week 1: VPC Fundamentals

#### Understanding VPCs
- [ ] VPC concepts and components
- [ ] Public vs Private subnets
- [ ] Route tables and routing
- [ ] Internet Gateway (IGW)
- [ ] NAT Gateway vs NAT Instance
- [ ] CIDR block planning

**VPC Architecture Overview:**
```
┌─────────────────────────────────────────────────────────────┐
│                    AWS VPC (10.0.0.0/16)                    │
│                                                             │
│  ┌───────────────────────┐  ┌───────────────────────┐     │
│  │  Public Subnet        │  │  Private Subnet       │     │
│  │  10.0.1.0/24          │  │  10.0.11.0/24        │     │
│  │  ┌─────────────────┐  │  │  ┌─────────────────┐ │     │
│  │  │   Web Server    │  │  │  │   App Server    │ │     │
│  │  │   (Public IP)   │  │  │  │   (Private IP)  │ │     │
│  │  └─────────────────┘  │  │  └─────────────────┘ │     │
│  └──────────┬────────────┘  └──────────┬────────────┘     │
│             │                           │                  │
│        ┌────▼─────┐                ┌───▼────┐             │
│        │ Internet │                │  NAT   │             │
│        │ Gateway  │                │ Gateway│             │
│        └────┬─────┘                └───┬────┘             │
└─────────────┼──────────────────────────┼──────────────────┘
              │                          │
         ┌────▼──────────────────────────▼─────┐
         │         Internet                     │
         └──────────────────────────────────────┘
```

#### VPC Design Best Practices
- [ ] Use multiple Availability Zones (minimum 2)
- [ ] Separate public and private subnets
- [ ] Plan CIDR blocks for future growth
- [ ] Use /16 for VPC, /24 for subnets
- [ ] Reserve IP space for peering
- [ ] Document your IP allocation strategy

**Example VPC CIDR Planning:**
```
Production VPC:    10.0.0.0/16
├─ Public Subnet AZ-A:   10.0.1.0/24  (256 IPs)
├─ Public Subnet AZ-B:   10.0.2.0/24  (256 IPs)
├─ Private Subnet AZ-A:  10.0.11.0/24 (256 IPs)
├─ Private Subnet AZ-B:  10.0.12.0/24 (256 IPs)
├─ Database Subnet AZ-A: 10.0.21.0/24 (256 IPs)
└─ Database Subnet AZ-B: 10.0.22.0/24 (256 IPs)

Development VPC:   10.1.0.0/16
Staging VPC:       10.2.0.0/16
```

> **💡 Pro Tip:** Avoid overlapping CIDR blocks between VPCs if you plan to use VPC peering or Transit Gateway.

#### Route Tables
- [ ] Main route table vs custom route tables
- [ ] Default routes (0.0.0.0/0)
- [ ] Local routes (automatic)
- [ ] Route priority and evaluation
- [ ] Route table associations

**Route Table Example:**
```
Public Subnet Route Table:
┌────────────────────┬─────────────────┬──────────┐
│ Destination        │ Target          │ Status   │
├────────────────────┼─────────────────┼──────────┤
│ 10.0.0.0/16        │ local           │ Active   │
│ 0.0.0.0/0          │ igw-xxxxx       │ Active   │
└────────────────────┴─────────────────┴──────────┘

Private Subnet Route Table:
┌────────────────────┬─────────────────┬──────────┐
│ Destination        │ Target          │ Status   │
├────────────────────┼─────────────────┼──────────┤
│ 10.0.0.0/16        │ local           │ Active   │
│ 0.0.0.0/0          │ nat-xxxxx       │ Active   │
└────────────────────┴─────────────────┴──────────┘
```

#### Creating a VPC
```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyVPC}]'

# Create public subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'

# Create private subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.11.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'

# Create and attach Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]'

aws ec2 attach-internet-gateway \
  --vpc-id vpc-xxxxx \
  --internet-gateway-id igw-xxxxx
```

### Week 2: Security Groups and Network ACLs

#### Security Groups (Stateful)
- [ ] Understand stateful firewall behavior
- [ ] Create and configure security groups
- [ ] Define inbound and outbound rules
- [ ] Security group chaining
- [ ] Best practices for security groups

**Security Group Architecture:**
```
┌──────────────────────────────────────────────────────┐
│            Security Group: Web-SG                    │
│                                                      │
│  Inbound Rules:                                      │
│  ┌────────┬──────┬──────────┬──────────────┐       │
│  │ Type   │ Port │ Source   │ Description  │       │
│  ├────────┼──────┼──────────┼──────────────┤       │
│  │ HTTP   │ 80   │ 0.0.0.0/0│ Public web   │       │
│  │ HTTPS  │ 443  │ 0.0.0.0/0│ Public web   │       │
│  │ SSH    │ 22   │ My-IP/32 │ Admin access │       │
│  └────────┴──────┴──────────┴──────────────┘       │
│                                                      │
│  Outbound Rules:                                     │
│  ┌────────┬──────┬──────────┬──────────────┐       │
│  │ Type   │ Port │ Dest     │ Description  │       │
│  ├────────┼──────┼──────────┼──────────────┤       │
│  │ All    │ All  │ 0.0.0.0/0│ All traffic  │       │
│  └────────┴──────┴──────────┴──────────────┘       │
└──────────────────────────────────────────────────────┘
```

**Security Group CLI Examples:**
```bash
# Create security group
aws ec2 create-security-group \
  --group-name web-servers-sg \
  --description "Security group for web servers" \
  --vpc-id vpc-xxxxx

# Add inbound rule for HTTPS
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Add inbound rule from another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-app-server \
  --protocol tcp \
  --port 8080 \
  --source-group sg-web-server
```

#### Network ACLs (Stateless)
- [ ] Understand stateless firewall behavior
- [ ] NACL rule numbering and evaluation
- [ ] Default NACL vs custom NACL
- [ ] Allow and deny rules
- [ ] NACL vs Security Group comparison

**NACL Example:**
```
┌──────────────────────────────────────────────────────┐
│         Network ACL: Public-Subnet-NACL              │
│                                                      │
│  Inbound Rules (Evaluated in order):                │
│  ┌──────┬────────┬──────┬────────┬────────┐        │
│  │ Rule │ Type   │ Port │ Source │ Action │        │
│  ├──────┼────────┼──────┼────────┼────────┤        │
│  │ 100  │ HTTP   │ 80   │ 0.0.0.0│ ALLOW  │        │
│  │ 110  │ HTTPS  │ 443  │ 0.0.0.0│ ALLOW  │        │
│  │ 120  │ SSH    │ 22   │ Office │ ALLOW  │        │
│  │ *    │ All    │ All  │ 0.0.0.0│ DENY   │        │
│  └──────┴────────┴──────┴────────┴────────┘        │
│                                                      │
│  Outbound Rules:                                     │
│  ┌──────┬────────┬──────┬────────┬────────┐        │
│  │ Rule │ Type   │ Port │ Dest   │ Action │        │
│  ├──────┼────────┼──────┼────────┼────────┤        │
│  │ 100  │ All    │ All  │ 0.0.0.0│ ALLOW  │        │
│  │ *    │ All    │ All  │ 0.0.0.0│ DENY   │        │
│  └──────┴────────┴──────┴────────┴────────┘        │
└──────────────────────────────────────────────────────┘
```

> **🔑 Key Difference:** Security Groups are STATEFUL (return traffic automatically allowed), NACLs are STATELESS (must explicitly allow return traffic).

#### Security Groups vs NACLs

| Feature | Security Groups | Network ACLs |
|---------|----------------|--------------|
| **State** | Stateful | Stateless |
| **Level** | Instance | Subnet |
| **Rules** | Allow only | Allow & Deny |
| **Rule Eval** | All rules | Rules in order |
| **Return Traffic** | Automatic | Must be explicit |
| **Default** | Deny all inbound | Allow all |
| **Use Case** | Instance-level control | Subnet-level defense |

**Defense-in-Depth Strategy:**
```
┌─────────────────────────────────────────┐
│         Internet (Threats)              │
└────────────────┬────────────────────────┘
                 │
        ┌────────▼─────────┐
        │  Network ACL     │ ← Layer 1: Subnet-level
        │  (Subnet Border) │
        └────────┬─────────┘
                 │
        ┌────────▼─────────┐
        │ Security Group   │ ← Layer 2: Instance-level
        │ (Instance ENI)   │
        └────────┬─────────┘
                 │
        ┌────────▼─────────┐
        │  EC2 Instance    │
        │  (Host Firewall) │ ← Layer 3: OS-level
        └──────────────────┘
```

### Week 3: VPC Flow Logs & Advanced Networking

#### VPC Flow Logs
- [ ] Enable VPC Flow Logs
- [ ] Log to CloudWatch Logs
- [ ] Log to S3
- [ ] Analyze flow log data
- [ ] Identify security anomalies
- [ ] Use Athena for log analysis

**VPC Flow Log Format:**
```
<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>

Example:
2 123456789010 eni-abc123 172.31.16.139 172.31.16.21 49152 22 6 20 4249 1418530010 1418530070 ACCEPT OK
```

**Enable VPC Flow Logs:**
```bash
# Flow logs to CloudWatch
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole

# Flow logs to S3
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::my-flow-logs-bucket/
```

**Athena Query for Rejected Connections:**
```sql
SELECT 
  srcaddr, 
  dstaddr, 
  dstport, 
  COUNT(*) as count
FROM vpc_flow_logs
WHERE action = 'REJECT'
  AND day BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY srcaddr, dstaddr, dstport
ORDER BY count DESC
LIMIT 100;
```

#### VPC Endpoints
- [ ] Understand interface endpoints (PrivateLink)
- [ ] Use gateway endpoints (S3, DynamoDB)
- [ ] Eliminate data transfer costs
- [ ] Improve security posture
- [ ] Configure endpoint policies

**Gateway Endpoint (S3):**
```bash
# Create S3 VPC endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-xxxxx
```

**Interface Endpoint (Secrets Manager):**
```bash
# Create interface endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxxx \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-east-1.secretsmanager \
  --subnet-ids subnet-xxxxx subnet-yyyyy \
  --security-group-ids sg-xxxxx
```

#### VPC Peering
- [ ] Connect VPCs within same region
- [ ] Cross-region VPC peering
- [ ] Transitive peering limitations
- [ ] Peering connection routing

**VPC Peering Architecture:**
```
┌──────────────────┐           ┌──────────────────┐
│   VPC A          │           │   VPC B          │
│   10.0.0.0/16    │◄─────────►│   10.1.0.0/16    │
│                  │  Peering  │                  │
│   App Tier       │           │   DB Tier        │
└──────────────────┘           └──────────────────┘
```

#### AWS Transit Gateway
- [ ] Hub-and-spoke network topology
- [ ] Connect multiple VPCs
- [ ] On-premises connectivity
- [ ] Route table management
- [ ] Transit Gateway attachments

**Transit Gateway Architecture:**
```
              ┌────────────────────┐
              │ Transit Gateway    │
              │  (Central Hub)     │
              └────────┬───────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   ┌────▼────┐    ┌────▼────┐   ┌────▼────┐
   │ VPC-1   │    │ VPC-2   │   │ VPC-3   │
   │ Prod    │    │ Dev     │   │ Staging │
   └─────────┘    └─────────┘   └─────────┘
```

#### AWS PrivateLink
- [ ] Expose services privately
- [ ] Consumer-provider model
- [ ] No VPC peering required
- [ ] Scalable private connectivity

### Week 4: AWS WAF, Shield & Advanced Security

#### AWS WAF (Web Application Firewall)
- [ ] Create Web ACLs
- [ ] Define WAF rules
- [ ] Use managed rule groups
- [ ] Block common attacks (SQLi, XSS)
- [ ] Rate limiting
- [ ] Geo-blocking

**WAF Rule Examples:**
```json
{
  "Name": "BlockSQLInjection",
  "Priority": 1,
  "Statement": {
    "ManagedRuleGroupStatement": {
      "VendorName": "AWS",
      "Name": "AWSManagedRulesSQLiRuleSet"
    }
  },
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "SQLiBlocked"
  }
}
```

**Rate Limiting Rule:**
```json
{
  "Name": "RateLimitRule",
  "Priority": 2,
  "Statement": {
    "RateBasedStatement": {
      "Limit": 2000,
      "AggregateKeyType": "IP"
    }
  },
  "Action": {
    "Block": {}
  }
}
```

#### AWS Shield
- [ ] Shield Standard (automatic, free)
- [ ] Shield Advanced (DDoS protection)
- [ ] DDoS Response Team (DRT)
- [ ] Cost protection
- [ ] Attack forensics

**Shield Protection Levels:**
```
Shield Standard (Free):
├─ Layer 3/4 DDoS protection
├─ Automatic detection and mitigation
└─ Protects all AWS resources

Shield Advanced ($3000/month):
├─ Enhanced detection and mitigation
├─ 24/7 DDoS Response Team (DRT)
├─ Cost protection (credit for scaling)
├─ Advanced metrics and reports
└─ WAF included at no extra cost
```

#### Network Firewall
- [ ] Deploy stateful firewall
- [ ] Create firewall rules
- [ ] Intrusion prevention (IPS)
- [ ] Domain filtering
- [ ] Centralized management

**Network Firewall Use Cases:**
- ✅ Traffic inspection between VPCs
- ✅ Filtering outbound traffic to internet
- ✅ IDS/IPS capabilities
- ✅ Domain allow/deny lists

## 🛠️ Hands-On Labs

### Lab 1: Build Secure 3-Tier VPC
```bash
#!/bin/bash
# Create a secure 3-tier VPC architecture

VPC_CIDR="10.0.0.0/16"
REGION="us-east-1"

# Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block $VPC_CIDR \
  --query 'Vpc.VpcId' \
  --output text)

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id $VPC_ID \
  --enable-dns-hostnames

# Create subnets
PUBLIC_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ${REGION}a \
  --query 'Subnet.SubnetId' \
  --output text)

PRIVATE_APP_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.11.0/24 \
  --availability-zone ${REGION}a \
  --query 'Subnet.SubnetId' \
  --output text)

PRIVATE_DB_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.21.0/24 \
  --availability-zone ${REGION}a \
  --query 'Subnet.SubnetId' \
  --output text)

echo "VPC Created: $VPC_ID"
echo "Public Subnet: $PUBLIC_SUBNET"
echo "Private App Subnet: $PRIVATE_APP_SUBNET"
echo "Private DB Subnet: $PRIVATE_DB_SUBNET"
```

### Lab 2: Security Group Tiers
```bash
# Web tier security group
aws ec2 create-security-group \
  --group-name web-tier-sg \
  --description "Web tier security group" \
  --vpc-id vpc-xxxxx

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-web \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# App tier security group
aws ec2 create-security-group \
  --group-name app-tier-sg \
  --description "App tier security group" \
  --vpc-id vpc-xxxxx

# Allow traffic from web tier only
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 8080 \
  --source-group sg-web

# Database tier security group
aws ec2 create-security-group \
  --group-name db-tier-sg \
  --description "Database tier security group" \
  --vpc-id vpc-xxxxx

# Allow traffic from app tier only
aws ec2 authorize-security-group-ingress \
  --group-id sg-db \
  --protocol tcp \
  --port 3306 \
  --source-group sg-app
```

### Lab 3: Analyze VPC Flow Logs
```python
#!/usr/bin/env python3
"""
Analyze VPC Flow Logs for security threats
"""
import boto3
from collections import Counter

def analyze_rejected_traffic():
    """Find most common rejected connection attempts"""
    logs = boto3.client('logs')
    
    query = """
    fields srcaddr, dstport, action
    | filter action = 'REJECT'
    | stats count() by srcaddr, dstport
    | sort count desc
    | limit 20
    """
    
    response = logs.start_query(
        logGroupName='/aws/vpc/flowlogs',
        startTime=int((datetime.now() - timedelta(days=1)).timestamp()),
        endTime=int(datetime.now().timestamp()),
        queryString=query
    )
    
    print("Top rejected connections in last 24 hours:")
    # Process and display results
```

## ✅ Phase 3 Checklist

Before moving to Phase 4, ensure you can:
- [ ] Design a multi-tier VPC architecture
- [ ] Configure Security Groups and NACLs effectively
- [ ] Explain the difference between stateful and stateless firewalls
- [ ] Enable and analyze VPC Flow Logs
- [ ] Set up VPC endpoints for private connectivity
- [ ] Configure AWS WAF rules
- [ ] Understand AWS Shield protection levels
- [ ] Implement network segmentation strategies

## 🎯 Success Criteria

You're ready for Phase 4 when you can:
1. Design a secure VPC from scratch
2. Troubleshoot network connectivity issues
3. Analyze flow logs for security incidents
4. Configure layered network security controls
5. Explain when to use each AWS network security service

> **💡 Pro Tip:** Network security is about defense-in-depth. Always use multiple layers: NACLs + Security Groups + WAF + Shield.

## 🔜 What's Next?

Excellent work securing your network! Now let's protect your data.

**Next:** [Phase 4: Data Security](phase-4-data-security.md) - Master encryption, KMS, and data protection in AWS

---

**Remember:** "A secure network is the foundation of a secure cloud!" 🌐🔒
