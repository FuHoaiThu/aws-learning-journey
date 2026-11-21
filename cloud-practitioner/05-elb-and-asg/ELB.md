# ELB (Elastic Load Balancing)

## Overview

- Elastic Load Balancer (ELB) is a managed AWS service that automatically distributes incoming traffic across multiple targets (e.g., EC2 instances).

## Why use a Load Balancer?

- Load Distribution: Distribute traffic across multiple instances.
- Single DNS: Provide one access point for applications.
- Fault Tolerance: Redirect traffic away from unhealthy instances.
- Health Checks: Regularly check instance health.
- SSL Termination: Handle HTTPS encryption/decryption.
- High Availability: Distribute traffic across multiple Availability Zones (AZs).

## Why use an Elastic Load Balancer?

- Fully Managed: AWS handles maintenance, upgrades, and high availability.
- Easy Configuration: Minimal management effort.
- AWS Integration: Works with EC2, Auto Scaling, CloudWatch, etc.
- Cost: Self-managed load balancers may have lower direct infrastructure costs but require more maintenance.

## Types of AWS Load Balancers

- Application Load Balancer (ALB) – Layer 7
  Protocols: HTTP, HTTPS, gRPC (Google Remote Procedure Call)
  Content-based routing (path, hostname, headers)
  DNS name provided by AWS
  Best for web applications and APIs

- Network Load Balancer (NLB) – Layer 4
  Protocols: TCP, UDP, TLS
  Ultra-high performance, millions of requests/second
  Supports static IP and Elastic IP
  Best for high-performance, low-latency applications

- Gateway Load Balancer (GWLB) – Layer 3
  Uses GENEVE protocol (port 6081)
  Routes traffic through virtual security appliances
  Supports firewalls and intrusion detection systems (IDS)

- Classic Load Balancer (CLB) – Layer 4 & 7
  Legacy load balancer
  Supports TCP, SSL/TLS, HTTP, HTTPS
  Not recommended for new applications

## Hands-on

### 1. Objectives

- Launch two EC2 web servers with different **User Data** scripts.
- Register the instances in an HTTP target group.
- Create an internet-facing Application Load Balancer (ALB).
- Use a single ALB DNS name to access both web servers.
- Observe target health checks and traffic distribution.
- Simulate one unhealthy target and verify failover to the healthy target.
- Restore the target and delete lab resources to minimize costs.

### 2. Architecture

```text
                 Browser / Client
                        |
                  HTTP (port 80)
                        |
                    demo-alb
                (Internet-facing)
                        |
               HTTP:80 Listener
                        |
                demo-target-group
                   /          \
                  /            \
        My First Instance   My Second Instance
          HTTP server         HTTP server
          User Data #1        User Data #2
```

**Key components**

| Resource              | Lab name / configuration | Purpose                                               |
| --------------------- | ------------------------ | ----------------------------------------------------- |
| EC2                   | `My First Instance`      | Web server with first response                        |
| EC2                   | `My Second Instance`     | Web server with different response                    |
| Target group          | `demo-target-group`      | Registers both EC2 instances on HTTP port 80          |
| ALB                   | `demo-alb`               | Distributes incoming HTTP traffic                     |
| Listener              | `HTTP:80`                | Forwards requests to `demo-target-group`              |
| Health check          | HTTP `/`, expected `200` | Determines whether a target is healthy                |
| ALB security group    | `demo-alb-sg`            | Allows inbound HTTP port 80 from clients              |
| Failure simulation SG | `demo-ec2-failure-sg`    | Temporarily blocks inbound HTTP on the first instance |

### 3. Steps and Results

#### Step 1 — Launch and verify EC2 instances

1. Created `My First Instance` and `My Second Instance` with **different User Data** scripts to serve distinguishable web pages.
2. Confirmed that both EC2 instances were running.
3. Opened each instance's **public IPv4 address** using `http://<public-ip>`.
4. **Observed:** Both web pages displayed correctly.

**Lesson:** Verify that each backend server works independently before configuring a load balancer.

#### Step 2 — Create a target group

Console: **EC2 → Target Groups → Create target group**

- Target type: **Instances**
- Name: `demo-target-group`
- Protocol / port: **HTTP / 80**
- VPC: Same VPC as the two EC2 instances
- Protocol version: **HTTP1**
- Health check protocol: **HTTP**
- Health check path: `/`
- Success code: `200`
- Registered both EC2 instances as targets.

**Observed:** Both instances appeared in the target group.

**Lesson:** A target can show **Unused** before the target group is connected to a suitable ALB listener or if its Availability Zone is not enabled. `Unused` is not the same as `Unhealthy`.

#### Step 3 — Create the Application Load Balancer

Console: **EC2 → Load Balancers → Create → Application Load Balancer**

- Name: `demo-alb`
- Scheme: **Internet-facing**
- IP address type: **IPv4**
- VPC: Same VPC as the targets
- Network mapping: At least **two subnets in different Availability Zones**, with internet-facing routing
- ALB security group: Allow inbound **HTTP/TCP port 80**
- EC2 security group: Allow inbound **HTTP/TCP port 80 from the ALB security group**
- Listener: **HTTP:80 → Forward to `demo-target-group`**

Waited for the ALB state to become **Active** and used its AWS-provided DNS name:

```text
http://<alb-dns-name>
```

**Observed:** The ALB DNS initially displayed the page from `My First Instance`.

**Lesson:** The ALB provides one DNS entry point; the listener rule determines which target group receives requests. An **Active** ALB alone does not guarantee that its targets are healthy.

#### Step 4 — Test load balancing

1. Opened the ALB DNS name in a browser over **HTTP**.
2. Refreshed repeatedly to send new requests.
3. **Observed:** Responses came from **both** `My First Instance` and `My Second Instance`.

**Lesson:** ALB distributes requests among eligible targets. Browser refreshes are **not guaranteed** to alternate strictly 1 → 2 → 1 → 2; connection reuse, routing algorithm, and stickiness settings can affect the result.

#### Step 5 — Test health checks and failure handling

1. Confirmed that the target group was being used by the ALB.
2. Simulated failure on `My First Instance` by changing its attached security groups so inbound HTTP was blocked. Used a separate lab security group (`demo-ec2-failure-sg`) to avoid disrupting the second instance.
3. Waited for health checks to detect the failure.
4. **Observed:**

| Target               | Health status | Traffic result                                                |
| -------------------- | ------------- | ------------------------------------------------------------- |
| `My First Instance`  | **Unhealthy** | No longer selected while another healthy target was available |
| `My Second Instance` | **Healthy**   | Continued serving requests                                    |

5. Refreshed the ALB DNS name.
6. **Observed:** The browser displayed only the response from `My Second Instance`.

**Lesson:** ALB health checks detect unhealthy targets and normally stop routing requests to them when healthy alternatives exist. This improves **fault tolerance** and availability.

**Important nuance:** If **all** targets in a target group are unhealthy, ALB can **fail open** and still route traffic to unhealthy targets.

#### Step 6 — Restore the first instance

1. Restored the original EC2 security group on `My First Instance` to allow HTTP from the ALB security group.
2. Allowed time for health checks to run again.
3. The recovery procedure was covered as part of the lab; the user subsequently confirmed the overall lab and cleanup were complete. No separate post-recovery health-status observation was recorded in the conversation.

**Lesson:** When a target becomes healthy again, ALB can automatically resume routing requests to it.
