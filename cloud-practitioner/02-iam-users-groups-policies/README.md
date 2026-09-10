# IAM Users, Groups and Policies

## Overview

This hands-on lab focuses on AWS IAM Users, User Groups, and Policies.

The goal is to understand how permissions are assigned to IAM users through groups and policies, and how AWS evaluates Allow and Deny permissions.

## What I Practiced

- Created an IAM User Group named `Developers`
- Created an IAM User named `dev-user`
- Added `dev-user` to the `Developers` group
- Tested IAM access without permissions
- Attached the AWS managed policy `IAMReadOnlyAccess`
- Configured MFA permissions for the IAM user
- Created a Customer Managed Policy
- Applied the principle of least privilege
- Tested Implicit Deny and Explicit Deny

## Permission Flow

```text
dev-user
   ↓
Developers
   ├── IAMReadOnlyAccess
   ├── AllowSelfManageMFA
   ├── AllowCreateTestGroups
   └── DenyCreateTestAdminGroup
```

## Custom Policy Example

The following policy allows users in the `Developers` group to create IAM groups whose names start with `Test`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:CreateGroup",
      "Resource": "arn:aws:iam::*:group/Test*"
    }
  ]
}
```

## Explicit Deny Example

The following policy prevents the creation of the `TestAdmin` group.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "iam:CreateGroup",
      "Resource": "arn:aws:iam::*:group/TestAdmin"
    }
  ]
}
```

Even though `TestAdmin` matches the `Test*` Allow policy, the Explicit Deny takes precedence.

## Key Takeaways

- IAM Users represent individual identities.
- IAM Groups make it easier to manage permissions for multiple users.
- Policies define what actions are allowed or denied.
- Users inherit permissions from the groups they belong to.
- Without an Allow permission, access is implicitly denied.
- Explicit Deny overrides Allow.
- Permissions should follow the principle of least privilege.
- AWS Managed Policies are created and maintained by AWS.
- Customer Managed Policies are created and managed within an AWS account.

## Result

Successfully practiced IAM permission management using Users, Groups, AWS Managed Policies, Customer Managed Policies, Implicit Deny, Explicit Deny, and least privilege.
