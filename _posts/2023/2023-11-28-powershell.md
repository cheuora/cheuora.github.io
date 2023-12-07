---
layout: post
title: windows powershell 에서 grep 사용하기
tags: [windows, powershell, grep]
---

번역 작업을 주로 markdown파일로 하고 있다. 이 때 유용한 기능이 macOS나 linux에서 사용하는 `grep` 명령어인데, 디렉토리 내 파일(텍스트 파일이어야 함) 중에 특정 단어가 포함되어 있는 파일 및 라인수를 다 찾아주기 때문이다. 

가끔 이 작업을 windows에서 할 일이 발생하는데 windows는 grep 명령이 없어서 상당해 불편했다. totalcommander의 검색 기능으로 찾기는 하지만, 이것도 좀 번거로왔다. 

그러던 중 windows powershell 에서 이와 비슷한 명령이 있다는것을 알았는데 바로 `Select-Sting` 이다. `grep`과 기능은 거의 비슷하다.

그런데 이게 손에 익지도 않고 `Select-String`이라는 명령어도 쓰려고 하면 잘 기억이 안났다. 그래서 찾아낸 방법이 powershell의 `New-Alias`기능이다. 

Powershell에서 `New-Alias`를 통해 `Select-String`명령에 `grep`이라고 Alias를 먹이는 명령이다. 

```powershell
> New-Alias -Name grep -Value Select-String
```

이렇게 하면 현재 세션에서만 적용이 되며 powershell 을 종료하면 사라진다.

이를 유지시키려면 세션 프로파일에 입력해 놓아야 한다. 

```powershell
New-Item -path $PROFILE -type file -force
notepad $PROFILE
```

이러면 노트패드에 `$profile$` 이란 이름의 파일이 열린다. 여기에 

```
New-Alias -Name grep -Value Select-String
```

을 입력하고 저장한다. 

이후 powershell 을 종료하고 다시 시작해도 Alias가 사라지지않고 `grep`명령이 그대로 실행된다. 



이제 powershell에서도 `grep` 명령을 쓸 수 있게 되었다. ㅋ



⭐︎참고 : [Set-Alias](https://learn.microsoft.com/ko-kr/powershell/module/microsoft.powershell.utility/set-alias?view=powershell-7.3)
