---
layout: post
title: ubuntu에서 ssh 사용자 추가 방법
tags: [ubuntu, user, ssh]
use_math: false
---

기록을 위해 남겨 본다..

1. 터미널로 EC2로그인
2. sudo adduser 사용자명 입력
3. passwd도 입력
    * 나머지 정보는 그냥 엔터로 넘어감
4. user 생성확인은 폴더 생성으로 확인
    * /home/사용자명 폴더 생성 확인
5. ssh접속에서 id/pw 접속을 허용하기 위한 설정 변경
    * sudo vi /etc/ssh/sshd_config 실행
    * PasswordAuthentication 을 찾아 옵션을 yes 로 설정
6. 변경 후 서버 재부팅
    * sudo shutdown -r now 실행


