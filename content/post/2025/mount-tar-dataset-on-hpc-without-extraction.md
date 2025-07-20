---
title: "Mounting Large Dataset Archives on Supercomputers"
description: "Supercomputer file systems often enforce quota limits. If a dataset contains a large number of small files, extracting it might exceed the quota. This post explains how to mount large dataset archives (tar files) on a supercomputer, enabling access to the files inside without extraction."
date: 2025-01-20T14:25:00+08:00
lastmod: 2025-01-20T14:25:00+08:00
categories:
  - Learning
tags:
  - Linux
  - HPC
---

## Background

I recently worked with the AffectNet dataset, which contains over a million files. I was using the Setonix supercomputer, which utilises the Lustre file system. A temporary storage partition at `/scratch` was provided, but it had a file count limit of 1 million. When I tried to extract the dataset, I ran into errors due to exceeding this limit.

The standard solution is to submit a ticket requesting a quota increase. But, as anyone familiar with Australian bureaucracy might expect, I submitted the request on Monday and only got approval by Friday after a few back-and-forth questions.

Instead of relying on others, I found an alternative solution to this issue.

## Concept

The dataset was packaged as a tar archive. While `.tar` files are technically archives rather than compressed files, I'll refer to them as "archives" for simplicity. It turns out that there's a way to mount such archives directly as directories without extracting them.

The key technology here is `FUSE` — *Filesystem in Userspace*. This allows users to mount filesystems without root privileges. After some research, I discovered a Python tool called [`ratarmount`](https://github.com/mxmlnkn/ratarmount) that supports this functionality.

## Step-by-Step Guide

Since I’m using a supercomputer environment, I first need to load the relevant module to use Python, then activate my virtual environment:

```bash
module load pytorch/2.2.0-rocm5.7.3
source /home/liyumin/software/venv/bin/activate
````

If you already have access to Python without loading modules, you can skip the steps above.

Next, install `ratarmount` using pip:

```bash
pip install ratarmount
```

Once installed, you can mount the archive with the following command:

```bash
ratarmount train_set.tar train_set
```

This mounts the `train_set.tar` archive to the `train_set` directory.

You might encounter an error like the following:

```text
Created mount point at: /scratch/pawsey1001/liyumin/datasets/AffectNet8/train_set
fusermount: mounting over filesystem type 0x0bd00bd0 is forbidden
```

This means the file system doesn’t allow mounting at that path. You can resolve this by choosing a different path, such as the `/tmp` directory:

```bash
ratarmount train_set.tar /tmp/train_set
```

If there are no errors, the mount was successful. You can now access the contents of `train_set.tar` simply by browsing the `/tmp/train_set` directory — no extraction needed.

When you're done, don't forget to unmount the directory:

```bash
fusermount -u /tmp/train_set
```
