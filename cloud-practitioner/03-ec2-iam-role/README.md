# EC2 IAM Role Hands-on

## 1. Overview

An **IAM Role** allows an EC2 instance to access AWS services without storing long-term Access Keys and Secret Access Keys directly on the instance.

```text
EC2 Instance
     |
     | Assume Role
     v
IAM Role
     |
     | Temporary Credentials
     v
AWS Services
```

This is safer than manually configuring credentials on an EC2 instance.

---

## 2. Lab Goal

The goal of this lab is to:

- Attach an IAM Role to an EC2 instance.
- Access AWS APIs from EC2.
- Verify the identity using AWS STS.
- Access S3 using the permissions provided by the Role.
- Verify that `AmazonS3ReadOnlyAccess` does not allow write operations.
- Avoid storing Access Keys on EC2.

Architecture:

```text
EC2 Instance
      |
      | AssumeRole
      v
EC2-S3-ReadOnly-Lab
      |
      | AmazonS3ReadOnlyAccess
      v
Amazon S3
```

---

# 3. IAM Role Configuration

Role used in this lab:

```text
EC2-S3-ReadOnly-Lab
```

Trusted entity:

```text
AWS Service: EC2
```

The trust relationship allows EC2 to assume the Role:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

Permission policy:

```text
AmazonS3ReadOnlyAccess
```

There are two different concepts here:

```text
Trust Policy
→ WHO can assume the Role?

Permission Policy
→ WHAT can the Role do?
```

For this lab:

```text
Trust Policy
→ EC2 can assume the Role

Permission Policy
→ Role has read-only access to S3
```

---

# 4. Initial State — EC2 Without IAM Role

The lab started with an existing EC2 instance without an IAM Role attached.

AWS CLI was already installed.

Check AWS CLI:

```bash
aws --version
```

Result:

```text
AWS CLI information displayed successfully.
```

Then check the current AWS identity:

```bash
aws sts get-caller-identity
```

Result:

```text
Error
```

The EC2 instance did not have AWS credentials available.

---

# 5. Check AWS CLI Credentials

Run:

```bash
aws configure list
```

Most credential-related values were:

```text
<not set>
```

This confirms that credentials had not been manually configured using:

```bash
aws configure
```

No Access Key or Secret Access Key was stored manually on the EC2 instance.

---

# 6. Test S3 Before Attaching IAM Role

Run:

```bash
aws s3 ls
```

Result:

```text
Access failed
```

Current state:

```text
EC2
 |
 | No IAM Role
 |
 v
AWS CLI
 |
 X
AWS API
```

The instance had AWS CLI installed, but installing AWS CLI alone does **not** give the instance AWS permissions.

---

# 7. Attach IAM Role to EC2

In AWS Console:

```text
EC2
→ Instances
→ Select instance
→ Actions
→ Security
→ Modify IAM role
```

Select:

```text
EC2-S3-ReadOnly-Lab
```

Then:

```text
Update IAM role
```

The EC2 instance does not need to be restarted after attaching the IAM Role.

---

# 8. Verify the IAM Role

After attaching the Role, run:

```bash
aws sts get-caller-identity
```

This time the command succeeded.

The ARN contained an assumed-role identity similar to:

```text
arn:aws:sts::<ACCOUNT-ID>:assumed-role/EC2-S3-ReadOnly-Lab/<SESSION>
```

Important part:

```text
assumed-role/EC2-S3-ReadOnly-Lab
```

This confirms that AWS CLI commands running on EC2 are using the IAM Role.

---

# 9. IAM Role Credentials

An important concept is that we did **not** run:

```bash
aws configure
```

and did not manually enter:

```text
AWS Access Key ID
AWS Secret Access Key
```

Instead:

```text
EC2
  |
  v
IAM Role
  |
  v
Temporary Credentials
  |
  v
AWS CLI
```

AWS manages these temporary credentials for the instance.

The application or AWS CLI can obtain and use the Role credentials without storing permanent IAM user credentials on the server.

---

# 10. Access S3 Using the IAM Role

Run:

```bash
aws s3 ls
```

Result:

```text
SUCCESS
```

The command was now authorized using:

```text
EC2
→ EC2-S3-ReadOnly-Lab
→ AmazonS3ReadOnlyAccess
→ S3
```

Comparison:

```text
BEFORE IAM ROLE

EC2
 |
 v
aws s3 ls
 |
 X
S3


AFTER IAM ROLE

EC2
 |
 v
IAM Role
 |
 v
AmazonS3ReadOnlyAccess
 |
 v
S3 ✓
```

---

# 11. Test Read-Only Permission

The Role had:

```text
AmazonS3ReadOnlyAccess
```

Therefore, we tested whether the EC2 instance could create an S3 bucket.

Command:

```bash
aws s3 mb s3://<UNIQUE-BUCKET-NAME>
```

Result:

```text
AccessDenied
```

This was the expected result.

The Role can perform permitted read/list operations but cannot create S3 resources.

```text
EC2
 |
 v
IAM Role
 |
 v
AmazonS3ReadOnlyAccess
 |
 +---- Read/List S3 --------> Allowed ✓
 |
 +---- Create S3 Bucket ----> AccessDenied ✗
```

The `AccessDenied` result proves that the IAM policy is restricting what the EC2 instance can do.

---

# 12. Least Privilege

This lab demonstrates the **Principle of Least Privilege**.

A resource should receive only the permissions required to perform its job.

For example, if an application only needs to read files from S3:

```text
Good

EC2
→ S3 ReadOnly
```

Giving it full S3 administration would provide unnecessary permissions:

```text
EC2
→ S3 FullAccess
```

Using limited permissions reduces the potential impact of mistakes or compromised workloads.

---

# 13. IAM Role vs IAM User Credentials

## Not Recommended

Manually placing long-term IAM user credentials on EC2:

```text
EC2
 |
 v
aws configure
 |
 +-- Access Key
 |
 +-- Secret Access Key
```

Problems include:

- Long-term credentials are stored on the server.
- Credentials can potentially be leaked.
- Manual credential rotation may be required.
- Credentials can accidentally be committed to source control.

## Recommended

Use an IAM Role:

```text
EC2
 |
 v
IAM Role
 |
 v
Temporary Credentials
 |
 v
AWS Services
```

Benefits:

- No manually stored long-term Access Key.
- Credentials are temporary.
- AWS handles credential rotation.
- Permissions are controlled using IAM policies.

---

# 14. Trust Policy vs Permission Policy

This distinction is important.

## Trust Policy

Defines:

> Who can assume this Role?

For this lab:

```text
EC2 Service
      |
      | sts:AssumeRole
      v
IAM Role
```

Example:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

## Permission Policy

Defines:

> What can the Role do after it has been assumed?

For this lab:

```text
AmazonS3ReadOnlyAccess
```

Therefore:

```text
EC2
 ↓
Trust Policy
 ↓
Can assume Role ✓
 ↓
Permission Policy
 ↓
Can read/list S3 ✓
Cannot create S3 bucket ✗
```

---

# 15. Important Commands

Check AWS CLI:

```bash
aws --version
```

Check current AWS identity:

```bash
aws sts get-caller-identity
```

Check AWS CLI configuration:

```bash
aws configure list
```

List S3 buckets:

```bash
aws s3 ls
```

Attempt to create an S3 bucket:

```bash
aws s3 mb s3://<UNIQUE-BUCKET-NAME>
```

For this lab, do **not** configure credentials manually:

```bash
aws configure
```

---

# 16. Troubleshooting

If:

```bash
aws sts get-caller-identity
```

fails after you believe a Role has been attached, first verify:

```text
EC2
→ Instances
→ Select instance
→ Security
→ IAM Role
```

The expected Role should be displayed.

If a newly created Role does not appear under:

```text
Modify IAM role
```

verify that the Role was created for the EC2 service.

Expected trusted service:

```text
ec2.amazonaws.com
```

For a simple lab, create the Role using:

```text
IAM
→ Roles
→ Create role
→ Trusted entity type: AWS service
→ Use case: EC2
```

---

# 17. Lab Results

## Without IAM Role

```text
aws --version
→ SUCCESS ✓

aws sts get-caller-identity
→ FAILED ✗

aws configure list
→ Credentials mostly <not set>

aws s3 ls
→ FAILED ✗
```

## After Attaching IAM Role

```text
aws sts get-caller-identity
→ SUCCESS ✓

Identity
→ assumed-role/EC2-S3-ReadOnly-Lab

aws s3 ls
→ SUCCESS ✓

aws s3 mb ...
→ AccessDenied ✗
```

The final `AccessDenied` is an **expected successful test**, because the Role only has read-only S3 permissions.

---

# 18. Key Takeaways

```text
IAM Role + EC2
│
├── EC2 can assume an IAM Role
├── Trust Policy defines who can assume the Role
├── Permission Policy defines what the Role can do
├── EC2 receives temporary credentials
├── No manual Access Key is required
├── AWS CLI can automatically use Role credentials
├── Permissions are restricted by IAM policies
└── Least Privilege should be applied
```

Most important flow:

```text
EC2
 ↓
Assume IAM Role
 ↓
Temporary Credentials
 ↓
AWS CLI / Application
 ↓
AWS API
```

And:

```text
AWS CLI installed
≠
AWS permission granted
```

Permissions come from the IAM identity being used.

---

# 19. Future S3 Lab

A bucket was intentionally **not created** during this lab.

When studying Amazon S3, continue this experiment with:

```text
S3 Bucket
    ↑
EC2 + S3 ReadOnly Role
    |
    ├── List bucket        ✓
    ├── Read object        ✓
    ├── Upload object      ✗
    └── Delete object      ✗
```

Then modify the IAM permissions and compare the results.

This will connect:

```text
IAM
+
EC2
+
S3
```

in a complete hands-on scenario.

---

## Status

**EC2 + IAM Role Hands-on — COMPLETED ✓**
