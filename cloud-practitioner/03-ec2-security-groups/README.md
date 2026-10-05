# EC2 Security Groups & Classic Ports

## 1. Overview

A **Security Group (SG)** is a fundamental component of network security in AWS.

Security Groups act as **virtual firewalls** that control traffic allowed to and from resources such as EC2 instances.

```text
Internet
   |
   v
Security Group
   |
   v
EC2 Instance
```

Security Groups regulate:

- Access to ports
- Authorized IPv4 and IPv6 ranges
- Inbound traffic
- Outbound traffic

---

# 2. Security Group Rules

Security Groups contain **allow rules only**.

```text
ALLOW  ✓
DENY   ✗
```

For example:

```text
Allow HTTP :80
Allow SSH  :22
```

If traffic does not match an applicable allow rule, the Security Group does not allow that traffic through.

Security Group rules can reference:

- IP addresses / CIDR ranges
- Other Security Groups

---

# 3. Inbound and Outbound Traffic

## Inbound Traffic

Inbound traffic flows **toward the EC2 instance**.

```text
Client
   |
   v
Security Group
   |
   v
EC2
```

Examples:

```text
SSH    :22
HTTP   :80
HTTPS  :443
```

## Outbound Traffic

Outbound traffic flows **from the EC2 instance to another destination**.

```text
EC2
 |
 v
Internet / Other Resource
```

For example, when an EC2 instance downloads packages:

```text
EC2
 |
 | HTTPS
 v
Package Repository
```

A typical/default Security Group configuration starts with:

```text
Inbound
→ No inbound traffic allowed unless an inbound rule permits it

Outbound
→ All outbound traffic allowed
```

---

# 4. Referencing IP Addresses

A Security Group rule can allow traffic from a specific IP address.

Example:

```text
Type: SSH
Protocol: TCP
Port: 22
Source: x.x.x.x/32
```

`/32` represents one specific IPv4 address.

Comparison:

```text
0.0.0.0/0
→ All IPv4 addresses

x.x.x.x/32
→ One specific IPv4 address
```

For direct SSH access from a known network, restricting port 22 to a specific trusted IP is safer than exposing SSH to the entire Internet.

---

# 5. Referencing Other Security Groups

Security Groups can reference other Security Groups.

Example:

```text
Internet
   |
   v
Load Balancer
SG-LoadBalancer
   |
   | HTTP
   v
EC2
SG-WebServer
```

The EC2 Security Group could contain:

```text
Type: HTTP
Port: 80
Source: SG-LoadBalancer
```

This is useful because the rule does not need to depend on individual instance IP addresses.

---

# 6. Security Groups and EC2

A Security Group acts outside the EC2 instance from the perspective of the instance.

If traffic is blocked by the Security Group, it does not reach the application running on EC2.

```text
Traffic
   |
   v
Security Group
   |
   +---- Allowed ----> EC2
   |
   +---- Blocked ----> X
```

A Security Group can be associated with multiple compatible resources/instances.

```text
          Security Group
          /     |      \
         v      v       v
       EC2-1  EC2-2   EC2-3
```

Security Groups are associated with a **VPC**, and the VPC belongs to an AWS Region.

```text
Region
  |
  v
VPC
  |
  v
Security Group
  |
  v
EC2
```

---

# 7. SSH Security

SSH should not normally be exposed to the entire Internet unless there is a specific reason.

Less restrictive:

```text
SSH :22
Source: 0.0.0.0/0
```

This allows connection attempts from any IPv4 address.

More restrictive for direct SSH from a known network:

```text
SSH :22
Source: <trusted-public-ip>/32
```

A useful design is to maintain separate Security Groups for different responsibilities.

Example:

```text
SG-Web
├── HTTP  :80
└── HTTPS :443

SG-SSH
└── SSH :22 → Trusted source
```

---

# 8. Classic Ports

Important ports to recognize:

| Port | Protocol | Purpose                            |
| ---: | -------- | ---------------------------------- |
|   21 | FTP      | File Transfer Protocol             |
|   22 | SSH      | Secure Shell / Linux remote access |
|   22 | SFTP     | Secure file transfer over SSH      |
|   80 | HTTP     | Unencrypted web traffic            |
|  443 | HTTPS    | TLS-encrypted web traffic          |
| 3389 | RDP      | Windows Remote Desktop             |

Important:

```text
Linux remote login
→ SSH :22

Windows remote desktop
→ RDP :3389

Website
→ HTTP :80

Secure website
→ HTTPS :443
```

FTP and SFTP should not be confused:

```text
FTP
→ Port 21
→ Does not use SSH

SFTP
→ Usually Port 22
→ Runs over SSH
```

---

# 9. Troubleshooting: Timeout vs Connection Refused

## Connection Timeout

If an application cannot be reached and the connection eventually times out, one of the first areas to investigate is the network path, including the Security Group.

Example:

```text
Browser
   |
   | HTTP :80
   v
Security Group
   |
   X
EC2
```

If port 80 is not allowed, the HTTP request cannot reach the EC2 instance through that Security Group.

## Connection Refused

If the connection is refused, check whether the application/service is running and listening on the expected port.

Example:

```text
Security Group
HTTP :80 ✓
    |
    v
EC2
    |
    v
Apache stopped ✗
```

These are useful troubleshooting heuristics, not absolute rules. Other network components and operating-system configuration can also affect connectivity.

---

# 10. Hands-on Lab — Security Group

## Goal

Verify how Security Group inbound rules affect access to an EC2 web server.

The existing EC2 instance had:

```text
SSH   TCP   22
HTTP  TCP   80
```

Apache was already running on the EC2 instance.

---

# 11. Step 1 — Verify HTTP Access

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

# 12. Step 2 — Remove HTTP Port 80

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

# 13. Step 3 — Verify Apache Is Still Running

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

# 14. Step 4 — Test the Website Locally

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

# 15. Step 5 — Restore HTTP Port 80

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

# 16. SSH and EC2 Instance Connect Experiment

Initially, SSH was configured as:

```text
SSH
TCP
22
0.0.0.0/0
```

The source was changed to:

```text
SSH
TCP
22
My IP /32
```

After this change, a new browser-based **EC2 Instance Connect** session failed with:

```text
Failed to connect to your instance
```

The SSH source was restored to:

```text
0.0.0.0/0
```

After restoring the rule, browser-based EC2 Instance Connect successfully connected again.

## Important Observation

`My IP` represents the public IP detected for the user's current network.

However, browser-based EC2 Instance Connect does not necessarily establish the SSH network connection from that same client public IP.

Therefore:

```text
SSH :22 → My IP
```

is particularly relevant when making a **direct SSH connection from the trusted client/network**, but it may not be sufficient for every EC2 Instance Connect configuration.

This does **not** mean opening SSH to `0.0.0.0/0` is recommended for production.

The broad SSH rule was used temporarily during this learning lab.

---

# 17. Direct SSH with a Key Pair

When directly connecting from a Linux machine with a `.pem` private key, the private key file permissions can be restricted:

```bash
chmod 400 <key-name>.pem
```

Then connect using:

```bash
ssh -i <key-name>.pem ec2-user@<PUBLIC-IP>
```

For this direct SSH model, the Security Group can be restricted to the trusted public IP:

```text
SSH
TCP
22
<TRUSTED-PUBLIC-IP>/32
```

Conceptually:

```text
Local Computer
Public IP: x.x.x.x
       |
       | SSH :22
       v
Security Group
Source: x.x.x.x/32
       |
       v
EC2
```

Never commit the `.pem` private key to GitHub.

---

# 18. Useful Commands

## Check Apache

```bash
sudo systemctl status httpd
```

## Start Apache

```bash
sudo systemctl start httpd
```

## Check the Web Server Locally

```bash
curl http://localhost
```

## Direct SSH

```bash
ssh -i <key-name>.pem ec2-user@<PUBLIC-IP>
```

## Restrict Private Key Permissions

```bash
chmod 400 <key-name>.pem
```

---

# 19. Troubleshooting Flow

When an EC2 website is inaccessible:

```text
Website inaccessible
        |
        v
Is EC2 running?
        |
        v
Check Security Group
        |
        +-- Correct port?
        |
        +-- Correct source?
        |
        v
Check application/service
        |
        v
sudo systemctl status httpd
        |
        v
Test locally
        |
        v
curl http://localhost
```

Example from this lab:

```text
Browser timeout
      |
      v
Apache active ✓
      |
      v
curl localhost ✓
      |
      v
Application works
      |
      v
Check Security Group
      |
      v
HTTP :80 missing
      |
      v
Restore HTTP :80
      |
      v
Website works ✓
```

---

# 20. Key Takeaways

```text
Security Group
│
├── Acts as a virtual firewall
├── Controls inbound and outbound traffic
├── Contains Allow rules only
├── Can reference IP/CIDR ranges
├── Can reference other Security Groups
├── Can be associated with multiple compatible resources
├── Is associated with a VPC
└── Filters traffic before it reaches the EC2 application
```

Default/common behavior to remember:

```text
Inbound
→ No inbound traffic unless allowed by rules

Outbound
→ Allow all traffic by default
```

Important ports:

```text
FTP    → 21
SSH    → 22
SFTP   → 22
HTTP   → 80
HTTPS  → 443
RDP    → 3389
```

Important troubleshooting lesson:

```text
External timeout
+
Application active
+
localhost works
=
Check Security Group / network path
```

---

# 21. Lab Results

```text
[✓] Review Security Group inbound rules
[✓] Verify HTTP port 80
[✓] Access EC2 website successfully
[✓] Remove HTTP port 80
[✓] Observe browser timeout
[✓] Verify Apache remains active
[✓] Test website using localhost
[✓] Restore HTTP port 80
[✓] Verify website becomes accessible again
[✓] Observe SG changes without restarting EC2
[✓] Test SSH source restriction
[✓] Observe EC2 Instance Connect behavior
[✓] Understand 0.0.0.0/0 vs /32
[✓] Review direct SSH using a Key Pair
```

## Status

**EC2 Security Groups & Classic Ports — COMPLETED**
