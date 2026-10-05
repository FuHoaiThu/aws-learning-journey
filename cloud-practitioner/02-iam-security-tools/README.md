# AWS IAM Security Tools — Theory & Hands-on

## Learning objectives

- Distinguish IAM Credentials Report from IAM Access Advisor.
- Audit credential status and review service-access history.
- Use AWS CLI on Ubuntu with `--profile aws-lab`.
- Apply least privilege without removing permissions solely because no access was recorded.

## 1. Theory

### IAM Credentials Report (account-level)

A CSV report covering IAM users and a root-user row, including password status and usage, MFA status, and access-key status and usage. Use it for account-wide credential auditing.

Important columns: `user`, `password_enabled`, `password_last_used`, `mfa_active`, `access_key_1_active`, `access_key_1_last_used_date`, `access_key_2_active`, `access_key_2_last_used_date`.

### IAM Access Advisor (service last accessed)

Shows service permissions and recorded last-accessed information for IAM users, groups, roles, and policies. Use it to review permissions and inform least-privilege decisions. An absent timestamp does **not** prove that a permission is unnecessary or that every possible access was unsuccessful. Check report scope, tracking limitations, workloads, and business needs first.

| Credentials Report              | Access Advisor                         |
| ------------------------------- | -------------------------------------- |
| Account-level credential audit  | Identity/policy service-access review  |
| Password, MFA, access keys      | Services and last-accessed information |
| Find credentials needing review | Inform least-privilege policy review   |

## 2. Hands-on results — AWS Console

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

## 4. Outstanding observations — not yet resolved

- Console listed **46** services while CLI returned **202**. The difference has **not** been reconciled; do not describe the two counts as matching. Potential checks include whether both views target the same IAM identity, use the same scope/filters, and were generated at comparable times.
- CLI filter for X-Ray returned no matching entry. An empty result `[]` means **no item matched the filter**, not proof of no X-Ray access.
- CLI filter `ServiceNamespace=='iam'` returned an empty result in the earlier attempt. A subsequent search for names containing `Identity` found only `Amazon Cognito Identity` (`cognito-identity`, `TotalAuthenticatedEntities: 0`), which is **not IAM**. IAM's CLI `LastAuthenticated` remains unverified; Console displayed **Today**.
- No further investigation was requested. Preserve these as unresolved notes rather than inventing an explanation.

## 5. Key takeaways

1. Credentials Report answers: **What is the credential status across the account?**
2. Access Advisor answers: **Which services have granted permissions and what last-access information is recorded?**
3. No recorded access alone is insufficient justification for removing a permission.
4. `AdministratorAccess` is broad: do not detach it merely to remove one service's permissions.
5. AWS CLI reports may require checking the exact identity, scope, filtering, and pagination before comparing them with Console views.

## 6. Git hygiene and security

Add to `.gitignore`:

```gitignore
credentials-report.csv
```

Do not publish CSV reports, access keys, passwords, tokens, account IDs, internal URLs, or identifying ARNs. Sanitize screenshots and terminal output before committing.

## 7. AWS documentation

- [IAM credential reports](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html)
- [Service last accessed information](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_last-accessed.html)
- [AWS CLI: generate-service-last-accessed-details](https://docs.aws.amazon.com/cli/latest/reference/iam/generate-service-last-accessed-details.html)
- [AWS CLI: get-service-last-accessed-details](https://docs.aws.amazon.com/cli/latest/reference/iam/get-service-last-accessed-details.html)
