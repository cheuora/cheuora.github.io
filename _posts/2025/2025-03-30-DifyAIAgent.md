---
layout: post
title: Dify에서 AI Agent(대리인) 추가
tags: [dify, AI Agent]
use_math: false
---

## Dify에서의 AI Agent (1.0.0부터 추가)

Dify 1.0.0 부터 플러그인이 확 바뀌었고 여러개가 추가되었다. 
그 중 AI Agent에 대해 정리를 해 볼까 합니다. 

### 1. AI Agent란


AI를 이용하여 스스로 어떤 도구가 필요한지 판단하여 작업을 진행하는 Agent를 의미합니다. 


![image-20250330225942916](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250330225942916.png)


### 2. Agent의 메리트

* 자율적 태스크 실행 
  * 정의된 전략을 기초로 모델, 툴을 동적으로 호출하고 유연하고 고도의 태스크를 자율적으로 실행할 수 있다. 
* 유연한 추론 전략
  * Dify의 Agent에서는 ‘Function Calling’ 과 ‘ReAct’ 라는 두 가지 추론 전략을 제공한다. 



#### 2.1 Function Calling



![image-20250330230111361](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250330230111361.png)


* 필요한 도구, 함수 등이 정의되면, 프롬프트를 분석하여 AI가 필요한 도구와 함수를 이용하는 것 

* 작업의 예시

  > “작년에 진행된 동계 올림픽은 어디에서 열렸어?”

  1. AI가 등록된 여러 도구들 중 검색도구가 필요하다고 판단
  2. 필요한 검색 툴을 호출한다 

* Function Calling 의 메리트

  * 미리 등록된 함수와 도구들이 있기 때문에 명확한 처리가 가능함. 복잡한 추론이 필요 없고 정해진 태스크를 정확히 실행한다.

#### 2.2 ReAct(Reason + Act)


![image-20250330230158874](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250330230158874.png)


* Reason(추론) : 현재의 상황과 목적을 추론
* Act(행동): 적절한 도구를 선택해 실행

* 작업의 예시

  > “내년 도쿄 올림픽의 일정을 알려줘”

  1. Reason : 최신 개최일시는 웹 서칭을 하는게 제일 좋다고 판단
  2. Act : 검색 도구로 “2025년 올림픽 개최일정” 을 검색
  3. Reason : 검색결과에 개최는 2025년 7월24일 부터라고 나온다. 이를 결과로 사용하자
  4. Act : 사용자에게 “2025년 7월24일 부터” 라고 전달한다 

* ReAct의 메리트

  * 유연한 대응력 : 상황에 맞게 적절한 도구를 선택하여 문제 해결 능력을 높임
  * 폭넓은 태스크 대응 : 정보검색, 데이터 조작, 의사결정, 외부API의 이용 등 모든 외부의존 태스크에 적용한다 



### 3. Dify 애플리케이션 제작

![image-20250330230416274](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250330230416274.png)


* AI Agent를 사용해 여행 플랜을 짜는 애플리케이션
* 예 : 6월10일부터 3박4일동안 강릉여행 계획을 50만원 예산으로 작성해 줘

#### 3.1 사전 준비

* Google Search Plugin 등록 : Dify Plugin설치-->마켓플레이스 --> Google Search를 설치한다.  API키는[SeriAPI](https://serpapi.com/users/sign_in) 에서 얻는다
* Wikipedia Plugin 등록 : DIfy Plugin 설치-->마켓플레이스 -->  Wikipedia Plugin을 설치한다
* Agent Plugin 등록 : DIfy Plugin 설치-->마켓플레이스 -->Dify Agent Stategies를 설치한다



다음은 이 애플리케이션의 제작 과정 입니다.



1. 시작 노드
   * place : 짧은 텍스트. 여행 목적지를 의미
   * budget : 짧은 텍스트. 예산을 의미
2. 대리인(Agent)
   * 에이전트 전략 : ‘Function Calling’ 과 ‘ReAct’가 있습니다. 여기서는 간단하기 때문에 ‘Function Calling’을 선택합니다.
   * Model : 사용할 LLM모델을 선택합니다. 여기서는 GPT-4o-mini를 선택했습니다. 
   * Tool List : 도구들의 선택지를 등록합니다. 여기에서는 Google Search와 WikiPedia 두 개를 골랐습니다. 대리인이 둘 중 알아서 골라 사용할 것입니다.
   * INSTRUCTION ; 이 대리인 기능의 개요를 적습니다. 
   * QUERY ; 도구에게 던질 질문입니다. 장소와 예산을 앞에서 받아 옵니다.
3. 끝 ; 출력은 앞 대리인 노드의 text로 선택합니다.



<iframe width="560" height="420" src="https://www.youtube.com/embed/G9nPl0p3UCU?si=9Q3HGOVPdgfrDwDJ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>



