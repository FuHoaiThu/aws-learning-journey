# AWS Access Keys, CLI and SDK

## 1. How can users access AWS

- Three opitons:
  AWS Management Console: protected by password + MFA
  AWS Command LIne Interface (CLI): protected by access keys
  AWS Software Developer Kit (SDK): for code: protected by access keys
- Access Keys are generated through the AWS Console.
- Users manage their own access keys
- Access Keys are secret, just like a password. Don't share them

## 2. AWS CLI

- A tool that enables you to interact with AWS services using commands in your command-line sheel
- Direct access to the public APIs of AWS service
- You can develop scripts to manage your resources
- It's open source
- Alternative to using AWS Management Console.

## 3. AWS SDK

- AWS Software Development Kit
- Language-specific APIs ( set of libraries )
- Enables you to access and manage AWS services programmatically
- Embedded within your application

## 4. CloudShell

AWS CloudShell is a browser-based shell that lets you run AWS CLI commands from the AWS Management Console."

## 5.Hands-on: AWS CLI on Windows

### 1. Check AWS CLI Installation

```powershell
aws --version
```

Result:

```text
aws-cli/2.17.0 Python/3.11.8 Windows/10 exe/AMD64
```

### 2. Check Existing Profiles

```powershell
aws configure list-profiles
```

No profile was configured initially.

### 3. Configure a CLI Profile

I created an access key for an IAM user and configured a named AWS CLI profile:

```powershell
aws configure --profile aws-lab
```

Configuration included:

- Access Key ID
- Secret Access Key
- Default AWS Region
- Output format: JSON

> Access keys and secret keys are never stored in this repository.

### 4. Verify the AWS Identity

```powershell
aws sts get-caller-identity --profile aws-lab
```

This command verifies which AWS identity is being used by the CLI.

### 5. Test IAM Permissions

List IAM users:

```powershell
aws iam list-users --profile aws-lab
```

List IAM groups:

```powershell
aws iam list-groups --profile aws-lab
```

Both commands succeeded because the IAM user used in this lab had administrator permissions.

### Authentication vs Authorization

This lab helped me understand the difference between:

**Authentication**

> Who are you?

The access key allows AWS to identify the IAM user.

**Authorization**

> What are you allowed to do?

IAM policies determine which AWS API actions the authenticated identity can perform.

Having valid credentials does not mean the user has permission to perform every AWS action.

### 6. Security Cleanup

After completing the lab, I deleted the administrator user's access key because it was only created for learning purposes.

Long-term administrator access keys should not be kept unnecessarily.
