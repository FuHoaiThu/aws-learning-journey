# IAM (Identity and Access Management )

## Users and Groups

- IAM = Identity and Access Management, Global service
- Root account created by default, shouldn't be used or shared
- Users are people within your organization, can be grouped
- Groups only contain users, not other groups
- Users don't have to belong to a group, and user can belong to multiple groups

## IAM permission

- Users or groups can be assigned JSON documents call policies
- These policies define the permissions of the users
- The least privilege principle: don't give more permission than a user needs
  Allow + Allow → Allow
  Allow + Implicit Deny → Allow
  Allow + Explicit Deny → Deny
