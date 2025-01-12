---
layout: post
title: 판매예측 시스템을 15분만에..?
tags: [LLM, dify, googleAppsScript]
use_math: false
---

## 판매 예측 시스템을 15분만에 만들어 보자..? 

이게 정말 가능한 일일까? 하지만 LLM과 Dify를 사용한다면 가능한 일일것 같다. 이를 같이 예제로 보도록 하겠다.

판매 예측을 하려면 당연하겠지만 데이터가 있어야 하는데 여기서는 구글 스프레드 시트에 있는 데이터를 이용하도록 하겠다. 

샘플 데이터는 다음과 같다. 

![image-20250111224206397](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111224206397.png)

이 스프레드 시트의 공유 설정은 다음과 같이 한다.

<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111224505945.png" alt="image-20250111224505945" style="zoom:67%;" />



### 전체 데이터 흐름

전체 데이터 흐름은 다음과 같다.

![image-20250111230646675](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111230646675.png)



사용자가 작성한 스프레드시트를 GAC(Google Apps Script)를 사용해 JSON으로 바꿔 DIFY의 LLM이 사용하기 쉽게 변환한다. 이를 GPT4같은 LLM에 분석작업에 활용하도록 하여 분석을 시킨 후 그 결과를 사용자에게 돌려 주는 것이다. 



### Google Apps Script 의 작성

GPT4 같은 LLM에 직접 구글 스프레드 시트 URL을 전달하는 것도 가능하다 테스트를 해보니 정확한 데이터 처리를 하지 못하는 것 같았다. 가급적 JSON형태의 텍스트 파일로 전달하는 게 더나을 것 같다. 이것이 GAS를 쓰는 목적이다. 

GAS를 새로 만들려면 신규에서 더보기에서 Google Apps Script를 선택한다

<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111230433320.png" alt="image-20250111230433320" style="zoom:67%;" />

선택하면 신규 스크립트 입력 페이지가 나오는데 여기에 코드를 입력해야 한다

![image-20250111231140819](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111231140819.png)

스크립트 명은 일단 ‘예측데이터스크립트’ 라고 했고 코드는 아래 코드를 참조하기 바란다.

참고로 spreadsheetId값은 위 스프레드 시트의 Id인데 스프레드시트 URL에서 따 올수 있다. (https://docs.google.com/spreadsheets/d/1gqCJJggpZAQxMfmX7ePZ9mPR9-zW8Tp26NFDRmxUWTU/edit?gid=0#gid=0 에서 1gqCJJggpZAQxMfmX7ePZ9mPR9-zW8Tp26NFDRmxUWTU 부분)

```
function doGet(e) {
  try {
    // 스프레드 시트의 ID 
    const spreadsheetId = "1gqCJJggpZAQxMfmX7ePZ9mPR9-zW8Tp26NFDRmxUWTU"; 
    const sheetName = "녹차류"; // 탭 명

    // 시트 객체 생성
    const spreadsheet = SpreadsheetApp.openById(spreadsheetId);
    const sheet = spreadsheet.getSheetByName(sheetName);

    // 시트가 없는 경우 오류 처리
    if (!sheet) {
      throw new Error(`Sheet "${sheetName}" not found.`);
    }

    // 시트 데이터 get()
    const data = sheet.getDataRange().getValues();

    // 헤더와 데이터의 분리
    const headers = data[0]; 
    const rows = data.slice(1);

    // JSON형식으로 데이터 가공
    const jsonData = rows
      .filter(row => row.some(cell => cell !== "")) // 공백 제거
      .map(row => {
        return headers.reduce((obj, header, index) => {
          obj[header.trim()] = row[index] !== undefined ? row[index] : null;
          return obj;
        }, {});
      });

    // 성공시 
    return ContentService.createTextOutput(
      JSON.stringify(jsonData, null, 2)
    ).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    // 에러시
    return ContentService.createTextOutput(
      JSON.stringify({ error: error.message }, null, 2)
    ).setMimeType(ContentService.MimeType.JSON);
  }
}

```

 입력을 완료했으면 배포를 해야 한다. 우측 상단의 ‘배포’ 버튼을 눌러 ‘새배포를’ 선택한다. 



![image-20250111231608211](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111231608211.png)

웹 앱에서 인증 정보 및 엑세스 권한은 위와 같이 설정한다. 그러면 웹 앱 배포 URL이 나타나는데 이를 Copy 해 둔다.

![image-20250111231720715](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111231720715.png)

### Dify 에서 워크플로우 설정

Dify에서 이제 워크 플로우를 작성한다.

#### 신규 --> HTTP 요청

‘시작’ 다음에 HTTP요청을 설정한다.

HTTP요청 설정은 다음과 같다.

![image-20250111232644317](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111232644317.png)

(1): API에는 GET으로 설정하고 앞에서 GAS에서 Copy한 웹 앱 URL를 붙여 넣는다. 

(2) , (3): 이 부분은 기본 설정으로 그대로 둔다. 

#### LLM설정

HTTPS 다음에 LLM이 오도록 한다.

![image-20250111233018314](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111233018314.png)

모델은 적절한 것을 선택하도록 하고(여기에선 gpt-4-mini를 쓰겠다) 컨텍스트는 앞 노드에서 body데이터를 사용한다. body가 json데이터로 들어올 것이며 json이 안들어온다면 script나 또는 script의 권한 부분을 잘 확인하기 바란다. 그리고 프롬프트는 다음과 같이 적었다.

```
당신은 이제 데이터 분석과 예측을 하는 AI애널리스트 입니다.

이하의 데이터를 기반으로 다음달의 매출합계를 예상해 보세요. 

과거의 판매 데이터, 계절, 날씨, 경제지표 등의 요인을 고려해 예측을 해 주세요. 또 예측의 근거를 간결하게 설명해 주세요

#### 입력 데이터
{context}

#### 예측 목표

1. 다음달 매출합계액(상품별, 그리고 전체)
2. 각 상품의 판매수량예측
3. 날씨와 경제지표의 영향을 고려한 예측


#### 출력 포멧

1. 총 매출합계예측:
  - 녹차 : {판매액}원
  - 말차라테 : {판매액}원
  - 합계 : {판매총액}원
  
  2. 판매수량예측
   - 녹차: {수량}개
   - 말차라테 : {수량}개
   
  3. 고찰
   - 날씨와 경제지표의 영향, 그외의 요인
   
  4. 제안
   - 다음달 판매액을 증가시키기위한 구체적인 제안

```



<img src="https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111233721319.png" alt="image-20250111233721319" style="zoom:67%;" />

#### 마무리

이제 ‘끝’ 을 붙여 마무리를 한다. 

![image-20250111233934490](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111233934490.png)



### 최종 실행 

Dify에서 해당 워크플로우를 발행을 하고 앱 실행을 하면 다음과 같이 작동되는 것을 확인할 수 있다.



![image-20250111234319030](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2025/images/image-20250111234319030.png)



다음 한달 예측 매출 및 수량에 대애 어느정도 일리는 있는 정도로 예측이 된 것 같다. 이제 앞으로 스프레드 시트의 데이터만 조정하면 다음달 예상 매출 및 수량을 구할 수 있는 시스템이 쉽게 만들어진 것이다.
