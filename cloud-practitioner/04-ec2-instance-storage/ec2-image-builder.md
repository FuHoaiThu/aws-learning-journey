# EC2 Image Builder

## Overview

- An AWS service used to automate the creation of virtual machine images and container images
- It automates the creation, maintenance, validation, and testing of EC2 Amazon Machine Images ( AMIs)
- It helps create standardized, secure, and up-to-date images.

## Key features

- Automation: Automatically builds and maintains EC2 AMIs.
- Customization: Installs software, applies updates, and configures images using build components.
- Validation and Testing: Verifies that images work correctly and meet requirements.
- Scheduling: Can run on a schedule, such as weekly or when updates are available.
- Distribution: Can distribute AMIs across multiple AWS Regions and accounts.

## How EC2 Image Builder Works

1. EC2 Image Builder: Starts the automated image-building process.
2. Builder EC2 Instance: Launches a temporary EC2 instance to build the image.
3. Build Components: Installs software, applies patches, and customizes the instance.
4. New AMI: Creates a new AMI from the configured instance.
5. Test EC2 Instance: Launches a temporary EC2 instance to validate and test the AMI.
6. AMI Distribution: Distributes the AMI to one or more AWS Regions.

## Pricing

- EC2 Image Builder has no additional service charge.
- You only pay for the underlying AWS resources used, such as:
  EC2 instances
  EBS volumes
  EBS snapshots
  Other applicable AWS resources"
