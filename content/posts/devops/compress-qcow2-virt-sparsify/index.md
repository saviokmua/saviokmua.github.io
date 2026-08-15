---
date: '2026-08-15'
lastmod: '2026-08-15'
draft: false
title: 'How to Compress qcow2 VM Images with virt-sparsify'
slug: 'compress-qcow2-virt-sparsify'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to reduce qcow2 VM disk image size by freeing unused blocks with virt-sparsify. Real-world example showing 67% space reduction on Proxmox."
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

If you see a gap between `disk size` and actual usage inside the VM, use `virt-sparsify` to reclaim the unused blocks.

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

## Step 5: Verify the Result

Check the image again:

```bash
qemu-img info /vz/images/100/vm-100-disk-0.qcow2
```

You now see:

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

VM 100 — clear candidate for sparsify. VM 102 — already optimized. Run `virt-sparsify` on each candidate following the same steps above.

Quick summary of total disk usage:

```bash
du -sh /vz/images/*/
```

---

## Alternative: qemu-img convert

There's another way to reclaim space — convert the image to a fresh qcow2 file:

```bash
qemu-img convert -O qcow2 /vz/images/100/vm-100-disk-0.qcow2 /tmp/vm-100-new.qcow2
```

Then replace the original:

```bash
mv /tmp/vm-100-new.qcow2 /vz/images/100/vm-100-disk-0.qcow2
```

What does this do? It reads the source image block by block and writes only the allocated blocks to a new file. Unused blocks are skipped. The result is a clean qcow2 without stale data.

### When to use qemu-img convert

- When `virt-sparsify` is not available or won't install
- When you want a completely fresh qcow2 structure (e.g., after corruption)
- When converting between formats (raw → qcow2, vmdk → qcow2)

### When virt-sparsify is better

| | virt-sparsify | qemu-img convert |
|---|---|---|
| Modifies in-place | Yes (`--in-place`) | No, creates a new file |
| Requires free space | No | Yes, for the output file |
| Speed | Faster | Slower (full copy) |
| Additional savings after sparsify | — | Minimal |

In our case, after `virt-sparsify` already reduced the image from 14.7 GB to 4.85 GB, running `qemu-img convert` won't give you significant additional savings. The sparsify operation already did the heavy lifting.

---

## Summary

The workflow is simple:

1. Check `qemu-img info` to see virtual vs disk size
2. If disk size is much larger than actual data — run `virt-sparsify --in-place`
3. Verify with `qemu-img info` again

One command, no data loss, your VMs keep working exactly as before.
