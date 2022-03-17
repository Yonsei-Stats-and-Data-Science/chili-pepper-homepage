---
title: "3. CPU node 사용법(R)"
author: "Jongmin Mun"
date: 2022-03-17T14:54:35+09:00
draft: false
---

# 3. CPU node 사용법(R)
2번 문서의 Step 1, 2, 3을 먼저 숙지하시기 바랍니다. 이 문서는 그 이후의 내용만을 다룹니다.

### 1. R 코드 작성
클러스터에서 실행할 `R` 코드를 local에서 작성합니다. 코드가 문제 없이 실행되는지 먼저 local에서 확인합니다. 그 후 실제로 실행할 코드를 작성하여 클러스터의 user home directory에 옮기거나, `Visual Studio Code`내에서 작성하여 저장합니다.

`R`은 `cpu-compute`에만 설치되어 있습니다. `R`은 conda environment를 사용하지 않습니다. `R` 패키지들은 user별 directory가 아닌 NAS 내의 공통 폴더에 저장됩니다. 실행할 `R` 코드를 script로 작성한 다음 Slurm batch script 내에 해당 script를 실행하도록 **Rscript xxx.R** command를 적어주면 됩니다.

아래의 샘플 코드는 `xgboost`와 `caret`을 설치 및 로드하고 learning과 prediction을 수행한 다음 prediction 결과 plot을 jpeg로 저장하는 코드입니다. Batch script를 작성할 때는 알고리즘의 output이 자동으로 저장되지 않으므로 파일로 결과를 저장하는 코드를 포함하는 것이 좋습니다. 아래 코드를 `R_test_cpu.R`로 저장하여 user home directory에 둡니다.

**install.packages()**에서
- **force = FALSE** 옵션은 이미 설치된 패키지를 또 설치하는 것을 막아줍니다[^fn4].
- **INSTALL_opts = c('--no-lock')** 옵션은 설치가 이전에 강제 중단 되어서 directory에 락이 걸렸을 때 락을 무시하고 설치하게 합니다[^fn5].

```R
install.packages("xgboost", force = FALSE, INSTALL_opts = c('--no-lock'))
install.packages("caret", force = FALSE, INSTALL_opts = c('--no-lock'))

library(xgboost)
library(caret)

boston = MASS::Boston
str(boston)

set.seed(12)
indexes = createDataPartition(boston$medv, p = .85, list = F)
train = boston[indexes, ]
test = boston[-indexes, ]

train_x = data.matrix(train[, -13])
train_y = train[,13]

test_x = data.matrix(test[, -13])
test_y = test[, 13]

xgb_train = xgb.DMatrix(data = train_x, label = train_y)
xgb_test = xgb.DMatrix(data = test_x, label = test_y)

xgbc = xgboost(data = xgb_train, max.depth = 2, nrounds = 50)
print(xgbc)

pred_y = predict(xgbc, xgb_test)

mse = mean((test_y - pred_y)^2)
mae = caret::MAE(test_y, pred_y)
rmse = caret::RMSE(test_y, pred_y)

cat("MSE: ", mse, "MAE: ", mae, " RMSE: ", rmse)

x = 1:length(test_y)
jpeg(file="R_plot.jpeg")
plot(x, test_y, col = "red", type = "l")
lines(x, pred_y, col = "blue", type = "l")
legend(x = 1, y = 38,  legend = c("original test_y", "predicted test_y"), 
       col = c("red", "blue"), box.lty = 1, cex = 0.8, lty = c(1, 1))
dev.off()
```
[^fn6]

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
작성한 코드를 `cpu-compute` node에서 실행하는 Slurm batch script를 작성합니다. 클러스터 소개 페이지의 [slurm job configurator](https://hpc.stat.yonsei.ac.kr/tools/job-configurator.html)를 사용하면 script를 쉽게 작성할 수 있습니다.
 

![slurm_config](/img/R_slurm_config.png)
- `Python`과 달리 conda environment를 사용하지 않으므로, conda activate에 체크합니다.
- 빈칸들을 채웁니다.
- Script란에 **Rscript xxx.R**라고 작성합니다. 이는 home directory에 있는 **xxx.R** 파일을 `R`로 실행하라는 의미입니다.
- **Print & Copy** 버튼을 누르면 내용이 클립보드에 복사됩니다. 

`R_test_cpu.job`이라는 이름으로 클러스터의 user home directory에 저장합니다.

Configurator로 생성한 Slurm batch script의 내용은 아래와 같습니다.
```
#!/bin/bash 
#
#SBATCH --job-name=R_test_cpu
#SBATCH --partition=all
#SBATCH --account=mjm
#SBATCH --mem=16gb
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=00:50:00
#SBATCH --output=/mnt/nas/users/mjm/R_test_cpu.log
#SBATCH --error=/mnt/nas/users/mjm/R_test_cpu.err
#SBATCH --nodelist=cpu-compute

Rscript R_test_cpu.R
```
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
**sbatch** 커맨드를 통해 job을 제출합니다. 할당되는 job 번호는 나중에 job 정보를 확인하거나 job을 취소할 때 이용되므로 기록해 놓아야 합니다.

**squeue**나 **smap -i**로 작업 현황을 확인하고, **cat xxx.log**이나 **tail -f xxx.err**으로 콘솔 출력이나 error를 확인합니다.

```bash
sbatch R_test_cpu.job
smap -i 1 # 작업 현황을 1초마다 갱신하여 보여줍니다. ctrl+c로 escape 할 수 있습니다.
cat R_test_cpu.log
tail -f R_test_cpu.err
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
[^fn1]: https://docs.conda.io/projects/conda/en/latest/release-notes.html
[^fn2]: https://github.com/conda/conda/issues/9399
[^fn3]: https://jstar0525.tistory.com/14
[^fn4]: https://stat.ethz.ch/pipermail/r-help/2010-May/239492.html
[^fn5]: https://github.com/lumiamitie/TIL/blob/master/rstudy/package_lock.md
[^fn6]: https://www.datatechnotes.com/2020/08/regression-example-with-xgboost-in-r.html