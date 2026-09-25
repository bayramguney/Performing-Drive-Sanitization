# Performing-Drive-Sanitization

# Assisted Lab: Performing Drive Sanitization

## Overview

This lab demonstrates the importance of **secure data destruction, drive sanitization, and hardware asset management** in cybersecurity.

As a security team member at **Structureality Inc.**, the objective was to prevent unauthorized access to sensitive information by testing different data removal techniques and understanding which methods provide true protection against data recovery.

During this lab, I performed:

- External drive preparation
- Disk partitioning and formatting
- File deletion and recovery testing
- Secure file deletion using `shred`
- Data recovery analysis using TestDisk
- Drive formatting
- Full drive sanitization using `dd`

---

# Scenario

Organizations must properly sanitize storage devices before:

- Reusing hardware
- Decommissioning systems
- Selling equipment
- Disposing of storage media
- Transferring devices between departments

Simply deleting files does not remove the actual data. Deleted files can often be recovered using forensic tools.

Secure sanitization ensures that sensitive information cannot be reconstructed.

---

# Lab Environment

| Component | Details |
|---|---|
| Virtual Machine | KALI |
| Operating System | Kali Linux |
| User Account | root |
| Password | Pa$$w0rd |
| Storage Device | `/dev/sdb` |
| Drive Size | 80 GiB |
| File System | FAT32 |

---

# CompTIA Security+ Objective

## Security+ SY0-701 Objective 4.2

**Explain the security implications of proper hardware, software, and data asset management.**

Topics covered:

- Data destruction
- Media sanitization
- Hardware lifecycle management
- Secure deletion methods
- Data recovery risks

---

# Tools Used

| Tool | Purpose |
|---|---|
| fdisk | Disk partition management |
| mkfs | File system creation |
| mount | Mount storage devices |
| TestDisk | File recovery analysis |
| shred | Secure file deletion |
| dd | Drive-level sanitization |

---

# Part 1: Prepare Portable Drive

## Objective

Prepare a new external storage device for use across multiple operating systems.

Tasks completed:

- Identified storage device
- Created a partition
- Formatted partition as FAT32
- Mounted the drive
- Copied test files

---

# Identify Storage Device

Displayed available disks:

```bash
fdisk -l
```

Identified:

```
/dev/sdb
```

as the 80 GiB storage device.

---

# Create Partition Using fdisk

Started disk management:

```bash
fdisk /dev/sdb
```

Created a new primary partition:

```
n
```

Selected:

```
Partition type: p
Partition number: 1
```

Accepted default sector values.

Verified partition:

```
p
```

Saved changes:

```
w
```

---

# Format Drive as FAT32

Created FAT32 file system:

```bash
mkfs -t fat /dev/sdb1
```

---

# Mount Storage Device

Created mount directory:

```bash
mkdir /mnt/SalesStorage
```

Mounted drive:

```bash
mount /dev/sdb1 /mnt/SalesStorage
```

Verified contents:

```bash
ls -l /mnt/SalesStorage
```

---

# Copy Test Data

Copied security testing files:

```bash
cp -r /usr/share/seclists/* /mnt/SalesStorage/
```

Verified files:

```bash
ls -l /mnt/SalesStorage
```

---

# Part 2: Delete and Recover Files

## Objective

Demonstrate that normal deletion does not securely remove data.

---

# Delete Directory

Removed the directory:

```bash
rm -r /mnt/SalesStorage/Miscellaneous
```

Verified deletion:

```bash
ls -l /mnt/SalesStorage
```

---

# Recover Deleted Files Using TestDisk

Started recovery tool:

```bash
testdisk /dev/sdb1
```

Recovery process:

1. Selected partition
2. Selected FAT32 filesystem
3. Chose **Undelete**
4. Selected deleted files
5. Copied recovered files to:

```
/root/Downloads
```

Verified recovery:

```bash
ls -l Downloads/Miscellaneous
```

---

# Lesson Learned

Normal deletion is not secure.

When files are deleted:

- File references are removed
- Storage locations are marked available
- Original data remains until overwritten

Recovery tools can often restore deleted files.

---

# Part 3: Secure File Deletion Using shred

## Objective

Securely destroy file contents and prevent recovery.

---

# View Sensitive Files

Displayed directory contents:

```bash
ls -lR /mnt/SalesStorage/Passwords
```

Viewed sample file:

```bash
cat /mnt/SalesStorage/Passwords/bt4-password.txt
```

---

# Secure Delete Using shred

Used:

```bash
find /mnt/SalesStorage/Passwords -type f -exec shred -uvz {} \;
```

Explanation:

| Option | Meaning |
|---|---|
| -u | Remove file after shredding |
| -v | Show progress |
| -z | Add final overwrite with zeros |

---

# Verify Destruction

Checked directory:

```bash
ls -lR /mnt/SalesStorage/Passwords
```

Attempted recovery:

```bash
testdisk /dev/sdb1
```

Recovered file contents were empty.

Example:

```bash
cat Downloads/Passwords/bt4-password.txt
```

Result:

```
No data available
```

---

# Lesson Learned

`shred` destroys file contents by overwriting data.

However:

- File names may remain
- Directory structure may remain
- Metadata may still exist

---

# Part 4: Format Drive

## Objective

Test whether formatting provides secure destruction.

---

# Unmount Drive

```bash
umount /mnt/SalesStorage
```

---

# Format Drive

```bash
mkfs -t fat /dev/sdb1
```

---

# Test Recovery

Attempted recovery:

```bash
testdisk /dev/sdb1
```

Result:

```
No file found. Filesystem may be damaged
```

---

# Lesson Learned

Formatting removes file system information but does not guarantee secure destruction.

Advanced forensic tools may still recover information.

Formatting is:

- Faster than manual deletion
- Suitable for low-sensitivity data
- Not a complete sanitization method

---

# Part 5: Drive Sanitization

## Objective

Completely overwrite storage media to prevent recovery.

---

# Restore Test Data

Mounted drive:

```bash
mount /dev/sdb1 /mnt/SalesStorage
```

Copied files:

```bash
cp -r /usr/share/seclists/* /mnt/SalesStorage/
```

---

# Perform Full Drive Overwrite

Used:

```bash
dd if=/dev/zero of=/dev/sdb bs=1M status=progress
```

Explanation:

| Command | Purpose |
|---|---|
| dd | Disk duplication utility |
| if=/dev/zero | Input source of zeros |
| of=/dev/sdb | Target drive |
| bs=1M | Block size |
| status=progress | Shows progress |

---

# Verify Drive Sanitization

Opened disk utility:

```bash
fdisk /dev/sdb
```

Checked partition table:

```
p
```

Verified:

```
v
```

Result:

- No partition table exists
- Previous data is removed

---

# Recovery Testing After Sanitization

Attempted:

```bash
testdisk /dev/sdb
```

Analysis showed:

```
No partitions found
```

The drive was successfully sanitized.

---

# Data Destruction Comparison

| Method | Data Recovery Possible? | Secure Sanitization? |
|---|---|---|
| Delete file | Yes | ❌ No |
| Format drive | Sometimes | ❌ No |
| shred | Very difficult | ⚠ Partial |
| dd overwrite | Extremely difficult | ✅ Yes |

---

# Security Concepts Learned

## Data Sanitization

Data sanitization is the process of permanently removing information from storage devices.

Common methods:

- Overwriting
- Cryptographic erase
- Physical destruction
- Secure wiping tools

---

## Hardware Asset Management

Organizations should sanitize devices before:

- Disposal
- Resale
- Recycling
- Employee reassignment

---

# Real-World Cybersecurity Applications

These skills apply to:

## Security Analysts

- Protecting sensitive information
- Investigating data exposure risks
- Understanding recovery techniques

## System Administrators

- Preparing retired hardware
- Managing storage lifecycle
- Implementing secure disposal procedures

## Incident Response Teams

- Preventing data leakage
- Handling compromised systems
- Preserving security controls

---

# Skills Demonstrated

✅ Linux disk management  
✅ Storage partitioning  
✅ FAT32 formatting  
✅ File recovery testing  
✅ Secure file deletion  
✅ Data sanitization  
✅ Hardware lifecycle security  
✅ Security+ asset management concepts  

---

# Key Takeaways

- Deleting files does not destroy data.
- Formatting is not the same as sanitization.
- Secure overwrite methods prevent data recovery.
- Proper media sanitization protects organizational information.

---

# Conclusion

This lab provided practical experience with Linux-based storage management and data destruction techniques.

By comparing deletion, recovery, shredding, formatting, and full-drive overwriting, I gained a better understanding of how cybersecurity professionals protect sensitive data throughout the hardware lifecycle.

These concepts directly support **CompTIA Security+ objectives and real-world security operations practices**.
