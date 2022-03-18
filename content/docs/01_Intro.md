---
title: "1. Introduction"
author: "Jongmin Mun"
date: 2022-03-14T22:24:04+09:00
draft: False
---

## 요약
Chili Pepper는

- 총 세 개의 컴퓨터로 구성된 컴퓨팅 클러스터입니다. Cpu node 하나와 gpu node 하나, 그리고 이 둘을 관리하는 proxy node로 구성되어 있습니다.
- Slurm이라는 job scheduler로 여러 사용자의 작업을 각 node에 효율적으로 할당합니다.
- 각 사용자는 자신만의 conda environment를 스스로 생성하여 사용합니다.
- 기본적으로 non-interactive입니다.사용자가 자신의 로컬 컴퓨터에서 작성 및 테스트한 코드를 서버에 제출하면 서버가 해당 코드를 실행합니다. 주피터 노트북 환경은 실행 가능하지만, 꼭 필요한 경우에 한하여 요청 시 제공합니다. 이는 노트북 환경은 작업 단위가 아닌 시간 단위로 실행되기 때문에 서버의 RAM 자원을 지나치게 많이 점유하고 다른 작업의 실행을 방해하기 때문입니다.
- 파일은 위 세 컴퓨터와 별개의 NAS(Network Attached Storage)에 저장되며, 모든 컴퓨터에 마운트되어 있으므로 한번 클러스터 내로 옮긴 파일은 클러스터 내부에서 다시 옮길 필요가 없습니다.

## 1. 클러스터  구성

### 1. cpu node
- Name: cpu-compute
- CPU: Intel(R) Xeon(R) Gold 5220 CPU @ 2.20GHz 32 cores
- RAM: 128GB
- OS: ubuntu 18.04.6 LTS
- conda version: 4.11.0
  

### 2. gpu-compute node
- Name: gpu-compute
- CPU: Intel(R) Xeon(R) Gold 5220 CPU @ 2.20GHz 16 cores
- GPU: NVIDIA® Tesla® T4 16GB x 2
- RAM: 80GB
- OS: ubuntu 18.04.6 LTS
- conda version: 4.6.14
- GPU driver version: 418.67

### 3. proxy node
- Name: proxy
- CPU: Intel(R) Xeon(R) Gold 5220 CPU @ 2.20GHz 2cores
- RAM: 4GB
- OS: ubuntu 18.04.6 LTS

### 특이사항
- Intel(R) Xeon(R) Gold 5220 CPU @ 2.20GHz가 18코어 제품임에도 불구하고 세 컴퓨터에 각각 32코어, 16코어, 2코어로 장착되어 있는 것은 이 세 컴퓨터가 네이버 클라우드 플랫폼에서 호스팅되고 있기 때문입니다. 네이버 클라우드 플랫폼에서 각 컴퓨터에 정해진 코어 수를 할당합니다. 

- proxy node는 사용자가 로그인하여 job을 제출하기 위한 용도로만 쓰이는 컴퓨터이므로 성능이 낮습니다.

- 사용자는 proxy에만 접속할 수 있으며, cpu-compute와 gpu-compute에는 접속할 수 없습니다.

## 2. Slurm job scheduler

### Job scheduler
HPC(High performance computing) 시스템은 개인용 컴퓨터와 달리 여러 사용자가 여러 node를 공유하며 사용합니다. 따라서 누구의 작업이 언제 어느 node에서 실행될지 결정해 주어야 합니다. 이러한 역할을 수행하는 것이 job scheduler입니다.

Job scheduler를 식당의 웨이터에 비유할 수 있습니다. 식당에 사람이 많으면 줄을 서서 기다려야 합니다. 웨이터는 각 손님 그룹의 수에 맞는 자리가 나면 그 그룹을 테이블로 안내합니다[^fn1].
![restaurant](https://raw.githubusercontent.com/Yonsei-Stats-and-Data-Science/chili-pepper-homepage/main/assets/restaurant_queue_manager.svg)

#### 용어
- Job: 사용자가 클러스터에서 실행하고자 하는 코드(bash, python, R 등을 모두 포함)

- Batch job submission: 사용자가 미리 작성한 코드(output을 파일로 저장하는 내용 포함)를 scheduler에게 제출하여 non-interactive하게 실행하는 것

### Slurm
Slurm(Simple Linux Utility for Resource Management)은 HPC에서 많이 채용하는 job scheduler입니다. 전 세계 TOP 500 슈퍼컴퓨터 중 60%가 slurm을 사용합니다[^fn2].

Slurm은 각 사용자의 cpu사용량 등 다양한 통계를 기반으로 작업의 우선순위를 결정할 수 있습니다. 예를 들어, University of Toronto의 Computer Science Department의 HPC는 사용 CPU 코어 수 * 사용 시간(초) 값이 낮은 사용자의 job을 먼저 실행합니다. RAM 사용량은 0.25GB당 CPU 1코어 사용으로, GPU 사용량은 GPU 1개당 CPU 16코어 사용으로 환산합니다[^fn3].

현재 Chili Pepper는 위와 같은 점수제가 아닌 FIFO(First In, First Out) 규칙을 사용하고 있습니다. 일정 기간 운영해본 뒤 상황에 맞추어 규칙을 변경할 예정입니다.

 
## 3. Conda environment

### Virtual environment
Virtual environment를 사용하면 한 컴퓨터 내에서 각 user가 독립적으로 Python과 R 패키지를 관리할 수 있습니다. 또한, 여러 컴퓨터에서 동일한 환경을 구축하여 패키지 버전 차이로 인한 문제 발생을 방지할 수 있습니다[^fn4]. 이는 특히 GPU driver, CUDA, cuDNN, 딥러닝 라이브러리의 버전 간 호환성이 중요한 딥러닝 job에서 유용합니다.

여러 사용자가 node를 공유하는 HPC에서 virtual environment의 사용은 필수적입니다. 각 사용자는 자신의 로컬 컴퓨터와 동일한 Python 및 R 환경을 node 내에 구축합니다. 그리고 로컬 컴퓨터에서 코드를 작성하고 이상 없이 실행되는지 테스트합니다. 문제 없이 실행되는 것이 확인된 코드를 slurm을 통해 클러스터에서 실행합니다. Virtual environment는 로컬에서 작성한 코드가 클러스터에서 문제 없이 작동하는 것을 보장합니다.

### Conda environment
Chili Pepper는 conda를 이용해 virtual environment를 구현합니다. Conda는 Python, R, Ruby, Lua, Scalca, Java, Javascript, C/C++, FORTRAN 등 다양한 언어를 지원할 뿐만 아니라 Windows, MacOS, Linux를 모두 지원합니다[^fn5].

Chili Pepper에서 job을 실행하기 위해서는 반드시 conda environment를 사용해야 합니다. 






[^fn1]: https://epcced.github.io/hpc-intro/13-scheduler/index.html

[^fn2]: https://wiki.hpc.odu.edu/en/slurm

[^fn3]: https://support.cs.toronto.edu/systems/slurmresource.html

[^fn4]: https://www.marquette.edu/high-performance-computing/py-venv.php

[^fn5]: https://docs.conda.io/en/latest/
