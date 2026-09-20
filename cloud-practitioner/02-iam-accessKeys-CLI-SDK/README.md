## AWS Access Keys, CLI, and SDK

### How Users Access AWS

There are three main ways to interact with AWS:

| Method                 | Usage                                                  |
| ---------------------- | ------------------------------------------------------ |
| AWS Management Console | Access AWS through a web browser                       |
| AWS CLI                | Access AWS services using command-line commands        |
| AWS SDK                | Access AWS services programmatically from applications |

### Access Keys

Access keys can be used as credentials when accessing AWS programmatically.

An access key consists of:

- Access Key ID
- Secret Access Key

Access keys are sensitive credentials and should never be shared, hard-coded in source code, or committed to GitHub.

For workloads, temporary credentials and IAM roles should be preferred over long-term access keys when possible.

### AWS CLI

AWS CLI (Command Line Interface) allows us to interact with AWS services from a terminal.

It can:

- Call AWS service APIs.
- Manage AWS resources using commands.
- Be used in scripts for automation.
- Provide an alternative to the AWS Management Console.

Basic command structure:

```text
aws <service> <command>
```

Example:

```text
aws iam list-users
```

### AWS SDK

AWS SDK (Software Development Kit) provides language-specific libraries that allow applications to interact with AWS services programmatically.

AWS provides SDKs for languages and platforms such as JavaScript, Python, Java, .NET, Go, mobile applications, and IoT devices.

### CloudShell

AWS CloudShell is a browser-based shell that lets you run AWS CLI commands from the AWS Management Console.

---

## Hands-on: AWS CLI on Windows

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

## Key Takeaways

- AWS CLI allows users to interact with AWS services from the command line.
- AWS CLI commands call AWS service APIs.
- Access keys are credentials and must be protected.
- IAM policies still control what an authenticated CLI user can do.
- Authentication and authorization are different concepts.
- Long-term access keys should only be created when necessary.
- Unused access keys should be removed.
