---
title: "5. iOS에서 클러스터 사용하기"
author: "Jongmin Mun"
date: 2022-03-19T14:15:35+09:00
draft: false
---

# 5. iOS에서 클러스터 사용하기
[2번 문서](https://hpc.stat.yonsei.ac.kr/docs/02_how-to-use-cpu-node_python/)에서 설명했듯이, 어느 환경에서든 `SSH` 접속만 가능하다면 클러스터 이용이 가능합니다. 이 문서는 iPad에서 [koder](https://apps.apple.com/kr/app/koder-code-editor/id1447489375)라는 앱을 이용해 클러스터를 이용하는 예제를 다룹니다.

## 1. 파일 시스템 접속
클러스터의 파일 시스템에 접속하여 파일을 열람하고 수정할 수 있습니다.
![koder_1](/img/koder_1.jpg)
1. 좌측 상단의 2번째 아이콘을 클릭하여 **FTP/SFTP** 설정으로 들어갑니다.
2. 좌측 하단의 + 버튼을 누르면 **New FTP Connection** 설정 창이 나옵니다.

![koder_2](/img/koder_2.jpg)
3. 아래 내용을 입력한 다음 **Create**를 누릅니다.
   1. 먼저, **Connection**을 **SFTP**로 바꿉니다.
   2. **Name**은 이 connection 설정의 이름을 의미합니다. 원하는 이름을 입력합니다.
   3. **Host Name**에 **hpc.stat.yonsei.ac.kr**을 입력합니다.
   4. **Username, Password**에 linux username, password를 입력합니다.
   5. **Port**에 Slack에서 안내한 `SSH` 포트번호를 입력합니다.

4. 화면 좌측 탭에 connection이 생성됩니다.
![koder_3](/img/koder_3.jpg)

5. 화면 좌측 탭에 connection이 생성됩니다. 이를 클릭하여 접속합니다.

6. User home directory의 파일들이 나옵니다. 파일을 클릭하면 오른쪽의 텍스트 에디터 창에 내용이 표시됩니다. 파일 내용을 수정한 뒤 저장하려면 우측 탭 상단에 있는 업로드 모양 아이콘을 누릅니다.
![koder_4](/img/koder_4.jpg)


## 2. SSH 접속
커맨드를 입력하고 job을 제출하려면 `SSH`로 접속해야 합니다. 

1. 좌측 탭 하단의 콘솔 모양 아이콘을 클릭하여 `SSH` 접속 절정 창을 엽니다.
![koder_ssh_1](/img/koder_ssh_1.jpg)

2. 아래 사진을 참고하여 정보를 입력하고 **Connect*를 누릅니다.
   1. 맨 위에는 **hpc.stat.yonsei.ac.kr**을 입력합니다.
   2. 그 다음 칸에는 linux username을 입력합니다.
   3. 그 다음 칸에는 linux user password를 입력합니다.
   4. 그 다음 칸에는 Slack에서 공지한 `SSH` 포트번호를 입력합니다.

3. SSH로 proxy node에 접속되고 터미널 환경이 나타납니다. 이제 Visual Studio Code에서 했던 것처럼 클러스터를 사용합니다.
![koder_ssh_2](/img/koder_ssh_2.jpg)



