# Module 3: Hands-On Lab - Multi-Account Setup
**Duration:** 15 minutes  
**Level:** Cloud Practitioner

## 🎯 Lab Objectives
By the end of this lab, you will have:
- Created a complete AWS Organization
- Set up organizational units for TESDA
- Created member accounts
- Applied basic security policies
- Configured consolidated billing

## 🏗️ What We'll Build Together

A complete multi-account structure for TESDA that you can use in real projects.

### Target Architecture
```
TESDA-Root (Main Organization)
├── Core-OU (Essential Services)
│   ├── Security-Account (All security tools)
│   └── Shared-Services-Account (Common resources)
├── Production-OU (Live Systems)
│   ├── LMS-Production (Student learning platform)
│   └── Assessment-Production (Testing and certification)
└── Non-Production-OU (Development & Testing)
    ├── Development-Account (Developers work here)
    └── Testing-Account (Test new features safely)
```

## 🚀 Lab Exercise 1: Create AWS Organization

### Prerequisites
- AWS account with administrative access
- MFA enabled on root account
- Billing information configured

### Step-by-Step Instructions

#### 1. Access AWS Organizations
```
1. Log into AWS Console
2. Search for "Organizations" in the services search
3. Click "AWS Organizations"
4. Click "Create an organization"
```

#### 2. Choose Organization Type
```
✅ Select: "Enable all features"
❌ Avoid: "Enable only consolidated billing"

Why? Full features give you SCPs and advanced management.
```

#### 3. Verify Organization Creation
```
Expected Result:
- Organization ID starts with "o-"
- Root account is listed as management account
- Billing is automatically consolidated
```

## 🗂️ Lab Exercise 2: Create Organizational Units

### Create Core-OU
```
1. In Organizations console, click "Organize accounts"
2. Click "Create organizational unit"
3. Name: "Core-OU"
4. Description: "Essential services and security"
5. Click "Create organizational unit"
```

### Create Production-OU
```
1. Click "Create organizational unit"
2. Name: "Production-OU"
3. Description: "Live systems serving students"
4. Click "Create organizational unit"
```

### Create Non-Production-OU
```
1. Click "Create organizational unit"
2. Name: "Non-Production-OU"
3. Description: "Development and testing environments"
4. Click "Create organizational unit"
```

### Verify OU Structure
```
Expected Structure:
Root
├── Core-OU
├── Production-OU
└── Non-Production-OU
```

## 🏢 Lab Exercise 3: Create Member Accounts

### Create Security Account
```
1. Click "Add an AWS account"
2. Select "Create an AWS account"
3. Account name: "TESDA-Security"
4. Email: tesda-security@tesda.gov.ph
5. IAM role name: "OrganizationAccountAccessRole"
6. Click "Create AWS account"
```

### Create LMS Production Account
```
1. Click "Add an AWS account"
2. Select "Create an AWS account"
3. Account name: "TESDA-LMS-Production"
4. Email: tesda-lms-prod@tesda.gov.ph
5. IAM role name: "OrganizationAccountAccessRole"
6. Click "Create AWS account"
```

### Create Development Account
```
1. Click "Add an AWS account"
2. Select "Create an AWS account"
3. Account name: "TESDA-Development"
4. Email: tesda-dev@tesda.gov.ph
5. IAM role name: "OrganizationAccountAccessRole"
6. Click "Create AWS account"
```

### Move Accounts to Appropriate OUs
```
1. Select "TESDA-Security" account
2. Click "Move" → Select "Core-OU"
3. Select "TESDA-LMS-Production" account
4. Click "Move" → Select "Production-OU"
5. Select "TESDA-Development" account
6. Click "Move" → Select "Non-Production-OU"
```

## 🔒 Lab Exercise 4: Configure Basic SCPs

### Create Prevent Account Closure Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PreventAccountClosure",
      "Effect": "Deny",
      "Action": [
        "account:CloseAccount"
      ],
      "Resource": "*"
    }
  ]
}
```

### Apply Policy Steps
```
1. Go to "Policies" in Organizations console
2. Click "Create policy"
3. Name: "PreventAccountClosure"
4. Description: "Prevents accidental account closure"
5. Copy and paste the JSON above
6. Click "Create policy"
7. Select the policy
8. Click "Attach" → Select "Root" OU
9. Click "Attach policy"
```

### Create Development Restrictions Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RestrictExpensiveServices",
      "Effect": "Deny",
      "Action": [
        "redshift:*",
        "sagemaker:*",
        "databrew:*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Apply to Non-Production OU
```
1. Create policy named "DevelopmentRestrictions"
2. Attach to "Non-Production-OU"
3. Verify policy is applied to development accounts
```

## 💰 Lab Exercise 5: Verify Consolidated Billing

### Check Billing Console
```
1. Go to AWS Billing and Cost Management
2. Click "Bills"
3. Verify you see charges from all accounts
4. Check "Consolidated Billing" section
5. Confirm all accounts are listed
```

### Set Up Budget Alert
```
1. Go to "Budgets" in Billing console
2. Click "Create budget"
3. Budget type: "Cost budget"
4. Budget name: "TESDA-Monthly-Budget"
5. Amount: ₱50,000 (adjust as needed)
6. Add email alerts at 80%, 90%, 100%
7. Click "Create budget"
```

## ✅ Lab Verification Checklist

### Organization Structure
- [ ] AWS Organization created successfully
- [ ] Three OUs created (Core, Production, Non-Production)
- [ ] Member accounts created and moved to appropriate OUs
- [ ] Account naming follows convention

### Security Policies
- [ ] Prevent Account Closure SCP applied to Root
- [ ] Development Restrictions SCP applied to Non-Production OU
- [ ] Policies are active and enforced

### Billing Configuration
- [ ] Consolidated billing is active
- [ ] All accounts appear in billing console
- [ ] Budget alert configured
- [ ] Email notifications set up

## 🎯 Real-World Application

This lab setup provides:
- **Immediate Use:** Can be deployed in TESDA's production environment
- **Scalability:** Easy to add more accounts as needed
- **Security:** Basic protections against common mistakes
- **Cost Control:** Centralized billing and budget monitoring

## 🔧 Troubleshooting Common Issues

### Account Creation Fails
```
Problem: Email already in use
Solution: Use unique email for each account
Example: tesda-lms-prod-001@tesda.gov.ph
```

### SCP Not Working
```
Problem: Policy doesn't seem to apply
Solution: 
1. Check policy syntax
2. Verify attachment to correct OU
3. Wait 5-10 minutes for propagation
```

### Billing Not Consolidated
```
Problem: Accounts show separate bills
Solution:
1. Verify accounts are in organization
2. Check organization has "All features" enabled
3. Contact AWS support if issue persists
```

## 📚 Next Steps
Continue to [Module 4: IAM Advanced Concepts & Best Practices](../04-iam-concepts/README.md)

---
**💡 Lab Tip:** Save your account IDs and OU IDs in a secure document for future reference.
