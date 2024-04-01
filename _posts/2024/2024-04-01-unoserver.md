---
layout: post
title: Linux에서 excel to pdf변환2
tags: [libreoffice, pdf, unoserver]
use_math: false
---

### Ubuntu 서버에서 excel 파일을 pdf로 바꾸기 2

예전에 ubuntu에서 excel 파일을 pdf로 바꾸기 위해 adobe에서 제공하는 open api를 사용한다고 했다. 

[이전 게시물](https://cheuora.github.io/2022/10/27/excelToPDF.html)

그런데 다시 이를 적용해 보려 하는데 API키도 재발급 받아야 하고 지원 언어도 제약이 많아 조금 불편했다. 

그래서 찾아본 것이 ubuntu에 libreoffice 를 설치하는 것인데, 단지 pdf 변환만을 위해 이 전체를 설치하는 게 좀 불편했다. 

그런데, unoserver란 것이 있는데 libreoffice에서 RESTAPI기능만을 구현한 것이 있었다. 이를 도커 컨테이너화하여 도커 허브에 누군가가 올려 놓았네...

![image-20240402002626324](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2024/images/image-20240402002626324.png)



적당한 Alpine_version을 선택하여(나는 'edge'를 선택했다) docker pull을 받고 docker 컨테이너를 구동시킨다. 내부 포트는 2004번을 쓰기 때문에 내,외부 포트를 2004로 매핑해 주었다. 

```
docker pull libreofficedocker/libreoffice-unoserver:edge
```

```
docker run -d --name unoserver -p 2004:2004 libreofficedocker/libreoffice-unoserver:edge
```

이제 unoserver 를 내부적으로 띄웠다.

자.. 이제 테스트를 해 보자. 클라이언트는 curl를 사용했다. exceltest.xlsx를 file.pdf로 바꾸는 예제이다.

```
curl -s -v \
   --request POST \
   --url http://127.0.0.1:2004/request \
   --header 'Content-Type: multipart/form-data' \
   --form "file=@/home/ubuntu/exceltest.xlsx" \
   --form 'convert-to=pdf' \
   --output 'file.pdf'
```



수행 결과이다.

```
*   Trying 127.0.0.1:2004...
* Connected to 127.0.0.1 (127.0.0.1) port 2004 (#0)
> POST /request HTTP/1.1
> Host: 127.0.0.1:2004
> User-Agent: curl/7.81.0
> Accept: */*
> Content-Length: 39785
> Content-Type: multipart/form-data; boundary=------------------------996e3a5643b7c2b7
>
} [39785 bytes data]
* We are completely uploaded and fine
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Content-Length: 157671
< Content-Type: application/pdf
< Last-Modified: Mon, 01 Apr 2024 15:39:51 GMT
< Date: Mon, 01 Apr 2024 15:39:51 GMT
<
{ [3917 bytes data]
* Connection #0 to host 127.0.0.1 left intact
```

정상 수행되었고 file.pdf로 현재 디렉토리에 생성됨을 확인할 수 있다. 

![image-20240402004127602](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2024/images/image-20240402004127602.png)



