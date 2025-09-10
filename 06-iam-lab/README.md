# Module 6: Hands-On Lab - User/Role Creation & MFA
**Duration:** 10 minutes  
**Level:** Cloud Practitioner

## 🎯 Lab Objectives
By the end of this lab, you will have:
- Created IAM users with MFA enforcement
- Set up cross-account roles
- Implemented a comprehensive MFA policy
- Tested the complete authentication flow

## 🔐 What We'll Build Together

A complete security setup that TESDA can use immediately, including:
- Test users with MFA requirements
- Cross-account access roles
- Automated MFA enforcement policy
- Emergency access procedures

## 🚀 Lab Exercise 1: Create IAM Users with MFA

### Why MFA is Critical for TESDA
```
Statistics:
- Password alone = 1 factor (something you know)
- MFA = Password + Phone/App = 2 factors
- Reduces security breaches by 99.9%
- Required for government compliance
```

### Step 1: Create Test User
```
1. Go to IAM Console → Users → Create User
2. User name: "tesda-pilot-user"
3. ✅ Enable console access
4. ✅ Users must create a new password at next sign-in
5. Set temporary password: "TempPass123!"
6. Click "Next"
7. Skip permissions for now (we'll add via groups)
8. Click "Create user"
```

### Step 2: Create User Group
```
1. Go to IAM Console → User groups → Create group
2. Group name: "TESDA-Pilot-Users"
3. Skip policy attachment for now
4. Click "Create group"
5. Select the group → Add users
6. Add "tesda-pilot-user" to the group
```

### Step 3: Test Initial Login
```
1. Copy the console sign-in URL
2. Open incognito/private browser window
3. Log in as "tesda-pilot-user"
4. Use temporary password "TempPass123!"
5. Set new password when prompted
6. Note: Limited access (no policies attached yet)
```

### Step 4: Set Up Virtual MFA Device
```
1. While logged in as test user:
2. Go to IAM → Users → tesda-pilot-user
3. Click "Security credentials" tab
4. Click "Assign MFA device"
5. Choose "Virtual MFA device"
6. Download Google Authenticator on your phone
7. Scan the QR code with the app
8. Enter two consecutive MFA codes
9. Click "Assign MFA"
```

### Step 5: Test MFA Login
```
1. Log out of the console
2. Log back in as "tesda-pilot-user"
3. Enter username and password
4. System should prompt for MFA code
5. Open Google Authenticator
6. Enter the 6-digit code
7. Verify successful login
```

## 🔄 Lab Exercise 2: Create Cross-Account Roles

### Scenario
Security team in Security Account needs to monitor Production Account safely.

### Step 1: Create Role in Production Account
```
1. Switch to Production Account (or simulate)
2. Go to IAM → Roles → Create role
3. Select "AWS account" as trusted entity
4. Enter Security Account ID: 123456789012 (use your actual ID)
5. ✅ Require MFA
6. Click "Next"
```

### Step 2: Attach Permissions
```
1. Search for "ReadOnlyAccess"
2. ✅ Select "ReadOnlyAccess" policy
3. Click "Next"
4. Role name: "SecurityTeamMonitoringRole"
5. Description: "Allows security team read-only access for monitoring"
6. Click "Create role"
```

### Step 3: Note the Role ARN
```
Copy this ARN for later use:
arn:aws:iam::PRODUCTION-ACCOUNT-ID:role/SecurityTeamMonitoringRole
```

### Step 4: Test Role Assumption (Simulation)
```
1. Go to Security Account
2. Create user: "security-analyst"
3. Attach policy allowing sts:AssumeRole
4. Test assuming the cross-account role
```

## 🛡️ Lab Exercise 3: Implement MFA Enforcement Policy

### The Challenge
How do we force ALL users to use MFA without locking them out during setup?

### Our Smart Solution
Create a policy that:
- ✅ Allows users to log in without MFA initially
- ✅ Allows users to set up their MFA device
- ❌ Blocks all other actions until MFA is configured
- ✅ Provides full access once MFA is active

### Step 1: Create the MFA Enforcement Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowViewAccountInfo",
            "Effect": "Allow",
            "Action": [
                "iam:GetAccountPasswordPolicy",
                "iam:GetAccountSummary",
                "iam:ListVirtualMFADevices"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowManageOwnPasswords",
            "Effect": "Allow",
            "Action": [
                "iam:ChangePassword",
                "iam:GetUser"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnMFA",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:ListMFADevices",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice"
            ],
            "Resource": [
                "arn:aws:iam::*:mfa/${aws:username}",
                "arn:aws:iam::*:user/${aws:username}"
            ]
        },
        {
            "Sid": "AllowListActions",
            "Effect": "Allow",
            "Action": [
                "iam:ListUsers",
                "iam:ListVirtualMFADevices"
            ],
            "Resource": "*"
        },
        {
            "Sid": "BlockMostAccessUnlessSignedInWithMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice",
                "sts:GetSessionToken",
                "iam:ChangePassword",
                "iam:GetAccountPasswordPolicy",
                "iam:GetAccountSummary"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

### Step 2: Create the Policy in AWS
```
1. Go to IAM Console → Policies → Create policy
2. Click "JSON" tab
3. Delete existing content
4. Copy and paste the JSON above
5. Click "Next: Tags" (skip tags)
6. Click "Next: Review"
7. Policy name: "TESDA-Force-MFA-Policy"
8. Description: "Enforces MFA for all user actions"
9. Click "Create policy"
```

### Step 3: Attach Policy to Test Group
```
1. Go to IAM Console → User groups
2. Select "TESDA-Pilot-Users" group
3. Click "Permissions" tab
4. Click "Add permissions" → "Attach policies"
5. Search for "TESDA-Force-MFA-Policy"
6. ✅ Select the policy
7. Click "Add permissions"
```

### Step 4: Test MFA Enforcement
```
1. Create another test user: "tesda-test-user-2"
2. Add to "TESDA-Pilot-Users" group
3. Log in as this user (without MFA setup)
4. Try to access EC2 console → Should be blocked
5. Try to access S3 console → Should be blocked
6. Go to IAM → Security credentials → Should work
7. Set up MFA device → Should work
8. After MFA setup, try EC2/S3 again → Should work
```

## 🔧 Lab Exercise 4: Create Emergency Access Procedure

### Why Emergency Access?
- What if MFA device is lost?
- What if Identity Center is down?
- How to handle urgent production issues?

### Step 1: Create Break-Glass Role
```
1. Go to IAM Console → Roles → Create role
2. Select "AWS account" as trusted entity
3. Enter current account ID
4. ✅ Require MFA (even for emergency)
5. Click "Next"
6. Attach "AdministratorAccess" policy
7. Role name: "TESDA-Emergency-Access"
8. Add description: "Emergency access for critical incidents"
```

### Step 2: Create Emergency Access Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::*:role/TESDA-Emergency-Access",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
```

### Step 3: Assign to Emergency Contacts
```
1. Create policy: "TESDA-Emergency-Access-Policy"
2. Create group: "TESDA-Emergency-Contacts"
3. Add 2-3 senior IT staff to the group
4. Attach the emergency access policy
5. Document the procedure for staff
```

## ✅ Lab Verification Checklist

### User and MFA Setup
- [ ] Test user created successfully
- [ ] MFA device assigned and working
- [ ] Login with MFA successful
- [ ] User can change their own password

### MFA Enforcement Policy
- [ ] Policy created and attached to group
- [ ] Users without MFA are blocked from most actions
- [ ] Users can still set up MFA when blocked
- [ ] Full access granted after MFA setup
- [ ] Policy doesn't interfere with normal operations

### Cross-Account Access
- [ ] Cross-account role created
- [ ] Role can be assumed from trusted account
- [ ] MFA required for role assumption
- [ ] Appropriate permissions granted

### Emergency Procedures
- [ ] Break-glass role created
- [ ] Emergency access policy implemented
- [ ] Emergency contacts identified and trained
- [ ] Procedure documented and tested

## 🎯 Real-World Application

This lab setup provides:
- **Production-Ready Security:** Can be deployed immediately
- **Scalable Design:** Works for 10 or 10,000 users
- **Compliance-Ready:** Meets government security requirements
- **User-Friendly:** Doesn't interfere with daily work

## 🔧 Troubleshooting Guide

### MFA Setup Issues
```
Problem: QR code won't scan
Solutions:
1. Try different authenticator app
2. Use manual entry instead of QR code
3. Check phone camera permissions
4. Ensure good lighting for QR scan
```

### Policy Not Working
```
Problem: Users still have access without MFA
Solutions:
1. Check policy syntax (use JSON validator)
2. Verify policy is attached to correct group
3. Wait 5-10 minutes for policy propagation
4. Check for conflicting policies
5. Test with fresh browser session
```

### Role Assumption Fails
```
Problem: Can't assume cross-account role
Solutions:
1. Verify account ID in trust policy
2. Check MFA requirement is met
3. Confirm user has sts:AssumeRole permission
4. Verify role ARN is correct
5. Check for typos in account numbers
```

## 📊 Success Metrics

After implementing this lab setup, you should see:
- **100% MFA adoption** within 30 days
- **Reduced password reset requests** by 60%
- **Faster incident response** with cross-account access
- **Improved compliance scores** in security audits
- **Better user satisfaction** with streamlined access

## 📚 Next Steps
Continue to [Module 7: Security Architecture Principles](../07-security-architecture/README.md)

---
**💡 Lab Tip:** Always test policies with a small group before rolling out organization-wide. This prevents accidentally locking out all users.
