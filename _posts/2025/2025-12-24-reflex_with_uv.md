---
layout: post
title: uv로 reflex 개발하기 
tags: [python, uv, reflex]
use_math: false
---



파이썬으로 프론트엔드를 만들 수 있게 도와주는 프레임워크가 있으니 바로 [reflex](reflex.dev)입니다.
이를 우분투 환경 + python3.10 환경에서 쓰려고 하니 최소 3.11이상 되어야 한다고 해서 이를 uv를 써서 가상환경에서 3.13버전(3.14버전은 경고가 뜨고 3.15버전은 뭔가 오류가 발생한다)을 활용한 기록을 여기에 남기려 합니다. 

uv의 설치는 되어 있다고 가정하고 python은 3.13버전으로 로컬에 설치하도록 하겠습니다.

```
> uv python install 3.13
> cd [프로젝트폴더]
```



가상환경을 설정하고 reflex를 인스톨 합니다.

```
프로젝트폴더> uv venv
```

```
프로젝트폴더> uv pip install reflex
```



reflex공홈에서는 설치후 `reflex init` 명령을 실행하라 하는데 uv에서는 다음과 같이 실행시킵니다.

```
프로젝트폴더> uv run -m reflex init
```

그럼 3가지 옵션이 뜨는데 저는 1) 기본 옵션으로 선택하여 진행했습니다.

공홈에서는 이 앱을 테스트용으로 띄우려면 `reflex run` 을 실행하라 했습니다만 우리는 uv환경이기 때문에 다음과 같이 실행합니다.

```
프로젝트폴더> uv run -m reflex run
```

이를 실행하고 나서 웹 페이지(http://[여러분의호스트]:3000)을 접속하면 웹 브라우저에 다음과 같은 문장이 적힌 페이지만 뜹니다.

```
Blocked request. This host ("여러분의호스트") is not allowed.
To allow this host, add "여러분의호스트" to `server.allowedHosts` in vite.config.js.
```

멘붕이 왔습니다. reflex 에서 vite를 사용한다는 것은 알겠는데 그럼 `vite.config.js` 파일을 찾아 수정을 해야 하는 걸까요? 

다행히 방법은 있었습니다. vite는 환경변수를 통해 설정을 읽을 수 있었습니다. 

```
export __VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS="your.host.name"
```

당연하겠지만 `your.host.name`  부분은 여러분의 서버 호스트 명이 들어갑니다. 이는 터미널 로그아웃을 하면 날라가는 값이기 때문에 처음부터 `.bashrc` 파일에 위 내용을 추가시키는게 더 편안할 겁니다.

환경변수 적용후 정상적으로 페이지가 보이는 것을 확인할 수 있네요

![image-20251224172740961](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251224172740961.png)

reflex 공식 홈페이지에서는 uv를 통해 쓰는 방법이 없어 한번 정리를 해 보았습니다. 이상으로 마치도록 하겠습니다. 



