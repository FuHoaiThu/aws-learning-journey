# IAM policies

## IAM Policies Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListMFADevices",
      "Effect": "Allow",
      "Action": ["iam:ListVirtualMFADevices"],
      "Resource": "*"
    }
  ]
}
```

- A policy consists of
  Version: policy language version, always include ""2012-10-17""
  Id: an identifier for the policy (optional)
  Statement: one or more individual statements (required)
- A statement consists of
  Sid: an identifier for the statement (optional)
  Effect: whether the statement allows or denies access (Allow, Deny)
- Principal: account / user / role to which this policy applied to (optional)
- Action: list of actions this policy allows or denies
- Resource: list of resources to which the actions applied to
- Condition: conditions for when this policy is in effect (optional)
