---
layout: post
title: 산타파이브 프로젝트
---



요새 SNS에 [산타파이브](https://colormytree.me) 라는 사이드 프로젝트가 화제다.

주니어 개발자 5명이 사이드 프로젝트로 시작한 프로젝트처럼 보이는데 크리스마스를 목전에 두고 인기를 얻은 모양이다. 그러다 보니 여러 파리들이 꼬이기 마련. 누군가가 API의 헛점을 발견했는데 이를 가지고 엄청난 공격을 하고, 이 공격 방법을 또 깃헙 및 디스코드에 전파를 한 모양이다. 관련 사과문 및 디스코드 로그는 [여기](https://despenser08.notion.site/despenser08/colormytree-me-API-0bcb19c3c1524dd5931e149471a9d4cc)를 참조하기 바란다.

처음 트위터에 윤리 문제로 산타파이브를 언급했을 때 나는 그냥 그런게 있나보다 했다. 그런데 어제 놀라운 소식을 들었다.

> 산타파이브 개발자 중 한명이 SSAFY졸업생 xxx라고 합니다. 지금 이 서비스 엄청 인기래요!



정말인가? (그런데 내가 만난적이 없는 교육생이었다) 그러고보니 산타파이브 서비스의 규모가 SSAFY에서 만든 프로젝트 정도로 느껴진게 우연은 아니었다. 위 얘기를 듣고 위의 디스코드 로그 사이트를 들어가 보았다. 

디스코드의 로그를 보니 어떤 취약점을 발견했고 그 결과는 저렇고 등등의 내용이 넘쳐났다. 중간에 일베스러운 글들도 상당히 많았다. 

주니어 개발자들이 사이드 프로젝트로 모여 개발을 하였으니 API의 설계가 사용자들의 사용 케이스(테스트 케이스)에 대해 전혀 고려를 하지 않았으리라 생각된다. 사실 사이드 프로젝트에서 이런건 어렵다. 공격자가 악의를 가지고 10만 캐릭터를 날릴 경우를 누가 감히 상정하겠는가? 

나를 비롯한 SSAFY 2학기 담당자들은 하나같이 칭친을 했다. 언제 이런 어마무시한(게임서버도 이정도는 아닐 것이다)부하를 자체적으로 경험하겠으며, 이렇게 세간에 주목을 받을 일이 또 있겠는가? 이런 경험을 스스로 만들어 했다는 게 기특할 따름이다.

이야기를 들어보니 위 졸업생은 D사의 인턴 기간이 끝나고 다시 취준생으로 돌아간 모양이다.  뭐 다시 D사에서 불러줄리는 없겠지만, 좋은 경험을 했으니 더 좋은 기회가 가지 않을까 생각된다.



