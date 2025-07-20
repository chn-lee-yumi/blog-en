---
title: "Linux SWAP Tip: Boost Performance with ZRAM"
description: "Improve SWAP performance by placing it on ZRAM – compressed memory that's significantly faster than traditional disk-based SWAP."
date: 2023-05-22T12:21:12+08:00
categories:
  - Tech Tips
tags:
  - Linux
---

## Key Takeaway

**ZRAM** (compressed RAM) allows you to dramatically improve your system's swap performance by placing your swap space on a block of compressed memory instead of a slow hard disk.

Swap space is used when your physical RAM is nearly full. When more memory is needed—for example, when opening a new tab in Chrome—but your RAM is exhausted, the operating system moves inactive memory data (such as a tab not being used) to swap space. The combined total of RAM and swap is referred to as **virtual memory**. So effectively, swap expands usable memory.

However, traditional swap space resides on the hard drive. And that’s where problems begin—hard drives are significantly slower than memory. When swapping starts, the system often slows down drastically due to high latency and increased I/O load.

But what if we could expand memory **without** that performance penalty?

That’s exactly what **ZRAM** achieves.

---

## What is ZRAM?

ZRAM allows the Linux kernel to create a block device that uses compressed RAM instead of physical storage. It’s already used in various devices like routers, Raspberry Pi, Orange Pi, NanoPi, and even smartphones.

Because it resides in memory and uses compression, ZRAM offers much better read/write speeds and latency compared to traditional swap on hard disks.

You can place your swap space on this compressed RAM block, gaining the benefit of extended memory capacity with little performance degradation.

Here’s a simplified performance comparison using the `fio` tool:

- **tmpfs** (pure memory) is the fastest.
- **ZRAM** comes next – slightly slower than tmpfs but still significantly faster than hard disk.
- **HDD/SSD** is by far the slowest.

ZRAM’s compressed memory "crushes" the hard disk in every aspect.

---

## Real-World Example

On my NAS (which runs multiple containers and a neural network model), I created a **4GB ZRAM** device as a swap partition. It’s fully utilised now, but it only uses **1.4GB of physical memory**, achieving a **compression ratio of 0.35**.

```shell
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 lz4             4G   4G  1.3G  1.4G       4 [SWAP]
````

ZRAM is managed with the `zramctl` command.

Each GB of ZRAM typically consumes around **6MB of overhead memory**. So, in theory, a 1TB ZRAM swap device would consume 6GB of memory—possible only if you have an absurd amount of RAM, which makes it more of a fun fact than a practical suggestion.

The compression process itself is handled by kernel threads like:

```
kworker/u8:1+flush-252:2
```

Here, `252:2` is the device number.

---

## How to Set Up ZRAM as Swap (Step-by-Step)

1. **Load the ZRAM module:**

```bash
modprobe zram
```

2. **Create a compressed memory device:**

```bash
zramctl --find --size 4G
```

3. **Check the created device:**

```bash
zramctl
```

You should see something like `/dev/zram0` with 4G size.

4. **Format as swap:**

```bash
mkswap /dev/zram0
```

5. **Enable swap:**

```bash
swapon /dev/zram0
```

6. **Verify with:**

```bash
free -h
```

You should now see an additional 4GB of swap space in memory!

---

## Final Thoughts

ZRAM provides a **cost-effective, high-performance** alternative to traditional disk-based swap. It expands your usable memory and can prevent your system from slowing down during heavy loads.

Of course, the trade-off is **CPU usage** for compression and decompression, but modern CPUs handle this very efficiently. For most use cases, this cost is negligible.

ZRAM isn’t limitless—its capacity is constrained by physical RAM and compression ratio. But when configured properly, it offers a **noticeable performance improvement** while also reducing disk wear and saving storage space.

If you're running a Linux system with limited RAM, or just want your swap to perform better, **ZRAM is an excellent solution** worth trying.
