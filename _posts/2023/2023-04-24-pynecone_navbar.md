---
layout: post
title: pynecone에서 navbar 샘플 코드 작성
tags: [pynecone, python]
---

pynecone의 기능을 테스트 하기 위해 간단한 회사 홈페이지 하나를 카피해 보고 있는데, 난관에 봉착한 일이 발생했고, 이를 해결하여 그 방법을 기록으로 남긴다. 

아래 페이지에서 네비게이션 바의 항목을 클릭하면 하위 섹션의 페이지가 해당 내용으로 바뀌게 하려고 한다. 이를 pynecone에서 하는 방법은... 없었다. 

![](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2023/2023-04-24-1.png)

`pc.tab`이라는 컴포넌트가 있기는 한데 위와같은 사용자화 내비게이션 바에서 적용하기에는 불가능했다. 그래서 찾아낸 게 `pc.cond`를 좀 과다하게 조합하여 사용하는 방법이었다. 

관련 내용은 깃헙에 [소스코드](https://github.com/cheuora/pyne)를 올렸으니 pyne.py의 코드를 참고하기 바란다. 

