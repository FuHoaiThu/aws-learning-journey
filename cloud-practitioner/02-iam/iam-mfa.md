# IAM MFA

## 1. Password Policy

- Strong password = higher security for your account
- You can setup a password policy:
  Set a minimum password length
  Require specific character types:
  - Including uppercase letters
  - Lowercase letters
  - Numbers
  - Non-alphanumeric characters
- Allow all IAM users to change their own password
- Require users to change their password after some time ( password expiration )
- Prevent password re-use

## 2. MFA (Multi-Factor Authentication)

- MFA = pasword you know + security device you own
- Main benefit: If a password is stolen or hacked, the account is not compromised

### 2-1. MFA devices options

- Virtual MFA device
  Google Authenitcator ( phone only )
  Authy ( phone only )
  -> Support for multiple tokens on a single device
- Universal 2nd Factor (U2F) Security Key
  Yubikey by Yubico ( 3rd party )
  -> Support for multiple root and IAM users using a single security key
- Hardware Key Fob MFA Device
