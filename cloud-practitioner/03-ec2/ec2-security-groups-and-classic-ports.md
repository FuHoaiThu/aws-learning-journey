# Security Groups and Classic Ports

## 1. Introduce to security groups

- A Security Group (SG) is a virtual firewall that controls traffic entering and leaving AWS resources, such as EC2 instances.
- They regulate:
  Access to Ports
  Authorised IP ranges - IPv4 and IPv6
  Control of inbound network ( from other to the instance )
  Control of outbound network ( from the instance to other )
  Internet
  │
  │ HTTP :80
  ↓
  Security Group
  │
  ↓
  EC2
  │
  └── Apache
- Security groups only contain allow rules
  ALLOW TCP :80
  ALLOW TCP :22 from My IP
- Security groups rules can reference by IP or by another security group
  By IP
  - SSH
  - Port: 22
  - Source: 123.123.123.123/32

  By another security group
  - Load Balancer
  - Security Group: SG-A
    ↓
    EC2
    Security Group: SG-B
    -> SG-B:
    HTTP :80
    Source: SG-A

- Can be attached to multiple instances
- Locked down to a region / VPC combination
- Does live outside the EC2 - if traffic is blocked the EC2 instance won't see it
- It's good to maintain one separate security group for SSH access
- If your application is not accessible ( timeout ), then it's a security group issue
- If your application gives a connection refused error, then it's an application error or it's not launched
- All inbound traffic is blocked by default
- All outbound traffic is authorized by default

## 2. Classic Ports

- 22 = SSH ( Secure Shell ) - log into a Linux instance
- 21 = FTP ( File Transfer Protocol )
- 22 = SFTP ( Secure File Transfer Protocol ) - upload files using SSH
- 80 = HTTP - access unsecured websites
- 443 = HTTPS - access secured websites
- 3389 = RDP ( Remote Desktop Protocol ) - log into a Windown instance "

## 3.Hands-on Lab — Security Group

### Goal

Verify how Security Group inbound rules affect access to an EC2 web server.

The existing EC2 instance had:

```text
SSH   TCP   22
HTTP  TCP   80
```

Apache was already running on the EC2 instance.

---

### Step 1 — Verify HTTP Access

The EC2 Public IPv4 address was opened in a browser:

```text
http://<PUBLIC-IP>
```

Result:

```text
Website displayed successfully ✓
```

Initial flow:

```text
Browser
   |
   | HTTP :80
   v
Security Group
   |
   | Allow :80
   v
EC2
   |
   v
Apache
   |
   v
Website ✓
```

This established the baseline before changing the Security Group.

---

### Step 2 — Remove HTTP Port 80

The Security Group was edited:

```text
EC2
→ Security
→ Security Group
→ Inbound rules
→ Edit inbound rules
```

The following rule was removed:

```text
HTTP
TCP
80
0.0.0.0/0
```

The SSH rule was kept.

No EC2 restart or Apache restart was performed.

The website was accessed again:

```text
http://<PUBLIC-IP>
```

Result:

```text
Connection timed out ✗
```

Flow:

```text
Browser
   |
   | HTTP :80
   v
Security Group
   |
   X  No Allow :80 rule
   |
EC2
```

This demonstrated that the Security Group prevented the external HTTP request from reaching the EC2 instance.

---

### Step 3 — Verify Apache Is Still Running

The EC2 instance was accessed through EC2 Instance Connect.

Inside the instance, Apache status was checked:

```bash
sudo systemctl status httpd
```

Result:

```text
Active: active (running)
```

This proved that the website timeout was **not caused by Apache being stopped**.

To exit the status screen:

```text
q
```

---

### Step 4 — Test the Website Locally

Inside the EC2 instance:

```bash
curl http://localhost
```

The website HTML was returned successfully.

Result:

```text
localhost:80 ✓
```

At this point:

```text
External browser → Public IP:80
→ TIMEOUT ✗

Apache
→ ACTIVE ✓

EC2 → localhost:80
→ WEBSITE ✓
```

Therefore:

```text
Application is healthy
+
External traffic fails
=
Investigate network access / Security Group
```

This is an important EC2 troubleshooting pattern.

---

### Step 5 — Restore HTTP Port 80

The HTTP inbound rule was added back:

```text
Type: HTTP
Protocol: TCP
Port: 80
Source: 0.0.0.0/0
```

The rule was saved.

No restart was required.

The website was accessed again:

```text
http://<PUBLIC-IP>
```

Result:

```text
Website displayed successfully ✓
```

Final experiment:

```text
HTTP :80 allowed
       |
       v
Website ✓

HTTP :80 removed
       |
       v
Timeout ✗

HTTP :80 restored
       |
       v
Website ✓
```

This demonstrates that Security Group rule changes can affect connectivity without restarting the EC2 instance or Apache.

---
