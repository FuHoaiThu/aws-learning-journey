# IAM Role

- Some AWS service will need to perform actions on your behalf. To do so, we will assign permissions to AWS services with IAM Roles.
- Common roles:
  EC2 Instance Roles
  Lambda Function Roles
  Roles for CloudFormation
- IAM Role provides temporary credentials instead of long-term credentials.

## Handon

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
