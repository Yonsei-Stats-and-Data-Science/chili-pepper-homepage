---
title: "How do I use the Chili Pepper Cluster?"
author: "Dongook Son"
date: 2022-02-21T12:12:53+09:00
draft: false
---

## Intro

This documentation will go over the basics of using the Chili Pepper cluster. Please go through this documentation step-by-step. Contact the server administrator via [email](mailto:jm.moon@yonsei.ac.kr) or use the Q&A channel in the [Slack Group](https://yonseidatasci-jnw9112.slack.com).

## Step 1 - Understanding the Cluster Structure

Chili Pepper cluster has a NAS which contains user home directories and other shared files. All users and groups are consistent across all nodes. The prefix for the user directory is `/mnt/nas/users/`. For example, the home directory for the user `dummyuser` will be `/mnt/nas/users/dummyuser/`. The home directory is the recommended directory for users to store scripts, data and configuration files for using the Chili Pepper cluster.

```txt
# with the tree command the home directory will typically look like this
/mnt/nas/users/dummyuser/
├── .bash_history
├── .bash_logout
├── .bashrc
├── .conda
├── .config
├── GettingStarted.md
├── .gnupg
├── .ipynb_checkpoints
├── .ipython
├── .jupyter
├── .local
├── logs
├── .npm
├── .profile
├── .python_history
├── some_script.sh
├── .ssh
└── .viminfo
```

## Step 2 - Data Transfer

With cluster access information(`SSH`) provided by the administrator, you can send and recieve files from and to the cluster with `scp`. The `dummyuser` can send various local files to the cluster in the following fashion. Note that for security purposes the default port for `SSH` is not `22`. The administrator will inform you of this information upon providing access information to the cluster.

### Sending a local file(`some_file.txt`) to the remote home directory

```bash
scp some_file.txt dummyuser@hpc.stat.yonsei.ac.kr:~/
```

### Sending a local directory(`some_files/`) to the remote home directory

```bash
scp -r some_files dummyuser@hpc.stat.yonsei.ac.kr:~/
# /mnt/nas/users/dummyuser/some_files/ will be created
```

### Recieving a remote file(`some_file.txt`) in the home directory to the current local directory

```bash
scp dummyuser@hpc.stat.yonsei.ac.kr:~/some_file.txt
```

### Recieving a remote directory(`/mnt/nas/users/dummyuser/some_files`) in the home directory to the current local directory

```bash
scp -r dummyuser@hpc.stat.yonsei.ac.kr:~/some_files ./
```

Other various options for `scp` exist. More information on this topic can be found in this [article](https://www.pcwdld.com/what-is-scp#wbounce-modal). Also users can use GitHub or GitLab to upload and download source code to the cluster. This will be handled in a seperate article.

## Step 3 - Writing a `SBATCH` script for SLURM.

From the homepage the [SLURM batch scripting tool](https://hpc.stat.yonsei.ac.kr/tools/job-configurator.html) is available. Let's look at the sample script(`/mnt/nas/users/dummyuser/test_script.sh`) created by using the tool. The first-half(line 1 ~ 11) of the script consists of directives and parameters for the `slurm` job. Each user can set the number of nodes, the time for the job to occupy the number of nodes, the location for the output logfile. There are more options available for submitting a job. Additional resources for `SBATCH` arguments can be found [here](https://slurm.schedmd.com/sbatch.html).

```bash
#!/bin/bash
# The interpreter used to execute the script

#“#SBATCH” directives that convey submission options:

#SBATCH --job-name=conda-env-create
#SBATCH --nodes=1
#SBATCH --time=15:00
#SBATCH --account=dummyuser
#SBATCH --partition=all
#SBATCH --output=/mnt/nas/users/dummyuser/conda.log

# The application(s) to execute along with its input arguments and options:

ENV_NAME=myenv
ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME

conda env remove --prefix $ENV_PATH
conda create -y --prefix $ENV_PATH python=3.8 pandas numpy scikit-learn
source activate $ENV_PATH && pip freeze
```

From line 15 to the end of the script are actual bash commands for the node to execute. 
- Line 15~16  creates two local variables(`ENV_NAME` and `ENV_PATH`). In the above script a conda environment named `myenv` will be created under `/mnt/nas/users/dummyuser/.conda/envs/myenv`. 
- Line 18 will remove the environment in `ENV_PATH` if it is present. 
- Line 19 will create a conda environment in `ENV_PATH` thanks to the `-y(--yes)` flag. This environment will have a Python interpreter of version `3.8` along with listed packages(`pandas, numpy and scikit-learn`).
- Line 20 will activate the conda environment in `ENV_PATH` and then sequentially run a `pip freeze` to the `stdout`. Note that the `stdout` is saved in the log file from line 11(`/mnt/nas/users/dummyuser/conda.log`).

To actually run a data science job, the only thing you have to do is to change the required packages for your environment, modify the `pip freeze` into `python your_script_to_run.py`.

## Step 4 - Submitting the script

The submission of the script from the above is very simple.

```bash
sbatch test_script.sh
```

You can check the current job queue with the following command.

```bash
squeue
```

When you want to cancel the job you have submitted, get the `JOBID` from the `squeue` command and use the `scancel` command in the following fashion. Suppose the `JOBID` is `23`.

```bash
scancel 23
```

Note that ordinary users cannot cancel jobs that belong to other users, but the administrator can.
