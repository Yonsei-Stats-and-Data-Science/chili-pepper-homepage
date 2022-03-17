---
title: "How to Use GPU Node for SLURM"
author: "Jongmin Mun"
date: 2022-03-15T14:54:35+09:00
draft: false
---

# How to use CPU node for SLURM


## Step 1 - terminal 앱 고르기
User는 `SSH`로 `proxy` node에 접속하여 클러스터를 사용합니다. 터미널 환경과 `vi` 에디터에 익숙한 user는 자신에게 친숙한 앱을 사용하면 됩니다. 그렇지 않은 경우, [Visual Studio Code](https://code.visualstudio.com/download)를 사용하는 것을 추천하며, 이 튜토리얼에서는 `Visual Studio Code`를 사용하는 것을 전제로 합니다. 추천 이유는 다음과 같습니다.
- Windows, MacOS, Linux에서 모두 사용 가능합니다.
- [Web Version](https://code.visualstudio.com/docs/editor/vscode-web)도 있기 때문에 모바일 디바이스에서도 사용할 수 있습니다.
- 또한 터미널과 에디터, 파일 브라우저가 통합되어 있기 때문에 불편하게 `vi`나 `nano`등의 CUI 기반 텍스트 에디터를 사용할 필요가 없으며, 파일 전송도 `scp`등의 복잡한 프로토콜을 사용할 필요 없이 drag & drop으로 수행할 수 있습니다.

터미널에서 ssh 접속만 할 수 있다면 어떤 기기에서도 `proxy` node에 접속하여 작업을 제출할 수 있으며, 터미널이 종료되어도 작업은 계속 실행됩니다. Standard output(Python, R에서 console에 출력되는 메시지)가 로그 파일에 기록되므로, 나중에 다시 터미널에 접속하여 job 실행 현황을 확인할 수 있습니다.

`Visual Studio Code`외에 다른 앱을 사용하실 경우 추천하는 앱은 다음과 같습니다.
- Windows 10: PowerShell보다는 [Windows Terminal]((https://docs.microsoft.com/ko-kr/windows/terminal/install))를 추천합니다.
- MacOS: 기본 터미널을 사용해도 되지만, [iTerm2](https://iterm2.com)를 추천합니다.
- iOS: [Blink](https://apps.apple.com/app/id1594898306)
- Android: [Termux](https://play.google.com/store/apps/details?id=com.termux&hl=ko&gl=US)
  

## Step 2 - proxy node SSH 접속 

### 1. Visual Studio Code extensions에서 Remote Development 설치[fn^3]
Microsoft가 제공하는 `Remote Development` extension pack을 설치합니다. Remote-WSL, Remote-Containers, Remote-SSH가 자동적으로 같이 설치됩니다.
![vscode_ssh_1](/assets/vscode_ssh_1.png)
![vscode_ssh_2](/assets/vscode_ssh_2.png)

### 2. Remote Explorer에서 SSH Targets를 선택 후, Add New 클릭
![vscode_ssh_3](/assets/vscode_ssh_3.png)

### 3. ssh 접속 커맨드 입력
아래와 같은 창이 뜨면 ssh 커맨드를 입력하여 `proxy node`에 접속합니다. 
![vscode_ssh_4](/assets/vscode_ssh_4.png)

아래 코드에 Slack으로 안내받은 ip, port, username을 넣어서 위 창에 입력하고 `Enter`키를 누르면 됩니다. `SSH`의 default port는 `22`이지만, 저희의 클러스터는 보안상 이유로 다른 port를 사용합니다.
```bash
ssh -p [port] [username]@[ip] 
```

### 4. SSH configuration file을 저장할 장소 선택
**Select SSH configuration file to update**가 나오면 맨 위 항목을 선택합니다.
![vscode_ssh_5](/assets/vscode_ssh_5.png)

**Host added!** 라는 메시지가 우측 하단에 나옵니다.
![vscode_ssh_6](/assets/vscode_ssh_6.png)

### 5. Remote Explore에서 Connect to Host in New Window 선택

![vscode_ssh_7](/assets/vscode_ssh_7.png)



### 6. 서버 Platform 선택
Linux를 선택합니다.
![vscode_ssh_8](/assets/vscode_ssh_8.png)

### 7. Password 입력
안내받은 password를 입력하여 로그인합니다. 
![vscode_ssh_9](/assets/vscode_ssh_9.png)

### 8. 파일 시스템 마운트
좌측 탭의 파일 모양 아이콘을 클릭하고 **Open Folder** 버튼을 클릭합니다.
![vscode_ssh_10](/assets/vscode_ssh_10.png)

기본적으로 user home directory 경로가 입력되어 있습니다. OK를 누릅니다.
![vscode_ssh_11](/assets/vscode_ssh_11.png)

### 9. 둘러보기
![vscode_ssh_12](/assets/vscode_ssh_12.png)
- 좌측 file explorer에서 파일을 관리합니다. Windows 탐색기나 MacOS Finder에서 drag&drop으로 파일을 옮길 수 있습니다. 클러스터 내부의 파일을 user의 local 컴퓨터로 가져오는 것도 drag&drop으로 가능합니다.

- `ctrl + shift + ~`키를 누르면 터미널이 열립니다. 여기서 서버 사용에 필요한 커맨드를 입력합니다.

- text editor에서 코드와 스크립트를 수정하고 이미지 파일 등을 열람합니다.


## Step 3 - 파일 시스템 구조 이해

NAS(Network Attached Storage)에 각 user의 home directory가 있습니다. NAS는 모든 node에 마운트되어 있으며, 모든 node에서 user명과 group명 및 관련 설정이 동일합니다. User명은 컴퓨팅 클러스터 사용 신청시 제출하신 이메일 주소의 @ 앞 부분과 동일합니다.

User home directory의 prefix는 `/mnt/nas/users/`입니다. 예를 들어, `dummyuser`라는 user의 home directory의 경로는 `/mnt/nas/users/dummyuser/`입니다. 다른 user의 home directory를 열람할 수 없도록 권한설정이 되어 있습니다. 각 user는 데이터와 코드, 설정 파일 등을 자신의 home directory 내에 저장합니다.

- Linux에서 directory를 이동하는 명령어는 `cd`입니다.
- home directory를 나타내는 기호는 `~`입니다.
- 현재 directory를 확인하는 명령어는 `pwd`입니다
- 파일 목록을 확인하는 명령어는 `ls`입니다. 

따라서 user는 proxy node아래의 명령어를 통해 자신의 홈 directory로 이동해 그 안에 있는 파일 목록을 확인할 수 있습니다.
```
cd ~
ls
``` 
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

## Step 3 - Data Transfer

파일을 옮길 때에는 크게 'SCP'와 'Git' 두 가지 방법이 있습니다.

### 방법 1. scp
`scp`는 `SSH`와 같은 port를 사용합니다. 아래 커맨드를 이용해 파일을 보내고 받을 수 있습니다. `dummyuser`를 본인의 user명으로 바꾸고 파일명, directory명을 적절히 바꾸면 됩니다.

### Sending a local file(`some_file.txt`) to the remote home directory

```bash
scp -P [port] some_file.txt dummyuser@hpc.stat.yonsei.ac.kr:~/
```

### Sending a local directory(`some_files/`) to the remote home directory

```bash
scp -P [port] -r some_files dummyuser@hpc.stat.yonsei.ac.kr:~/
# /mnt/nas/users/dummyuser/some_files/ will be created
```

### Recieving a remote file(`some_file.txt`) in the home directory to the current local directory

```bash
scp -P [port] dummyuser@hpc.stat.yonsei.ac.kr:~/some_file.txt
```

### Recieving a remote directory(`/mnt/nas/users/dummyuser/some_files`) in the home directory to the current local directory

```bash
scp -P [port] -r dummyuser@hpc.stat.yonsei.ac.kr:~/some_files ./
```
이 외의 scp 옵션들은 [여기서](https://www.pcwdld.com/what-is-scp#wbounce-modal) 확인할 수 있습니다. 

윈도우에서는 [winscp](https://winscp.net/eng/download.php) 를 이용해 `scp`를 drag & drop으로 편리하게 이용할 수 있습니다.

MacOS에서는 [filezilla](https://filezilla-project.org)를 사용할 수 있습니다. 이에 대한 안내는 별도의 글로 작성되어 있습니다.

### 방법 2. Git
`Git`이 이미 설치되어 있으므로, 클러스터 내 적절한 directory에서 `git clone`을 사용합니다.

## Step 4. Conda environment 생성
- `cpu-compute` node에는 conda version 4.11.0이 설치되어 있으며 Python version을 3.10까지 지원합니다[fn^1].
- `gpu-compute` node에는 conda version 4.6.14가 설치되어 있으며 Python version을 3.8까지 지원합니다.

여기서는 `cpu-compute` node에서 conda environment를 생성하는 방법을 설명합니다. local에서 작성한 코드가 `cpu-compute` node에서 오류 없이 작동하도록 하기 위해, local과 `cpu-compute` node에서 동일한 conda environment를 구축해야 합니다. 

### 1. local에서 conda environment 생성
이 섹션의 작업은 모두 user의 local 컴퓨터에서 진행합니다.

[miniconda](https://docs.conda.io/en/latest/miniconda.html)를 설치한 다음, local 컴퓨터의 터미널에서 아래 커맨드로 virtual environment를 설정합니다. `testEnv` 자리에 원하는 이름을 넣고, `python=` 뒤에 사용할 Python version을 명시합니다. 

```bash
conda create -n testEnv python=3.6
```

성공적으로 생성되면 아래와 같은 결과가 나옵니다.

```text
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate testEnv
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

아래 커맨드를 통해 virtual environment가 제대로 생성되었는지 확인합니다.
```bash
conda info --env
```

```text
# conda environments:
#
base                  *  /opt/miniconda3
testEnv                  /opt/miniconda3/envs/testEnv
```

Virtual environment에 진입한 뒤 패키지를 설치합니다. pip로 설치되는 패키지들은 conda로 설치된 패키지에 대한 정보를 모르기 때문에 의존성 충돌이 발생할 수 있으므로 conda만을 사용해서 설치하실 것을 권장합니다. [anaconda 웹사이트](https://anaconda.org/anaconda/scikit-learn)에서 패키지명을 검색해서 linux-64를 지원하는 버전이 어디까지인지를 확인하고 설치하는 것을 추천합니다. 위 사이트는 설치 커맨드도 제공합니다. 여러 패키지를 설치할 경우 한 커맨드 내에 명시하면 conda가 자동으로 dependency 충돌을 검사해 줍니다.
```bash
conda activate testEnv

#For example,
conda install -c conda-forge lightgbm==2.0.7 scikit-learn matplotlib keras
```

또는 user가 사용할 중요한 패키지들이 있을 경우 environment를 생성할 때 사용할 패키지 목록을 지정할 수도 있습니다. 
아래 커맨드로 설치된 패키지 목록을 확인합니다. 
```bash
conda list
```

Environment를 지우려면 아래 커맨드를 입력합니다.
```bash
conda remove --name testEnv --all
```
### 2. `cpu-compute` node에 local과 동일한 conda environment 구축하기

**conda env export** 커맨드를 이용해 environment 전체를 `.yml` 파일로 만들고 이를 이용해 `cpu-compute` 노드에서 environment를 구축하는 것이 동일한 environment를 만드는 가장 이상적인 방법입니다. 또는 설치된 패키지 목록과 버전만을 `requirements.txt`로 추출하여 노드에서 `cpu-compute`에서 설치할 수 있습니다. 그러나 이 두 방법은 user의 local 컴퓨터가 linux가 아니면 오류가 발생할 확률이 매우 높습니다. 이는 유저가 패키지를 설치할 때 자동으로 설치되는 dependency들의 버전이 OS별로 다를 수 있기 때문입니다. 예를 들어 `lightgbm` 2.0.7버전은 Python 버전이 3.6일 때 linux와 MacOS에서 둘 다 설치 가능하지만, 이 패키지의 dependency 중 하나인 `libgfortran`은 MacOS에서는 4.0.0버전이 설치되지만 linux에서는 3.0.0버전까지만 지원되기 때문에 MacOS에서 만든 environment에서 추출한 yml 파일이나 list txt 파일을 클러스터에서 사용하면 오류가 발생합니다[^fn2]. 이 오류는 적절한 조치를 통해 해결할 수 있을 때도 있지만 해결하기 힘들 때도 있습니다.

따라서 local에서 만든 environment를 cluster로 옮기기보다는, local의 environment에서 사용하는 Python 버전과 중요 패키지들의 버전을 그대로 사용하여 cluster 내에서 environment를 생성하는 것을 추천하며, 이 문서에서는 그 절차를 안내합니다.


1. Conda environment 생성 batch script를 작성합니다.
    ```bash
    #!/bin/bash
    #SBATCH --job-name=conda-env-create
    #SBATCH --nodes=1
    #SBATCH --mem=4gb
    #SBATCH --partition=all
    #SBATCH --nodelist=cpu-compute
    #SBATCH --output=testEnv.out
    #SBATCH --error=testEnv.err
    CONDA_BIN_PATH=/opt/miniconda/bin
    ENV_NAME=testEnv #local에서와 같은 이름으로 입력
    ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME
    $CONDA_BIN_PATH/conda env remove --prefix $ENV_PATH
    $CONDA_BIN_PATH/conda create -y --prefix $ENV_PATH python=3.6 
    source $CONDA_BIN_PATH/activate $ENV_PATH
    conda install -c conda-forge lightgbm==2.0.7 scikit-learn matplotlib keras pandas numpy
    ```
    
    위 내용에서
    - 7번 라인의 **#SBATCH --output=testEnv.out**의 output log 파일명
    - 8번 라인의 **#SBATCH --error=testEnv.err**의 error log 파일명
    - 10번 라인의 environment name
    - 13번 라인의 python version
    - 14번 라인의 패키지 설치 커맨드
    를 알맞게 수정한 뒤  local에서 `testEnv.sh`로 저장하여 `scp`로 클러스터 내 user home directory로 전송합니다. 또는 proxy node에 접속한 뒤 home directory에서

    ```bash
    vi testEnv.sh
    ```
    를 입력하여 linux의 텍스트 에디터를 실행한 뒤 `ctrl+v`로 위 내용을 붙여넣기하고 `esc`, `:wq`, `ENTER`를 차례로 눌러 저장합니다.

2. 작성한 스크립트 실행하기. proxy node에 SSH접속한 뒤 home directory에서 
    ```bash
    sbatch testEnv.sh
    ```  
    를 입력해 slurm batch job submission을 수행합니다. 작업이 노드에서 성공적으로 실행되면

    ```text
    Submitted batch job 401
    ```
    와 같은 메시지가 뜨고 job 번호가 할당됩니다.

    ```bash
    squeue
    ```
    커맨드를 통해 작업 실행 현황을 확인할 수 있습니다.

    ```text
    JOBID PARTITION     NAME     USER ST    TIME  NODES NODELIST(REASON)
    402     all       conda-en    mjm  R    0:01    1  cpu-compute
    ``` 

    또는 아래 커맨드를 통해 실시간으로 작업 실행 현황을 확인할 수 있습니다.
    ```bash
    smap -i 1
    ```

    다음 커맨드를 통해 output log, error log파일의 내용을 확인할 수 있습니다.
    ```bash
    cat testEnv.err
    ```
    
    error log 또는 output log는 다음 커맨드를 통해 실시간으로 확인할 수 있습니다.
    ```bash
    tail -f testEnv.out
    ```  
 

## Step 2. make env file and upload file to user’s workspace

local에서 만들어진 env 파일을 서버로 옮길 때는 **scp**를 사용한다. 간단한 사용법은 아래와 같다.

```bash
# scp 사용법
# File upload
scp [FILEPATH] ghk@[IP address]:~/ # upload from local, ~/ means home directory
# File download
scp ghk@[IP address]:[FILEPATH] ./ # scp [server file path] [local save path]

# directory 전체 다운, 업로드 할 때는 -r 옵션을 사용한다.
```

Windows 사용자라면 **[WinSCP](https://winscp.net/eng/download.php)** 라는 이름의 프로그램을 사용하면 local과 서버 간 파일 전송을 보다 쉽게 처리할 수 있다. 

<aside>
💡 env file을 이용한 conda 환경 설정은 local과 서버 작업 환경을 동일하게 설정할 수 있는 신뢰할 수 있는 방법이다. 그러나 conda 환경 설정 과정이 너무 번거롭다면 requirements.txt를 만들어 패키지 버전만 관리해도 충돌을 방지할 수 있다.

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


# Refernece
[fn^1]: https://docs.conda.io/projects/conda/en/latest/release-notes.html
[fn^2]: https://github.com/conda/conda/issues/9399
[fn^3]: https://jstar0525.tistory.com/14