# Amazon FSx

## Overview

- Fully managed file storage service on AWS.
- Provides high-performance file systems based on established file system technologies.
- Main types:
  FSx for Windows File Server
  FSx for Lustre
  FSx for NetApp ONTAP
  FSx for OpenZFS

## Amazon EFx for Windows File Server

- A fully managed, highly reliable, and scalable Windows native shared file system
- Built on Windows File Server
- Supports SMB (Server Message Block): A network protocol used to share files and folders, commonly on Windows.
- Supports NTFS (New Technology File System): A Windows file system that supports file permissions and access control.
- Integrates with Microsoft Active Directory (AD): A service used to manage users, groups, authentication, and access.
- Accessible from AWS and on-premises infrastructure through appropriate network connectivity.
- Supports Multi-AZ deployments and automatic backups.
- Use cases: Windows applications, enterprise shared folders, corporate file servers.

## Amazon FSx for Lustre

- Fully managed, high-performance, scalable file storage for HPC (High Performance Computing).
- Lustre is derived from Linux and Cluster.
- Use cases: Machine Learning, Data Analytics, Video Processing, Financial Modeling, ...
- Performance capabilities:
  Throughput (GB/s): can scale to hundreds of GB/s.
  IOPS (Input/Output Operations Per Second): Number of read/write operations per second; can reach millions.
  Latency: can achieve sub-millisecond latency.
- Integrates with Amazon S3 for processing large datasets.
- Deployment types:
  Scratch: Temporary storage for short-term processing.
  Persistent: Durable storage for longer-term workloads.
