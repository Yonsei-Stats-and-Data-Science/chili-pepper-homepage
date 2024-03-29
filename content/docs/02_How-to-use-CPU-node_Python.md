---
title: "2. CPU node 사용법(Python)"
author: "Jongmin Mun"
date: 2022-03-03T14:54:35+09:00
draft: false

---

# CPU node에서 Python 코드 실행하기

R 사용자는 Step 1-3을 숙지한 뒤 다음 문서로 넘어가세요.

## Step 1 - terminal 앱 고르기

User는 `SSH`로 `proxy` node에 접속하여 클러스터를 사용합니다. 터미널 환경과 `vi` 에디터에 익숙한 user는 자신에게 친숙한 앱을 사용하면 됩니다. 그렇지 않은 경우 [`Visual Studio Code`](https://code.visualstudio.com/download)를 사용하는 것을 추천합니다. 이 문서에서는 `Visual Studio Code`를 사용하는 것을 전제로 합니다. 추천 이유는 다음과 같습니다.

- Windows, MacOS, Linux에서 모두 사용 가능합니다.
- 터미널과 에디터, 파일 브라우저가 통합되어 있습니다.
  - 불편하게 `vi`나 `nano`등의 CLI용 텍스트 에디터를 사용할 필요가 없습니다.
  - 파일 전송시 `scp`등의 복잡한 프로토콜을 사용하지 않고 drag & drop으로 수행할 수 있습니다.

`Visual Studio Code`외에 다른 앱을 사용하실 경우 추천하는 앱은 다음과 같습니다.

- Windows 10: [WSL2(Windows Subsystem for Linux 2)](https://docs.microsoft.com/ko-kr/windows/wsl/install)와 [Windows Terminal]((https://docs.microsoft.com/ko-kr/windows/terminal/install))을 설치하여 사용하는 것을 추천합니다.
- MacOS: 기본 터미널을 사용해도 되지만, [iTerm2](https://iterm2.com)를 추천합니다.
- iOS: [5번 문서](https://hpc.stat.yonsei.ac.kr/docs/05_ipad)를 참조하세요.
- Android: [Termux](https://play.google.com/store/apps/details?id=com.termux&hl=ko&gl=US)

### proxy node

- `proxy` node는 user가 로그인하여 파일을 정리하고 `cpu-compute`나 `gpu-compute` node에 job을 제출하는 용도로만 쓰는 컴퓨터입니다. 
  - 따라서 `proxy` node는 성능이 낮습니다. `proxy` node에서는 Python 작업 등을 실행하지 마세요.
- 터미널에서 `SSH` 접속만 할 수 있다면 어떤 기기에서도 `proxy` node에 접속하여 job을 제출할 수 있습니다.
- 일단 job을 제출하면, 터미널이 종료되고 user와 `proxy` node 간의 연결이 끊겨도 job은 계속 `cpu-compute`나 `gpu-compute` node에서 실행됩니다.
- Standard output(Python, R에서 console에 출력되는 메시지)이 로그 파일에 기록되므로, 나중에 다시 터미널에 접속하여 job 실행 현황을 확인할 수 있습니다.

## Step 2 - proxy node SSH 접속

[Visual Studio Code](https://code.visualstudio.com/download)를 설치하고 아래 안내에 따라 설정합니다.

### 1. Visual Studio Code extensions에서 Remote Development 설치[^fn3]

Microsoft가 제공하는 `Remote Development` extension pack을 설치합니다. Remote-WSL, Remote-Containers, Remote-SSH가 자동적으로 같이 설치됩니다.
![vscode_ssh_1](/img/vscode_ssh_1.png)
![vscode_ssh_2](/img/vscode_ssh_2.png)

### 2. Remote Explorer에서 SSH Targets를 선택 후, Add New 클릭

![vscode_ssh_3](/img/vscode_ssh_3.png)

### 3. ssh 접속 커맨드 입력

아래와 같은 창이 뜨면 `SSH` 커맨드를 입력하여 `proxy node`에 접속합니다. 
![vscode_ssh_4](/img/vscode_ssh_4.png)

아래 커맨드에 Slack으로 안내받은 port, username을 넣어서 위 창에 입력하고 `Enter`키를 누르면 됩니다. `SSH`의 default port는 `22`이지만, 저희는 보안상 이유로 다른 port를 사용합니다.

```bash
ssh -p [port] [username]@hpc.stat.yonsei.ac.kr 
```

또는 안내받은 proxy node의 ip를 입력해도 됩니다.

```bash
ssh -p [port] [username]@[ip]
```

### 4. SSH configuration file을 저장할 장소 선택

**Select SSH configuration file to update**가 나오면 맨 위 항목을 선택합니다.
![vscode_ssh_5](/img/vscode_ssh_5.png)

**Host added!** 라는 메시지가 우측 하단에 나옵니다.
![vscode_ssh_6](/img/vscode_ssh_6.png)

### 5. Remote Explore에서 Connect to Host in New Window 선택

![vscode_ssh_7](/img/vscode_ssh_7.png)

### 6. 서버 Platform 선택

Linux를 선택합니다.
![vscode_ssh_8](/img/vscode_ssh_8.png)

### 7. Password 입력

안내받은 password를 입력하여 로그인합니다. 
![vscode_ssh_9](/img/vscode_ssh_9.png)

### 8. 파일 시스템 마운트

좌측 탭의 파일 모양 아이콘을 클릭하고 **Open Folder** 버튼을 클릭합니다.
![vscode_ssh_10](/img/vscode_ssh_10.png)

기본적으로 user home directory 경로가 입력되어 있습니다. OK를 누릅니다.
![vscode_ssh_11](/img/vscode_ssh_11.png)

### 9. 둘러보기

![vscode_ssh_12](/img/vscode_ssh_12.png)

- 좌측 file explorer에서 파일을 관리합니다. Windows 탐색기나 MacOS Finder에서 drag&drop으로 파일을 옮길 수 있습니다. 클러스터 내부의 파일을 user의 local 컴퓨터로 가져오는 것도 drag&drop으로 가능합니다.

- `ctrl + shift + ~`키를 누르면 터미널이 열립니다. 여기서 서버 사용에 필요한 커맨드를 입력합니다. 터미널은 여러 개 띄울 수 있습니다.

- text editor에서 코드와 스크립트를 수정하고 이미지 파일 등을 열람합니다.

### 10. user password 변경

모든 user의 초기 password가 다 동일하기 때문에, 각 user는 첫 접속 시 password를 변경할 것을 권장합니다. 터미널에서 아래 커맨드를 입력하여 password를 변경합니다.

```bash
passwd
```

## Step 3. 파일 시스템 구조 이해

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

## Step 4. Conda environment 생성

- `cpu-compute` node에는 conda version 4.11.0이 설치되어 있으며 Python version을 3.10까지 지원합니다[^fn1].
- `gpu-compute` node에는 conda version 4.6.14가 설치되어 있으며 Python version을 3.8까지 지원합니다.

### Step 4.1. Conda environment 생성 batch script 작성

여기서는 `cpu-compute` node에서 conda environment를 생성하는 방법을 설명합니다. 두 가지 방법 중 하나를 선택하여 진행합니다. 
- 4.1.1. 직접 파이썬 버전과 설치할 패키지 목록을 명시해서 생성
- 4.1.2. yml 파일을 통해 생성

### 4.1.1. 직접 파이썬 버전과 설치할 패키지 목록을 명시해서 생성

Conda environment 생성 slurm batch script를 작성합니다. 아래는 script 예시입니다.

```bash
#!/bin/bash
#SBATCH --job-name=testEnv 
#SBATCH --nodes=1
#SBATCH --time=99:59:59
#SBATCH --mem=4gb
#SBATCH --partition=all
#SBATCH --nodelist=cpu-compute
#SBATCH --output=testEnv.log
#SBATCH --error=testEnv.err
CONDA_BIN_PATH=/opt/miniconda/bin
ENV_NAME=testEnv 
ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME #environment의 경
$CONDA_BIN_PATH/conda env remove --prefix $ENV_PATH #envenvironment가 이미 존재하면 삭제
$CONDA_BIN_PATH/conda create -y --prefix $ENV_PATH python=3.8.12 
source $CONDA_BIN_PATH/activate $ENV_PATH #conda를 환경변수에 등록하여 conda명령어만으로 conda가 실행되도록 함
conda install -y lightgbm scikit-learn pandas numpy #설치할 패키지 목록. -y는 yes/no question에 yes로 답하도록 함. 설치 중에 상호작용이 불가능하므로 -y 옵션이 꼭 필요함
```
conda install -y 뒤에 설치를 원하는 패키지명을 입력합니다.
위 내용에서
- 2번 라인의 **#SBATCH --job-name=testEnv**의 job name을 원하는 이름으로 변경합니다.
- 3번 라인의 #SBATCH --time=99:59:59 은 작업 시간 최대 허용치를 의미합니다.
- 7번 라인의 **#SBATCH --output=testEnv.log**의 output log 파일명과 경로, 8번 라인의 **#SBATCH --error=testEnv.err**의 error log 파일명과 경로를 원하는 데로 변경합니다.
- 10번 라인의 environment name을 원하는 이름으로 변경합니다.
- environment 저장 경로를 바꾸고 싶을 경우 11번 라인의 ENV_PATH=/mnt/nas/users/\$(whoami)/.conda/envs/\$ENV_NAME 에서 \$(whoami)와 \$ENV_NAME 사이 내용을 원하는 경로로 변경합니다. 이렇게 경로를 바꿀 경우, Python 코드를 실행하는 job을 제출할 때 바뀐 ENV_PATH를 써 주어야 합니다.
- 13번 라인의 python version을 원하는 버전으로 변경합니다. 
- 15번 라인의 패키지 설치 커맨드에서 conda install -y 뒤에 설치를 원하는 패키지 이름을 입력합니다.
  
위 내용에 따라 job script를 알맞게 수정하여 `Visual Studio Code`에서 작성한 뒤, 클러스터 내 user home directory에 `[your_env_name].job`으로 저장합니다(e.g.  `testEnv.job`).


### 4.1.2. yml 파일을 이용해 생성
아래는 yml 파일을 이용해 environment를 생성하는 scipt 예시입니다.
```bash
#!/bin/bash
#SBATCH --job-name=testEnvGPU
#SBATCH --nodes=1
#SBATCH --mem=8gb
#SBATCH --partition=all
#SBATCH --nodelist=gpu-compute
#SBATCH --time=30:50:00
#SBATCH --output=/mnt/nas/users/mjm/testEnvGPU.log
#SBATCH --error=/mnt/nas/users/mjm/testEnvGPU.err
CONDA_BIN_PATH=/opt/miniconda/bin
cd /mnt/nas/users/$(whoami)/.conda/envs/ #environment 저장 경로
conda env create --file /mnt/nas/users/mjm/environment.yml
```
- #SBATCH --output, #SBATCH --error를 원하는 경로와 파일명으로 변경합니다.
- environment 저장 경로를 바꾸고 싶을 경우 cd /mnt/nas/users/\$(whoami)/.conda/envs/ 에서 \$(whoami) 뒷부분을 원하는 경로로 변경합니다. 이렇게 경로를 바꿀 경우, Python 코드를 실행하는 job을 제출할 때 바뀐 ENV_PATH를 써 주어야 합니다.
- conda env create --file 뒷쪽에 yml 파일 경로를 입력합니다.
  

### Step 4.2. 작성한 스크립트 실행하기.
`Visual Studio Code` 하단 터미널에
   
   ```bash
   sbatch [your_env_name].job
   ```
  를 입력해 slurm batch job submission을 수행합니다. 작업이 노드에서 성공적으로 실행되면
   
   ```text
   Submitted batch job 401
   ```
와 같은 메시지가 뜨고 job 번호가 할당됩니다. 할당되는 job 번호는 나중에 squeue를 통해 정보를 확인하거나 job을 취소할 때 이용되므로 기록해 놓아야 합니다.
   
   ```bash
   squeue
   ```
커맨드를 통해 작업 실행 현황을 확인할 수 있습니다.
   
   ```text
   JOBID PARTITION     NAME     USER ST    TIME  NODES NODELIST(REASON)
   402     all       testEnv    mjm  R    0:01    1  cpu-compute
   ```
또는 아래 커맨드를 통해 실시간(1초 단위)으로 작업 실행 현황을 확인할 수 있습니다. **ctrl+c**로 escape 할 수 있습니다.
   
   ```bash
   smap -i 1 
   ```
다음 커맨드를 통해 output log, error log파일의 내용을 확인할 수 있습니다.
   
   ```bash
   cat [your_job-name].log
   cat [your_job-name].err
   ```
log 파일의 내용을 다음 커맨드를 통해 실시간으로 확인할 수 있습니다. **ctrl+c**로 escape할 수 있습니다.
   
   ```bash
   tail -f [your_job_name].log
   tail -f [your_job_name].err
   ```
터미널을 여러 개 띄워 놓고 각각에 `smap`과 `tail` 커맨드를 입력하면 편리하게 실행 상황을 모니터링할 수 있습니다.
   
위 샘플 코드는 약 3분 안에 작업이 완료됩니다. smap에서 작업 목록이 사라진 후 **cat**으로 로그 파일을 열어서
   
   ```
   pandas-1.1.3         | 10.5 MB   |            |   0% 
   pandas-1.1.3         | 10.5 MB   | #######1   |  71% 
   pandas-1.1.3         | 10.5 MB   | ########## | 100% 
   Preparing transaction: ...working... done
   Verifying transaction: ...working... done
   Executing transaction: ...working... done
   ```
   
와 같은 기록이 남아 있으면 conda environment 생성이 완료된 것입니다.
   
작업이 끝나기 전에 취소하려면 **scancel** 커맨드를 사용합니다.
   
   ```bash
   scancel [job_number]
   ```

## 

가상환경을 생성한 후에 패키지를 추가로 설치할 때는 위의 job script에서 가상환경을 삭제하고 다시 만드는 부분

```bash
CONDA_BIN_PATH/conda env remove --prefix ENV_PATH 
CONDA_BIN_PATH/conda create -y --prefix ENV_PATH python=3.8.12
```

을 삭제하고 아래와 같이 패키지를 설치하는 커맨드를 추가해 준 다음 job을 제출합니다. -y는 설치 중간에 yes/no 를 물을 경우 자동으로 yes를 입력하게 하는 옵션입니다. 설치 중에 컴퓨터를 직접 조작할 수 없기 때문에 꼭 사용해야 하는 옵션입니다. 아래는 `pyyaml` 패키지를 설치하는 job script 예시입니다.

```
```bash
   #!/bin/bash
   #SBATCH --job-name=install_package 
   #SBATCH --nodes=1
   #SBATCH --time=99:59:59
   #SBATCH --mem=4gb
   #SBATCH --partition=all
   #SBATCH --nodelist=cpu-compute
   #SBATCH --output=testEnv.log
   #SBATCH --error=testEnv.err
   CONDA_BIN_PATH=/opt/miniconda/bin
   ENV_NAME=testEnv 
   ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME #environment의 경
   source $CONDA_BIN_PATH/activate $ENV_PATH 
   conda install -y pyyaml #설치할 패키지 목록. -y는 yes/no question에 yes로 답하도록 함. 설치 중에 상호작용이 불가능하므로 -y 옵션이 꼭 필요함
```

가상환경 생성 및 패키지 설치 중 일어나는 오류 중 상당수가 Python에서 기본으로 제공하는 패키지를 설치하려 하거나, 설치할 패키지의 이름을 잘못 입력했기 때문에 발생합니다. 예를 들어, 위에서 설치한 `pyyaml` 패키지는 실제로 Python에서 import할 때는 `import yaml`로 입력하는데, `conda install yaml`이라고 job script에 쓸 경우 오류가 발생합니다.

## Step 5. Slrum batch script 작성하여 서버에 제출하기

### 5.1. Python 코드 작성

이제 클러스터에서 실행할 Python 코드를 local에서 작성합니다. 코드가 오류 없이 작동하는지 local에서 확인합니다. 그 후 코드 파일을 user home directory에 옮기거나, `Visual Studio Code`내에서 작성하여 저장합니다.

아래는 tree 기반 boosting 알고리즘인 LightGBM으로 mnist dataset을 분류하는 코드입니다. Boosting round를 10회 수행하고 학습 결과를 csv파일로 저장합니다[^fn4]. Batch script를 작성할 때는 알고리즘의 output이 자동으로 저장되지 않으므로 파일로 결과를 저장하는 코드를 포함하는 것이 좋습니다. 단, 콘솔에 출력되는 내용은 output log에 자동으로 기록됩니다. 아래 코드를 `python_test_cpu.py`로 저장하여 user home directory에 둡니다.

```python
import numpy as np
from time import process_time
from lightgbm import LGBMClassifier
from sklearn.metrics import accuracy_score, log_loss
from sklearn.datasets import fetch_openml
from sklearn.model_selection import train_test_split


def lgb(n=10, c=0, sequence=1):

    mnist = fetch_openml('mnist_784')
    x_train, x_test, y_train, y_test = train_test_split(mnist.data, mnist.target, test_size=0.33, random_state=42)



    proba_test = np.zeros((n, y_test.shape[0], len(np.unique(y_test))))
    proba_train = np.zeros((n, y_train.shape[0], len(np.unique(y_train))))

    test_score = []
    train_score = []

    tr_time = []
    seq = []
    while(n):

        model = LGBMClassifier(n_estimators=sequence)

        t0 = process_time()
        model.fit(x_train, y_train)
        tr_time.append(process_time() - t0)
        test_score.append(accuracy_score(y_test, model.predict(x_test)))
        train_score.append(accuracy_score(y_train, model.predict(x_train)))
        proba_test[c, ] = model.predict_proba(x_test)
        proba_train[c, ] = model.predict_proba(x_train)

        seq.append(sequence)
        sequence *= 2
        n -= 1
        c += 1

        ce_train = []
        ce_test = []

    for i in range(10):
        ce_test.append(log_loss(y_test, proba_test[i]))
        ce_train.append(log_loss(y_train, proba_train[i]))

        np.savetxt('round'+ str(i) + 'proba_test.csv', proba_test[i])
        np.savetxt('round'+ str(i) + 'proba_train.csv', proba_train[i])

    np.savetxt('test_score.csv', test_score, delimiter=',')
    np.savetxt('train_score.csv', train_score, delimiter=',')

    np.savetxt('ce_test.csv', ce_test, delimiter=',')
    np.savetxt('ce_train.csv', ce_train, delimiter=',')


if __name__ == '__main__':
    lgb()
```

### 5.2. 현재 클러스터 자원 사용량 확인

아래 커맨드를 통해 `cpu-compute` 노드의 cpu와 RAM 사용 현황을 볼 수 있습니다.

```bash
sinfo -o "%n %e %m %a %c %C"
```

아래와 같은 결과가 나옵니다.

```
HOSTNAMES FREE_MEM MEMORY AVAIL CPUS CPUS(A/I/O/T)
cpu-compute 105589 128916 up 32 0/32/0/32
gpu-compute 53318 80532 up 16 0/16/0/16
```

- CPUS의 A/I/O/T는 allocated/idle/other/total을 의미합니다. 
- 자신의 job이 바로 실행되기를 원한다면, Slurm batch script를 작성할 때 
  - RAM 용량을 FREE_MEM보다 적게 설정해야 합니다. 
  - CPU 코어 개수를 CPUS idle보다 적게 설정해야 합니다.
- 현재 가용 자원보다 더 많은 자원을 요구하는 script를 작성하면, job이 바로 실행되지 않습니다. 대기 상태에 있다가 다른 user들의 job이 끝나고 자원이 반환되면 job이 실행됩니다.

### 5.3. Slurm batch script 작성

앞선 단계에서 만든 해당 conda environment를 activate하고 코드를 실행하는 Slurm batch script를 작성합니다. 클러스터 소개 페이지의 [slurm job configurator](https://hpc.stat.yonsei.ac.kr/tools/job-configurator.html)를 사용하면 script를 쉽게 작성할 수 있습니다. 

![slurm_config](/img/slurm_config.png)

- Conda activate에 체크합니다.
- 빈칸들을 채웁니다. 사용 시간을 넉넉하게 입력할 것을 권장합니다.
- Script란에 **python xxx.py**라고 작성합니다. 이는 home directory에 있는 **xxx.py** 파일을 Python으로 실행하라는 의미입니다. Job script를 작성하거나 sbatch 명령어를 사용할 때, visual studio code의 explorer에서 파일명을 마우스 우클릭하고 경로 복사를 사용하면 편리합니다.
- **Print & Copy** 버튼을 누르면 내용이 클립보드에 복사됩니다. 

Slurm batch script의 내용은 아래와 같습니다.

```bash
#!/bin/bash 
#
#SBATCH --job-name=python_test_cpu
#SBATCH --partition=all
#SBATCH --account=mjm
#SBATCH --mem=4gb
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=01:00:00
#SBATCH --output=/mnt/nas/users/mjm/python_test_cpu.log
#SBATCH --error=/mnt/nas/users/mjm/python_test_cpu.err
#SBATCH --nodelist=cpu-compute

CONDA_BIN_PATH=/opt/miniconda/bin
ENV_NAME=testEnv
ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME
source $CONDA_BIN_PATH/activate $ENV_PATH

python python_test_cpu.py
```

`python_test_cpu.job`이라는 이름으로 클러스터의 user home directory에 저장합니다.

Script 윗부분의 #SBATCH 옵션들의 의미는 다음과 같습니다.

- **—job-name**: 수행할 작업의 이름
- **—mem**: memory limit
- **—nodelist**: 작업할 노드의 이름
- **—ntasks**: 작업의 수
- **—cpus-per-task**: 각 작업에서 사용할 cpu 코어의 수
- **—time**: 작업 제한시간
- **—accoun**t: 해당 작업을 수행하는 계정의 이름
- **—partition**: group of nodes with specific characteristics
- **--nodelist**: 사용할 node의 이름
- **—output**: 코드 실행 결과 log 파일. 확장자는 out이나 log가 가능합니다.
- **—error**: 코드 실행 결과 log
  error 파일과 log 파일의 파일명과 저장 경로는 원하는 데로 수정할 수 있습니다.
  sbatch에 대한 더 자세한 정보는 [Slurm 공식 웹페이지](https://slurm.schedmd.com/sbatch.html)를 참조하세요.

### 5.4. Slurm batch script 실행

Conda environment를 만들 때처럼, **sbatch** 커맨드를 통해 job을 제출합니다. 할당되는 job 번호는 나중에 squeue를 통해 정보를 확인하거나 job을 취소할 때 이용되므로 기록해 놓아야 합니다.

Step 4에서처럼, `ctrl+shift+~`를 눌러 터미널을 여러 개 띄우고 **smap -i**로 작업 현황을 확인하고, **tail -f xxx.out**, **tail -f xxx.err**으로 콘솔 출력이나 error를 확인합니다. 작업은 4분 정도 걸립니다.

```bash
sbatch python_test_cpu.job
```

현재 작업이 자원을 얼마나 할당받았는지 확인하려면 다음 커맨드를 사용합니다. NumCPUs=4가 코어를 4개 할당받았다는 뜻이고, mem=4G가 RAM을 4gb 할당받았다는 뜻입니다. 이 커맨드는 다른 user가 제출한 job에 대해서도 사용할 수 있습니다.

```bash
scontrol show job [job number]
```

```text
UserId=mjm(1003) GroupId=mjm(1003) MCS_label=N/A
   Priority=4294901694 Nice=0 Account=mjm QOS=(null)
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=1 Reboot=0 ExitCode=0:0
   RunTime=00:00:05 TimeLimit=01:00:00 TimeMin=N/A
   SubmitTime=2022-03-17T15:31:01 EligibleTime=2022-03-17T15:31:01
   StartTime=2022-03-17T15:31:01 EndTime=2022-03-17T16:31:01 Deadline=N/A
   PreemptTime=None SuspendTime=None SecsPreSuspend=0
   LastSchedEval=2022-03-17T15:31:01
   Partition=all AllocNode:Sid=proxy:30897
   ReqNodeList=cpu-compute ExcNodeList=(null)
   NodeList=cpu-compute
   BatchHost=cpu-compute
   NumNodes=1 NumCPUs=4 NumTasks=1 CPUs/Task=4 ReqB:S:C:T=0:0:*:*
   TRES=cpu=4,mem=4G,node=1,billing=4
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=4 MinMemoryNode=4G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   Gres=(null) Reservation=(null)
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/mnt/nas/users/mjm/python_test_cpu.job
   WorkDir=/mnt/nas/users/mjm
   StdErr=/mnt/nas/users/mjm/python_test_cpu.err
   StdIn=/dev/null
   StdOut=/mnt/nas/users/mjm/python_test_cpu.log
   Power=
```

`Visual Stuio Code`의 file explorer는 실시간으로 변화가 반영되지 않습니다. 새로고침 버튼을 눌러 주면 변화가 반영되고 output 파일이 explorer에 보입니다.

## terminal에서는 ssh 접속이 되지만 Visual Studio Code에서는 되지 않을 때
sudo 권한이 필요하므로 조교에게 문의하세요.

해결방법:
```bash
lsof +D ./.vscode-server
lsof +D ./.vscode-remote-containers
```
로 이 폴더를 사용하는 프로세스 번호를 확인한 후(e.g. 46088)
```bash
sudo kill 46088
```

그 후 아래 폴더를 삭제합니다.
```bash
sudo rm -rf .vscode-server
sudo rm -rf .vscode-remote-containers
```
## 더 알아보기

[Submitting a slurm job script](https://ubccr.freshdesk.com/support/solutions/articles/5000688140-submitting-a-slurm-job-script)

[SLRUM Job Examples](https://doc.zih.tu-dresden.de/jobs_and_resources/slurm_examples/)

## References

[^fn1]: https://docs.conda.io/projects/conda/en/latest/release-notes.html
[^fn2]: https://github.com/conda/conda/issues/9399
[^fn3]: https://jstar0525.tistory.com/14
[^fn4]: https://www.kaggle.com/samanemami/script-lightgbm-mnist