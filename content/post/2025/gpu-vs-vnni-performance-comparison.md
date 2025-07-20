---
title: "Simple Performance Comparison: GPU vs VNNI"
description: "A comparison of model inference performance between an NVIDIA GPU and Intel Xeon Gold 5318Y using the VNNI instruction set."
date: 2025-06-14T12:14:00+10:00
categories:
  - Tinkering
tags:
  - AI
---

## Background

I've recently been exploring model quantisation and happen to have a server with two Intel Xeon Gold 5318Y processors. The Xeon Gold 5318Y supports the VNNI (Vector Neural Network Instructions) instruction set. I wanted to see how current CPU performance compares with GPUs when running AI models.

## Test Environment

I used the Stable Diffusion 1.5 model for this test. The code is as follows:

### GPU

```python
from diffusers import StableDiffusionPipeline
import torch

model_id = "sd-legacy/stable-diffusion-v1-5"
pipe = StableDiffusionPipeline.from_pretrained(model_id, torch_dtype=torch.float16)
pipe = pipe.to("cuda")

prompt = "a photo of an astronaut riding a horse on mars"
image = pipe(prompt).images[0] 
````

### CPU

```python
from optimum.intel.openvino import OVDiffusionPipeline

model_id = "OpenVINO/stable-diffusion-v1-5-int8-ov"
pipeline = OVDiffusionPipeline.from_pretrained(model_id)

prompt = "sailing ship in storm by Rembrandt"
images = pipeline(prompt, num_inference_steps=20).images
```

## Test Results

The CPU used int8 quantisation, and actual computation should have been accelerated via the VNNI instruction set. The GPU used float16 precision.

With both 5318Y CPUs, the speed was 2.08 seconds per iteration. The K80 GPU achieved 1.44 seconds per iteration. The GTX 2080 Ti reached 3.60 iterations per second (note the units: s/it vs it/s).

Using the CPU as a baseline:

* K80 is roughly **1.44×** faster than the CPU: `2.08 / 1.44 = 1.44`
* GTX 2080 Ti is about **7.49×** faster than the CPU: `3.60 * 2.08 = 7.49`

It seems that running quantised models directly on CPUs is becoming more viable. While it’s not particularly fast, the speed is acceptable and roughly on par with GPUs from a decade ago. That said, if you compare against the latest GPUs (especially those supporting float8), the CPU performance would still lag far behind.
