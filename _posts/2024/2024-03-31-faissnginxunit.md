---
layout: post
title: nginx unit으로 faiss서버 만들기
tags: [nginx unit, faiss]
use_math: false
---

### faiss 수행 코드를 nginx unit + flask로 구현

(여기에 나온 코드의 자세한 사항은 이전 게시물 [faiss와 vectorDB의 비교](https://cheuora.github.io/2023/10/04/vectordb.html)를 참조하기 바란다)

faiss 를 수행시키려면 pytorch + faiss 를 설치해야 한다. 처음 환경인 ubuntu 20.04버전은 python3.8이 디폴트 버전이다. pytorch 공식 사이트를 가 보면 python3.8 이상 지원으로 써 있지만, 막상 python3.8에서 pytorch를 설치하면 오류가 뜬다(예전에는 안떴는데... 기술의 발전은 많은 기존 코드들을 죽여버린다). 

`ERROR: Package 'networkx' requires a different Python: 3.8.10 not in '>=3.9'`

networkx라는 것이 필요하며 이는 python3.9 이상 버전을 요구한다는 것이다. 어떻게 할 까 하다가 결국 대상 환경인 ubuntu의 버전을 업그레이드 하여 python 기본 버전을 올리기로 했다. nginx unit는 파이썬 기본 버전을 타기 때문이다. 

[구글링 결과(ubuntu os update)](https://www.google.com/search?q=ubuntu+os+update&oq=ubuntu+&gs_lcrp=EgZjaHJvbWUqDAgCECMYJxiABBiKBTIGCAAQRRg5MgwIARAjGCcYgAQYigUyDAgCECMYJxiABBiKBTIGCAMQRRg7MgoIBBAAGLEDGIAEMgoIBRAAGLEDGIAEMgYIBhBFGD0yBggHEEUYPNIBCDM2NDJqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8)

20.04 -> 22.04 LTS버전으로 업데이트를 하였고 python기본 버전이 3.10이 된다. 이제 설치 가능하다. 

venv가상환경에서 pytorch 설치, faiss설치 후 nginx unit를 설치해 준다.

<u>nginx unit설치 후 구동시 unitd프로세스를 root유저로 띄워야 한다. 나중에 transformer를 사용할 텐데 root유저로 구동하지 않으면 다음과 같은 오류가 난다. (참고로 이 상황은 chatGPT도 몰랐다. 방법은 내가 스스로 찾았다)</u>

```
There was a problem when trying to write in your cache folder (/root/.cache/huggingface/hub). You should set the environment variable TRANSFORMERS_CACHE to a writable directory.
```

unit을 root유저로 띄우는 방법은 다음과 같다

`sudo unitd --user root`



이제 유사어 검색을 위한 npy파일 생성 및 검색 코드를 만들어 보자

npy 생성 코드는 앞 게시물과 달라진 것은 없다. 

```
import pandas as pd
import numpy as np
from sentence_transformers import SentenceTransformer

df = pd.read_csv("./ChatbotData.csv") #from https://github.com/songys/Chatbot_data
sentences = df['Q'].to_list()
embedder = SentenceTransformer("all-MiniLM-L6-v2")

embeddings = embedder.encode(sentences)

np.save("./embeddings.npy", embeddings)
```

이제 flask용 RESTAPI 서버를 만든다.

```
import pandas as pd
import numpy as np
import faiss
import os
import json
from sentence_transformers import SentenceTransformer

from flask import Flask
from flask import request, Response
app = Flask(__name__)
INDEX_FILE = "/home/ubuntu/faiss_test/sts.index"

#변환기 불러오기
#embedder = SentenceTransformer("Huffon/sentence-klue-roberta-base")
embedder = SentenceTransformer("all-MiniLM-L6-v2")

#데이터 불러오기
df = pd.read_csv("/home/ubuntu/faiss_test/ChatbotData.csv")

# faiss 계산하기
if os.path.exists(INDEX_FILE):
    #index파일이 존재하면 이것만 읽어들인다.
else:
    #임베딩 벡터 불러오기
    embeddings = np.load("/home/ubuntu/faiss_test/embeddings.npy")

    index = faiss.IndexFlatL2(embeddings.shape[1]) # 초기화 : 벡터의 크기를 지정
    index.add(embeddings) # 임베딩을 추가
    faiss.write_index(index,INDEX_FILE)


top_k = 10

@app.route("/")
def hello_world():
    query = request.args.get("q")
    query_embedding = embedder.encode(query, normalize_embeddings=True, convert_to_tensor=True)
    distances, indices = index.search(np.expand_dims(query_embedding, axis=0),top_k)

    temp = df.iloc[indices[0]]
    data = temp[['Q','A','label']].head(10).to_json().encode().decode('unicode_escape')
    return Response(json.dumps(data, ensure_ascii=False), mimetype='application/json')
```



이제 이 애플리케이션 파일을 unit에 등록할 json을 만든다.

```
{
    "listeners": {
        "*:8080": {
            "pass": "applications/flask"
        }
    },

    "applications": {
        "flask": {
            "type": "python",
            "path": "/home/ubuntu/faiss_test/",
            "home": "/home/ubuntu/faiss_test/.venv/",
            "module": "wsgi",
            "callable": "app"
        }
    }
}
```

이제 unit 를 등록해주면 된다.

```
sudo curl -X PUT --data-binary @config.json --unix-socket /var/run/control.unit.sock http://localhost/config
```



```
root       12725       1  0 Mar29 ?        00:00:00 unit: main v1.32.1 [unitd --user root]
root       12728   12725  0 Mar29 ?        00:00:00 unit: controller
root       12729   12725  0 Mar29 ?        00:00:00 unit: router
root       12732   12725  0 Mar29 ?        00:00:00 unit: "flask" prototype
root       12733   12732  0 Mar29 ?        00:00:09 unit: "flask" application
ubuntu     17818   17680  0 15:18 pts/0    00:00:00 grep --color=auto unit
```



![image-20240331002344514](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2024/images/image-20240331002344514.png)

속도는 faiss유사 검색을 단독 데몬으로 돌리는 만큼 나오는 것 같았다. 웹 사이트에서 혹시 nginx unit에 대한 벤치마킹 자료가 있는지 찾아보았는데.. 있었다. 

[Nginx Unit discover and Benchmark](https://medium.com/@le_moment_it/nginx-unit-discover-and-benchmark-part-3-c38661194cd0)

이 중 python 에 대한 데이터는 다음과 같다 (Unicorn + Nginx vs nginx unit)

![img](https://miro.medium.com/v2/resize:fit:1000/1*6x-2wRi6w9mQXNMQJ_Nc6Q.png)

이 표를 보면 Client/Second 에서 접속자 수가 100Client/sec일 때 응답 시간이 4㎳ vs 1197㎳로 어마하게 차이가 났다. 

이정도면 nginx unit으로 애플리케이션을 올릴 값어치는 있지 않을까? 
