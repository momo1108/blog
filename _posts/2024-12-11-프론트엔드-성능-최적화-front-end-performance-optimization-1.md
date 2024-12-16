---
layout: post
title: 프론트엔드 성능 최적화(Front-End Performance Optimization)-1
date: '2024-12-11 14:53:15 +0900'
category: [프론트엔드]
tags: [최적화,lighthouse]
math: true
---

# 프론트엔드 성능 최적화
이번 사이드 프로젝트 [Store Soljik](https://github.com/momo1108/StoreSoljik){: target='_blank' } 의 기능들을 개발하면서, 새로운 기술 스택에 대한 경험들을 얻었다. 사실 기능 구현에 있어서는 다른 사이드 프로젝트에서도 재미있는 기능들을 구현해봤다고 생각하기 때문에 내가 진심으로 하고싶었던 부분이 바로 성능 최적화 부분이다.

성능 최적화 부분은 사용자 잔존율과 이탈률에 커다란 영향을 미친다. 간단하게 말하면 웹사이트의 성능이 저하되면(속도가 느리면) 사용자가 이탈하고, 성능이 향상되면(속도가 빠르면) 사용자가 늘어난다는 간단하면서도 중요한 영역이다.

성능 최적화는 그 영역이 정말 다양한데, 그 중에서도 Front-End 에 관련된 최적화 내용들만 찾아서 종합해보려 한다.

---

## 프론트엔드 성능 최적화의 종류
여러 블로그들이나 도큐먼트, 영상, 책들을 살펴본 결과 성능 최적화는 크게 2가지로 나눌 수 있을 것 같다.

1. 로딩 성능 최적화
2. 렌더링 성능 최적화

### 로딩 성능
**클라이언트가 서버로부터 웹 페이지, 필요한 기타 리소스를 다운로드할 때의 성능**이다. 대표적으로 HTML, JS, CSS, 기타 이미지, 폰트 파일 등등이 있다.

만약 JS 파일이나 이미지, 폰트 파일의 크기가 크면 인터넷 속도가 느린 경우 그만큼 웹 페이지 로딩이 느려지고, 사용자가 화면을 보기위한 시간이 느려진다.

따라서 JS 파일의 경우는 목적에 따라서 코드를 분할하고(code splitting), 이미지 파일이나 폰트 파일 등은 개수를 줄이거나 크기를 줄이거나 필요한 시점에 로딩하는 방식들이 있다.

그 외에도 리소스들의 우선순위를 정해 중요한 리소스 먼저 다운로드할 수 있다.

### 렌더링 성능
로딩 과정이 끝난 후 **다운로드된 리소스로 화면을 그리는 과정의 성능**이다. 코드를 실행하여 화면에 보여 주는 과정으로 자바스크립트 코드가 가장 큰 영향을 미친다.

따라서 코드를 얼마나 잘 작성했는지에 따라 화면이 그려지는 속도와 사용자 인터랙션의 자연스러움이 달라진다.

렌더링 성능의 치적화를 위해서는 자신의 서비스의 유형과 프레임워크, 브라우저의 동작 원리 등의 개발 지식이 필요하다.

---

## 웹 어플리케이션의 동작 과정
먼저 웹 어플리케이션이 사용자 관점에서 어떻게 동작하는지부터 정리를 해보아야 성능 측정과 최적화 방식을 이해하는데 좋을 것 같다.

MDN 의 웹 도큐먼트를 살펴보면 [Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API){: target='_blank' } 가 등장하는데, 이게 바로 웹 어플리케이션의 성능을 측정하기 위한 기준들을 제공하는 API 이다.

웹 어플리케이션의 성능을 이해하기 위한 [가이드](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API#guides){: target='_blank' } 부분을 보면, Performance API 를 이해하는데 가장 핵심적인 컨셉들을 설명하고 있다.

번역해서 정리해보자면 다음과 같다.

Performance Data
: Performance API 는 다양한 성능 데이터를 수집, 접근하고 작업에 사용할 수 있다. Performance API 에서 사용되는 대부분의 지표들은 브라우저가 자동으로 수집하므로 우리는 그냥 가져다 쓰면 된다. 단, Element Timing, User Timing, Server Timing 지표들은 브라우저에게 어떤것을 측정할 것인지 직접 지정해줘야 한다.

High Precision Timing
: non-monotonic 한 Date 의 timestamp 와 달리 monotonic clock 방식으로, 1ms 미만 시간 기준의 높은 정확도의 측정값들을 사용할 수 있다.

Resource Timing
: 이미지나 JS, CSS 등의 리소스들을 fetch 하는 네트워크 타이밍을 측정한다.

Navigation Timing
: 한 페이지에서 다른 페이지로 이동하는 네비게이션 타이밍을 측정한다.

User Timing
: 웹 어플리케이션에 맞춘 Performance Data 를 측정한다.

Server Timing
: 서버와 유저 에이전트간 request-response 사이클에 관련된 지표들을 수집한다.

Long Animation Frame Timing
: Long Animation Frames(LoaFs: 50ms 이상 걸리는 렌더링 업데이트. 60fps 를 위해서는 16ms 안에 렌더링되어야 한다.) 은 느린 UI 업데이트를 유발하므로, 끊기는 애니메이션과 안좋은 조작감 등 유저 경험을 악화시킬 수 있다. Long Animation Frames API 를 통해 Long Animation Frames 의 정보를 얻고 근본 원인을 찾아낼 수 있다.

Monitoring bfcache blocking reasons
: 현재 문서가 왜 back/forward cache (bfcache: history navigation(이전 페이지, 이후 페이지) 을 위해 현대 브라우저에서 제공하는 최적화 기능) 사용이 금지됐는지 알려준다.

다양한 컨셉들이 있지만, 이중에서도 웹 어플리케이션의 성능에 가장 연관이 높아보이는 Resource Timing 과 Navigation Timing 에 대해서 좀 더 알아보려고 한다.

### Resource Timing
![timestamp-diagram](/assets/img/captures/1_timestamp-diagram.png){: width='700' .normal }

위 그림은 w3c 에서 제공하는 Processing Model 로 **Resource Timing** 과 **Navigation Timing** 을 포함한다. 각각 [PerformanceNavigationTiming Interface](https://w3c.github.io/navigation-timing/#dom-performancenavigationtiming){: target='_blank' } 에서 정의하고 있는 타이밍 속성들을 나타내고, 소괄호로 묶여있는 것은 다른 origin 문서에서의 네비게이션에서는 동작하지 않을 수 있다는 뜻이다.

먼저 노란색 위주의 [Resource Timing](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API/Resource_timing){: target='_blank' } 을 살펴보자.

#### 개념
Resource Timing 은 **어플리케이션의 리소스를 불러올 때 네트워크의 시간 데이터를 받아서 분석할 수 있는 API** 이다. 이를 활용해 페이지 로드의 일부 혹은 자바스크립트(ex. fetch api 등)로 불러오는 특정 리소스(이미지나 스크립트 파일)를 불러오는데 걸린 시간을 구할 수 있다.

문서 내의 모든 리소스들은 [PerformanceResourceTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming){: target='_blank' } (*[PerformanceEntry](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry){: target='_blank' } 인터페이스를 확장한 인터페이스*)의 entry 로서 표현된다.(`{ ..., entryType:"resource" }`)

리소스 각각의 PerformanceResourceTiming entry 들을 위해 리소스 로딩 타임라인은 전부 high-resolution timestamps 를 통해 리다이렉트 시작/끝, dns lookup 시작/끝, request/response 시작/끝 등의 시간들이 기록된다. 이런 시간 정보 외에도 리소스의 사이즈나 fetch 를 일으킨 리소스의 타입 등 리소스 관련 정보도 properties 로 제공된다.

#### API
그럼 이제 Resource Timing 에서 제공하는 **타임스탬프 api** 들을 흐름에 맞게 살펴보자.

1. `startTime`: 리소스 로딩 프로세스가 시작되기 직전
2. `redirectStart`: redirect 를 일으키는 fetch 시점
3. `redirectEnd`: 최근 redirect 에 대한 응답의 last byte 까지 받은 직후
4. `workerStart`: Service Worker 쓰레드가 시작되기 직전
5. `fetchStart`: 브라우저가 리소스 fetch 를 시작하기 직전
6. `domainLookupStart`: 브라우저가 리소스의 domain name lookup 을 시작하기 직전
7. `domainLookupEnd`: 브라우저가 리소스의 domain name lookup 을 완료한 직후
8. `connectStart`: 유저 에이전트가 서버에서 리소스를 받아오기 위한 연결 설정을 시작하기 직전
9. `secureConnectionStart`: 리소스가 secure connection 을 사용하는 경우,브라우저가 현재 연결 보안을 위한 handshake 프로세스를 시작하는 시점(ex. https 연결의 tls handshake)
10. `connectEnd`: 브라우저가 서버로부터 리소스를 받아오기 위한 연결 설정을 완료한 직후
11. `requestStart`: 브라우저가 서버/캐시/로컬 리소스를 요청하기 직전
12. `responseStart`: 브라우저가 서버/캐시/로컬 리소스 응답의 first byte 를 받은 직후
13. `responseEnd`: 브라우저가 리소스 응답의 last byte 를 받은 직후 or transport connection 이 끝나기 직전 (둘 중 더 빠른 시점)

종합해보면 네트워크 요청을 통해 리소스를 가져오는 과정은 다음과 같다.

`redirect 가 시작` - `Service Worker 가 시작` - `리소스를 fetch 하기 시작` - `리소스의 domain name lookup` - `연결 설정` - `리소스를 요청` - `응답 완료 and 연결 종료`

또한 위의 타임스탬프들을 조합해 TCP handshake, DNS lookup, 리다이렉트 시간 등 다양한 시간 지표([Typical Resource Timing Metrics](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API/Resource_timing#typical_resource_timing_metrics){: target='_blank' })들을 구해낼 수 있다.

### Navigation Timing
다음은 Navigation Timing 이다.

#### 개념
Navigation Timing 은 **한 페이지에서 다른 페이지로의 navigating** 에 관련된 지표들과 관련되어있다. 예를들어, **document 를 load/unload** 하는데 걸린 시간을 계산하거나 혹은 **DOM 생성이 완료**되고 **DOM 과 상호작용이 가능**해해지는데 걸린 시간을 기록한다.

오직 현재 document 만 포함되기 때문에 보통 하나의 [PerformanceNavigationTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming){: target='_blank' } 객체만을 사용한다. 이 객체는 [PerformanceEntry](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry){: target='_blank' } 인터페이스의 확장(`{ ..., entryType: "navigation" }`)이고, [PerformanceResourceTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming){: target='_blank' } 인터페이스를 상속한 인터페이스다. 따라서 document 를 fetching 하는 과정의 타임스탬프들 또한 모두 사용 가능하다.

![Navigation Timing](/assets/img/captures/2_navigation_timing.png){: .normal }

#### API
이제 Navigation Timing 에서 제공하는 **타임스탬프 api** 들을 흐름에 맞게 살펴보자.

1. startTime: 항상 0 으로 설정됨.
2. unloadEventStart: 이전 document 가 있는경우, 현재 document 의 unload 이벤트 핸들러가 시작되기 직전
3. unloadEventEnd: 이전 document 가 있는경우, 현재 document 의 unload 이벤트 핸들러가 실행 완료된 직후
4. domInteractive: DOM 생성이 끝나고 javascript 로 상호작용이 가능해진 시점
5. domContentLoadedEventStart: 현재 document 의 DOMContentLoaded 이벤트 핸들러가 시작되기 직전
6. domContentLoadedEventEnd: 현재 document 의 DOMContentLoaded 이벤트 핸들러가 실행 완료된 직후
7. domComplete: document 와 모든 sub-resource 들의 로딩이 완료된 시점
8. loadEventStart: 현재 document 의 load 이벤트 핸들러가 시작되기 직전
9. loadEventEnd: 현재 document 의 load 이벤트 핸들러가 실행 완료된 직후

종합해보면 리소스를 불러온 후, 페이지가 렌더링되는 과정은 다음과 같다.

(이전 document 가 있는 경우)`window.onbeforeunload, window.onunload 이벤트 핸들러` - `DOMContentLoaded 이벤트 핸들러` - `window.onload 이벤트 핸들러`

이 이벤트들은 HTML 문서의 라이프 사이클에 관여하는 핵심 이벤트들이다.

각각의 이벤트에 대한 설명을 [모던 JavaScript 튜토리얼](https://ko.javascript.info/onload-ondomcontentloaded){: target='_blank' } 에서 인용해 아래처럼 정리해봤다.

#### beforeunload
사용자가 사이트를 떠나려 할 때, 변경되지 않은 사항들을 저장했는지 확인시켜줄 때 사용한다.

#### unload
사용자가 진짜 떠나기 전에 사용자 분석 정보를 담은 통계자료를 전송하고자 할 때 사용한다.

#### DOMContentLoaded
브라우저가 HTML을 전부 읽고 DOM 트리를 완성하는 즉시 발생한다. 이미지 파일(\<img\>)이나 스타일시트 등의 기타 자원은 기다리지 않는다. DOM이 준비된 것을 확인한 후 원하는 DOM 노드를 찾아 핸들러를 등록해 인터페이스를 초기화할 때 사용할 수 있다. 이 이벤트는 document 객체에서 발생해야 하므로 **document.addEventListener** 메서드를 사용해서 설정해야 한다.

```html
<script>
    function ready() {
        alert('DOM이 준비되었습니다!');
        // 이미지가 로드되지 않은 상태이기 때문에 사이즈는 0x0으로 출력된다.
        alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`);
    }
    document.addEventListener("DOMContentLoaded", ready);
</script>
<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

문서가 로드된 후 실행되므로, 핸들러 내부에서는 핸들러 코드 아래쪽에 있는 모든 요소(img, p, div, ...)에 id를 통해 접근할 수 있지만 이미지나 외부 스타일시트는 적용되지 않은 상태라서 img 의 크기는 0으로, div 의 크기는 기본 설정된 크기로 출력된다.

브라우저가 HTML 문서를 처리할 때, \<script> 태그의 내용은 DOM 조작 관련 로직이 있을 수 있어서 DOM 트리의 구성을 멈추고 \<script> 태그의 내용이 먼저 실행된다. 아래의 코드를 살펴보자.

```html
<script>
  document.addEventListener("DOMContentLoaded", () => {
    alert("DOM이 준비되었습니다!");
  });
</script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js"></script>

<script>
  alert("라이브러리 로딩이 끝나고 인라인 스크립트가 실행되었습니다.");
</script>
```

이 코드에서 가장 먼저 DOMContentLoaded 이벤트 핸들러를 등록하지만, 아래 스크립트들이 모두 실행될 때 까지 DOM 트리의 구성이 멈추기 때문에, 아래의 스크립트들이 모두 실행된 이후에 이벤트 핸들러가 실행된다.

##### 예외
마지막으로 위 규칙들에 대한 예외가 몇가지 존재한다.

1. `async` 속성을 사용한 스크립트는 DOMContentLoaded 를 블락하지 않는다.

    [**async, defer 속성이란?**](https://ko.javascript.info/script-async-defer){: target='_blank' }
    > defer 와 async 모두 스크립트가 백그라운드에서 실행된다는 특징은 동일하다. 
    > - defer 는 페이지 생성을 막지 않고 DOM 이 준비된 후 실행되지만 `DOMContentLoaded` 이벤트 발생 전에는 실행된다.(defer 는 src 속성이 존재하는 외부 스크립트에서만 동작한다. src 가 없는 경우 무시된다.)
    > ```html
        <p>...스크립트 앞 콘텐츠...</p>
        <script>
        document.addEventListener('DOMContentLoaded', () => alert("`defer` 스크립트가 실행된 후, DOM이 준비되었습니다!")); // 이벤트 핸들러는 먼저 등록되지만 defer 스크립트가 완료된 후에 실행이 된다.
        </script>
        <script defer src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>
        <p>...스크립트 뒤 콘텐츠...</p> <!-- 직전의 외부 스크립트의 실행 완료를 기다리지 않고 바로 페이지에 출력된다. -->
      ```
    > - async 는 페이지와는 완전 독립적으로 동작한다. async 스크립트와 `DOMContentLoaded` 이벤트, 또 async 스크립트끼리는 서로의 실행을 기다리지 않습니다.
    > ```html
        <p>...스크립트 앞 콘텐츠...</p>
        <script>
          document.addEventListener('DOMContentLoaded', () => alert("DOM이 준비 되었습니다!"));
          // 아래의 async 스크립트 실행 전 핸들러가 실행되고, 컨텐츠도 모두 보인다.
        </script>
        <script async src="https://javascript.info/article/script-async-defer/long.js"></script>
        <script async src="https://javascript.info/article/script-async-defer/small.js"></script>
        <p>...스크립트 뒤 콘텐츠...</p>
      ```
    {: .prompt-info }
2. `document.createElement('script')` 메서드로 동적 생성된 스크립트는 DOMContentLoaded 를 블락하지 않습니다.
3. 간혹 스타일시트의 영향을 받는 요소의 속성을 사용해야하는 경우를 위해 외부 스타일시트 중 \<script> 바로 이전에 스타일 시트를 불러오면 스크립트의 실행이 스타일시트를 불러올 때 까지 블락됩니다.
  ```html
    <link type="text/css" rel="stylesheet" href="style.css">
    <script>
      // 이 스크립트는 위 스타일시트가 로드될 때까지 실행되지 않습니다.
      alert(getComputedStyle(document.body).marginTop);
    </script>
  ```

#### load
HTML로 DOM 트리를 만드는 게 완성되었을 뿐만 아니라 이미지, 스타일시트 같은 외부 자원도 모두 불러와서 적용이 끝났을 때 발생한다. 따라서 이미지 사이즈를 확인할 때 등 화면에 뿌려지는 요소의 실제 크기를 확인할 수 있다.

---

## 성능 측정 도구
성능 최적화에는 다양한 방식들이 있지만, 이러한 방식들이 얼마나 효과적인지를 알기 위해서는 최적화 적용 전/후 시점의 성능을 각각 측정할 필요가 있다.

이러한 관점에서 성능을 측정하기 위한 다양한 지표들과 측정 도구가 필요하다. 대부분의 개발자들이 사용하는 도구들과 지표들이 있

---

서비스 워커
유저 에이전트