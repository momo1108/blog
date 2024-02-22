---
layout: post
title: packet sniffing 프로젝트 뜯어보기
date: '2024-02-22 15:35:31 +0900'
category: [자기계발]
---

# 앞서서
이 포스트는 내가 원하는 기능의 앱을 개발하기 위한 기본 과정을 참조할 수 있는 깃헙 프로젝트를 직접 분석해보는 포스트입니다.

실제로 이 프로젝트를 사용하는게 아닙니다.

![감사콩](/assets/img/emoji/감사콩.png){: width='100' .normal }

## 프로젝트의 구조
이 프로젝트는 따져보자면 Windows Service 템플릿의 프로젝트이다. 앱의 설정을 담당하는 `App.config` 와 엔트리포인트를 담당하는 `Program.cs` 두 개의 베이스코드를 제외하면 모두 직접 생성한 서비스들이다.

## Program.cs
엔트리 포인트인 메인 메서드가 있는 Program.cs 파일에 스태틱 변수나 메서드들이 많이 있다.

가장 먼저나온 구문이다.

```cs
public static bool IsConsole = Console.OpenStandardInput(1) != Stream.Null;
```

IsConsole 변수는 단순히 bool 값이고, 변수에 해당하는 값은 `Console.OpenStandardInput(1)` 로 받은 1바이트 크기의 인풋 스트림이 Null 이 아닌지의 여부이다.