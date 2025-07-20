---
title: "Azure Linux Server Auto-reboot Issue"
description: "Troubleshooting Azure Linux server auto-reboot issue. The cause is Azure auto-installing patches and rebooting the server."
date: 2024-09-15T23:15:00+10:00
lastmod: 2024-09-15T23:15:00+10:00
categories:
  - Tinkering
tags:
  - Linux
---

# Problem Description

One day I logged into the server and noticed that the tasks I had kept running with `screen` were gone. I checked the `uptime` and found the system boot time didn’t match my expectations. Then I checked the Azure VM `Activity log` and found the following entry:

```text
Install OS update patches on virtual machine | Succeeded | 23 hours ago
````

The timestamp of this log exactly matched the server’s reboot time.

Next, I went to `Operation > Updates` and clicked on `Update settings` at the top. I noticed that `Patch orchestration` was set to `Azure Managed - Safe Deployment`.

Changing it to `Customer Managed Schedules` should solve the problem.

I feel this default setting is quite problematic and really catches people off guard.
