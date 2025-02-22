---
layout: post
title: AI가 개발자를 대체할까?
tags: [AI, Developer]
use_math: false
---

## Dify로 gitlab 개발환경 MR발생시 GPT를 통한 자동 리뷰 시스템 구축(1)

현업에선 팀 단위로 MR등 발생시 사람이 코드 리뷰를 합니다.  보통 경력자들이 모여서 하기 때문에 의미있는 리뷰 정보들이 충분히 나올 수 있습니다.

하지만 부트캠프 같은 교육기관에서는 모두 초심자인 교육생의 입장에서 팀 활동을 합니다. 서로 모여서 코드 리뷰를 한다 하더라도 의미있는 정보들이 교환될 가능성은 현업보다 더 낮습니다. 

의미있는 리뷰 정보를 전달하기 위해서는 외부의 리뷰가 필요했고 이를 GPT와 같은 생성형AI에 맡기는건 어떨까 생각했습니다. 요새같이 생성형 AI광풍이 부는 시대에 누구나 할 수 있는 생각일겁니다. 



제가 있는 부트캠프는 gitlab에서 코드를 관리합니다. 그래서 gitlab에 webhook를 설정하고 연결되는 서버 프로그램을 만들어 거기에서 생성형 AI로 정보를 넘기고 분석 결과를 받아오는 형태로 될 것입니다. 

먼저 Dify 의 Flow에서 이를 위해 구성하면 다음과 같습니다.

![image-20250222012208441](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222012208441.png)



이제 “**시작**” 단계 부터 보도록 하겠습니다. 

<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222012502210.png" alt="image-20250222012502210" style="zoom: 33%;" />

입력 데이터는 총 5개를 받습니다. 

* Private_token : gitlab 에서 발행한 Access Token값입니다. 
* prjid : MR등 이벤트가 발생하는 프로젝트의 id 입니다. 
* siteurl : gitlab의 server url 입니다. (예: https//exeample.gitlab.com/)
* branch: 정보를 얻어올 브랜치 입니다.(거의 master브랜치가 될 것입니다)
* state: 어떤 상태의 MR을 가져올 것인가에 대한 정보 입니다(opened vs closed)



이제 다음 단계로 **GetMRList** 노드를 보도록 하겠습니다.

<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222013541301.png" alt="image-20250222013541301" style="zoom:33%;" />

스크롤을 내리면 오류처리 부분이 나옵니다.

<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222013635257.png" alt="image-20250222013635257" style="zoom:33%;" />

GetMRList노드의 특성은 “코드” 입니다. DIfy는 JavaScript및 Python Script를 지원하는데 저는 Python을 사용했습니다. 

```python
import requests
import json


def main(prvtoken:str, prjid:str, siteurl:str, branch:str, state:str) -> dict:
	url = f"{siteurl}/api/v4/projects/{prjid}/merge_requests"
	headers = {"Private-Token": prvtoken}
	params = {
		"state": state,
		"target_branch": branch
	}
	response = requests.get(url, headers=headers, params=params)

	urlList = []

	if response.status_code == 200:
		mrs = response.json()

		# web_url만 추출
		urls = [mr["web_url"] for mr in mrs if "web_url" in mr]

		# 결과 출력
		for url in urls:
			urlList.append(url)
	else:
		print("Fail to call API")


	return {
		"results": urlList,
		"count" : len(urlList)
	}
```

입력 변수는 앞 **시작** 노드에서 받은 데이터들을 그대로 받아옵니다. 이를 gitlab API에 전달하는 URL 및 Param 형태로 가공한 뒤 request를 get으로 던지고 응답을 받은뒤에, 우리가 필요한 것은 여기에 해당하는 merge request 의 URL이므로 이 URL만 추출하여 결과를 딕셔너리로 돌려주는 구조입니다(항상 데이터를 딕셔너리로 돌려주어야 합니다). 노드의 입력 변수와 출력 변수는 이 코드의 입출력과 일치해야 합니다. 입력변수 5개와 출력 변수 필드 2개를 노드에 설정합니다.

오류가 발생할 때 오류 처리 방법도 노드에서 정할 수 있는데 여기에서는 분기를 만들어 성공 분기와 실패 분기를 나누었습니다. 실패시에는 “**끝**” 노드로 빠져 오류 코드 및 메시지를 출력하게 했으며 성공하면 다음 노드인 “**반복**”로 넘어갑니다. 아래 이미지는 오류처리를 위한 “**끝**”노드 입니다.

<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222123917100.png" alt="image-20250222123917100" style="zoom:33%;" />