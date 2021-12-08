---
title: "Creating Your Custom Jupyter Kernel from a Virtual Environment"
date: 2021-11-28T22:24:04+09:00
draft: False
---

## Creating a Python Kernel

### Admin Prerequisite

Admin must enable `conda` for all users by creating a symlink. This will let users create their own virtual environment by using the `conda create` command.

```bash
ln -s /opt/conda/bin/conda /usr/local/bin/conda
```

### User Instructions

#### Python

1. Users can launch a terminal session with either JupyterHub or SSH connections. Launch a `bash` session.

```bash
bash
```

2.  Users then can create a virtual environment in the following way. Note that specifying `ipykernel` package will make the user easily create jupyter kernels from that environment. Also users can create virtual environments with various python versions(3.6 ~ 3.9). Activate the environment. If any warnings occur, read the warning and do the recommended procedure. Most of the time you will need to refresh your shell with `bash`.

```bash
# create environemnt
conda create -n NAME_OF_VIRTUAL_ENV python=3.8 ipykernel

# refresh your shell
bash

# activate environemnt
conda activate NAME_OF_VIRTUAL_ENV
```

3.  Install packages via `pip`.

```bash
# directly
pip install PACKAGE_NAME

# requirements.txt
pip install -r YOUR_REQUIREMENTS.txt
```

4.  Add the virtual environment as a kernel. This will be only available to each user.

```bash
python -m ipykernel install --user --name NAME_OF_VIRTUAL_ENV --display-name "[displayKenrelName]"
```

#### R

1. Launch a `bash` session via JupyterHub or SSH.

```bash
bash
```

2.  Users then can create a virtual environment in the following way. 

```bash
# create environemnt
conda create -n NAME_OF_VIRTUAL_ENV r-essentials r-base r-irkernel

# refresh your shell
bash

# activate environemnt
conda activate NAME_OF_VIRTUAL_ENV
```

3.  Install packages via `install.packages()`.

```bash
# directly
Rscript -e 'install.packages(c("dplyr"))'
```

4.  Add the virtual environment as a kernel. This will be only available to each user.

```bash
Rscript -e "IRkernel::installspec(name='myRkernel', displayname='My R Kernel')"
```


#### Removing Kernels

To remove kernels use the `jupyter` command in terminal.

View your current kernel list with the following command from a bash terminal.
```bash
jupyter kernelspec list
```

Remove kernel with the following command.
```bash
jupyter kernelspec remove KERNELNAME
```


