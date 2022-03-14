---
title: "Overview"
author: "Jongmin Mun"
date: 2022-03-14T22:24:04+09:00
draft: False
---

## 요약
Chilli Pepper는

- 총 세 개의 컴퓨터로 구성된 컴퓨팅 클러스터입니다. Cpu node 하나와 gpu node 하나, 그리고 이 둘을 관리하는 proxy node로 구성되어 있습니다.
- Slurm이라는 job scheduler로 여러 사용자의 작업을 각 node에 효율적으로 할당합니다.
- 각 사용자는 자신만의 conda environment를 스스로 생성하여 사용합니다.
- 기본적으로 non-interactive입니다. 사용자가 자신의 로컬 컴퓨터에서 작성 및 시험한 코드를 서버에 제출하면 서버가 해당 코드를 실행합니다.
- 주피터 노트북 환경은 실행 가능하지만, 꼭 필요한 경우에 한하여 요청 시 제공합니다. 이는 노트북 환경은 작업 단위가 아닌 시간 단위로 실행되기 때문에 서버의 RAM 자원을 지나치게 많이 점유하고 다른 작업의 실행을 방해하기 때문입니다.

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

- proxy node는 사용자가 접속하여 job을 제출하기 위한 용도로만 쓰이는 컴퓨터이므로 성능이 낮습니다.

- 사용자는 proxy에만 접속할 수 있으며, cpu-compute와 gpu-copute에는 접속할 수 없습니다.

## 2. Job scheduler

## 3. Conda environment

## 4. 
