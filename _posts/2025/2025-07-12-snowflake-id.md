---
layout: post
title: snowflake id 생성법
tags: [snowflake, flake-idgen]
use_math: false
---



『가상면접 사례로 배우는 대규모 시스템 설계 기초』에서 보면 Snowflake 를 소개한 부분이 있습니다.(Snowflake 데이터베이스가 아니라 unique id generator입니다) 책에서는 id의 구조를 설명하는데 그쳤는데 실제 적용시에 어려움을 겪는 학생들이 많아 여기에 적용 코드를 정리해 보았습니다

snowflake id의 비트 구조는 다음과 같다(총 64비트)

| 비트 수 |         내용         |            비고             |
| :-----: | :------------------: | :-------------------------: |
|    1    |  부호 비트 (항상 0)  | 이 부분은 대부분 비워놓는다 |
|   41    | 타임스탬프 (ms 단위) |                             |
|   10    |     머신/노드 ID     |                             |
|   12    |     시퀀스 번호      |                             |



snowflake unique id 를 지원하는 것은 언어별로 다 라이브러리가 있습니다. 여기에서는 js에 있는 `flake-idgen`을 보도록 하겠습니다

[flake-idgen](https://www.npmjs.com/package/flake-idgen) 및 biguint-format 을 npm으로 설치합니다

```
npm install -save flake-idgen biguint-format
```



이제 코드를 만든다

```javascript
const FlakeId = require('flake-idgen');
const intformat = require('biguint-format');

const flake = new FlakeId({ epoch: 1300000000000, datacenter: 9, worker: 7 });

const id = flake.next();

console.info(id)
console.info(flake.id);  // Flake ID 출력
console.info(flake.datacenter)
console.info(flake.worker);  // 워커 ID 출력
console.info(intformat(id, 'dec'));  // 숫자 형식
console.info(intformat(id, 'hex', {prefix: '0x'}));  // 16진수 형식
const base64_id = id.toString('base64');
console.info(base64_id);  // 문자열 형식

const decodedBuffer = Buffer.from(base64_id, 'base64');
console.info(intformat(decodedBuffer, 'dec'));  // 디코딩된 버퍼 출력

```



이 코드의 실행 결과 입니다. 



```
➜  node flaketest.js 
<Buffer 1a 54 50 70 26 92 70 00>
295
9
7
1897229785643511808
0x1a54507026927000
GlRQcCaScAA=
1897229785643511808
➜  
```



생성된 unique id는 기본적으로 Buffer 타입입니다. 첫번째 출력이 그것입니다. 두 번째는 id인데 worker 와 datacenter 를 조합하여 자동으로 만들어 주는 부분입니다. 세번째와 네번째는 설정된 datacenter 및 worker의 값들 입니다. 

생성된 unique id는 Buffer타입이기 때문에 이를 바로 쓰기는 어렵습니다. 이를 사용하려면 다른 타입으로 바꾸어야 하는데 다섯번째는 decimal 로, 여섯번째는 hex로 변환한 것입니다. 일곱번째는 base64 인코딩입니다. 사실 앞의 방법보다 base64가 길이도 짧고 해서 이걸 쓰는게 제일 나은 것 같습니다. 마지막 출력은 이를 디코딩한 결과를 decimal로 복원한 것입니다. 



저번 학생들에게 snowflake id 를 설명하다가 어떤 학생이 javascript 에서 구현이 잘 안되었다고 한 적이 있습니다. 변수의 길이가 짤린다는 것이었습니다. javascript로 구현시 Buffer로의 구현을 몰랐기 때문에 그랬을 거라고 추측합니다. 혹시 나중에 snowflake 를 쓰려는 친구들이 있다면 참고 바랍니다. 



P.S: epoch 값은 시간의 start time 을 의미 합니다. 예전 unix에서 시스템은 항상 1970-01-01 을 기준으로 썼고 지금 이 값는 서비스마다 달리 가져가고 있습니다. (Twitter는 `1288834974657`(2006년), Discord는 `1420070400000`(2015년)입니다.)

