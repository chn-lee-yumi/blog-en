---
title: "Troubleshooting ROCm Errors in the PyTorch Environment on Setonix"
description: "Setonix is a hybrid CPU-GPU supercomputer housed at the Pawsey Centre in Western Australia. This post documents troubleshooting steps for PyTorch and ROCm errors on Setonix."
date: 2024-07-12T19:55:00+10:00
lastmod: 2024-07-12T21:55:00+10:00
categories:
  - Learning
tags:
  - Linux
  - HPC
---

## Introduction

Setonix is the most powerful supercomputer in the Southern Hemisphere — and the most unstable one I’ve ever used. It crashes about once a month: firmware updates, Lustre filesystem issues, you name it.

Because it uses AMD Instinct MI250X GPUs, the PyTorch backend is based on ROCm. This setup introduces all kinds of weird problems.

**If you’re training deep learning models, stay away from AMD cards. They’ve ruined my youth.**

## PyTorch Environment: Unable to Install Packages

First, load the module and create a virtual environment:

```bash
module load pytorch/2.2.0-rocm5.7.3
python3 -m venv venv
````

After this, a `venv` directory will appear in your current working directory. Go to `venv/bin` and check with `ls -l`. You’ll see that `python3` is linked to the wrong path:

```text
lrwxrwxrwx 1 liyumin pawsey1001    7 Mar 17 12:03 python -> python3
lrwxrwxrwx 1 liyumin pawsey1001  100 Mar 17 12:03 python3 -> /usr/bin/python3
```

To fix it:

```bash
rm python3
ln -s /software/setonix/2023.08/containers/modules-long/quay.io/pawsey/pytorch/2.2.0-rocm5.7.3/bin/python3 python3
```

Now it's correctly linked:

```text
lrwxrwxrwx 1 liyumin pawsey1001    7 Mar 17 12:03 python -> python3
lrwxrwxrwx 1 liyumin pawsey1001  100 Mar 17 12:03 python3 -> /software/setonix/2023.08/containers/modules-long/quay.io/pawsey/pytorch/2.2.0-rocm5.7.3/bin/python3
```

The `pip` executable is missing in this environment. Install it manually:

```bash
source ./activate
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py
```

Then install `jupyterlab` and other packages:

```bash
pip install jupyterlab
```

If your home partition quota is full, move `.local` to a project directory:

```bash
mv ~/.local /software/projects/pawsey1001/`whoami`/.local/
ln -s /software/projects/pawsey1001/`whoami`/.local/ ~/.local
```

After installing Jupyter, add it to your `PATH` by appending this line to `~/.bashrc`:

```bash
export PATH=$PATH:~/.local/bin
```

If Jupyter still fails to start:

```text
File "/home/liyumin/.local/bin/jupyter", line 5, in <module>
  from jupyter_core.command import main
ModuleNotFoundError: No module named 'jupyter_core'
```

Edit the file `~/.local/bin/jupyter` and change the **first line** to point to the Python interpreter in your virtual environment:

```python
#!/software/projects/pawsey1001/liyumin/venv/bin/python3
# -*- coding: utf-8 -*-
import re
import sys
from jupyter_core.command import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

Now `jupyter lab` should work correctly.

Here’s the SLURM batch script, save it as `batch_script`:

```bash
#!/bin/bash -l
#SBATCH --account=pawsey1001-gpu
#SBATCH --gres=gpu:1
#SBATCH --partition=gpu
#SBATCH --time=12:00:00
#SBATCH --job-name=jupyter_notebook

host=$(hostname)
port="8888"
pfound="0"
while [ $port -lt 65535 ] ; do
  check=$( ss -tuna | awk '{print $4}' | grep ":$port *" )
  if [ "$check" == "" ] ; then
    pfound="1"
    break
  fi
  : $((++port))
done
if [ $pfound -eq 0 ] ; then
  echo "No available communication port found to establish the SSH tunnel."
  exit
fi

echo "*****************************************************"
echo "Setup - from your laptop do:"
echo "ssh -L ${port}:${host}:${port} \$USER@setonix.pawsey.org.au"
echo "*****************************************************"
echo ""

export OMP_NUM_THREADS=1
module load pytorch/2.2.0-rocm5.7.3
source /home/liyumin/software/venv/bin/activate

srun -N 1 -n 1 -c 8 --gres=gpu:1 --gpus-per-task=1 --gpu-bind=closest jupyter lab \
  --no-browser \
  --port=${port} --ip=0.0.0.0 \
  --notebook-dir=${dir}
```

Replace the virtual environment path with your own in the `source` command.

Submit the job: `sbatch batch_script`

Check the status: `squeue --me`

Once running, check the `slurm-{task_id}.out` file. It will include output like this:

```text
ssh -L 8888:nid002240:8888 liyumin@setonix.pawsey.org.au
...
http://127.0.0.1:8888/lab?token=xxxxxxxx
```

Run the `ssh -L` command on your local machine and open the provided URL in your browser. Done!

To cancel the job: `scancel {task_id}` or shut down from `File > Shut Down` in Jupyter Lab.

## Running a Model Immediately Fails

```text
MIOpen Error: ... Internal error while accessing SQLite database: attempt to write a readonly database
...
RuntimeError: miopenStatusInternalError
```

No solution found online. No reply to my support ticket either. So I figured it out myself.

Turns out only *some* nodes have this issue. So I consider it an environmental bug.

To avoid these problematic nodes, use the `--exclude` parameter in your `sbatch` command:

```bash
sbatch --exclude=nid00[2056,2112,2826,2860,2868,2872,2928,2930,2932,2942,2946,2948,2984,2986,2988,2990,2994,3000] batch_script
```

## Works Fine on NVIDIA, Loss Goes NaN on AMD

While testing an open-source model, adding a simple `nn.Linear` layer caused the loss to immediately become `NaN`.

Same code worked fine on NVIDIA cards.

I investigated further — manually set the layer’s weights and verified the output values. Turns out AMD’s `nn.Linear` produced incorrect values.

By lowering the `batch_size` and `lr`, I found the bug disappeared when `batch_size <= 2`.

Still puzzled, I later discovered the code enabled AMP (automatic mixed precision). Disabling AMP fixed everything.

Although AMD officially supports AMP, this bug made me too scared to use it in the future.
