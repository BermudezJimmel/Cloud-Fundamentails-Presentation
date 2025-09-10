# Module 4: IAM Advanced Concepts & Best Practices
**Duration:** 25 minutes  
**Level:** Cloud Practitioner

## 🎯 Learning Objectives
By the end of this module, you will:
- Understand IAM components and when to use each
- Know the difference between users and roles
- Be able to implement security best practices
- Understand advanced IAM features

## 🔐 What is IAM?

### Simple Explanation
IAM (Identity and Access Management) is like a security system for your office building. It controls:
- **Who** can enter (authentication)
- **What** they can do once inside (authorization)
- **When** they can access resources (conditions)

### Why IAM Matters for TESDA
- Protect student data and academic records
- Ensure only authorized staff access systems
- Meet government compliance requirements
- Prevent accidental or malicious changes

## 👥 IAM Components Explained

### 1. Users vs Roles - When to Use Each

#### IAM Users
**What they are:** Real people who need long-term access
**When to use:**
- Staff members who regularly access AWS
- Service accounts for applications
- Long-term access requirements

**Example:**
```
User: maria.santos@tesda.gov.ph
Role: Curriculum Developer
Access: Development account, specific S3 buckets
Duration: Permanent (until employment ends)
```

#### IAM Roles
**What they are:** Temporary access for services or short-term needs
**When to use:**
- EC2 servers need to access other AWS services
- Cross-account access
- Temporary access for contractors

**Example:**
```
Role: TESDA-EC2-S3-Access
Purpose: Allow web servers to read/write student files
Duration: Only while the server is running
```

### 2. Policy Types Made Simple

#### AWS Managed Policies
**What they are:** Pre-built by AWS (like templates)
**Advantages:**
- Maintained by AWS
- Regular security updates
- Best practices built-in

**Examples:**
```
ReadOnlyAccess - Can view everything, change nothing
PowerUserAccess - Full access except user management
DatabaseAdministrator - Full database management
```

#### Customer Managed Policies
**What they are:** Custom policies you create
**When to use:**
- Specific TESDA requirements
- Combination of multiple AWS services
- Custom business logic

**Example - TESDA LMS Access:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::tesda-student-files/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Inline Policies
**What they are:** Attached directly to one user/role
**When to use:** Rarely - only for very specific exceptions
**Why avoid:** Harder to manage and reuse

### 3. Permission Boundaries (Advanced Safety Net)

**What they are:** Sets maximum permissions a user can have
**Analogy:** Like a speed limit - even if you have a fast car, you can't exceed it

**Use Case for TESDA:**
```
Scenario: Give department heads ability to create users
Problem: Don't want them to create users with more access than themselves
Solution: Permission boundary limits what they can grant
```

**Example Permission Boundary:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["ap-southeast-1", "ap-southeast-2"]
        }
      }
    }
  ]
}
```
*Translation: "No matter what permissions are granted, can only work in Southeast Asia regions"*

### 4. Access Analyzer (Your Security Assistant)

**What it does:**
- Finds unused permissions
- Identifies overly broad access
- Suggests improvements
- Monitors external access

**How to use:**
1. Go to IAM Console → Access Analyzer
2. Create analyzer for your organization
3. Review findings regularly
4. Remove unused permissions

## 🔧 Advanced IAM Features

### Condition Keys (Smart Access Control)

**Purpose:** Add context-based restrictions to policies

#### Time-Based Access
```json
"Condition": {
  "DateGreaterThan": {
    "aws:CurrentTime": "2024-01-01T00:00:00Z"
  },
  "DateLessThan": {
    "aws:CurrentTime": "2024-12-31T23:59:59Z"
  }
}
```
*Translation: "Only allow access during 2024"*

#### IP Address Restrictions
```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": ["203.0.113.0/24", "198.51.100.0/24"]
  }
}
```
*Translation: "Only allow access from TESDA office IP addresses"*

#### MFA Requirements
```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```
*Translation: "Must use MFA to perform this action"*

### Policy Variables (Dynamic Permissions)

**Purpose:** Create flexible policies that adapt to the user

#### User-Specific Folders
```json
"Resource": "arn:aws:s3:::tesda-user-files/${aws:username}/*"
```
*Translation: "Each user can only access their own folder"*

#### Department-Based Access
```json
"Resource": "arn:aws:s3:::tesda-dept-files/${saml:Department}/*"
```
*Translation: "Users can only access their department's files"*

### Cross-Account Access (Secure Resource Sharing)

**Scenario:** Security team in Security Account needs to monitor Production Account

**Solution:** Cross-account role assumption

#### Step 1: Create Role in Production Account
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::SECURITY-ACCOUNT-ID:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### Step 2: Security team assumes role when needed
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::PROD-ACCOUNT:role/SecurityMonitoringRole \
  --role-session-name SecurityReview
```

### Temporary Credentials (STS and Assume Role)

**What is STS?** Security Token Service - provides temporary credentials

**Benefits:**
- Credentials expire automatically
- No long-term secrets to manage
- Can be revoked immediately if needed

**Use Cases:**
- Applications running on EC2
- Mobile apps accessing AWS
- Cross-account access
- Federated users from Active Directory

## 🛡️ Security Best Practices for TESDA

### 1. Principle of Least Privilege
**Rule:** Give minimum access needed to do the job

**Implementation:**
```
❌ Bad: Give everyone PowerUserAccess
✅ Good: Create specific roles for each job function

Examples:
- TESDA-Instructor: Access to LMS content only
- TESDA-IT-Support: Infrastructure management only
- TESDA-Auditor: Read-only access to logs and configs
```

### 2. Regular Access Reviews
**Frequency:** Quarterly for all users, monthly for privileged access

**Process:**
1. Export all users and their permissions
2. Review with department managers
3. Remove access for departed staff
4. Reduce over-privileged accounts
5. Document all changes

### 3. Automated Policy Compliance
**Tools to use:**
- AWS Config Rules
- AWS Security Hub
- Custom Lambda functions

**Example Config Rule:**
```
Rule: "iam-user-mfa-enabled"
Purpose: Ensure all IAM users have MFA enabled
Action: Alert security team if MFA is disabled
```

### 4. Centralized Identity Source Integration
**Recommendation:** Connect to TESDA's Active Directory

**Benefits:**
- Single source of truth for user identities
- Automatic user provisioning/deprovisioning
- Consistent password policies
- Simplified user management

## 🎯 TESDA-Specific IAM Strategy

### Role-Based Access Control (RBAC)

#### Administrative Roles
```
TESDA-SuperAdmin: Full AWS access (IT Director only)
TESDA-ITAdmin: Infrastructure management
TESDA-SecurityAdmin: Security tools and monitoring
TESDA-BillingAdmin: Cost management and billing
```

#### Functional Roles
```
TESDA-Instructor: LMS content management
TESDA-Registrar: Student data management
TESDA-Assessor: Testing and certification tools
TESDA-Developer: Application development access
```

#### Temporary Roles
```
TESDA-Contractor: Limited access for external vendors
TESDA-Auditor: Read-only access for compliance reviews
TESDA-Support: Break-glass access for emergencies
```

### Department-Based Policies

#### Curriculum Development Department
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*"
      ],
      "Resource": [
        "arn:aws:s3:::tesda-curriculum/*",
        "arn:aws:s3:::tesda-learning-materials/*"
      ]
    }
  ]
}
```

#### Assessment Department
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:Describe*",
        "rds:List*"
      ],
      "Resource": "arn:aws:rds:*:*:db:tesda-assessment-*"
    }
  ]
}
```

## ✅ Implementation Checklist

### Week 1: Foundation
- [ ] Audit existing IAM users and roles
- [ ] Document current access patterns
- [ ] Plan role-based access structure
- [ ] Create naming conventions

### Week 2: Role Creation
- [ ] Create administrative roles
- [ ] Create functional roles
- [ ] Create department-specific policies
- [ ] Test role assumptions

### Week 3: User Migration
- [ ] Migrate users to appropriate roles
- [ ] Remove direct policy attachments
- [ ] Enable MFA for all users
- [ ] Document new access procedures

### Week 4: Monitoring Setup
- [ ] Configure Access Analyzer
- [ ] Set up automated compliance checks
- [ ] Create access review procedures
- [ ] Train staff on new processes

## 📚 Next Steps
Continue to [Module 5: AWS IAM Identity Center Implementation](../05-identity-center/README.md)

---
**💡 Pro Tip:** Start with AWS managed policies and customize only when necessary. This reduces maintenance overhead and improves security.
