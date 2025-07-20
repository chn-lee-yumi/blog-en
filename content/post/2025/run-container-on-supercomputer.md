---
title: "Running Containers with Singularity on a Supercomputer"
description: "Since installing software on a supercomputer can be difficult for users, using Singularity to run containers is an excellent solution."
date: 2025-04-20T21:45:00+10:00
lastmod: 2025-04-20T21:45:00+10:00
categories:
  - Learning
tags:
  - Linux
  - HPC
---

## Introduction

Back when DeepSeek went viral, I was curious to see if it could be run natively on a supercomputer, specifically [Setonix](https://pawsey.org.au/systems/setonix/). The DeepSeek team provides several deployment methods. But, as you might expect, installing software on a supercomputer is difficult — you're typically limited to using the pre-installed modules. Fortunately, most supercomputers support containerised workflows.

After many rounds of testing (side note: [SGLang's Docker image was missing dependencies — not exactly plug-and-play](https://github.com/sgl-project/sglang/issues/4630)), I eventually decided to use the [AMD-packaged vLLM Docker image](https://hub.docker.com/r/rocm/vllm/tags), since the GPU in Setonix is the MI250X.

This post mainly serves as a command log so I can refer back to it later. In the end, I wasn’t successful in running the full version of DeepSeek, because it uses 8-bit quantisation, which isn’t supported on the MI250X. It requires an MI300 series GPU. It's theoretically possible to convert the model to 16-bit and run it across four nodes, but that’ll have to wait for another time.

## Step-by-step Instructions

Load the Singularity module:

```bash
module load singularity
````

Pull the Docker image:

```bash
singularity pull docker://rocm/vllm:rocm6.3.1_instinct_vllm0.8.3_20250410
```

After the pull completes, a `.sif` file will be created in the current directory.

Run the container:

```bash
singularity shell vllm_rocm6.3.1_instinct_vllm0.8.3_20250410.sif
```
