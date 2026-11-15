# EC2

- EC2 is one of the most popular of AWS' offering
- EC2 = Elastic Compute Cloud ( IaaS )
- It mainly consists in the capability of:
  Renting virtual machines ( EC2 instances )
  Storing data on virtual drives ( EBS )
  Distributing load across machines ( ELB )
  Scaling the services using an auto-scaling group ( ASG )

## 1. EC2 sizing and cofiguration options

- Operating system ( OS ): Linux, Windows or Mac OS
- How much compute power and cores ( CPU )
- How much ramdom-access memory ( RAM )
- How much storage space:
  Network-attached ( EBS and EFS )
  Hardware ( EC2 Instance Store )
- Network card: speed of the card, Public IP address
- Firewall rules: security group
- Bootstrap script ( configure at first launch ): EC2 User Data

## 2. EC2 User Data

- It is possible to bootstrap our instances using an EC2 User data script
- Bootstraping means launching commands when a machine starts
- That script is only run once at the instance first launch
- EC2 user data is used to automate boot tasks such as:
  Installing updates
  Installing software
  Downloading common files from the internet
  Anything you can think of
- The EC2 user data script runs with the root user

## 3. AMI

- Provides the information required to launch an EC2 instance.

## 4. Key pair

- A key pair is used for authentication when connecting to an EC2 instance, for example via SSH
- It consists of:
  Public key → stored on the EC2 instance.
  Private key → kept by you. This is sensitive information; do not share it in any way."
