# Amazon Machine Image (AMI) — Theory & Hands-on

## 1. Learning objectives

- Understand what an Amazon Machine Image (AMI) is.
- Distinguish AMIs from EBS snapshots and EC2 User Data.
- Create a custom AMI from an existing Amazon Linux EC2 instance.
- Launch a new EC2 instance from the AMI and verify that software, configuration, and files are preserved.
- Clean up lab resources to reduce unnecessary charges.

## 2. AMI fundamentals

An **Amazon Machine Image (AMI)** is a template used to launch Amazon EC2 instances. It includes the information required to boot an instance, such as an operating system, software and configuration stored on included volumes, and block device mappings.

### Common AMI sources

- **AWS-provided AMIs** — for example, Amazon Linux.
- **Custom AMIs** — created from an instance configured for a particular workload.
- **AWS Marketplace AMIs** — images offered by third-party publishers.

### AMI versus EBS Snapshot versus EC2 User Data

| Feature | Purpose |
| --- | --- |
| AMI | Template for launching EC2 instances with a prepared environment. |
| EBS Snapshot | Point-in-time backup of an EBS volume, used to create new volumes and to back EBS-based AMIs. |
| EC2 User Data | Bootstrapping instructions supplied to an EC2 instance, commonly processed by cloud-init on launch. |

**Important:** Creating a new instance from an AMI does **not** automatically copy the original instance's **User Data attribute**. However, files and software created by earlier User Data scripts may be present in the AMI if stored on included EBS volumes. Cloud-init state or caches may also exist on the image; this is distinct from copying the User Data attribute.

### Regional scope

AMIs are regional resources. An AMI can be copied to another AWS Region when needed.

## 3. Lab environment

| Resource | Value |
| --- | --- |
| AWS Region | `ap-southeast-2` (Sydney) |
| Original EC2 instance | `MyEc2Instance` |
| Operating system | Amazon Linux |
| Original web server | Apache (`httpd`), port `80` |
| Additional web server | Nginx, port `8080` |
| Custom AMI name | `custom-nginx-ami-lab` |
| Test EC2 instance | `EC2-From-Custom-AMI` |
| Test security group | `custom-ami-lab-sg` |

> The original EC2 instance was reused from previous labs. Do not terminate it as part of this lab.

## 4. Hands-on steps

### Step 1 — Verify the source instance

In **AWS Console → EC2 → Instances**, confirm `MyEc2Instance` is running and connect using EC2 Instance Connect.

Identify the operating system:

```bash
cat /etc/os-release
```

### Step 2 — Install and configure Nginx

Check whether Nginx is installed:

```bash
nginx -v
```

For **Amazon Linux 2023**, install it if necessary:

```bash
sudo dnf install -y nginx
```

> Installation commands can differ on Amazon Linux 2. Confirm the OS version first.

An initial attempt to start Nginx failed:

```bash
sudo systemctl start nginx
sudo systemctl status nginx --no-pager -l
```

The error was:

```text
bind() to 0.0.0.0:80 failed (98: Address already in use)
```

Identify the process listening on port 80:

```bash
sudo ss -ltnp | grep ':80 '
```

**Finding:** Apache (`httpd`) already used port `80`.

To avoid disrupting Apache, configure Nginx to listen on port `8080` instead.

Check port availability:

```bash
sudo ss -ltnp | grep ':8080 '
```

Back up and edit the Nginx configuration:

```bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
sudo nano /etc/nginx/nginx.conf
```

Change the server block from:

```nginx
listen       80;
listen       [::]:80;
```

to:

```nginx
listen       8080;
listen       [::]:8080;
```

Verify that the file was saved:

```bash
sudo grep -n 'listen' /etc/nginx/nginx.conf
sudo nginx -t
```

Start Nginx and enable it at boot:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx
curl -I http://localhost:8080
```

**Observed result:** Nginx started successfully after changing the port.

### Step 3 — Create a custom HTML page

Confirm the configured document root:

```bash
sudo grep -n 'root ' /etc/nginx/nginx.conf
```

For the default document root `/usr/share/nginx/html`, create the test page:

```bash
sudo tee /usr/share/nginx/html/index.html > /dev/null <<'EOF_HTML'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AWS Custom AMI Lab</title>
</head>
<body>
    <h1>Hello from Custom AMI!</h1>
    <p>This website is running on Amazon EC2 with Nginx.</p>
    <p>Created for AWS AMI Hands-on Lab.</p>
</body>
</html>
EOF_HTML
```

Test locally:

```bash
curl http://localhost:8080
sudo systemctl is-enabled nginx
```

**Observed result:** The page was served successfully, and Nginx was enabled to start at boot.

### Step 4 — Create the custom AMI

1. Open **EC2 → Instances** in `ap-southeast-2`.
2. Select `MyEc2Instance`.
3. Choose **Actions → Image and templates → Create image**.
4. Set the image name to `custom-nginx-ami-lab`.
5. Keep the required EBS volumes included.
6. Leave **No reboot** unchecked if a reboot is acceptable; this allows AWS to reboot the source instance for improved file-system consistency.
7. Create the image.
8. Open **EC2 → AMIs** and wait until its state becomes **Available**.

**Observed result:** The custom AMI reached **Available**.

> Creating an EBS-backed AMI involves EBS snapshots. A reboot may briefly interrupt services on the source instance. Plan this for a suitable maintenance window if the instance hosts important workloads.

### Step 5 — Launch a new instance from the AMI

1. Open **EC2 → AMIs → Owned by me**.
2. Select `custom-nginx-ami-lab`.
3. Choose **Launch instance from AMI**.
4. Name the new instance `EC2-From-Custom-AMI`.
5. Select a compatible instance type and suitable key pair.
6. Configure a public IPv4 address if browser access is needed.
7. Configure a security group to allow required access, including TCP port `8080` from **My IP**.
8. Leave **User Data** empty.
9. Launch and wait for **Running**.

Connect to the new instance and run:

```bash
nginx -v
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx
sudo grep -n 'listen' /etc/nginx/nginx.conf
curl http://localhost:8080
```

Optionally test from a browser:

```text
http://<PUBLIC-IP>:8080
```

**Observed result:** Nginx was already installed and running, its port `8080` configuration was retained, and the custom HTML page appeared without reinstalling or recreating anything.

### Step 6 — Clean up resources

1. **Terminate** `EC2-From-Custom-AMI` only.
2. **Deregister** `custom-nginx-ami-lab`.
3. **Delete associated EBS snapshots** no longer needed.
4. Delete the lab security group `custom-ami-lab-sg` if it is no longer in use.
5. Preserve `MyEc2Instance`; **Stop** it when not in use to avoid EC2 compute charges.
6. Check for any remaining EBS volumes, snapshots, public IPv4 allocations, and Recycle Bin retention.

**Reported result:** The test instance was terminated, the AMI and snapshots were deleted, and security group cleanup was completed.

## 5. Lessons learned

1. **An AMI captures a prepared environment.** The new EC2 instance inherited installed software, saved configuration, and the website files.
2. **Port conflicts prevent services from starting.** Apache was already bound to TCP port `80`, so Nginx was moved to `8080`.
3. **Validate configuration changes.** Checking `grep`, `nginx -t`, `systemctl`, `ss`, and `curl` helped isolate and resolve the startup failure.
4. **Service startup settings matter.** `systemctl enable nginx` helps the service start automatically after boot.
5. **AMI is different from User Data.** User Data is not automatically copied as a new instance attribute, even though its effects on included disks can persist.
6. **Cleanup matters.** Stopped EC2 instances do not incur ordinary instance compute charges, but EBS storage, snapshots, and some IP resources can still incur charges.

## 6. Final outcome

**Status: Completed.** Successfully created a custom AMI from an existing Amazon Linux EC2 instance, launched a new instance from that image, verified that Nginx and the HTML page were preserved, and cleaned up the test resources.