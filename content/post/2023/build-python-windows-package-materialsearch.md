---
title: "Tutorial: Building a Python Windows Distribution Package"
description: "A guide to building a Python distribution package for Windows users"
date: 2023-12-18T19:06:00+11:00
categories:
  - Tinkering
tags:
  - Windows
  - Python
---

## Introduction

In my previous open-source project [MaterialSearch](https://github.com/chn-lee-yumi/MaterialSearch), many users requested a Windows-integrated version because configuring the environment on Windows can be quite troublesome—especially for beginners who may not know how to install everything properly. So I decided to try building a standalone Windows package.

## Steps

1. Create a new folder called `MaterialSearchWindows`.

2. Download the [project source code](https://github.com/chn-lee-yumi/MaterialSearch/archive/refs/tags/v0.0.0-20231218.zip), extract it, and copy the contents into the `MaterialSearchWindows/MaterialSearch` directory.

3. Download Python. Since PyTorch supports up to Python 3.10 on Windows, we’ll use that version. Download the [Windows installer (64-bit)](https://www.python.org/ftp/python/3.10.11/python-3.10.11-amd64.exe) and install it as a user into a specific folder—directly into `MaterialSearch`.

4. Use pip to install dependencies. Run the following command:

   ```powershell
   .\python -m pip install -r .\MaterialSearch\requirements.txt --index-url=https://download.pytorch.org/whl/cu118 --extra-index-url=https://pypi.org/simple/
   ```

5. Create a new batch file called `run.bat` with the following content. Execute it to let the model download completely:

   ```powershell
   SET TRANSFORMERS_CACHE=..\huggingface
   cd MaterialSearch
   ..\python main.py
   ```

6. After the model is downloaded, modify `run.bat` with the following:

   ```powershell
   :: Configure asset scan paths, separated by commas
   SET ASSETS_PATH=C:/Users/Administrator/Pictures,C:/Users/Administrator/Videos
   :: Configure the device: cpu or cuda
   SET DEVICE=cpu
   SET DEVICE_TEXT=cpu
   :: Do not modify below
   SET PATH=%PATH%;..\
   SET TRANSFORMERS_OFFLINE=1
   SET TRANSFORMERS_CACHE=..\huggingface
   cd MaterialSearch
   ..\python main.py
   ```

7. Download [FFMpeg](https://www.gyan.dev/ffmpeg/builds/ffmpeg-git-full.7z), extract it, and copy `ffmpeg.exe` into the `MaterialSearchWindows` directory.

8. Finally, compress the entire folder into a zip package. To launch the program later, simply run `run.bat`.

The final directory structure should look like this:

```text
MaterialSearchWindows
 |
 |- run.bat (the batch script you created)
 |- MaterialSearch (project code directory)
     |
     |- main.py
     |- .env
     |- ... (other code files)
 |- python.exe
 |- huggingface (transformers model cache)
 |- ... (other Python-related files)
```