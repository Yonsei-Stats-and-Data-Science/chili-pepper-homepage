---
title: "[Tip] Filezilla로 파일 전송하기"
author: "Jongmin Mun"
date: 2022-03-15T14:54:35+09:00
draft: false
---
# Filezilla로 파일 전송하기

[Filezilla](https://filezilla-project.org)는 MacOS, Windows, Linux에서 사용할 수 있는 FTP 클라이언트입니다. Filezilla로 `scp`도 이용할 수 있기 때문에, drag & drop으로 편리하게 파일을 전송할 수 있습니다.

Filezilla를 실행한 뒤 파일->사이트 관리자를 누릅니다.
![filezilla_click](/assets/filezilla_sitemanager_click.png)

좌측 하단의 'New site'를 클릭하고, 오른쪽에
- 프로토콜: SFTP - SSH File Transfer Protocol
- 호스트: hpc.stat.yonsei.ac.kr
- 포트: 공지된 SSH port
- 로그온 유형: 비밀번호 묻기
- 사용자: 본인의 user name
을 입력하고 연결을 클릭합니다.

![filezilla_connect](/assets/filezilla_connect.png)

Password를 묻는 창이 나오면 리눅스 user password를 입력합니다. '알 수 없는 호스트키' 창이 나오면 확인을 클릭합니다.

아래와 같은 창이 나오면 drag & drop으로 로컬과 서버 양방향으로 파일을 전송할 수 있습니다.

![filezilla_connect](/assets/filezilla_explorer.png)

## Reference
[fn^1] https://www.bettertechtips.com/how-to/use-scp-filezilla/
