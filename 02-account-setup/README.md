# Module 2: Account Setup & Billing Consolidation
**Duration:** 20 minutes  
**Level:** Cloud Practitioner

## 🎯 Learning Objectives
By the end of this module, you will:
- Understand the step-by-step account creation process
- Know how to implement billing best practices
- Be able to configure basic security controls

## 🔧 What is an AWS Account?

An AWS account is like your personal workspace in the cloud. Each account has its own resources, users, and billing.

## 📋 Step-by-Step Account Creation Process

**Watch:** [How do I create and activate a new AWS account?](https://www.youtube.com/watch?v=lIdh92JmWtg)

### Step 1: Root Account Setup
**What to do:**
- Use a company email (not personal)
- Create a strong password (12+ characters)
- This account has "super admin" powers

**Best Practices:**
```
✅ Use: tesda-aws-root@tesda.gov.ph
❌ Avoid: john.doe@gmail.com

✅ Password: T3sda@AWS2024!SecureP@ss
❌ Avoid: password123
```

### Step 2: Enable MFA Immediately
**What is MFA?**
- MFA = Multi-Factor Authentication
- Like having two locks on your door
- Use Google Authenticator or similar app

**Why Critical:**
- Password alone = 1 factor (something you know)
- MFA = Password + Phone/App = 2 factors
- Reduces security breaches by 99.9%

**Implementation Steps:**
1. Go to IAM Console → Users → Root User
2. Click "Assign MFA Device"
3. Choose "Virtual MFA Device"
4. Scan QR code with authenticator app
5. Enter two consecutive codes

### Step 3: Create Organizational Units (OUs)
**What are OUs?**
- Think of OUs as folders for organizing accounts
- Apply policies to multiple accounts at once

**TESDA OU Structure:**
```
Root Organization
├── Core-OU (Essential services)
├── Production-OU (Live systems)
├── Non-Production-OU (Dev/Test)
└── Sandbox-OU (Learning/Experimentation)
```

### Step 4: Create/Invite Member Accounts
**Options:**
- **Create New Account:** AWS creates it for you
- **Invite Existing Account:** Add existing AWS account to organization

**Naming Convention:**
```
tesda-security-prod
tesda-lms-prod
tesda-lms-dev
tesda-shared-services
```

### Step 5: Configure Consolidated Billing
**Benefits:**
- All accounts share one payment method
- Get volume discounts across all accounts
- Simplified financial management

## 💰 Billing Best Practices

### Cost Allocation Tags
**Purpose:** Label resources by department/project

**Example Tags:**
```json
{
  "Department": "Curriculum-Development",
  "Project": "LMS-Upgrade",
  "Environment": "Production",
  "CostCenter": "TESDA-IT-001"
}
```

### Budget Alerts
**Setup Process:**
1. Go to AWS Billing Console
2. Create Budget → Cost Budget
3. Set monthly limit (e.g., ₱50,000)
4. Configure alerts at 80%, 90%, 100%
5. Add email notifications

### Reserved Instance Sharing
**What it means:**
- Buy capacity in advance for discounts
- Share discounts across all accounts in organization
- Save 30-60% on compute costs

### Detailed Billing Reports
**Available Reports:**
- Cost and Usage Reports (CUR)
- Daily spend by service
- Monthly cost allocation by tags
- Forecasting and trends

## 🔒 Security Controls

### Service Control Policies (SCPs)
**Purpose:** Rules that prevent dangerous actions

**Example SCP - Prevent Account Closure:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "account:CloseAccount"
      ],
      "Resource": "*"
    }
  ]
}
```

### Cross-Account Roles
**Purpose:** Safe way to access other accounts

**Use Cases:**
- Security team accessing all accounts for monitoring
- Developers accessing production for troubleshooting
- Auditors reviewing configurations

### CloudTrail Organization Trail
**What it does:**
- Records every action taken (audit log)
- Centralized logging across all accounts
- Required for compliance

**Setup:**
1. Enable in Security Account
2. Configure S3 bucket for log storage
3. Set up log file validation
4. Configure CloudWatch integration

### Config Organization Rules
**Purpose:** Automatically check if settings are correct

**Example Rules:**
- Ensure all S3 buckets are encrypted
- Verify MFA is enabled for all users
- Check that security groups don't allow unrestricted access

## 🏛️ TESDA-Specific Considerations

### Government Compliance
- Regulatory Requirements
- Audit trail requirements
- Access control documentation
- Regular security assessments

### Budget Management
- Quarterly budget reviews
- Department-wise cost allocation
- Training program cost tracking
- ROI measurement for cloud adoption

### Integration Planning
- Connection to existing TESDA systems
- Active Directory integration
- Legacy system migration planning
- Disaster recovery requirements

## ✅ Implementation Checklist

### Immediate Actions (Week 1)
- [ ] Create root account with strong credentials
- [ ] Enable MFA on root account
- [ ] Set up basic OU structure
- [ ] Configure consolidated billing
- [ ] Create first member account

### Security Setup (Week 2)
- [ ] Enable CloudTrail organization trail
- [ ] Configure basic SCPs
- [ ] Set up cross-account roles
- [ ] Implement Config rules
- [ ] Document access procedures

### Ongoing Management
- [ ] Monthly billing reviews
- [ ] Quarterly access reviews
- [ ] Annual security assessments
- [ ] Regular backup testing
- [ ] Staff training updates

## 📚 Next Steps
Continue to [Module 3: Hands-On Lab - Multi-Account Setup](../03-hands-on-lab/README.md)

---
**💡 Pro Tip:** Always test new configurations in a development account before applying to production.
