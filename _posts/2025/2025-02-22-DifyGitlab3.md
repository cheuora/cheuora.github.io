---
layout: post
title: Dify와 gitlab연동(3)
tags: [dify, gitlab, codereview]
use_math: false
---

## Dify로 gitlab 개발환경 MR발생시 GPT를 통한 자동 리뷰 시스템 구축(3)



이제 DIfy에서의 플로우 셋팅은 완료되었습니다.

이 Flow는 Dify에서 제공하는 UI를 쓸 수도 있지만 API서버 형태로도 활용을 할 수 있습니다. 아래는 API엑세스를 선택하면 보여주는 안내 페이지 입니다.

![image-20250222163013401](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222163013401.png)



이 API를 사용하려면 API키가 필요한데 이 키는 우측 상단의 “**API키**” 를 클릭하여 얻을 수 있습니다. 



이제 이 Dify API와 gitlab 의 Webhook을 연결할 간단한 서버 프로그램을 만들어 보겠습니다. gitlab의 Webhook에서 바로 Dify API를 바라보게 할 수도 있지만, 변수에 제약이 있고 Dify API가 여러명이 동시에 접속할 경우에 원활한 접속 보장이 없기 때문에 중간에 nginx unit을 사용해 별도의 서버를 두겠습니다.

DIfy API 에 접속하는 코드는 다음과 같습니다(파일명: difyserver.py)

```python
import requests
import json

API_KEY = "****-&&&&&&&**********" # Dify API
BASE_URL = "http://gitlab.example.com/v1"


# Run workflow
def run_workflow(inputs, response_mode, user):
    url = f"{BASE_URL}/workflows/run"
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }
    data = {
        "inputs": inputs,
        "response_mode": response_mode,
        "user": user
    }
    response = requests.post(url, headers=headers, data=json.dumps(data))

    if response.status_code == 200:
        if response_mode == "blocking":
            result = response.json()
            if "data" in result and "outputs" in result["data"] and "error_message" in result["data"]["outputs"]:
                return result["data"]["outputs"]["error_message"]
            else:
                print("Error: 'error_message' not found in the API response.")
        elif response_mode == "streaming":
            for chunk in response.iter_content(chunk_size=None):
                if chunk:
                    print(chunk.decode())
    else:
        print(f"Request failed with status code {response.status_code}")
```



inputs에 앞에서 정의한 Dify flow의 ‘**시작**’ 노드에 들어갈 변수들을 딕셔너리로 받아 flow를 동작시키고 결과를 받아오는 함수 입니다. 이제 이를 활용해 gitlab에서 Webhook를 받고 처리하는 코드를 만들어 보겠습니다.

```python
from flask import Flask, request, jsonify
import requests
import threading
import difyserver

app = Flask(__name__)

# GitLab 설정
GITLAB_URL = "https://gitlaburl"

def process_webhook(data):
    response_mode = "blocking"
    user = "sungjoon kim"
    results = difyserver.run_workflow(data, response_mode, user)

@app.route("/gitlab-webhook", methods=["POST"])
def gitlab_webhook():
    data = request.json
    GITLAB_TOKEN = request.headers.get("X-Gitlab-Token")
    # MR 열림 이벤트 확인
    if data.get("object_kind") == "merge_request" and data.get("object_attributes").get("state") == "opened":
        try:
            project_id = data["project"]["id"]

            # run workflow with GITLAB_TOKEN, project_id, GITLAB_URL, "master", "opened"

            inputs = {"PrivateToken": GITLAB_TOKEN,
                      "prjid": str(project_id),
                      "siteurl": GITLAB_URL,
                      "branch": "master",
                      "state": "opened",}

            threading.Thread(target=process_webhook, args=(inputs,)).start()
            return jsonify({"status": "OK"}),200
        except:
            return jsonify({"status": "Error in Dify Server"}),500


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```



Flask를 써서 간단하게 만든 서버입니다. 눈여겨 볼 부분은 `process_webhook()` 함수를 쓰레드로 실행시킨 부분입니다. 이는 Dify 플로우에서 LLM의 답변을 기다리는 시간이 좀 길어서(길게는 10초 까지 걸렸습니다) 응답을 받는 gitlab Webhook에서 실행시마다 `Internal error occurred while delivering this webhook. Error: Net::ReadTimeout` 오류 로그를 쌓는 일이 발생했습니다. 이를 해결하기 위해서는 Webhook의 waiting 시간을 좀 길게(10초 이상) 설정할 수 있는데 이는 gitlab에서 시스템 어드민 권한이 있어야 가능한 일이어서 차선책으로 먼저 쓰레드로 Dify flow를 실행을 시작시킨뒤 바로 200응답을 주는 걸로 했습니다.



다음은 gitlab 프로젝트에서의 WebHook설정 입니다.

![image-20250222174956432](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222174956432.png)

URL부분에는 위 Flask를 구동시킬 URL을 입력하시고 Secret Token 부분에 gitlab프로젝트의 Access token(왼쪽 메뉴바에서 Access tokens에서 발행) 값을 넣고 Trigger 는 ‘Merge Request events’ 를 선택합니다. 



마지막으로 앞에서 작성한 Flask 코드를 구동시킵니다. 저는 외부에 있는 ubuntu서버를 활용하여 구동을 하였으며 nginx unit에 물려 서버를 띄웠습니다. nginx unit이 아닌 nginx로 하셔도 되고 다른 WSGI서버로 물려도 상관 없습니다. 이 부분까지 여기서 다루지는 않겠습니다. 



아래는 MR 신규 등록후 자동으로 관련 리뷰를 코멘트로 단 결과 화면 입니다.



![image-20250222231004383](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222231004383.png)



### 이 작업을 하고 느낀점

* Dify자체에 gitlab 플러그인이 있기는 하지만 여기에서 하려는 작업들을 하기에는 부족하여 코드 노드를 만들었습니다. 
* 이 리뷰 기능만 필요하다면 굳이 Dify flow를 만들지 않고 python 프로그래밍으로 해도 될 뻔한 작업이었던 것 같습니다.
* Flow를 백본으로 하여 이렇게 만들면 생각보다 여러 장점이 있는 것 같습니다.
  1. 간결한 코딩 : 많은 코드들이 분리가 되어 흐름에 배치되므로 변수 조정, 상속 등 고려해야 할 사항이 많이 줄었습니다. 기본 문법만으로도 가능합니다.
  2. 확장성 : 코드만으로 하는 것과는 달리 플랫폼이 주는 확장성이 있습니다. 예를 들어 지금은 LLM을 Gemini로 사용했지만, 쉽게 GPT나 다른 모델로 별도의 수정 없이 사용이 가능하며, 중간에 ‘지식’ 노드를 추가해 우리 조직만의 코딩 규칙을 적용하고 이를 기반으로 리뷰를 요청하는 것도 가능합니다.



