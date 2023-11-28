---
layout: post
title: windows powershell 에서 grep 사용하기
tags: [windows, powershell, grep]
---

번역 작업을 주로 markdown파일로 하고 있다. 이 때 유용한 기능이 macOS나 linux에서 사용하는 `grep` 명령어인데, 디렉토리 내 파일(텍스트 파일이어야 함) 중에 특정 단어가 포함되어 있는 파일 및 라인수를 다 찾아주기 때문이다. 

가끔 이 작업을 windows에서 할 일이 발생하는데 windows는 grep 명령이 없어서 상당해 불편했다. totalcommander의 검색 기능으로 찾기는 하지만, 이것도 좀 번거로왔다. 

그러던 중 windows powershell 에서 이와 비슷한 명령이 있다는것을 알았는데 바로 `Select-Sting` 이다. `grep`과 기능은 거의 비슷하다.

그런데 이게 손에 익지도 않고 `Select-String`이라는 명령어도 쓰려고 하면 잘 기억이 안났다. 그래서 찾아낸 방법이 powershell의 `Set-Alias`기능이다. 

Powershell에서 `Set-Alias`를 통해 `Select-String`명령에 `grep`이라고 Alias를 먹이는 명령이다. 

```
> Set-Alias -Name grep -Value Select-String
```



이제 powershell에서도 `grep` 명령을 쓸 수 있게 되었다. ㅋ



⭐︎참고 : [Set-Alias](https://learn.microsoft.com/ko-kr/powershell/module/microsoft.powershell.utility/set-alias?view=powershell-7.3)
