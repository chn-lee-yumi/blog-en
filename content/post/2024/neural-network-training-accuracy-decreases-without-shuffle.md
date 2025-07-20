---
title: "Debugging: Neural Network Training Starts with High Accuracy then Decreases"
description: "A strange phenomenon where neural network training starts with high accuracy and gradually decreases. The root cause: not setting shuffle=True."
date: 2024-08-18T22:02:00+10:00
lastmod: 2024-08-18T22:02:00+10:00
categories:
  - Learning
tags:
  - AI
---

## Phenomenon

During neural network training, the training accuracy (Train ACC) starts off very high, but gradually decreases. See the example below:

```text
Epoch 	 Time 	 Train Loss 	 Train ACC 	 Val Loss 	 Val ACC 	 Test Loss 	 Test ACC 	 LR
1	 197.8234 	 0.0053 	 0.8645 	 0.0412 	 0.1443 	 0.0412 	 0.1443 	 0.0100
2	 108.6638 	 0.0084 	 0.7311 	 0.0272 	 0.1443 	 0.0272 	 0.1443 	 0.0100
3	 108.4892 	 0.0095 	 0.6777 	 0.0267 	 0.1443 	 0.0267 	 0.1443 	 0.0100
4	 108.8819 	 0.0087 	 0.7102 	 0.0269 	 0.1443 	 0.0269 	 0.1443 	 0.0100
5	 108.8337 	 0.0065 	 0.7712 	 0.0504 	 0.1443 	 0.0504 	 0.1443 	 0.0100
6	 109.4179 	 0.0061 	 0.8071 	 0.0624 	 0.1443 	 0.0624 	 0.1443 	 0.0100
7	 109.2300 	 0.0057 	 0.8349 	 0.0762 	 0.1443 	 0.0762 	 0.1443 	 0.0075
8	 109.2820 	 0.0101 	 0.6432 	 0.0245 	 0.1443 	 0.0245 	 0.1443 	 0.0075
````

The key issue is: Train ACC starts off very high, but Val ACC is very low. As epochs increase, Train ACC decreases, while Val ACC almost stays unchanged.

## Debugging Process

I first removed the Argumentation part of the code, but the problem remained.
Since I am using distributed training, I tried running with only 1 process, but the issue was still there.

Finally, I took out my old single-machine training code and debugged it part by part. In the end, I found the root cause: the `Dataloader` had `shuffle=False`. Once I enabled shuffle, the training became normal.

Why was shuffle disabled before? I can't really remember... In distributed training, the `DistributedSampler` can be set with `shuffle=False`. The code looks like this:

```python
trainsampler = DistributedSampler(train, shuffle=True)
trainloader = DataLoader(train, batch_size=128, num_workers=6, sampler=trainsampler)
```