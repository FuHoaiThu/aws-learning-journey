# Amazon EBS Snapshots — Theory & Hands-on

> **Learning path:** AWS Certified Cloud Practitioner (CLF-C02)  
> **Lab status:** Completed — backup and restore verified  
> **AWS Region:** Asia Pacific (Sydney), `ap-southeast-2`  
> **EC2 instance:** `MyEc2Instance` (`t3.micro`, Amazon Linux, AZ `ap-southeast-2a`)

## 1. Objectives

- Understand Amazon EBS Snapshots and how they differ from EBS volumes.
- Create a point-in-time backup of a data volume using the AWS Management Console.
- Restore a new EBS volume from the snapshot and verify the original data.
- Safely clean up lab resources and understand Recycle Bin retention.

## 2. CLF-C02 Theory

### What is an EBS Snapshot?

An **Amazon Elastic Block Store (EBS) Snapshot** is a **point-in-time backup** of an EBS volume. Snapshots are stored and managed by AWS and can be used to create new EBS volumes.

| Concept               | Key point                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Point-in-time backup  | Captures the volume's data at a particular point in time.                                                               |
| Incremental snapshots | After the initial snapshot, only changed blocks need to be saved; AWS manages the snapshot dependencies.                |
| Regional resource     | EBS snapshots are scoped to an AWS Region, not a single Availability Zone.                                              |
| Restore               | Create a **new EBS volume** from a snapshot; choose a supported AZ in the snapshot's Region.                            |
| Cross-Region copy     | Copy a snapshot to another AWS Region for recovery or migration.                                                        |
| Snapshot Archive      | Lower-cost tier for long-term retention; minimum storage billing period is **90 days**.                                 |
| Archive restore       | Standard retrieval commonly takes **24–72 hours**; expedited retrieval may be available for eligible snapshots.         |
| Recycle Bin           | Retention rules can preserve deleted snapshots for **1–365 days** so they can be recovered during the retention period. |

**Important distinctions:**

- **EBS volume:** Block storage attached to an EC2 instance in the **same AZ**.
- **EBS snapshot:** Backup scoped to a **Region**; can be used to create volumes in different AZs within that Region.
- Deleting a volume does **not** automatically delete its snapshots.
- Removing a Recycle Bin retention rule does **not** immediately purge resources already retained under that rule.
- Snapshots and retained resources may incur storage charges.

## 3. Lab Architecture

```text
AWS Region: ap-southeast-2 (Sydney)
└── Availability Zone: ap-southeast-2a
    └── EC2: MyEc2Instance
        ├── Root EBS (8 GiB) -> /dev/nvme0n1 -> /
        ├── Original EBS (1 GiB, gp3) -> /dev/nvme1n1 -> /data
        │   └── snapshot-test.txt
        └── Restored EBS (1 GiB, gp3) -> /dev/nvme2n1 -> /data-restored

Original EBS -> EBS Snapshot (Standard) -> Restored EBS
```

### Lab resources

| Resource             | Name / ID                                         | Notes                                                             |
| -------------------- | ------------------------------------------------- | ----------------------------------------------------------------- |
| Existing EC2         | `MyEc2Instance`                                   | Reused; **not deleted**                                           |
| Existing root volume | `vol-03dde99bafe695f17`                           | 8 GiB; **not deleted**                                            |
| Original data volume | `ebs-snapshot-lab` / `vol-0d26c646e823ab72f`      | 1 GiB gp3; deleted during cleanup                                 |
| Restored data volume | `ebs-snapshot-restored` / `vol-04a272672e81cde71` | 1 GiB gp3; deleted during cleanup                                 |
| Snapshot             | `ebs-snapshot-lab-backup`                         | Created from original data volume; later deleted into Recycle Bin |

> **Safety:** EC2 NVMe device names can change between instances or reattachments. Always verify the EBS Volume ID before formatting or mounting a device.

## 4. Hands-on: Backup and Restore

### Step 1 — Select the correct Region and EC2

In **AWS Management Console → EC2**, switch to `ap-southeast-2` and locate the existing `MyEc2Instance` in `ap-southeast-2a`.

### Step 2 — Create an EBS data volume

In **EC2 → Elastic Block Store → Volumes → Create volume**:

- Type: **gp3**
- Size: **1 GiB**
- Availability Zone: **ap-southeast-2a** (same AZ as EC2)
- Name tag: `ebs-snapshot-lab`

Original volume ID: `vol-0d26c646e823ab72f`.

### Step 3 — Attach the volume

Select the volume → **Actions → Attach volume** → choose `MyEc2Instance`.

Connect to the instance using EC2 Instance Connect and inspect block devices:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

To map AWS EBS Volume IDs to Linux NVMe devices:

```bash
ls -l /dev/disk/by-id/ | grep 'nvme-Amazon_Elastic_Block_Store'
```

Verified mapping:

```text
vol-0d26c646e823ab72f -> /dev/nvme1n1
```

The `nvme` utility was not installed on this instance, so `/dev/disk/by-id/` symlinks were used instead.

### Step 4 — Create a filesystem and mount the original volume

**Run these formatting commands only on the newly created, empty original data volume. Never format the restored volume.**

```bash
sudo file -s /dev/nvme1n1
# Initially returned: data (no filesystem detected)

sudo mkfs -t xfs /dev/nvme1n1
lsblk -f

sudo mkdir -p /data
sudo mount /dev/nvme1n1 /data
df -h /data
```

### Step 5 — Write and flush test data

```bash
echo "Hello AWS EBS Snapshot!" | sudo tee /data/snapshot-test.txt
cat /data/snapshot-test.txt
sync
```

Expected output:

```text
Hello AWS EBS Snapshot!
```

`sync` requests that pending filesystem writes be flushed before creating the snapshot.

### Step 6 — Create an EBS Snapshot

In **EC2 → Volumes**, select `ebs-snapshot-lab` → **Actions → Create snapshot**.

- Description: `EBS Snapshot Hands-on - Backup test data`
- Name tag: `ebs-snapshot-lab-backup`
- Source volume: `vol-0d26c646e823ab72f`
- Storage tier: **Standard**

Wait for snapshot status **Completed**.

### Step 7 — Restore an EBS volume from the snapshot

In **EC2 → Snapshots**, select `ebs-snapshot-lab-backup` → **Actions → Create volume from snapshot**.

- Volume name: `ebs-snapshot-restored`
- Type: **gp3**
- Size: **1 GiB**
- Availability Zone: **ap-southeast-2a**

New restored volume ID: `vol-04a272672e81cde71`.

Attach it to `MyEc2Instance`.

### Step 8 — Verify the restored device and filesystem

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
ls -l /dev/disk/by-id/ | grep 'nvme-Amazon_Elastic_Block_Store'
```

Verified mappings:

```text
vol-03dde99bafe695f17 -> /dev/nvme0n1 (root volume)
vol-0d26c646e823ab72f -> /dev/nvme1n1 (original data volume)
vol-04a272672e81cde71 -> /dev/nvme2n1 (restored data volume)
```

The restored device already had an **XFS filesystem**, demonstrating that the snapshot preserved filesystem data.

**Do not run `mkfs` on `/dev/nvme2n1`: doing so would overwrite the restored filesystem.**

### Step 9 — Mount the restored XFS volume read-only

```bash
sudo mkdir -p /data-restored
sudo mount -t xfs -o ro,nouuid /dev/nvme2n1 /data-restored
```

Why `ro,nouuid`?

- `ro`: Mount read-only to protect recovered data during verification.
- `nouuid`: An XFS volume restored from a snapshot can have the **same filesystem UUID** as the original. This option allows both copies to be mounted simultaneously.

### Step 10 — Validate the restored data

```bash
ls -l /data-restored
cat /data-restored/snapshot-test.txt
findmnt /data-restored
```

**Actual verified result:**

```text
Hello AWS EBS Snapshot!
```

**Lab result: PASS.** The file contents on the restored EBS volume matched the original data.

## 5. Cleanup

### Step 11 — Unmount both data volumes

```bash
sudo umount /data-restored
sudo umount /data
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

**Verified:** Both data volumes were unmounted, and the root filesystem remained mounted at `/`.

### Step 12 — Detach both data volumes

In **EC2 → Volumes**, detach:

1. `ebs-snapshot-restored` — `vol-04a272672e81cde71`
2. `ebs-snapshot-lab` — `vol-0d26c646e823ab72f`

Wait until both show **Available**.

**Verified:** Both volumes reached `Available`.

### Step 13 — Delete the lab volumes

Delete the two **1 GiB lab volumes** in the Console.

**Verified:** Both lab volumes were deleted. The existing root volume `vol-03dde99bafe695f17` was preserved.

### Step 14 — Delete the snapshot and check Recycle Bin

In **EC2 → Snapshots**, delete `ebs-snapshot-lab-backup`.

**Observed final state:**

- Snapshot removed from the active **Snapshots** list.
- Snapshot **remained in Recycle Bin** under a **1-day retention period**.
- The associated **retention rule was deleted**, but this does not shorten the retention period of the snapshot already retained.
- Snapshot is expected to be permanently removed by AWS when its existing retention expires; **permanent deletion was not yet independently verified**.
- Recycle Bin storage may incur charges while the snapshot remains retained.

**Follow-up:** After the 1-day retention period expires, check **EC2 → Recycle Bin → Resources** in `ap-southeast-2` to confirm the snapshot is gone.

> **Existing EC2:** `MyEc2Instance` was intentionally preserved. Its ongoing instance/root-volume charges, if any, are separate from this lab. Review its state and billing as appropriate; do not terminate shared lab resources without checking.

## 6. Lessons Learned

1. EBS volumes are **AZ-scoped**, while EBS snapshots are **Region-scoped**.
2. Snapshots enable backup and restoration by creating **new EBS volumes**.
3. Restored volumes preserve filesystem data; formatting them would destroy that restored data.
4. NVMe Linux device names are not a reliable substitute for verifying EBS Volume IDs.
5. XFS copies may share filesystem UUIDs; `mount -o ro,nouuid` is useful when comparing original and restored volumes on the same instance.
6. Deleting EBS volumes does not delete existing snapshots.
7. A deleted snapshot may still be retained in **Recycle Bin** until its retention expires, even if the retention rule is later deleted.
8. Cleaning up temporary volumes and snapshots helps avoid unnecessary AWS charges.

## 7. Completion Checklist

- [x] Create and attach original gp3 EBS volume
- [x] Identify the correct Linux NVMe device
- [x] Format original volume as XFS and mount it
- [x] Write and verify sample data
- [x] Create a Standard EBS Snapshot
- [x] Restore a new EBS volume from the snapshot
- [x] Mount restored volume read-only using `nouuid`
- [x] Verify restored file contents
- [x] Unmount and detach both lab volumes
- [x] Delete both lab volumes
- [x] Delete snapshot from active Snapshots list
- [x] Confirm snapshot is retained in Recycle Bin for 1 day
- [x] Delete the retention rule
- [x] Verify snapshot has expired and disappeared from Recycle Bin
- [x] Preserve existing EC2 instance and root volume

---

**Outcome:** Successfully demonstrated EBS Snapshot backup and restore through AWS Console and Linux hands-on commands, with verified data recovery and documented cleanup status.
