---
layout: post
title: Dify와 gitlab연동(2)
tags: [dify, gitlab, codereview]
use_math: false
---

## Dify로 gitlab 개발환경 MR발생시 GPT를 통한 자동 리뷰 시스템 구축(2)



이제 “**반복**” 노드를 보도록 하겠습니다. 

![image-20250222132924303](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222132924303.png)



반복 노드는 입력 변수는 Array로 받으며 이 Array의 길이 만큼 반복을 합니다. 반복 노드 내에서 Array내 item들을 가지고 처리를 할 수 있습니다.

반복 노드 내에서 처음 처리하는 하위 노드는 ‘코드’인 “**ADD DIFFS**” 입니다. 파이썬 코드를 이용했습니다.

![image-20250222133517679](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222133517679.png)



입력값은 Array내 Item 입니다. MR URL을 받아서 바뀐 부분들인 `diffs` 를 구하기 위한 URL을 만듭니다. 코드는 다음과 같습니다.

```python

def main(arg1: str) -> dict:

    mrIID = arg1.split("/")[-1]

    apiUrl = "https://gitlab.example.com/api/v4/projects/5935/merge_requests/" + mrIID +"/diffs"

    return {
        "result": apiUrl
    }
```



다음 노드는 ‘HTTP요청’ 인 “**GET DIFF**” 입니다. 앞 노드에서 받은 URL을 gitlab API에 전달하고 Response를 받습니다. 

![image-20250222142621379](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222142621379.png)

gitlab API사용을 위한 인증을 위해 헤더에 Private-Token섹션을 만들고 여기에 “**시작**” 에서 받고 있는 PrivateToken 값을 할당합니다.

Response의 데이터의 구조는 다음과 같습니다(출력변수 참조)

* body
* status_code
* headers
* files

이 데이터들은 이제 다음 노드인 **LLM**으로 넘어갑니다



![image-20250222144814687](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222144814687.png)

**LLM** 노드에서는 앞 노드에서 받은 데이터를 근거로 분석이 수행됩니다. 여기에서는 아직은 무료로 사용할 수 있는 Gemini를 사용했습니다. LLM모델의 설정은 우측 상단의 사용자 이름을 클릭해 설정 → 모델제공자 에서 등록을 할 수 있습니다. 

컨텍스트는 앞 노드에서 body 데이터를 할당합니다. 그리고 프롬프트는 다음과 같이 설정했습니다. 

```
Prompt:
You are an AI designed to automate code reviews for a team of six junior developers. Below is the Diff information from a GitLab Merge Request.
Focus on the modified code and provide feedback specifically on these changes. The feedback should include:
﻿﻿﻿An explanation of the functional changes in the modified code.
﻿﻿﻿Advice on code style and best practices relevant to the changes.
﻿﻿﻿Suggestions for improving the performance and efficiency of the modified code.
﻿﻿﻿Explanations of related concepts or technologies in a way that junior developers can understand.
﻿﻿﻿Recommendations for additional learning materials or resources.
﻿﻿﻿Identification of any issues or mistakes in the modified code, with suggestions on how to fix them. Provide example code if necessary.
When writing the feedback, be kind and clear. The goal is to help junior developers grow. Please provide the review in Korean.
Diff Information:

/컨텍스트

```
마지막 `/컨텍스트` 부분은 직접 `/`를 타이핑해 입력합니다.


이 노드의 출력은 text로 기본 설정됩니다. 



다음 노드는 ‘코드’ 노드인 “**MAKENOTESURL**” 입니다. 여기에서는 앞에서의 리뷰 결과와 diff를 위해 사용된 URL을 입력 변수로 받고 URL만 코멘트를 위한 URL로 변경하여 다음 노드로 전달합니다. 

![image-20250222150924008](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222150924008.png)

여기에서의 코드는 다음과 같습니다.

```python

def replace_last_segment(url: str, old_segment: str, new_segment: str) -> str:
    if url.endswith(old_segment):
        return url[: -len(old_segment)] + new_segment
    return url

def main(url: str, text: str) -> dict:
    # URL중 뒤의 /diffs를 삭제하고 /notes를 붙인다
    updated_url = replace_last_segment(url, "/diffs", "/notes")
    
    return {
        "result": updated_url,
        "comments" : text,
    }
```



이제 ‘반복’ 의 마지막 노드인 ‘HTTP요청’ 노드인 “**WRITE NOTES**” 입니다. 여기에서는 MR에 코멘트를 남기는 URL을 받아 gitlab API에 전달을 하는 역할을 합니다.

![image-20250222152811598](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222152811598.png)

gitlab API 인증을 위해 헤더에 Private-Token을 설정했고 코멘트의 내용은 본문의 body에 넣었습니다. 출력으로는 앞의 **GET DIFFS** 노드와 동일하게 body, status_code, header, files 입니다.



마지막 ‘**끝**’ 노드는 앞에서의 status_code 를 Array로 받고 이를 출력하는 것으로 끝을 냅니다.

![image-20250222153852052](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250222153852052.png)



이제 다음 글에서 이 결과를 가지고 gitlab의 webhook와 연결을 해 보도록 하겠습니다.



