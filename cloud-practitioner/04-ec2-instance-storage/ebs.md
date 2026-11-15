# EBS (Elastic Block Store)

## 1. What's an EBS Volume?

- An EBS ( Elastic Block Store ) Volumn is a network drive you can attach to your instances while they run.
- It allows your instances to persist data, even after their termination
- They can only be mounted to one instance at a time
- They are bound to a specific availabilit zone

## 2. EBS Volume

- It's a network drive ( not a physical drive )
  It uses the network to communicate the instance, which means there might be a bit of latency
  It can be detached from an EC2 instance and attached to another one quickly
- It's locked to an AZ
  To move a volume across, you first need to snapshot it
- Have a provisioned capacity ( size in GBs, and IOPS )
  You get billed for all the provisoned capacity
  You can increase the capacity of the drive over time

## 3. EBS - Delete on Termination attribute

- Controls the EBS behavior when an EC2 instance terminates
  By default, the root EBS volume is deleted ( attribute enabled )
  By default, any other attached EBS volume is not deleted ( attribute disabled )
- This can be controlled by the AWS console / AWS CLI
- Use case
  Preserve root volume when instance is terminated

## 4. Hands-on Steps

### Step 1 — Launch an EC2 instance

1. Open **AWS Console → EC2 → Instances → Launch instances**.
2. Create an instance named `ec2-ebs-lab` using **Amazon Linux 2023**.
3. Configure a suitable instance type and a **root EBS volume of 8 GiB**.
4. Ensure **Delete on Termination** is enabled for the root volume.
5. Launch the instance and confirm its state is **Running**.
6. Record its **Availability Zone**.

> Connection was performed through **EC2 Instance Connect** in the AWS Console. Connection prerequisites (network, security group, public connectivity where applicable) must be satisfied.

### Step 2 — Create a data EBS volume

1. Open **EC2 → Elastic Block Store → Volumes → Create volume**.
2. Configure:
   - **Type:** `gp3`
   - **Size:** `2 GiB`
   - **Availability Zone:** Same AZ as `ec2-ebs-lab`
   - **Name tag:** `ebs-data-lab`
3. Create the volume.
4. Verify that its state is **Available** (not attached to an instance).

### Step 3 — Attach the data volume

1. Select `ebs-data-lab` in **Volumes**.
2. Choose **Actions → Attach volume**.
3. Select `ec2-ebs-lab` and use the suggested device name (for example `/dev/sdf`).
4. Confirm that the volume state changes to **In-use**.

> On Nitro-based instances, the device can appear inside Linux as `/dev/nvme...`, even if `/dev/sdf` was selected in the Console.

### Step 4 — Inspect the block devices

Connect using **EC2 Instance Connect** and run:

```bash
lsblk
```

Observed devices:

```text
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk
├─nvme0n1p1   259:1    0   8G  0 part /
├─nvme0n1p127 259:2    0   1M  0 part
└─nvme0n1p128 259:3    0  10M  0 part /boot/efi
nvme1n1       259:4    0   2G  0 disk
```

- `/dev/nvme0n1` was the **8 GiB root disk**.
- `/dev/nvme1n1` was the **2 GiB data disk**, initially without a mountpoint.

### Step 5 — Check and create a filesystem

Before formatting, inspect the data disk:

```bash
sudo file -s /dev/nvme1n1
```

The volume had **no existing filesystem**. For this new, empty lab volume only, create an XFS filesystem:

```bash
sudo mkfs -t xfs /dev/nvme1n1
sudo file -s /dev/nvme1n1
```

The second command should identify **XFS filesystem data**.

> **Warning:** `mkfs` can destroy existing data. Never format the root disk (`/dev/nvme0n1`). Do **not** format a previously used volume merely because it is not mounted.

### Step 6 — Mount the data volume

```bash
sudo mkdir -p /data
sudo mount /dev/nvme1n1 /data
lsblk
df -h /data
```

After mounting, `/dev/nvme1n1` should show `/data` under **MOUNTPOINTS**.

**Important distinction:**

1. **Attach:** Connect the block device to EC2.
2. **Format:** Create a filesystem if the new disk does not already have one.
3. **Mount:** Make the filesystem accessible at a Linux directory such as `/data`.

A volume being **In-use** does not by itself mean the filesystem is ready for ordinary file access.

### Step 7 — Write and read a test file

```bash
echo "Hello AWS EBS Storage!" | sudo tee /data/test-ebs.txt
ls -l /data
cat /data/test-ebs.txt
df -h /data
```

Expected file content:

```text
Hello AWS EBS Storage!
```

### Step 8 — Unmount and detach

First unmount the filesystem safely:

```bash
sudo umount /data
lsblk
```

Verify that the 2 GiB disk no longer shows `/data` as its mountpoint.

Then in **EC2 → Volumes**, select `ebs-data-lab` → **Actions → Detach volume**. Wait until the state returns to **Available**.

**Observed behavior:** EC2 continued running, while the detached EBS volume remained available with its stored data.

### Step 9 — Reattach and verify persistence

1. In **EC2 → Volumes**, attach `ebs-data-lab` back to `ec2-ebs-lab` in the same AZ.
2. Confirm the volume is **In-use**.
3. In EC2 Instance Connect, identify the correct data disk:

```bash
lsblk -f
```

4. If the device is still `/dev/nvme1n1`, mount it again:

```bash
sudo mount /dev/nvme1n1 /data
cat /data/test-ebs.txt
```

The test file should still contain:

```text
Hello AWS EBS Storage!
```

**Do not run `mkfs` again.** The existing XFS filesystem and data persist across detach and reattach. Device names can change after reattachment; verify the device before mounting.

### Step 10 — Inspect Delete on Termination

In **EC2 → Instances → select `ec2-ebs-lab` → Storage / block device settings**, inspect the attached volumes and their deletion settings.

For this lab:

- **Root volume (8 GiB):** Delete on Termination enabled.
- **Data volume (2 GiB):** Preserved when the instance was terminated.

### Step 11 — Cleanup and confirm behavior

1. In **EC2 → Instances**, terminate `ec2-ebs-lab`.
2. In **EC2 → Volumes**, check that the **8 GiB root EBS** is automatically deleted.
3. Confirm that the **2 GiB data EBS** remains available after EC2 termination.
4. Manually delete `ebs-data-lab` once no longer needed.

**Actual cleanup results reported:**

| Resource          | Result                               |
| ----------------- | ------------------------------------ |
| EC2 `ec2-ebs-lab` | Terminated                           |
| Root EBS (8 GiB)  | Automatically deleted                |
| Data EBS (2 GiB)  | Preserved after instance termination |
| Data EBS (2 GiB)  | Manually deleted afterward           |
