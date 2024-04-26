---
layout: post
title: 우분투에서 docker 컨테이터와 방화벽
tags: [ufw, docker]
use_math: false
---

우분투에서는 ufw로 방화벽을 컨트롤 한다. 

우분투 서버에서 ufw로 방화벽을 설정하고 특정 포트를 allow하면 그것만 외부에서 접속이 되고 나머지 포트는 안돼야 하는데 방화벽이 무색하게 모든 포트에서 접속이 가능한 경우가 있다. docker 컨테이터를 띄우면 ufw가 전혀 반영되지 않은 채 모든 포트가 열린다. 이에 대한 해결 책을 찾았기에 여기에 기록한다

(참조 : https://d-life93.tistory.com/431)



#### 1. docker 컨테이터에는ufw가 적용되지 않는 문제

* docker 는 기본적으로 ufw가 아닌 iptables의 적용을 받는다.
* ufw를 적용하기 위해서는 iptables 의 적용을 disable시키면 된다.  그 방법은 docker daemon 파일을 만드는 것이다.

```
sudo vi /etc/docker/daemon.json
```

```
{
    "iptables" : false
}
```

이와 같이 파일을 작성, 저장하고 docker 서비스를 재시작한다.

```
sudo systemctl start docker
```



#### 2. docker 컨테이너 간의 통신 문제

이렇게 하면 문제가 있다. <u>docker 컨테이너 간에 통신이 되지 않는다.</u>  ufw는 routed를 허용하지 않도록 기본 설정이 되어 있기 때문이다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FTCWgh%2Fbtrv4Ky5pjd%2F9wK5DTfmI7NqDjhUNmQJk0%2Fimg.png)

ufw의 routed를 아래와 같이 허용하면 된다.

```
sudo ufw default allow routed
sudo ufw reload
```

