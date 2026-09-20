# AWS IAM Roles — Theory & Hands-on Lab

## 1. Learning objectives

- Understand why AWS services use IAM roles.
- Distinguish IAM users, roles, trust policies, and permissions policies.
- Create an EC2 service role and inspect it using AWS CLI.
- Attach and detach a managed policy, verify the result, and clean up.
- Avoid launching billable compute resources during this lab.

## 2. IAM Roles: theory

An IAM role is an AWS identity with permissions that trusted principals can assume. Roles are commonly used to provide temporary credentials to workloads without embedding long-term access keys in source code.

Common examples:

| Role                        | Example purpose                                                                 |
| --------------------------- | ------------------------------------------------------------------------------- |
| EC2 instance role           | Let applications on EC2 call AWS APIs.                                          |
| Lambda execution role       | Let a Lambda function access required AWS services and write logs.              |
| CloudFormation service role | Let CloudFormation manage resources using the permissions assigned to the role. |

### Trust policy versus permissions policy

- **Trust policy:** Who can assume the role? For this lab, the trusted principal is `ec2.amazonaws.com`.
- **Permissions policy:** Which actions can the role perform, and on which resources?

Example EC2 trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

When an EC2 instance uses an instance profile associated with an IAM role, applications can obtain temporary role credentials through the EC2 credential provider chain. Do not hard-code access keys. Grant only the actions and resources required by the workload (least privilege).

> `AmazonS3ReadOnlyAccess` is a broad AWS-managed policy. It was used only to inspect attachment and detachment in this lab. For a real workload, prefer a narrowly scoped policy for the specific bucket and actions required.

## 3. Lab: EC2-S3-ReadOnly-Lab (IAM configuration only)

**Cost-conscious scope:** Created and managed an IAM role only. No EC2 instance or S3 bucket was created for this lab. IAM role/policy management itself has no additional charge, but unrelated resources in the AWS account may still incur charges.

### Step 1 — Create the role in AWS Console

1. Open **IAM → Roles → Create role**.
2. Select **AWS service → EC2**.
3. Attach the AWS-managed policy `AmazonS3ReadOnlyAccess`.
4. Name the role `EC2-S3-ReadOnly-Lab` and create it.
5. Inspect **Trust relationships** to verify `Principal.Service` is `ec2.amazonaws.com`.

### Step 2 — Select the existing CLI profile

The credentials were configured in the named profile `aws-lab`, not the default profile. Without `--profile aws-lab`, the CLI reported `Unable to locate credentials`.

```powershell
aws sts get-caller-identity --profile aws-lab
```

Optional: select this profile for the current PowerShell session:

```powershell
$env:AWS_PROFILE = "aws-lab"
```

Never commit access keys, secret keys, session tokens, or local AWS credentials files to GitHub.

### Step 3 — Inspect role and trust policy

```powershell
aws iam get-role --role-name EC2-S3-ReadOnly-Lab --profile aws-lab
```

The response includes role metadata and `AssumeRolePolicyDocument`. **It does not list attached permissions policies.**

### Step 4 — List attached managed policies

```powershell
aws iam list-attached-role-policies --role-name EC2-S3-ReadOnly-Lab --profile aws-lab
```

Expected while the managed policy is attached:

```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "AmazonS3ReadOnlyAccess",
      "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
    }
  ]
}
```

To list **inline policy names** instead:

```powershell
aws iam list-role-policies --role-name EC2-S3-ReadOnly-Lab --profile aws-lab
```

### Step 5 — Detach and verify

In **IAM → Roles → EC2-S3-ReadOnly-Lab → Permissions**, remove `AmazonS3ReadOnlyAccess` from the role. Do not attempt to delete the AWS-managed policy itself.

Verify:

```powershell
aws iam list-attached-role-policies --role-name EC2-S3-ReadOnly-Lab --profile aws-lab
```

Observed result:

```json
{
  "AttachedPolicies": []
}
```

This confirms no managed policies are attached directly to the role. It does **not** prove that the role has no effective permissions from any other source. Detaching the policy does not delete the role or change its trust policy.

### Step 6 — Cleanup

After ensuring the role is not needed and has no remaining dependent resources or policies:

```powershell
aws iam delete-role --role-name EC2-S3-ReadOnly-Lab --profile aws-lab
aws iam get-role --role-name EC2-S3-ReadOnly-Lab --profile aws-lab
```

Expected verification after successful deletion: `NoSuchEntity`.

**Status:** Role deletion was requested as the final cleanup step; record it as verified only after observing `NoSuchEntity` in your terminal.

## 4. CLI command reference

| Command                                                                                 | Purpose                                  |
| --------------------------------------------------------------------------------------- | ---------------------------------------- |
| `aws sts get-caller-identity --profile aws-lab`                                         | Identify the current CLI identity.       |
| `aws iam get-role --role-name EC2-S3-ReadOnly-Lab --profile aws-lab`                    | Read role metadata and trust policy.     |
| `aws iam list-attached-role-policies --role-name EC2-S3-ReadOnly-Lab --profile aws-lab` | List directly attached managed policies. |
| `aws iam list-role-policies --role-name EC2-S3-ReadOnly-Lab --profile aws-lab`          | List inline policy names.                |
| `aws iam delete-role --role-name EC2-S3-ReadOnly-Lab --profile aws-lab`                 | Delete the role after cleanup.           |

## 5. Key takeaways

1. **Trust policy = who can assume the role.**
2. **Permissions policy = what the role can do.**
3. `get-role` displays trust policy, not the list of attached managed policies.
4. `list-attached-role-policies` displays directly attached managed policies; `list-role-policies` displays inline policy names.
5. Detaching a permissions policy does not delete the role or alter its trust policy.
6. Prefer roles and temporary credentials for AWS workloads instead of hard-coded long-term access keys.
7. Follow least privilege: avoid broad managed policies in production when narrower permissions suffice.

## 6. Lab limitations and next step

This was an **IAM configuration lab**, not a runtime access test. We did not launch EC2, associate an instance profile, call S3 from an EC2 instance, or demonstrate an actual S3 `AccessDenied`/successful read. Those require a separate lab after checking Free Tier eligibility and potential EC2, EBS, IPv4, and S3 charges.
