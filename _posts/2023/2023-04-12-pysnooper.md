---
layout: post
title: pysnooper 소개 및 기록...
tags: [pysnooper]
---

가끔 소셜 네트워크에 재미있는 라이브러리들이 올라오는데 오늘 소개할 것은 snooper() 라이브러리이다. 

with 구문내에 있는 코드들의 각 변수들에 어떤 값이 들어 있는지 찍어주고 해당 코드의 수행 시간까지 알려준다. 샘플 코드를 수행해 보고 결과를 python3.8과 python3.11의 수행 차이를 비교해 보았다. 

샘플 코드 

```python
import pysnooper
import random

def foo():
    lst = []
    for i in range(10):
        lst.append(random.randrange(1, 1000))

    with pysnooper.snoop():
        lower = min(lst)
        upper = max(lst)
        mid = (lower + upper) / 2
        print(lower, mid, upper)

foo()
```

이 코드에서 `with pysnooper.snoop()` 내의 코드를 변수 메모리 상황, 수행속도 보여준다. 

아래는 macOS python3.11 에서 실행한 결과를 보여준다.

```bash
> python3 snooper.py
Source path:... /Users/cheuora/works/snooper.py
New var:....... lst = [411, 373, 361, 277, 955, 465, 278, 432, 281, 279]
New var:....... i = 9
13:54:43.579331 line        10         lower = min(lst)
New var:....... lower = 277
13:54:43.579454 line        11         upper = max(lst)
New var:....... upper = 955
13:54:43.579492 line        12         mid = (lower + upper) / 2
New var:....... mid = 616.0
13:54:43.579527 line        13         print(lower, mid, upper)
277 616.0 955
13:54:43.579568 line         9     with pysnooper.snoop():
Elapsed time: 00:00:00.000277

```

참고로 ubuntu python3.8 환경에서 동일한 코드를 수행해 보았다. 

```
~$ python3 snooper.py
Source path:... snooper.py
New var:....... lst = [191, 908, 665, 383, 51, 708, 335, 894, 200, 92]
New var:....... i = 9
04:54:37.187162 line        10         lower = min(lst)
New var:....... lower = 51
04:54:37.187426 line        11         upper = max(lst)
New var:....... upper = 908
04:54:37.187503 line        12         mid = (lower + upper) / 2
New var:....... mid = 479.5
04:54:37.187568 line        13         print(lower, mid, upper)
51 479.5 908
Elapsed time: 00:00:00.000539
```

OS의 차이는 있겠지만, 수행 시간에서 3.8 버전이 3.11 버전에 비해 약 2배 가까이 수행시간이 더 걸린다는 것을 알 수 았다. 

