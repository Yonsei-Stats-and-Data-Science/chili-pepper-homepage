---
title: "3. CPU node 사용법(R)"
author: "Jongmin Mun"
date: 2022-03-17T14:54:35+09:00
draft: false
---

# 3. CPU node 사용법(R)
Step 1, 2, 3은 2번 문서와 내용이 같고 Step 4, 5만 다릅니다. 따라서 이 문서에서는 Step 4, 5만 작성합니다.

## Step 4. Conda environment 생성
- `cpu-compute` node에는 conda version 4.11.0이 설치되어 있습니다.[fn^1].
- `gpu-compute` node에는 conda version 4.6.14가 설치되어 있습니다.

여기서는 `cpu-compute` node에서 conda environment를 생성하는 방법을 설명합니다. local에서 작성한 코드가 `cpu-compute` node에서 오류 없이 작동하도록 하기 위해, local과 `cpu-compute` node에서 동일한 conda environment를 구축해야 합니다. 

### 1. local에서 conda environment 생성
이 섹션의 작업은 모두 클러스터가 아니라 user의 local 컴퓨터에서 진행합니다.

[miniconda](https://docs.conda.io/en/latest/miniconda.html)를 설치한 다음, local 컴퓨터의 터미널에서 아래 커맨드로 virtual environment를 설정합니다. `testEnv` 자리에 원하는 이름을 넣습니다.

```bash
conda create -n r_env r-base
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
r_env                  /opt/miniconda3/envs/r_env
```

Virtual environment에 진입한 뒤 패키지를 설치합니다. Conda에서 R 패키지 버전 관리가 잘 되지 않고 있기 때문에, conda로 설치하는 것은 추천하지 않습니다.

**install.packages()**를 사용하여 패키지를 설치합니다. CRAN mirror를 명시해 주어야 합니다. [CRAN mirror의 목록](https://cran.r-project.org/mirrors.html)을 참고하세요.
```bash
conda activate r_env

#For example,
Rscript -e 'install.packages(c("dplyr", "ggplot2", "lightgbm"), repos="https://cloud.r-project.org/")'
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

- **conda env export** 커맨드를 이용해 environment 전체를 `.yml` 파일로 만들고 이를 이용해 `cpu-compute` 노드에서 environment를 구축하는 것이 동일한 environment를 만드는 가장 이상적인 방법입니다.
- 또는 설치된 패키지 목록과 버전만을 `requirements.txt`로 추출하여 노드에서 `cpu-compute`에서 설치할 수 있습니다.
- 그러나 이 두 방법은 user의 local 컴퓨터가 linux가 아니면 오류가 발생할 확률이 매우 높습니다. 이는 유저가 패키지를 설치할 때 자동으로 설치되는 dependency들의 버전이 OS별로 다를 수 있기 때문입니다.
  - 예를 들어 `lightgbm` 2.0.7버전은 Python 버전이 3.6일 때 linux와 MacOS에서 둘 다 설치 가능하지만, 이 패키지의 dependency 중 하나인 `libgfortran`은 MacOS에서는 4.0.0버전이 설치되지만 linux에서는 3.0.0버전까지만 지원되기 때문에 MacOS에서 만든 environment에서 추출한 yml 파일이나 list txt 파일을 클러스터에서 사용하면 오류가 발생합니다[^fn2].
  - 이 오류는 적절한 조치를 통해 해결할 수 있을 때도 있지만 해결하기 힘들 때도 있습니다.

따라서 local에서 만든 environment를 cluster로 옮기기보다는, local의 environment에서 사용하는 Python 버전과 중요 패키지들의 버전을 그대로 사용하여 cluster 내에서 environment를 생성하는 것을 추천하며, 이 문서에서는 그 절차를 안내합니다.


1. Conda environment 생성 batch script를 작성합니다.
    ```bash
    #!/bin/bash
    #SBATCH --job-name=conda-env-create
    #SBATCH --nodes=1
    #SBATCH --mem=4gb
    #SBATCH --partition=all
    #SBATCH --nodelist=cpu-compute
    #SBATCH --output=testEnvR.log
    #SBATCH --error=testEnvR.err
    CONDA_BIN_PATH=/opt/miniconda/bin
    ENV_NAME=testEnvR #local에서와 같은 이름으로 입력
    ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME
    $CONDA_BIN_PATH/conda env remove --prefix $ENV_PATH
    $CONDA_BIN_PATH/conda create -y --prefix $ENV_PATH r-base
    source $CONDA_BIN_PATH/activate $ENV_PATH
    Rscript -e 'install.packages(c("dplyr", "ggplot2", "lightgbm"), repos="https://cloud.r-project.org/")'
    ```
    
    위 내용에서
    - 7번 라인의 **#SBATCH --output=testEnv.out**의 output log 파일명
    - 8번 라인의 **#SBATCH --error=testEnv.err**의 error log 파일명
    - 10번 라인의 environment name
    - 14번 라인의 패키지 설치 커맨드
    를 알맞게 수정하여 `Visual Studio Code`에서 작성한 뒤,클러스터 내 user home directory에 `testEnvR.job`으로 저장합니다.


2. 작성한 스크립트 실행하기. `Visual Studio Code` 하단 터미널에
    ```bash
    sbatch testEnv.job
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

    또는 아래 커맨드를 통해 실시간(1초 단위)으로 작업 실행 현황을 확인할 수 있습니다.
    ```bash
    smap -i 1 # ctrl+c로 escape 할 수 있습니다.
    ```

    다음 커맨드를 통해 output log, error log파일의 내용을 확인할 수 있습니다.
    ```bash
    cat testEnv.err
    ```
    
    error log 또는 output log는 다음 커맨드를 통해 실시간으로 확인할 수 있습니다.
    ```bash
    tail -f testEnv.log
    ```  
 

## Step 5. Slrum batch script 작성하여 서버에 제출하기

### 1. Python 코드 작성
이제 클러스터에서 실행할 Python 코드를 local에서 작성합니다. 머신 러닝 코드일 경우, Epoch 수를 작게 하는 등의 작업을 통해 빨리 실행되는 코드를 가지고 코드가 문제 없이 실행되는지 먼저 local에서 확인합니다. 그 후 실제로 실행할 코드를 작성하여 클러스터의 user home directory에 옮기거나, `Visual Studio Code`내에서 작성하여 저장합니다.

아래는 tree 기반 boosting 알고리즘인 LightGBM으로 mnist dataset을 분류하는 코드입니다. Matplotlib으로 boost round에 대한 loss curve와 accuracy curve의 plot을 만들어 `loss_curve.jpg`라는 파일로 저장합니다[fn^4]. Batch script를 작성할 때는 알고리즘의 output이 자동으로 저장되지 않으므로 파일로 결과를 저장하는 코드를 꼭 포함해야 합니다. 아래 코드를 `cpu_test_python.py`로 저장하여 user home directory에 둡니다.

```python
import numpy as np
from time import process_time
import matplotlib.pyplot as plt
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

    
    fig, ax1 = plt.subplots()

    l1 = ax1.plot(seq, ce_test, ':', label='loss-test', color='r')
    l2 = ax1.plot(seq, ce_train, ':', label='loss-train', color='b')
    ax1.set_xlabel("Boost rounds")
    ax1.set_ylabel("Cross Entropy")

    ax2 = ax1.twinx()
    l3 = ax2.plot(seq, test_score, label='accuracy-test', color='r')
    l4 = ax2.plot(seq, train_score, label='accuracy-train', color='b')
    ax2.set_ylabel("Accuracy")

    lb = l1 + l2 + l3 + l4
    label = [l.get_label() for l in lb]

    ax1.legend(lb, label, loc=0)
    plt.title('loss curve/ accuracy')
    plt.savefig('loss_curve.jpg', dpi=500)


if __name__ == '__main__':
    lgb()
```
### 2. 현재 클러스터 자원 사용량 확인
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
- 현재 가용 자원보다 더 많은 자원을 요구하는 script를 작성하면, job이 바로 실행되지 않습니다. 대기 상태에 있다가 다른 사용자들의 job이 끝나고 자원이 반환되면 job이 실행됩니다.


### 3. Slurm batch script 작성
앞선 단계에서 만든 해당 conda environment를 activate하고 코드를 실행하는 Slurm batch script를 작성합니다. 클러스터 소개 페이지의 [slurm job configurator](https://hpc.stat.yonsei.ac.kr/tools/job-configurator.html)를 사용하면 script를 쉽게 작성할 수 있습니다. 

![slurm_config](/assets/slurm_config.png)
- Conda activate에 체크합니다.
- 빈칸들을 채웁니다.
- Script란에 **python xxx.py**라고 작성합니다. 이는 home directory에 있는 **xxx.py** 파일을 Python으로 실행하라는 의미입니다.
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
  
sbatch에 대한 더 자세한 정보는 [Slurm 공식 웹페이지](https://slurm.schedmd.com/sbatch.html)를 참조하세요.

### 4. Slurm batch script 실행
Conda environment를 만들 때처럼, **sbatch** 커맨드를 통해 job을 제출합니다. 할당되는 job 번호는 나중에 job 정보를 확인하거나 job을 취소할 때 이용되므로 기록해 놓아야 합니다.

**squeue**나 **smap -i**로 작업 현황을 확인하고, **cat xxx.log**이나 **tail -f xxx.err**으로 콘솔 출력이나 error를 확인합니다.

```bash
sbatch python_test_cpu.job
smap -i 1 # 작업 현황을 1초마다 갱신하여 보여줍니다. ctrl+c로 escape 할 수 있습니다.
cat python_test_cpu.log
tail -f python_test_cpu.err
```

현재 작업이 자원을 얼마나 할당받았는지 확인하려면 다음 커맨드를 사용합니다. NumCPUs=4가 코어를 4개 할당받았다는 뜻이고, mem=4G가 RAM을 4gb 할당받았다는 뜻입니다.

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
작업이 완료되면 **squeue** `Visual Stuio Code`의 file explorer는 실시간으로 변화가 반영되지 않습니다. 새로고침 버튼을 눌러 주면 변화가 반영되고 output 파일이 explorer에 보입니다.
![vscode_file](/assets/vscode_file.png)

작업이 끝나기 전에 취소하려면 **scance** 커맨드를 사용합니다.
```bash
scancel [job number]
```
## 더 알아보기

[Submitting a slurm job script](https://ubccr.freshdesk.com/support/solutions/articles/5000688140-submitting-a-slurm-job-script)

[SLRUM Job Examples](https://doc.zih.tu-dresden.de/jobs_and_resources/slurm_examples/)


# Refernece
[fn^1]: https://docs.conda.io/projects/conda/en/latest/release-notes.html
[fn^2]: https://github.com/conda/conda/issues/9399
[fn^3]: https://jstar0525.tistory.com/14
[fn^4]: https://docs.anaconda.com/anaconda/user-guide/tasks/using-r-language/