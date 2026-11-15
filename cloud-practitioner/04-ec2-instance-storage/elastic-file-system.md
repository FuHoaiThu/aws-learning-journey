# Elastic File System

## Overview

- Managed NFS ( network file system ) that can be mounted on 100s of EC2
- EFS works with Linux EC2 instances in multi AZ
- Highly available, scalble, expensive ( 3x gp2 ), pay per use, no capacity planing

## EFS Infrequent Access (EFS-IA)

- Storage class that is cost-optimized for files not accessed every day
- Up to 92% lower cost compared to EFS Standard
- EFS will automatically move your files to EFS-IA based on the last time they were accessed
- Enable EFS-IA with a Lifecycle Policy
- Transparent to the applications accessing EFS"
