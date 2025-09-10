# Module 5: AWS IAM Identity Center Implementation
**Duration:** 25 minutes  
**Level:** Cloud Practitioner

## 🎯 Learning Objectives
By the end of this module, you will:
- Understand what Single Sign-On (SSO) is and why TESDA needs it
- Know how to implement AWS IAM Identity Center
- Be able to create permission sets for different roles
- Understand integration with existing Active Directory

## 🔑 What is Single Sign-On (SSO)?

### Simple Analogy
Imagine having one key that opens all doors in your building. SSO is like that - one login for all AWS accounts and applications.

### Traditional Login Problems
```
❌ Without SSO:
- Staff need separate passwords for each AWS account
- 5 accounts = 5 different passwords to remember
- Password resets multiply across accounts
- Security team can't easily remove access
- Difficult to enforce consistent policies
```

### SSO Solution
```
✅ With SSO:
- One login for all AWS accounts
- Centralized user management
- Consistent security policies
- Easy access removal when staff leave
- Better audit trails
```

## 🏛️ Why Does TESDA Need SSO?

### User Convenience
- **Staff Productivity:** No time wasted on multiple logins
- **Reduced Support:** Fewer password reset requests
- **Better Adoption:** Staff more likely to use secure practices

### Better Security
- **Centralized Control:** Manage all access from one place
- **Consistent Policies:** Same security rules across all accounts
- **Faster Response:** Quickly disable access for departed staff
- **Audit Compliance:** Complete view of who accessed what

### Easier Management
- **Single Source:** Connect to existing TESDA Active Directory
- **Automated Provisioning:** New staff automatically get appropriate access
- **Role-Based Access:** Access based on job function, not individual setup

## 🏗️ AWS IAM Identity Center Architecture

### Visual Overview
```
TESDA Active Directory (Your existing user database)
           ↓ (Connects to)
    AWS IAM Identity Center (The bridge)
           ↓ (Provides access to)
Multiple AWS Accounts (All your cloud resources)
```

### How It Works
1. **Staff logs in** with their TESDA username/password
2. **Identity Center authenticates** against Active Directory
3. **System checks permissions** based on their role/department
4. **Access granted** to appropriate AWS accounts and applications
5. **All actions logged** for compliance and auditing

## 🚀 Step-by-Step Implementation for TESDA

### Step 1: Enable AWS IAM Identity Center

#### Prerequisites Check
```
✅ AWS Organizations enabled
✅ Management account access
✅ Appropriate permissions (OrganizationAccountAccessRole)
✅ Decision on identity source (AD vs built-in)
```

#### Enable Process
```
1. Go to AWS Console → Search "IAM Identity Center"
2. Click "Enable IAM Identity Center"
3. Choose your region (recommend: Asia Pacific - Singapore)
4. Confirm organization integration
5. Wait for setup completion (2-3 minutes)
```

### Step 2: Configure Identity Source

#### Option A: Built-in Identity Store (Simple Start)
**Best for:** Small organizations or pilot projects
**Pros:** Quick setup, no external dependencies
**Cons:** Manual user management, no integration with existing systems

```
Setup Steps:
1. Identity Center → Settings → Identity source
2. Select "Identity Center directory"
3. Create users manually in the console
4. Manage groups and permissions in AWS
```

#### Option B: Active Directory Integration (Recommended for TESDA)
**Best for:** Organizations with existing AD infrastructure
**Pros:** Leverages existing user database, automatic sync
**Cons:** Requires network connectivity and AD configuration

```
Setup Steps:
1. Identity Center → Settings → Identity source
2. Select "Active Directory"
3. Choose connection method:
   - AWS Managed Microsoft AD
   - Self-managed Active Directory
4. Configure directory connection
5. Test connectivity and sync
```

#### Option C: External Identity Provider
**Best for:** Organizations using Google Workspace, Azure AD, etc.
**Examples:** Google Workspace, Microsoft Azure AD, Okta

### Step 3: Create Permission Sets (Role Templates)

#### What are Permission Sets?
Permission sets are like job descriptions that define what people can do in AWS accounts.

#### TESDA Permission Sets Design

##### Administrative Permission Sets

**TESDA-SuperAdmin**
```json
{
  "Name": "TESDA-SuperAdmin",
  "Description": "Full administrative access for IT Directors",
  "ManagedPolicies": [
    "arn:aws:iam::aws:policy/AdministratorAccess"
  ],
  "SessionDuration": "PT4H",
  "RequireMFA": true
}
```

**TESDA-ITAdmin**
```json
{
  "Name": "TESDA-ITAdmin",
  "Description": "Infrastructure management for IT staff",
  "ManagedPolicies": [
    "arn:aws:iam::aws:policy/PowerUserAccess"
  ],
  "InlinePolicy": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Deny",
        "Action": [
          "iam:CreateUser",
          "iam:DeleteUser",
          "organizations:*"
        ],
        "Resource": "*"
      }
    ]
  }
}
```

##### Functional Permission Sets

**TESDA-Developer**
```json
{
  "Name": "TESDA-Developer",
  "Description": "Development access for application developers",
  "ManagedPolicies": [
    "arn:aws:iam::aws:policy/AmazonEC2FullAccess",
    "arn:aws:iam::aws:policy/AmazonS3FullAccess",
    "arn:aws:iam::aws:policy/AmazonRDSFullAccess"
  ],
  "InlinePolicy": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Deny",
        "Action": "*",
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "aws:RequestedRegion": ["ap-southeast-1"]
          }
        }
      }
    ]
  }
}
```

**TESDA-ReadOnly**
```json
{
  "Name": "TESDA-ReadOnly",
  "Description": "View-only access for auditors and managers",
  "ManagedPolicies": [
    "arn:aws:iam::aws:policy/ReadOnlyAccess"
  ],
  "SessionDuration": "PT8H"
}
```

**TESDA-LMS-Manager**
```json
{
  "Name": "TESDA-LMS-Manager",
  "Description": "LMS management for curriculum staff",
  "InlinePolicy": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ],
        "Resource": [
          "arn:aws:s3:::tesda-lms-content/*",
          "arn:aws:s3:::tesda-student-files/*"
        ]
      },
      {
        "Effect": "Allow",
        "Action": [
          "rds:DescribeDBInstances",
          "rds:DescribeDBClusters"
        ],
        "Resource": "*"
      }
    ]
  }
}
```

### Step 4: Assign Users/Groups to Accounts

#### Create Groups Based on TESDA Structure

**IT Department Groups**
```
TESDA-IT-Directors → TESDA-SuperAdmin permission set
TESDA-IT-Managers → TESDA-ITAdmin permission set
TESDA-IT-Support → TESDA-Developer permission set
```

**Academic Department Groups**
```
TESDA-Curriculum-Managers → TESDA-LMS-Manager permission set
TESDA-Instructors → TESDA-ReadOnly permission set
TESDA-Assessment-Staff → TESDA-LMS-Manager permission set
```

**Administrative Groups**
```
TESDA-Finance → TESDA-ReadOnly permission set (billing accounts only)
TESDA-Auditors → TESDA-ReadOnly permission set (all accounts)
TESDA-Executives → TESDA-ReadOnly permission set (dashboard access)
```

#### Assignment Process
```
1. Identity Center → AWS accounts
2. Select target account (e.g., Production)
3. Click "Assign users or groups"
4. Select group (e.g., TESDA-IT-Managers)
5. Select permission set (e.g., TESDA-ITAdmin)
6. Click "Submit"
7. Repeat for all account/group combinations
```

### Step 5: Configure MFA Requirements

#### MFA Policy Configuration
```
1. Identity Center → Settings → Authentication
2. Configure MFA settings:
   - Require MFA: Always
   - MFA types: Authenticator apps, Security keys
   - Remember device: 30 days
   - Emergency access: Configure break-glass procedure
```

#### Supported MFA Methods
```
✅ Authenticator Apps: Google Authenticator, Authy, Microsoft Authenticator
✅ Hardware Tokens: YubiKey, RSA SecurID
✅ SMS (not recommended for high security)
✅ Built-in authenticators: Windows Hello, Touch ID
```

## 🔗 TESDA Integration Scenarios

### Scenario 1: New Employee Onboarding
```
1. HR adds Maria Santos to Active Directory
2. Maria is added to "TESDA-Curriculum-Staff" AD group
3. Identity Center automatically syncs the change
4. Maria gets access to appropriate AWS accounts
5. Maria logs in with her TESDA credentials
6. System prompts for MFA setup on first login
7. Maria can now access LMS development tools
```

### Scenario 2: Employee Role Change
```
1. John moves from IT Support to IT Manager
2. HR updates his AD group membership
3. Identity Center syncs the change within 1 hour
4. John's AWS access automatically updates
5. Old permissions removed, new permissions added
6. All changes logged for audit purposes
```

### Scenario 3: Employee Departure
```
1. HR disables Maria's AD account
2. Identity Center detects the change
3. All AWS access immediately revoked
4. Active sessions terminated within 15 minutes
5. Audit log shows exact time of access removal
```

## 🔍 Real-World Benefits for TESDA

### Security Improvements
- **Reduced Attack Surface:** Fewer passwords to compromise
- **Faster Incident Response:** Centralized access control
- **Better Compliance:** Complete audit trails
- **Consistent Policies:** Same security rules everywhere

### Operational Efficiency
- **Reduced Help Desk:** 60% fewer password reset requests
- **Faster Onboarding:** New staff productive on day one
- **Easier Offboarding:** One click to remove all access
- **Simplified Management:** Manage 1000s of users from one console

### Cost Savings
- **Reduced Administrative Overhead:** Less time managing access
- **Improved Productivity:** Staff spend more time on core work
- **Better Resource Utilization:** Clear visibility into who uses what
- **Compliance Efficiency:** Automated audit report generation

## ✅ Implementation Checklist

### Pre-Implementation (Week 1)
- [ ] Document current user access patterns
- [ ] Map TESDA organizational structure to AWS accounts
- [ ] Plan permission set design
- [ ] Prepare Active Directory integration
- [ ] Communicate changes to staff

### Implementation (Week 2)
- [ ] Enable IAM Identity Center
- [ ] Configure identity source (AD integration)
- [ ] Create permission sets
- [ ] Test with pilot group (5-10 users)
- [ ] Validate MFA setup process

### Rollout (Week 3-4)
- [ ] Migrate department by department
- [ ] Provide user training sessions
- [ ] Monitor for issues and feedback
- [ ] Adjust permission sets as needed
- [ ] Document final configuration

### Post-Implementation (Ongoing)
- [ ] Monthly access reviews
- [ ] Quarterly permission set audits
- [ ] Annual security assessments
- [ ] Continuous user training
- [ ] Regular backup of configuration

## 🔧 Troubleshooting Common Issues

### AD Sync Problems
```
Problem: Users not appearing in Identity Center
Solutions:
1. Check AD connector status
2. Verify network connectivity
3. Confirm AD group membership
4. Force manual sync
5. Check Identity Center logs
```

### Permission Issues
```
Problem: User can't access expected resources
Solutions:
1. Verify permission set assignment
2. Check account assignment
3. Confirm MFA completion
4. Review session duration limits
5. Check for conflicting policies
```

### MFA Setup Problems
```
Problem: Users can't set up MFA
Solutions:
1. Verify MFA is enabled in settings
2. Check supported MFA types
3. Provide clear setup instructions
4. Test with different authenticator apps
5. Configure emergency access procedures
```

## 📚 Next Steps
Continue to [Module 6: Hands-On Lab - User/Role Creation & MFA](../06-iam-lab/README.md)

---
**💡 Pro Tip:** Start with a pilot group of 5-10 technical users to validate the setup before rolling out to the entire organization.
