---
title: "How do I use the Chili Pepper Cluster?"
author: "Dongook Son"
date: 2022-02-21T12:12:53+09:00
draft: true
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

Other various options for `scp` exist. More information on this topic can be found in this [article](https://www.pcwdld.com/what-is-scp#wbounce-modal).

## Step 3 - Writing a `SBATCH` script for SLURM.

