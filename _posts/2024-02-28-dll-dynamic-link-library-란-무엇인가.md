---
layout: post
title: DLL(Dynamic-Link Library)란 무엇인가
date: '2024-02-28 17:45:48 +0900'
category: [Windows, DLL]
tags: [dll,shared library,com]
---

# DLL(Dynamic-Link Library) 이란?
DLL 은 Microsoft Windows 의 shared library 이다. DLL 은 함수, 데이터, 리소스를 조합하여 만들 수 있다.

> **Shared Library 란?**
>
> 여러 컴퓨터 프로그램이나 다른 library 에서 runtime 에 사용하게 디자인된 컴퓨터 파일이다. library 안에 executable code 를 포함하고 있으며 이를 한번 메모리에 불러와 유지되며 동시에 사용할 수 있다. 이와 다르게 static library 는, linker[^fn1] 가 executable image file 을 빌드할 때 static library 의 코드를 안에 복사해 넣기 때문에 각각의 프로그램들이 해당 코드의 복사본을 따로 가지고 있는 셈이다.
>
> shared library 를 사용하는 프로그램을 실행할 때, 운영 체제는 loadtime 이나 runtime 에  shared library 를 프로그램 실행 파일 외부의 다른 파일에서 메모리로 불러옵니다.
>
> 대부분의 현대 운영체제는 shared library 와 executable files 에 대해 동일한 형식을 사용한다. 이는 두 가지 주요 장점이 있는데, 첫번째는 하나의 로더만 구축하고 유지하면 되기 때문에 이로 인한 추가적인 복잡한 작업도 감수할만 하다고 간주한다. 둘째는 executable files이 shared library 로 사용될 수 있게된다(symbol table 이 있는 경우). 이와 관련해 Unix ELF, Mach-O 및 Windows PE와 같은 예시가 있다.
{: .prompt-info }

---

## 개요
Windows 운영체제의 많은 기능들이 DLL 형태로 제공된다. 또한 Windows 운영체제에서 프로그램을 실행할 때 다양한 기능들을 DLL 로부터 제공받을 수 있다.

예시) 한 프로그램에서 다양한 모듈들을 사용하는데, 각각의 모듈들은 각각의 DLL 들에 포함되어 있다.

이렇게 DLL 을 사용하면 코드의 모듈화, 재사용성, 메모리 효율성, 저장공간 확보 등의 장점을 챙길 수 있어 더 빠른 로딩과 실행, 적은 용량의 디스크에도 저장이 가능해진다.

다만 프로그램에서 DLL 을 사용할 때, *의존성(dependency)* 이라는 개념이 프로그램 오작동을 일으킬 수 있다. 프로그램이 DLL 을 사용하면 프로그램과 DLL 간의 의존성이 생기는데, 만약 다른 프로그램에서 이 의존성을 덮어쓰거나 깨버리면 원래의 프로그램이 오작동할 수 있다.

단 .NET Framework 에서는 대부분의 의존성 문제가 assemblies 를 사용해서 해결되어있다.

---

## DLL 자세히 살펴보기
DLL 은 코드와 데이터를 가지고 있으며, 여러 프로그램에서 동시에 사용될 수 있다. 간단한 예시로 Windows 운영체제에서 *Comdlg32 DLL* 은 일반 대화상자와 관련된 기능을 수행한다. 모든 프로그램들은 각각 이 DLL 의 기능을 대화상자를 생성하는데 사용할 수 있다. 이런식으로 코드의 재활용과 메모리 효율을 상승시킨다.

DLL 을 사용함으로서 프로그램을 분리된 컴포넌트로 모듈화시킬 수 있다. 예를들어 회계 관련 프로그램이 모듈 형태로 판매될 수 있다. *회계 모듈*을 구매한 회사에서 이 모듈을 설치하고, *메인 프로그램*의 *runtime* 에 모듈을 불러온다. 이렇게 *회계 모듈*이 *메인 프로그램*과 따로 분리되어있기 때문에, 프로그램의 로딩은 더 빨라지고 *회계 모듈*은 기능이 사용될 때만 불러온다.

더불어 기능 등의 다양한 업데이트도 프로그램의 다른 부분에 영향을 주지않는 선에서 각각의 모듈에 쉽게 적용이 가능하다. 예를들어 우리가 급여대장 프로그램을 가진 사장님이면 매년 세금의 변화를 프로그램에 적용해야 한다. 세금처럼 자주 변화하는 내용이 한 DLL 에 따로 분리되어있다면, 관련 변화 내용을 프로그램 전체를 빌드하거나 설치하지 않고도 적용하여 업데이트할 수 있다.

Windows 운영체제 내부 DLL 의 일부 예시 파일
- ActiveX Controls(`.ocx`) 파일. 예시로는 날짜를 선택할 수 있게 해주는 calendar control
- Control Panel(`.cpl`) 파일. 제어판의 모든 항목들이 특화된 dll 들이다. `Windows\System` 폴더에 위치하고 있다.(내 노트북 Windows 11 에서는 System32,SysWOW64 폴더)
- Device Drive(`.drv`) 파일. 예시로는 프린터기의 프린팅을 조작하는 프린터 드라이버.

DLL 파일 대부분이 `.dll` 확장자를 가지지만, 개발자의 의도에 따라 원하는 확장자를 선택할 수 있다. 추가로 resource 만 사용해 만들어진 DLL 은 resouce DLL 이라 부른다. 예를들어 icon 라이브러리는 종종 `.icl` 확장자를, font 라이브러리는 `.fon` `.fot` 라이브러리를 가진다.

---

## DLL 의 장점
프로그램에서 DLL 을 사용하면 좋은 점들이 있다.

적은 resource 사용량
: 여러 프로그램들이 같은 기능의 library 를 사용하는 경우에, DLL 이 디스크나 메모리에 로드되는 코드의 중복을 줄여줄 수 있다. 이를 통해 foreground 의 프로그램 뿐 아니라 Windows 운영체제의 프로그램도 성능에 지대한 영향을 미칠 수 있다.

더욱 세분화(modular)된 구조
: DLL 은 modular program(세분화된 프로그램)의 개발을 증진시켜준다. 다양한 버전의 언어를 요구하는 대규모 프로그램이나 세분화된(modular) 구조를 요구하는 프로그램의 개발을 도와주는 것이다.

배포와 설치의 간편화
: DLL 내부의 함수가 업데이트나 오류개선이 필요할 때, DLL 의 배포와 설치 후 프로그램이 DLL 과 다시 링크하는 등의 작업이 필요없다. 또 여러 프로그램들이 같은 DLL 을 사용하는 경우, 모든 프로그램이 같이 업데이트나 오류개선을 받을 수 있다. 이 장점은 정기적으로 업데이트/오류개선이 되는 서드파티 DLL 을 사용하는 경우 더 부각된다.

---

## DLL 종속성(dependencies)
어떤 프로그램이나 DLL 이 다른 DLL 의 함수를 사용할 때, 서로간의 종속성이 생성된다. 이때부터 프로그램은 더이상 self-contained(독립된) 프로그램이 아니며, 종속성이 고장날 경우 문제가 발생할 수 있다.

예를들어 아래의 경우 프로그램이 작동하지 않을수도 있다.

- 종속된 DLL 의 버전 업데이트
- 종속된 DLL 의 오류 개선
- 종속된 DLL 이 이전 버전으로 덮어쓰기 된 경우
- 종속된 DLL 의 컴퓨터에서 삭제된 경우

위의 경우들은 DLL conflicts(충돌) 이라 불린다. 만약 backward compatibility(이전 버전과의 호환성) 이 적용되지 않으면, 프로그램이 작동하지 않을 수 있다.

윈도우 2000 이후의 운영체제에서 종속성 문제들을 최소화하기 위해 아래와 같은 변화들이 발표됐다.

Windows File Protection
: *Windows File Protection* 내에서 운영체제는 *unauthorized agent*(허가받지 않은 프로그램, 사용자 등) 가 *system DLL* 들을 업데이트하거나 삭제하는 것을 방지한다. 프로그램 설치시에 system DLL 로서 정해져 있는 DLL 을 제거하거나 업데이트하려고 할 때, *Windows File Protection* 은 *digital signature*(전자 서명)이 유효한지 확인한다.

Private DLLs
: *Private DLLs* 는 shared DLL 로부터 비롯되는 변화들로부터 영향을 받지 않게 해준다. *Private DLLs* 는 특정 버전별 정보 혹은 비어 있는 `.local` 파일을 사용하여 프로그램에서 사용되는 DLL의 버전을 강제한다. *Private DLLs* 를 사용하려면 DLL을 프로그램 루트 폴더에 위치시킨다. 새 프로그램의 경우 DLL에 버전별 정보를 추가하고, 이미 있던 프로그램의 경우 비어 있는 `.local` 파일을 사용한다. 각 method 는 운영체제에게 프로그램 루트 폴더에 있는 *Private DLLs* 을 사용하라고 알려준다.

---

## DLL troubleshooting tools
DLL 의 문제들을 해결하기 위한 몇가지 도구들이 존재한다. 관련된 자세한 내용은 [도큐먼트(DLL troubleshooting tools)](https://learn.microsoft.com/en-us/troubleshoot/windows-client/setup-upgrade-and-drivers/dynamic-link-library#dll-troubleshooting-tools){: target='_blank' } 를 확인하자.

---

## DLL 개발
나만의 DLL 을 개발할 때 고려해야할 문제들과 요구사항들이 있다.

### DLL 종류
어플리케이션에서 DLL 을 불러올 때, export 된 DLL 함수들을 호출할 수 있는 2가지 링크 방법이 있다.

#### Load-time dynamic linking
load-time dynamic linking 에서는 어플리케이션이 exported DLL function 을 local function 처럼 명시적 호출(explicit call)을한다. load-time dynamic linking 을 사용하기 위해서는, 어플리케이션을 compile 하고 link 할 때 헤더파일(.h)을 제공하고 library(.lib)를 import 해야 한다. 이래야 linker[^fn1] 가 DLL 을 로드하는데 필요한 정보와 exported DLL function 위치를 찾는데 필요한 정보를 시스템에 제공한다.

#### Run-time dynamic linking
Run-time dynamic linking 에서는 어플리케이션이 `LoadLibrary` 함수나 `LoadLibraryEx` 함수를 호출해 runtime 에 DLL 을 불러온다. DLL 을 성공적으로 불러온 후 `GetProcAddress` 함수를 사용해 호출하고싶은 exported DLL function 의 주소를 얻을 수 있다. Run-time dynamic linking 을 사용할 때는 library file 을 import 할 필요가 없다.

#### 사용 기준
아래의 리스트는 어플리케이션이 언제 Load-time dynamic linking 과 Run-time dynamic linking 을 사용해야 하는지에 대한 기준을 설명한다.

Startup performance
: 어플리케이션이 처음 시작할 때의 성능이 중요한 경우, run-time dynamic linking 을 사용해야 한다.

Ease of use
: load-time dynamic linking 에서는 exported DLL functions 들이 마치 local functions 와 비슷하게 사용할 수 있어 호출이 편해진다.

Applicatino logic
: run-time dynamic linking 에서는 어플리케이션이 원하는만큼 다른 모듈들을 불러오도록 branch(분기) 시킬 수 있다. 다양한 언어 버전에서 사용되는 어플리케이션을 개발할 때 중요하다.

---

## 참조
- <https://en.wikipedia.org/wiki/Dynamic-link_library>{: target='_blank' }
- <https://learn.microsoft.com/en-us/troubleshoot/windows-client/setup-upgrade-and-drivers/dynamic-link-library#dll-troubleshooting-tools>{: target='_blank' }

---

## 각주
[^fn1]: <https://en.wikipedia.org/wiki/Linker_(computing)>{: target='_blank' } 참조