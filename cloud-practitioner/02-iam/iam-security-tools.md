# IAM security tools

## 1. IAM Credentials Report ( account-level )

- A report that list all account's users and the status of their various credentials.
- The report is generated as a CSV file.
- The report also includes the root user.
- Key information:
  Password: Is it enabled? When was it last used?
  MFA: Is it activated?
  Access Keys: Are they active? When were they last used?
- Purpose: To audit the security status of credentials across the entire AWS account.
- Example: An admin wants to identify IAM users who have not enabled MFA → use the IAM Credentials Report.

## 2. IAM Access Advisor ( user-level )

- Access advisor shows the service permission granted to a user and when those services were last accessed.
- Two key pieces of information:
  Service permissions granted: Services for which permissions have been granted.
  Last accessed: The last time the service was accessed.
- Purpose: To use this information to adjust IAM policies and apply the principle of Least Privilege.
- Example: An IAM user is granted permissions for S3, EC2, and DynamoDB but does not use DynamoDB during the monitoring period.
  → The admin needs to verify if the DynamoDB permission is still necessary and then consider revoking it.
- Access Advisor also supports IAM groups, roles, and policies, not just IAM users.

## 3. Hands-on in — AWS Console

### Credentials Report

| Check                                         | Observed result |
| --------------------------------------------- | --------------: |
| IAM users (excluding root)                    |               2 |
| IAM users with `mfa_active=false`             |               2 |
| IAM users with at least one active access key |               1 |
| Root MFA                                      |         Enabled |

Follow-up: Check `password_enabled` before assessing MFA exposure; verify actual access-key usage and dependencies before disabling a key.

### Access Advisor

- Console listed **46 services** for the selected IAM user.
- Some services showed no access within the tracking period.
- IAM showed **Last accessed: Today** in Console.
- AWS X-Ray appeared as not accessed in the inspected Console view.
- The granting policy identified was the AWS-managed `AdministratorAccess` policy.

**Important:** `AdministratorAccess` grants broad permissions; detaching it to remove only X-Ray access would also remove its other grants. Review role requirements and test narrower replacement policies before changing access.

## 3. AWS CLI lab — Ubuntu

All commands explicitly use `--profile aws-lab`. Replace placeholders locally; never commit real ARNs, account IDs, job IDs, or credential reports.

### Check active identity

```bash
aws sts get-caller-identity --profile aws-lab
```

### Generate Credentials Report

```bash
aws iam generate-credential-report --profile aws-lab
```

Observed transition: `STARTED` → `COMPLETE`.

### Download and decode Credentials Report

```bash
aws iam get-credential-report \
  --profile aws-lab \
  --query 'Content' \
  --output text | base64 --decode > credentials-report.csv

ls -lh credentials-report.csv
```

Observed: CSV downloaded successfully. Keep it private and out of Git.

### List IAM users and identify the intended ARN

```bash
aws iam list-users \
  --profile aws-lab \
  --query 'Users[].{UserName:UserName,Arn:Arn}' \
  --output table
```

### Generate service-last-accessed report

```bash
aws iam generate-service-last-accessed-details \
  --arn 'YOUR_IAM_USER_ARN' \
  --profile aws-lab
```

Observed: command returned a `JobId`.

### Retrieve the report

```bash
aws iam get-service-last-accessed-details \
  --job-id 'YOUR_JOB_ID' \
  --profile aws-lab
```

Observed: `JobStatus=COMPLETED`, with `ServicesLastAccessed` returned.

### Count services

```bash
aws iam get-service-last-accessed-details \
  --job-id 'YOUR_JOB_ID' \
  --profile aws-lab \
  --query 'length(ServicesLastAccessed)' \
  --output text
```

Observed CLI count: **202 services**.

### Display service names and last-accessed timestamps

```bash
aws iam get-service-last-accessed-details \
  --job-id 'YOUR_JOB_ID' \
  --profile aws-lab \
  --query 'ServicesLastAccessed[].{Service:ServiceName,LastAccessed:LastAuthenticated}' \
  --output table
```

### Inspect IAM and X-Ray entries by namespace

```bash
aws iam get-service-last-accessed-details \
  --job-id 'YOUR_JOB_ID' \
  --profile aws-lab \
  --query "ServicesLastAccessed[?ServiceNamespace=='iam']" \
  --output json

aws iam get-service-last-accessed-details \
  --job-id 'YOUR_JOB_ID' \
  --profile aws-lab \
  --query "ServicesLastAccessed[?ServiceNamespace=='xray']" \
  --output json
```

### Inspect report metadata

```bash
aws iam get-service-last-accessed-details \
  --job-id 'YOUR_JOB_ID' \
  --profile aws-lab \
  --query '{Status:JobStatus,Type:JobType,Count:length(ServicesLastAccessed)}' \
  --output json
```

Observed `JobType`: `SERVICE_LEVEL`.
