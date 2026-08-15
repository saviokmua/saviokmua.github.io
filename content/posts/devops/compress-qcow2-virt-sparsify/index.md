---
date: '2026-08-15'
lastmod: '2026-08-15'
draft: false
title: 'How to Compress qcow2 VM Images with virt-sparsify'
slug: 'compress-qcow2-virt-sparsify'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to reduce qcow2 VM disk image size by freeing unused blocks with virt-sparsify and qemu-img convert. Real-world example showing 67% space reduction on Proxmox."
keywords: ['qcow2 compress', 'virt-sparsify', 'qcow2 optimization', 'proxmox disk space', 'vm disk compression', 'qemu-img']
tags: ['qcow2', 'virt-sparsify', 'proxmox', 'vm', 'disk optimization']
categories: ['DevOps']
hiddenFromHomePage: false
featuredImage: ''
---

### How to Compress qcow2 VM Images with virt-sparsify

---

## Step 1: We Have a VM Disk Image

Suppose you have a Proxmox VM with ID 100. Its disk image is located at:

```
/vz/images/100/vm-100-disk-0.qcow2
```

You suspect that the qcow2 file is larger than it should be — the VM used to have more data, but some of it was deleted. The blocks, however, were never reclaimed by the host.

---

## Step 2: Check the Image Info

Run:

```bash
qemu-img info /vz/images/100/vm-100-disk-0.qcow2
```

You get:

```
image: vm-100-disk-0.qcow2
file format: qcow2
virtual size: 30 GiB (32212254720 bytes)
disk size:    14.7 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
```

Also check the actual file size on disk:

```bash
ls -lh /vz/images/100/vm-100-disk-0.qcow2
du -h /vz/images/100/vm-100-disk-0.qcow2
```

---

## Step 3: Pay Attention to Two Key Values

From the `qemu-img info` output, focus on these two lines:

```
virtual size: 30 GiB
disk size:    14.7 GiB
```

What do they mean?

- **virtual size: 30 GiB** — this is the disk size the VM sees. The guest OS thinks it has a 30 GB drive.

- **disk size: 14.7 GiB** — this is how much space the qcow2 file actually occupies on the Proxmox host.

If `disk size` is significantly smaller than `virtual size` — that's normal. But if `disk size` is much larger than the actual data inside the VM — you have wasted space.

In our case: the VM only uses about 5 GB of real data, but the qcow2 file holds 14.7 GB. That's ~10 GB of stale blocks that can be reclaimed.

---

## Step 4: If There's Waste — Run virt-sparsify

If you see a gap between `disk size` and actual usage inside the VM, use `virt-sparsify` to zero out the unused blocks.

First, install the tool (on Proxmox host):

```bash
apt install libguestfs-tools
```

Stop the VM (the disk must not be in use):

```bash
qm stop 100
```

Run sparsify in-place:

```bash
virt-sparsify --in-place /vz/images/100/vm-100-disk-0.qcow2
```

Wait for completion:

```
Sparsify in-place operation completed with no errors
```

---

## Step 5: Check After virt-sparsify

Check the image after sparsify:

```bash
qemu-img info /vz/images/100/vm-100-disk-0.qcow2
```

You see:

```
virtual size: 30 GiB
disk size:    14.7 GiB
```

**The file size on disk did NOT change.** This is important.

What `virt-sparsify --in-place` actually did — it zeroed out the unused blocks inside the qcow2 file. The file is still 14.7 GB, but now it contains lots of zero blocks. qcow2 doesn't automatically shrink when blocks are zeroed — it just marks them as sparse.

To actually reduce the file size, you need the next step.

---

## Step 6: Run qemu-img convert to Actually Shrink the File

Now convert the image to a fresh qcow2 file — this will skip the zeroed blocks and create a smaller file:

```bash
qemu-img convert -O qcow2 /vz/images/100/vm-100-disk-0.qcow2 /tmp/vm-100-new.qcow2
```

Wait for it to finish, then replace the original:

```bash
mv /tmp/vm-100-new.qcow2 /vz/images/100/vm-100-disk-0.qcow2
```

---

## Step 7: Verify the Final Result

Check the image one more time:

```bash
qemu-img info /vz/images/100/vm-100-disk-0.qcow2
```

Now you see:

```
image: vm-100-disk-0.qcow2
file format: qcow2
virtual size: 30 GiB (32212254720 bytes)
disk size:    4.85 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false
```

Compare:

| Metric | Before | After |
|--------|--------|-------|
| Virtual size | 30 GiB | 30 GiB |
| Disk size | 14.7 GiB | 4.85 GiB |
| Space saved | — | ~9.85 GiB (67%) |

The virtual size stayed the same — the VM still sees a 30 GB disk. But the physical footprint on the host dropped from 14.7 GB to 4.85 GB.

---

## The Full Workflow

The complete process is two commands:

```bash
# Step 1: Zero out unused blocks inside the image
virt-sparsify --in-place /vz/images/100/vm-100-disk-0.qcow2

# Step 2: Re-create the qcow2 file, skipping zeroed blocks
qemu-img convert -O qcow2 /vz/images/100/vm-100-disk-0.qcow2 /tmp/vm-100-new.qcow2 && \
mv /tmp/vm-100-new.qcow2 /vz/images/100/vm-100-disk-0.qcow2
```

`virt-sparsify` alone does NOT reduce the file size — it only marks blocks as sparse. `qemu-img convert` is what actually creates the smaller file.

---

## Bonus: Check All VMs at Once

To scan all qcow2 images on your storage and find which ones waste the most space:

```bash
for f in /vz/images/*/vm-*.qcow2; do
  echo "=== $f ==="
  qemu-img info "$f" | grep -E "virtual size|disk size"
  echo
done
```

Example output:

```
=== /vz/images/100/vm-100-disk-0.qcow2 ===
virtual size: 30 GiB (32212254720 bytes)
disk size:    14.7 GiB

=== /vz/images/101/vm-101-disk-0.qcow2 ===
virtual size: 50 GiB (53687091200 bytes)
disk size:    42.3 GiB

=== /vz/images/102/vm-102-disk-0.qcow2 ===
virtual size: 20 GiB (21474836480 bytes)
disk size:    1.2 GiB
```

VM 100 — clear candidate for compression. VM 102 — already optimized. Run the two-step process on each candidate.

Quick summary of total disk usage:

```bash
du -sh /vz/images/*/
```

---

## Summary

| Step | Command | What it does |
|------|---------|--------------|
| 1 | `virt-sparsify --in-place` | Zeroes out unused blocks inside qcow2 |
| 2 | `qemu-img convert -O qcow2` | Creates a new, smaller qcow2 file |

**Important:** `virt-sparsify` alone does NOT reduce the file size. You must run `qemu-img convert` after it to actually reclaim the space.

One command cleans the inside, the second command shrinks the file. Together they give you the full 67% reduction.
