---
title: "How to Use GPU Node for SLURM"
author: "Gwnaghee Kim"
date: 2022-03-15T14:54:35+09:00
draft: false
---

# How to use GPU node for SLURM

## Step 1. Export your conda setting

conda 환경을 동일하게 맞춰주기 위해 local에서 생성된 가상환경으로부터 환경설정 파일을 만들고, 서버에서 conda 가상환경을 만들 때 사용한다. 

```bash
# export conda setting

conda activate [YOUR ENV NAME]
conda env export -n [ENV NAME] -f [FILENAME].yml --no-builds # 이러면 조금은 해결되긴 함

# create environment from file

conda create --name [YOUR ENV NAME] python = [VESTION] # same env name in yml file
conda env create -f [FILENAME].yml

conda env create -p [prefix path] -f [filename].yml
```

**—no-builds** 옵션은 서로 다른 OS에서 conda 가상환경 파일 내 패키지들의 버전 충돌을 방지하기 위한 것이다.  다만 경우에 따라 완전히 문제를 예방하지는 못하기 때문에 사용자들이 직접 env 파일을 수정하는 상황이 발생할 수 있다. 이런 경우 **ResolvePackageNotFound** 아래의 패키지들을 env 파일에서 삭제해주면 문제가 해결된다.

```bash
conda env create -f test.yml
Collecting package metadata: done
Solving environment: failed

# yml 파일에서 아래에 등장하는 패키지들을 지워주면 된다.
ResolvePackageNotFound: 
  - libgfortran==3.0.1=h93005f0_2 # --no-builds는 패키지 버전 옆의 빌드 정보를 제거한다.
  - pyzmq==17.0.0=py36h1de35cc_1
  - python==3.6.6=h4a56312_1003
  - prompt_toolkit==1.0.15=py36haeda067_0
  - libiconv==1.15=h1de35cc_1004
  - sqlite==3.25.3=ha441bb4_0
  - six==1.11.0=py36h0e22d5e_1
  - cryptography==2.3.1=py36hdbc3d79_1000
  - openssl==1.0.2p=h1de35cc_1002
  - libxml2==2.9.8=hf14e9c8_1005
  - libcxxabi==4.0.1=hebd6815_0
  - matplotlib==2.2.3=py36h0e0179f_0
  - ptyprocess==0.5.2=py36he6521c3_0
```

## Step 2. make env file and upload file to user’s workspace

로컬에서 만들어진 env 파일을 서버로 옮길 때는 **scp**를 사용한다. 간단한 사용법은 아래와 같다.

```bash
# scp 사용법
# File upload
scp [FILEPATH] ghk@[IP address]:~/ # upload from local, ~/ means home directory
# File download
scp ghk@[IP address]:[FILEPATH] ./ # scp [server file path] [local save path]

# directory 전체 다운, 업로드 할 때는 -r 옵션을 사용한다.
```

Windows 사용자라면 **[WinSCP](https://winscp.net/eng/download.php)** 라는 이름의 프로그램을 사용하면 로컬과 서버 간 파일 전송을 보다 쉽게 처리할 수 있다. 

<aside>
💡 env file을 이용한 conda 환경 설정은 로컬과 서버 작업 환경을 동일하게 설정할 수 있는 신뢰할 수 있는 방법이다. 그러나 conda 환경 설정 과정이 너무 번거롭다면 requirements.txt를 만들어 패키지 버전만 관리해도 충돌을 방지할 수 있다.

</aside>

```bash
conda install --force-reinstall -y -q -c conda-forge --file requirements.txt

# --force-reinstall : Install the package even if it already exists.
# -y : Yes, do not ask for confirmation.
# -q : Quiet, do not display progress bar.
# -c : Channels, additional channels to search for packages
# conda-forge is recommended
```

## Step 3. make SLRUM batch script and run code in server

앞선 단계에서 conda환경이 잘 만들어졌다면 해당 가상환경을 activate하여 코드를 돌리는 SLURM batch script를 작성할 수 있다. 사용자들의 이해를 돕기 위해 TensorFlow 공식 페이지에 게시된 [초보자용 튜토리얼](https://www.tensorflow.org/tutorials/quickstart/beginner?hl=ko) 코드를 SLURM을 통해 실행시키는 예제를 공유한다. 먼저, 튜토리얼 코드는 다음과 같다.

```python
# tensor.py

import tensorflow as tf

mnist = tf.keras.datasets.mnist

(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(input_shape=(28, 28)),
  tf.keras.layers.Dense(128, activation='relu'),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)

model.evaluate(x_test,  y_test, verbose=2)
```

```bash
#!/bin/bash
#
#SBATCH --job-name=gpu-tensor-test
#SBATCH --mem=4gb
#SBATCH --nodelist=gpu-compute
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --gres=gpu:1
#SBATCH --time=00:20:00
#SBATCH --account=ghk
#SBATCH --partition=all
#SBATCH --output=/mnt/nas/users/ghk/code/gputest.log

CONDA_BIN_PATH=/opt/miniconda/bin
ENV_NAME=tensor
ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME

## $CONDA_BIN_PATH/conda env remove --prefix $ENV_PATH
## $CONDA_BIN_PATH/conda create -y --prefix $ENV_PATH python=3.8 tensorflow pandas numpy

source $CONDA_BIN_PATH/activate $ENV_PATH
## pip uninstall keras
## pip install keras==2.6.0
python /mnt/nas/users/ghk/code/tensor.py
```

- **—job-name**: 수행할 작업의 이름
- **—mem**: memory limit
- **—nodelist**: 작업할 노드의 이름
- **—ntasks**: 작업의 수
- **—cpus-per-task**: 각 작업에서 사용할 cpu 코어의 수
- **—gres=gpu** : 작업에서 사용할 gpu의 개수, gpu-compute 노드에는 총 2개의 gpu가 사용 가능하다.
- **—time**: 작업 제한시간
- **—accoun**t: 해당 작업을 수행하는 계정의 이름
- **—partition**: group of nodes with specific characteristics
- **—output**: 코드 실행 결과 log

제시한 대로 python 파일과 batch script 파일이 잘 만들어졌다면 **sbatch** 명령어를 입력하여 계산을 실행할 수 있다.

```bash
sbatch tensor.sh
smap -i 1 # 작업 현황을 1초마다 갱신하여 보여준다. ctrl+c로 escape 할 수 있다.
```

```bash
ghk@proxy:~/code$ cat gputest.log
2022-03-15 14:27:34.877232: I tensorflow/core/platform/cpu_feature_guard.cc:142] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  SSE4.1 SSE4.2 AVX AVX2 AVX512F FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
2022-03-15 14:27:34.887611: I tensorflow/core/common_runtime/process_util.cc:146] Creating new thread pool with default inter op setting: 2. Tune using inter_op_parallelism_threads for best performance.
2022-03-15 14:27:38.063857: I tensorflow/compiler/mlir/mlir_graph_optimization_pass.cc:185] None of the MLIR Optimization Passes are enabled (registered 2)
Epoch 1/5
1875/1875 [==============================] - 36s 19ms/step - loss: 0.2993 - accuracy: 0.9140
Epoch 2/5
1875/1875 [==============================] - 18s 10ms/step - loss: 0.1436 - accuracy: 0.9575
Epoch 3/5
1875/1875 [==============================] - 17s 9ms/step - loss: 0.1080 - accuracy: 0.9675
Epoch 4/5
1875/1875 [==============================] - 19s 10ms/step - loss: 0.0866 - accuracy: 0.9739
Epoch 5/5
1875/1875 [==============================] - 52s 28ms/step - loss: 0.0750 - accuracy: 0.9762
313/313 - 4s - loss: 0.0782 - accuracy: 0.9779
[0.078231580555439, 0.9779000282287598]
```

SLURM batch script를 사용자들이 보다 편하게 만들 수 있도록 [SLURM Job Configurator](https://hpc.stat.yonsei.ac.kr/tools/job-configurator.html) 를 새롭게 작성하였다. 사용자들은 gpu 옵션을 체크하거나 해제하여 gpu-compute node 사용 여부를 결정할 수 있다. 해당되는 옵션을 체크하고 **Print** 버튼을 누르면 손쉽게 batch script를 작성할 수 있다.

## 더 알아보기

[Submitting a slurm job script](https://ubccr.freshdesk.com/support/solutions/articles/5000688140-submitting-a-slurm-job-script)

[SLRUM Job Examples](https://doc.zih.tu-dresden.de/jobs_and_resources/slurm_examples/)

[TensorFlow on the HPC Clusters](https://researchcomputing.princeton.edu/support/knowledge-base/tensorflow)