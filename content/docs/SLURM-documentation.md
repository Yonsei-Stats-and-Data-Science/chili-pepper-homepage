---
title: "SLURM Documentation"
author: "Gwanghee Kim"
date: 2022-03-08T22:39:38+09:00
draft: False
---

## 1. What is SLURM?

Slurm is an open source, fault-tolerant, and highly scalable cluster management and job scheduling system for large and small Linux clusters. Slurm requires no kernel modifications for its operation and is relatively self-contained. 

If you need more information, Please Visit [https://slurm.schedmd.com/overview.html](https://slurm.schedmd.com/overview.html)

## 2. Basic SLURM Command

- sbatch
    - Submit a batch script to SLURM
    
    ```bash
    $ sbatch [YOUR_SCRIPT]
    
    # output
    > Submitted batch job 210 # job_id: 210
    ```
    
- squeue
    - View the queue
    
    ```bash
    $ squeue
    
    # output
    > JOBID PARTITION     NAME     USER   ST   TIME   NODES  NODELIST(REASON)
      210   all           test1    ghk    R    0:29   1      cpu-compute
    ```
    
- scancel
    - Cancel a SLURM job
    
    ```bash
    $ scancel [YOUR_JOBID]
    ```
    
- sinfo
    - See the state of system
    
    ```bash
    $ sinfo
    
    # output
    > PARTITION AVAIL  TIMELIMIT  NODES  STATE  NODELIST
      all*      up     infinite   2      idle   cpu-compute,gpu-compute
    ```
    
- smap
    - graphically view information about SLURM job

For more information about SLURM command, please visit website below.

* [SLURM Workload Manager](https://slurm.schedmd.com/quickstart.html)

* [SLURM Command Cheatsheet](https://slurm.schedmd.com/pdfs/summary.pdf)

* [SLURM Reference Sheet(NeSI)](https://support.nesi.org.nz/hc/en-gb/articles/360000691716) 

* [Submit your first job(NeSI)](https://support.nesi.org.nz/hc/en-gb/articles/360000684396)

## 3. How to make SLURM batch script?

'Job submission file' is the official SLURM name for the file you use to submit your program and ask for resources from the job scheduler. In this document, we will be using it ‘batch script’ or ‘script’.

### Basic example

```bash
#!/bin/bash 
# 
# SBATCH --job-name=basic
# SBATCH --mem=1gb
# SBATCH --ntasks=1
# SBATCH --time=01:00
# SBATCH --output=/mnt/nas/users/testuser/basic.log
```

Asking 1 tasks, running for no more than 1 minutes limit memory less than 1gb. If any problem with your job, log file(in this case, 'basic.log') have information to help troubleshoot the issue.

You can also use gpu-nodes by using '—gres' option. Here is an example.

```bash
#!/bin/bash 
# 
#SBATCH --job-name=basic
#SBATCH --nodes=1
#SBATCH --gres=gpu:2 # max : 2
#SBATCH --time=01:00
#SBATCH --account=testuser
#SBATCH --partition=all
#SBATCH --output=/mnt/nas/users/testuser/torch.log

source activate /opt/miniconda/envs/pytorch && python /mnt/nas/users/testuser/main.py
```

```python
# main.py

import torch

print('is cuda avaiable? ', torch.cuda.is_available())
print('how many cuda devices? ',torch.cuda.device_count())
print('get first cuda device name => ', torch.cuda.get_device_name(0))
print('*** MEMORY INFO ***')
t = torch.cuda.get_device_properties(0).total_memory print('total memory => ', t)
print('total memory: ', t)
```

The job can then be submitted through sbatch

```bash
$ sbatch basic.sh
$ cat torch.log
> is cuda avaiable?  True
> how many cuda devices?  2
> get first cuda device name =>  Tesla T4
> ***MEMORY INFO***
> total memory =>  16879583232
```

Beacuse we only have 2 gpu machine, this option can’t be set more than 2 

For the convenience of users, we provide SLURM job configurator page in our website. Please visit [SLURM job configurator](https://hpc.stat.yonsei.ac.kr/tools/job-configurator.html) and make your own SLURM batch script!