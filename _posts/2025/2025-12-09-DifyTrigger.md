---
layout: post
title: Dify에서의 새로운 기능 - 트리거
tags: [dify, RAG, AIAgent]
use_math: false
---



Dify의 버전이 1.10.x로 넘어가면서 새로운 기능이 나왔습니다. 바로 트리거(Trigger)기능입니다. 

포스팅한 [예전글](https://cheuora.github.io/2025/02/22/DifyGitlab1.html)을 보면 gitlab에서 웹 훅을 쓰려고 별도의 flask 앱을 만들었었습니다. 어렵지는 않지만 서버앱을 별도로 띄워야 하는 불편함이 있었는데 이 기능이 나왔다고 해서 바로 ‘gitlab MR발생시 코드 리뷰 앱’에 적용해 보았습니다. 

버전 1.10.x부터 신규 워크 플로우를 생성하면 다음과 같은 선택 옵션이 뜹니다. 

![image-20251209203558154](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251209203558154.png)

새롭게 추가된 것이 트리거 입니다. 트리거를 클릭하면 다음과 같이 하위 옵션이 뜹니다.

![image-20251209203651097](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251209203651097.png)



트리거는 세가지 종류가 있습니다. 특정 앱에서 어떤 액션이 행해지면 동작하는 트리거(위에서 추천 부분)가 있고 설정 시간마다 자동으로 돌아가는 트리거(일정 트리거), 그리고 웹훅을 통해 실행되는 웹훅 트리거가 그것입니다. 여기서는 gitlab의 웹훅 트리거를 쓸 것이기 때문에 웹훅 트리거를 선택합니다.

선택을 하면 웹훅 트리거의 이름으로 시작 노드가 생성됩니다. 

![image-20251209204731815](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251209204731815.png)

웹훅 URL은 임의로 자동생성됩니다. 수정할 수는 없습니다. 접속할 서비스에서 POST로 넘길 것이므로 POST로 선택합니다. URL중에 ‘localhost’ 부분은 Dify를 설치한 서버의 도메인 호스트로 변경하여 gitlab 웹훅에 입력하셔야 합니다. 

![image-20251209210512034](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251209210512034.png)

<div style='text-align:center'>(gitlab의 webhook설정화면 - localhost부분은 DIfy가 설치된 호스트 도메인입니다)</div>

이전하고 달라진 부분은 custom webhook templete의 설정입니다. 여기는 Request body부분입니다. 이전에는 flask의 파이썬 코드에서 설정했던 부분을 여기에서 설정했습니다. master 브랜치의 MR중 opened만 고르겠다는 설정입니다.



![image-20251209211433206](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251209211433206.png)



<div style='text-align:center'>(gitlab의 webhook설정화면)</div>

DIfy의 웹훅 트리거 노드에서는 다음과 같이 받습니다(Request Body Parameters).

![image-20251209212451973](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/assets/image-20251209212451973.png)

<div style='text-align:center'>(Dify의 웹훅 노드 설정)</div>



이제 다음 노드부터는 이전의 gitlab 워크플로우와 동일합니다. 다만 prvToken, siteurl을 받아오는 곳이 `X_Gitlab_Token` 및 `X_Gitlab_Instance` 로 바뀌었습니다. 

이후 설정 방법은 이전 포스트를 참고하시기 바랍니다.

[Dify와 gitlab연동(2)](https://cheuora.github.io/2025/02/22/DifyGitlab2.html)

