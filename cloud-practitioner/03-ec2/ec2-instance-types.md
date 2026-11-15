# EC2 Instance Types

- You can use different types of EC2 instances that are optimised for different use cases.
  General Purpose
  Compute Optimized
  Memory Optimized
  Storage Optimized
  Accelerated Computing
  HPC Optimized

- AWS has the following naming convention:
  ex: m5.2xlarge
  m: Instance class
  5: generation
  2xlarge: size within the instance class

## 1. General Purpose

- Great for a diversity of workloads such as web servers or code repositories.
- Balance between:
  Compute
  Memory
  Networking

## 2. Compute Optimized

- Great for compute-intensive taks that require high performance processors:
  Batch processing workloads
  Media transcoding
  High performance web servers
  High performance computing (HPC)
  Scientific modeling and machine learning
  Dedicated gaming servers

## 3. Memory Optimized

- Fast performance for workloads that process large data sets in memory
- Use cases:
  High performance, relational / non-relational databases
  Distributed web scale cache stores
  In-memory databases optimized for BI ( business itelligence )
  Applications performing real-time processing of big unstructrured data

## 4. Storage Optimized

- Great for storage-intensive tasks that require high, sequential read and write access to large data sets on local storage.
- Use cases:
  High frequency online transaction processing (OLTP)
  Relational and NoSQL databases
  Cache for in-memory databases (ex: Redis )
  Data warehousing applications
  Distributed file systems"
