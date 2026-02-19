# 🎯 Phase 1: Foundations

> **Building a solid foundation for Cloud Security Engineering**

**Estimated Time:** 2 months (8 weeks)  
**Effort:** 10-15 hours per week

## 📋 Overview

Before diving into AWS-specific security, you need a strong foundation in networking, operating systems, programming, and core security concepts. This phase ensures you have the prerequisite knowledge to excel in cloud security.

## 🎓 Learning Objectives

By the end of this phase, you will:
- ✅ Understand networking fundamentals and the OSI model
- ✅ Be comfortable with Linux command line and system administration
- ✅ Write basic Python scripts for automation
- ✅ Grasp core security principles (CIA triad, Zero Trust, encryption)
- ✅ Navigate the AWS Management Console
- ✅ Understand basic cloud computing concepts

## 📚 Topics & Progress Tracker

### Week 1-2: Networking Fundamentals

#### OSI Model & TCP/IP
- [ ] Understand the 7 layers of the OSI model
- [ ] Learn TCP vs UDP differences
- [ ] Master the three-way handshake
- [ ] Understand IP addressing (IPv4 and IPv6)
- [ ] Learn about CIDR notation and subnetting

**Key Concepts:**
```
┌─────────────────────────────────────────────────────┐
│              OSI MODEL (7 Layers)                   │
├─────────────────────────────────────────────────────┤
│ 7. Application  │ HTTP, DNS, SMTP, SSH             │
│ 6. Presentation │ SSL/TLS, Encryption               │
│ 5. Session      │ Session Management                │
│ 4. Transport    │ TCP, UDP (Port numbers)           │
│ 3. Network      │ IP, ICMP, Routing                 │
│ 2. Data Link    │ MAC addresses, Switches           │
│ 1. Physical     │ Cables, Signals                   │
└─────────────────────────────────────────────────────┘
```

#### Subnetting Practice
- [ ] Calculate subnet masks
- [ ] Determine network and broadcast addresses
- [ ] Understand CIDR notation (e.g., 10.0.0.0/16)
- [ ] Practice subnet design for different scenarios

**Example: Common AWS VPC CIDR blocks**
```
10.0.0.0/16     → 65,536 IP addresses (typical VPC)
10.0.1.0/24     → 256 IP addresses (typical subnet)
10.0.1.0/28     → 16 IP addresses (small subnet)
```

> **💡 Pro Tip:** Use online subnet calculators initially, but practice manual calculation to truly understand the concepts.

#### DNS Deep Dive
- [ ] Understand DNS hierarchy and record types (A, AAAA, CNAME, MX, TXT)
- [ ] Learn DNS resolution process
- [ ] Understand DNS caching
- [ ] Learn about Route 53 basics

#### Network Protocols
- [ ] HTTP/HTTPS fundamentals
- [ ] SSH protocol and key-based authentication
- [ ] FTP/SFTP differences
- [ ] Common port numbers (22, 80, 443, 3389, etc.)

**Common Ports to Memorize:**
| Port | Protocol | Service |
|------|----------|---------|
| 22   | TCP      | SSH     |
| 80   | TCP      | HTTP    |
| 443  | TCP      | HTTPS   |
| 3389 | TCP      | RDP     |
| 3306 | TCP      | MySQL   |
| 5432 | TCP      | PostgreSQL |

### Week 3-4: Linux System Administration

#### Command Line Basics
- [ ] Navigate filesystem (cd, ls, pwd)
- [ ] File operations (cp, mv, rm, touch, mkdir)
- [ ] File permissions (chmod, chown, chgrp)
- [ ] Text processing (cat, grep, sed, awk, cut)
- [ ] Process management (ps, top, kill, systemctl)

**Essential Commands Cheat Sheet:**
```bash
# File Permissions
chmod 600 private_key.pem    # Read/write for owner only
chmod 755 script.sh          # Executable by all, writable by owner
ls -la                        # List all files with permissions

# Finding Files
find /var/log -name "*.log" -mtime -7    # Find logs modified in last 7 days
grep -r "ERROR" /var/log/                # Search for ERROR in all log files

# System Information
uname -a                      # System information
df -h                         # Disk space
free -h                       # Memory usage
uptime                        # System uptime
```

#### User & Permission Management
- [ ] Understand Linux file permission model (rwx)
- [ ] User and group management
- [ ] sudo and privilege escalation
- [ ] Understanding /etc/passwd and /etc/shadow

**Permission Breakdown:**
```
-rwxr-xr-x  1 user group  4096 Jan 01 12:00 file.sh
│││││││││
│││││││└┴─ Execute (1)
││││││└──── Read (4)
│││││└───── Write (2)
││││└────── Group permissions
│││└─────── Owner permissions
││└──────── File type (- = file, d = directory)
│└───────── Sticky bit/SUID/SGID
```

#### Networking Commands
- [ ] ifconfig / ip addr
- [ ] netstat / ss
- [ ] ping, traceroute
- [ ] nslookup, dig
- [ ] tcpdump basics
- [ ] iptables fundamentals

```bash
# Network Diagnostics
ip addr show                  # Show IP addresses
ss -tulpn                     # Show listening ports
ping -c 4 google.com          # Test connectivity
traceroute google.com         # Trace route to host
dig example.com              # DNS lookup
tcpdump -i eth0 port 80      # Capture HTTP traffic
```

#### Shell Scripting
- [ ] Bash scripting basics
- [ ] Variables and loops
- [ ] Conditionals (if/else)
- [ ] Functions
- [ ] Input/output redirection

**Simple Security Script Example:**
```bash
#!/bin/bash
# Check for failed SSH login attempts

echo "=== Failed SSH Login Attempts ==="
grep "Failed password" /var/log/auth.log | awk '{print $1, $2, $3, $11}' | sort | uniq -c | sort -nr

echo "=== Unique IP addresses with failed logins ==="
grep "Failed password" /var/log/auth.log | grep -oP '\d+\.\d+\.\d+\.\d+' | sort | uniq -c | sort -nr
```

#### Log Analysis
- [ ] Understand common log locations (/var/log)
- [ ] Parse system logs
- [ ] Understand syslog format
- [ ] Use journalctl (systemd)

### Week 5-6: Python for Security

#### Python Basics
- [ ] Variables, data types, and operators
- [ ] Control structures (if, for, while)
- [ ] Functions and modules
- [ ] Lists, dictionaries, tuples, sets
- [ ] File I/O operations
- [ ] Exception handling

**Python Security Script Example:**
```python
#!/usr/bin/env python3
"""
Simple S3 bucket permission checker
"""
import boto3

def check_s3_public_access():
    """Check for publicly accessible S3 buckets"""
    s3 = boto3.client('s3')
    buckets = s3.list_buckets()['Buckets']
    
    print("Checking S3 buckets for public access...")
    
    for bucket in buckets:
        bucket_name = bucket['Name']
        try:
            # Check bucket ACL
            acl = s3.get_bucket_acl(Bucket=bucket_name)
            
            for grant in acl['Grants']:
                grantee = grant.get('Grantee', {})
                if grantee.get('Type') == 'Group':
                    uri = grantee.get('URI', '')
                    if 'AllUsers' in uri or 'AuthenticatedUsers' in uri:
                        print(f"⚠️  WARNING: {bucket_name} has public access!")
                        print(f"   Permission: {grant['Permission']}")
        except Exception as e:
            print(f"❌ Error checking {bucket_name}: {str(e)}")

if __name__ == "__main__":
    check_s3_public_access()
```

#### Boto3 - AWS SDK for Python
- [ ] Install and configure boto3
- [ ] Basic S3 operations
- [ ] EC2 instance management
- [ ] IAM policy manipulation
- [ ] Error handling and pagination

```python
import boto3

# Initialize AWS clients
ec2 = boto3.client('ec2', region_name='us-east-1')
s3 = boto3.client('s3')

# List EC2 instances
response = ec2.describe_instances()
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        print(f"Instance ID: {instance['InstanceId']}")
        print(f"State: {instance['State']['Name']}")
```

#### Useful Python Libraries for Security
- [ ] **requests** - HTTP library
- [ ] **paramiko** - SSH implementation
- [ ] **boto3** - AWS SDK
- [ ] **json** - JSON parsing
- [ ] **argparse** - Command-line arguments
- [ ] **logging** - Proper logging

### Week 7-8: Security Fundamentals

#### The CIA Triad
- [ ] **Confidentiality** - Preventing unauthorized access
- [ ] **Integrity** - Ensuring data accuracy and trustworthiness
- [ ] **Availability** - Ensuring systems are accessible when needed

```
┌────────────────────────────────────────┐
│          CIA TRIAD                     │
│                                        │
│         Confidentiality                │
│              ╱   ╲                    │
│             ╱     ╲                   │
│            ╱       ╲                  │
│           ╱         ╲                 │
│    Integrity ─────── Availability     │
│                                        │
└────────────────────────────────────────┘

Confidentiality → Encryption, Access Controls, MFA
Integrity → Hashing, Digital Signatures, Checksums
Availability → Redundancy, Backups, DDoS Protection
```

> **🔑 Key Takeaway:** Every security decision should be evaluated against the CIA triad. Sometimes you need to balance trade-offs between these three pillars.

#### Authentication vs Authorization
- [ ] Understand the difference between AuthN and AuthZ
- [ ] Multi-Factor Authentication (MFA)
- [ ] Single Sign-On (SSO)
- [ ] Role-Based Access Control (RBAC)

**Authentication (AuthN):** "Who are you?"
- Username/password
- MFA tokens
- Biometrics
- Certificate-based

**Authorization (AuthZ):** "What can you do?"
- IAM policies
- RBAC
- ABAC (Attribute-Based Access Control)
- ACLs

#### Encryption Basics
- [ ] Symmetric vs Asymmetric encryption
- [ ] Common algorithms (AES, RSA)
- [ ] Hashing (SHA-256, MD5)
- [ ] TLS/SSL fundamentals
- [ ] PKI and Certificate Authorities

**Encryption Types:**
```
Symmetric (Same key for encrypt/decrypt)
├─ AES-256 (AWS default)
├─ AES-128
└─ DES (deprecated)

Asymmetric (Public/Private key pair)
├─ RSA (2048-bit, 4096-bit)
├─ ECC (Elliptic Curve)
└─ Used for: Key exchange, digital signatures

Hashing (One-way function)
├─ SHA-256 (recommended)
├─ SHA-512
└─ MD5 (deprecated - use only for checksums)
```

#### Zero Trust Security Model
- [ ] Understand "never trust, always verify"
- [ ] Principle of least privilege
- [ ] Micro-segmentation
- [ ] Continuous verification

**Zero Trust Principles:**
1. Verify explicitly (always authenticate and authorize)
2. Use least privileged access (just enough permissions)
3. Assume breach (segment access, verify, and monitor)

#### Common Attack Vectors
- [ ] SQL Injection
- [ ] Cross-Site Scripting (XSS)
- [ ] Cross-Site Request Forgery (CSRF)
- [ ] Man-in-the-Middle (MITM)
- [ ] Phishing and social engineering
- [ ] DDoS attacks
- [ ] Privilege escalation

### Cloud Computing Basics

#### AWS Account Setup
- [ ] Create AWS Free Tier account
- [ ] **Enable MFA on root account** (CRITICAL!)
- [ ] Create IAM user for daily use
- [ ] Set up billing alerts ($10, $50, $100 thresholds)
- [ ] Install and configure AWS CLI

**AWS CLI Configuration:**
```bash
# Install AWS CLI
pip install awscli

# Configure credentials
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region: us-east-1
# Default output format: json

# Test configuration
aws sts get-caller-identity
aws s3 ls
```

> **⚠️ CRITICAL SECURITY PRACTICE:** 
> 1. NEVER use root account for daily tasks
> 2. ALWAYS enable MFA on root account
> 3. NEVER commit AWS credentials to Git

#### Core AWS Services Overview
- [ ] EC2 (Elastic Compute Cloud)
- [ ] S3 (Simple Storage Service)
- [ ] VPC (Virtual Private Cloud)
- [ ] IAM (Identity and Access Management)
- [ ] RDS (Relational Database Service)
- [ ] Lambda (Serverless compute)

#### AWS Shared Responsibility Model
- [ ] Understand what AWS manages vs what you manage
- [ ] Security "of" the cloud vs security "in" the cloud

```
┌─────────────────────────────────────────────────────┐
│         AWS SHARED RESPONSIBILITY MODEL             │
├─────────────────────────────────────────────────────┤
│  CUSTOMER (Security IN the Cloud)                   │
│  ├─ Customer Data                                   │
│  ├─ Platform, Applications, IAM                     │
│  ├─ Operating System, Network & Firewall Config    │
│  ├─ Client-side Data Encryption                     │
│  └─ Network Traffic Protection                      │
├─────────────────────────────────────────────────────┤
│  AWS (Security OF the Cloud)                        │
│  ├─ Compute, Storage, Database, Networking          │
│  ├─ Hardware / AWS Global Infrastructure            │
│  ├─ Regions, Availability Zones, Edge Locations     │
│  └─ Physical Security of Data Centers               │
└─────────────────────────────────────────────────────┘
```

## 🛠️ Hands-On Labs

### Lab 1: Linux Security Hardening
```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Configure firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable

# 3. Disable root login via SSH
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# 4. Set up automatic security updates
sudo apt install unattended-upgrades
```

### Lab 2: Python AWS Automation
```python
# Create a script that lists all EC2 instances and their security groups
import boto3

ec2 = boto3.client('ec2', region_name='us-east-1')

response = ec2.describe_instances()
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        instance_id = instance['InstanceId']
        security_groups = [sg['GroupName'] for sg in instance['SecurityGroups']]
        print(f"Instance: {instance_id}")
        print(f"Security Groups: {', '.join(security_groups)}")
        print("---")
```

### Lab 3: Set Up AWS Account Security
- [ ] Enable MFA on root account
- [ ] Create IAM user with admin access
- [ ] Enable MFA on IAM user
- [ ] Create billing alarm
- [ ] Review CloudTrail (should be enabled by default)

### Lab 4: Subnet Calculation Practice
```
Exercise: Design a VPC with the following requirements:
- VPC CIDR: 10.0.0.0/16
- 3 public subnets (256 IPs each)
- 3 private subnets (256 IPs each)
- Each subnet in a different AZ

Solution:
Public Subnets:
- 10.0.1.0/24 (us-east-1a)
- 10.0.2.0/24 (us-east-1b)
- 10.0.3.0/24 (us-east-1c)

Private Subnets:
- 10.0.11.0/24 (us-east-1a)
- 10.0.12.0/24 (us-east-1b)
- 10.0.13.0/24 (us-east-1c)
```

## 📚 Recommended Resources

### Books
- **"The Practice of Network Security Monitoring"** by Richard Bejtlich
- **"Linux Command Line and Shell Scripting Bible"** by Richard Blum
- **"Python Crash Course"** by Eric Matthes
- **"Cryptography Engineering"** by Ferguson, Schneier, and Kohno

### Online Courses
- **[Linux Essentials - Linux Academy](https://www.pluralsight.com/)**
- **[Python for Everybody - Coursera](https://www.coursera.org/)**
- **[AWS Cloud Practitioner Essentials](https://aws.amazon.com/training/)**

### Practice Platforms
- **OverTheWire: Bandit** - Linux command line practice
- **HackerRank** - Python practice
- **TryHackMe** - Cybersecurity fundamentals

## ✅ Phase 1 Checklist

Before moving to Phase 2, ensure you can:
- [ ] Subnet a network and understand CIDR notation
- [ ] Navigate Linux command line confidently
- [ ] Write a basic Python script using boto3
- [ ] Explain the CIA triad with examples
- [ ] Describe the difference between symmetric and asymmetric encryption
- [ ] Set up a secure AWS account with MFA enabled
- [ ] Understand the AWS Shared Responsibility Model
- [ ] Create and manage EC2 instances via AWS CLI

## 🎯 Success Criteria

You're ready for Phase 2 when you can:
1. Design a simple network with multiple subnets
2. Write a bash script to parse log files
3. Create a Python script that interacts with AWS
4. Explain core security concepts to a non-technical person
5. Navigate AWS Management Console comfortably

> **💡 Pro Tip:** Don't rush this phase! A strong foundation makes everything else easier. Take time to practice hands-on labs.

## 🔜 What's Next?

Congratulations on completing Phase 1! You now have the foundational knowledge needed for cloud security.

**Next:** [Phase 2: IAM Deep Dive](phase-2-iam.md) - Master AWS Identity and Access Management

---

**Questions or stuck?** Remember, everyone struggles with these concepts at first. Keep practicing, and don't hesitate to revisit topics!
