---
title: "How to Use GPU Node for SLURM"
author: "Gwnaghee Kim, Jongmin Mun"
date: 2022-03-15T14:54:35+09:00
draft: false
---

# yml 파일 이용해 local과 동일한 conda 환경 구축하기

**conda env export** 커맨드를 이용해 environment 전체를 `.yml` 파일로 만들고 이를 이용해 `cpu-compute` 노드에서 environment를 구축하는 것이 가장 이상적인 방법입니다. 하지만 이 방법은 user의 local 컴퓨터가 linux가 아니면(특히 Windows일 경우) 자잘한 오류가 발생합니다[^fn2]. 차선책으로 설치된 패키지 목록과 버전만을 `requirements.txt`로 추출하여 노드에서 `cpu-compute`에서 설치할 수 있습니다.


1. User의 local 컴퓨터에서에서 아래의 커맨드를 통해 virtual environment로부터 환경설정 파일을 추출합니다. **—no-builds**는 서로 다른 OS에서 conda environment 내 패키지들의 버전 충돌을 방지하기 위한 옵션입니다. 

    ```bash
    # export conda setting
    conda activate [YOUR ENV NAME]
    conda env export > [ENV NAME] -f [FILENAME].yml --no-builds # OS 문제를 해결해주는 옵션이지만, 완전히 해결되지 않을 수도 있습니다.
    ```


2. `yml` 파일을 `scp`를 통해 클러스터 내의 user home directory로 전송합니다. 위의 Step 3 또는 [여기를](https://hpc.stat.yonsei.ac.kr/docs/) 참조하세요.

3. Conda environment 생성 batch script를 작성합니다.
    ```bash
    #!/bin/bash

    #“#SBATCH” directives that convey submission options:

    #SBATCH --job-name=conda-env-create
    #SBATCH --nodes=1
    #SBATCH --time=50:00
    #SBATCH --account=dummyuser
    #SBATCH --partition=all
    #SBATCH --nodelist=cpu-compute
    #SBATCH --output=testEnv.out
    #SBATCH --error=testEnv.err

    # The application(s) to execute along with its input arguments and options:
    CONDA_BIN_PATH=/opt/miniconda/bin

    ENV_NAME=testEnv #local에서와 같은 이름으로 입력

    ENV_PATH=/mnt/nas/users/$(whoami)/.conda/envs/$ENV_NAME
    $CONDA_BIN_PATH/conda env remove --prefix $ENV_PATH

    conda env create -p ENV_PATH -f testEnv.yml
    ```
    
    위 내용에서
    - x번 라인의 **#SBATCH --output=testEnv.out**의 output log 파일명
    - x번 라인의 **#SBATCH --error=testEnv.err**의 error log 파일명
    - x번 라인의 environment name
    - xx번 라인의 yml 파일 이름
    을 원하는 데로 수정한 뒤  local에서 `testEnv.sh`로 저장하여 `scp`로 클러스터 내 user home directory로 전송합니다. 또는 proxy node에 접속한 뒤 home directory에서

    ```bash
    vi testEnv.sh
    ```
    를 입력하여 linux의 텍스트 에디터를 실행한 뒤 `ctrl+v`로 위 내용을 붙여넣기하고 `esc`, `:wq`, `ENTER`를 차례로 눌러 저장합니다.

4. 작성한 스크립트 실행하기. proxy node에 SSH접속한 뒤 home directory에서 
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

    다음 커맨드를 통해 error log파일의 내용을 확인할 수 있습니다.
    ```bash
    cat testEnv.err
    ```
    Local 환경이 linux가 아니었을 경우 거의 항상 다음과 같은 오류가 발생합니다. 

    ```bash
    ResolvePackageNotFound:
     - libgfortran=5.0.0
    ```


    이 경우 user가 직접 env 파일을 수정해야 합니다. 이런 경우 **ResolvePackageNotFound** 아래에 나오는 패키지들을 `yml` 파일에서 삭제해주면 문제가 해결될 수 있습니다.

    ```bash
    vi testEnv.yml
    ```
    로 `yml` 파일을 열면 아래와 같은 결과가 나오는데,

    ```text
    "testEnv.yml" 36L, 673C                                                     1,1           Top
    name: testEnv
    channels:
       - conda-forge
       - anaconda
       - defaults
    dependencies:
       - ca-certificates=2020.10.14
       - certifi=2020.6.20
       - joblib=1.1.0
       - libblas=3.9.0
       - libcblas=3.9.0
       - libcxx=12.0.0
       - libffi=3.3
       - libgfortran=5.0.0
       - libgfortran5=9.3.0
       - liblapack=3.9.0

  화살표 키를 움직여 삭제해야 하는 줄까지 이동한 다음 `dd`를 입력하면 해당 줄이 삭제됩니다. 그 후 `esc`, `:wq`, `ENTER`를 차례로 눌러 저장합니다.
 
  이 방법으로 문제가 해결될 수도 있고 아닐 수도 있습니다. 이 튜토리얼에서는 문제가 해결되지 않습니다. 

  [conda 패키지 목록](https://anaconda.org/search?q=platform%3Alinux-64+type%3Aconda+libgfortran)에서 `libgfortran`을 검색하고 platform 옵션을 linux-64로 설정하면, 버전이 3.0.0까지 밖에 없는 것을 알 수 있습니다. 반면 osx-64(MacOS)는 5.0.0 버전이 존재합니다. 이 때문에 MacOS에서 만든 virtual environment가 linux OS를 사용하는 `cpu-compute` 노드에서 재현될 수 없었던 것입니다. 
