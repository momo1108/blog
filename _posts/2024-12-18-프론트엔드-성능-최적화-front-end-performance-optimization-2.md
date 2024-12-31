---
layout: post
title: 프론트엔드 성능 최적화(Front-End Performance Optimization)-2
date: '2024-12-18 16:07:21 +0900'
category: [프론트엔드]
tags: [최적화,lighthouse]
math: true
---

# 성능 최적화
이번 포스트에서는 실제로 사용할 수 있는 성능 최적화 방법들을 알아보자.

이전 포스트에서 성능 최적화를 크게 두가지로 분류한다고 정리했었다.

1. **로딩 성능 최적화**
  - **클라이언트가 서버로부터 웹 페이지, 필요한 기타 리소스를 다운로드할 때의 성능**.(ex. HTML, JS, CSS, 기타 이미지, 폰트 파일, ...)
  - 만약 JS 파일이나 이미지, 폰트 파일의 크기가 크면 인터넷 속도가 느린 경우 그만큼 웹 페이지 로딩이 느려지고, 사용자가 화면을 보기위한 시간이 느려진다.
  - JS 파일의 경우는 목적에 따라서 코드를 분할하고(code splitting), 이미지 파일이나 폰트 파일 등은 개수를 줄이거나 크기를 줄이거나 필요한 시점에 로딩하는 최적화 방식들이 존재한다.
  - 그 외에도 리소스들의 우선순위를 정해 중요한 리소스 먼저 다운로드하도록 설정하는 방식이 존재한다.
2. **렌더링 성능 최적화**
  - 로딩 과정이 끝난 후 **다운로드된 리소스로 화면을 그리는 과정의 성능**. 코드를 실행하여 화면에 보여 주는 과정으로 자바스크립트 코드가 가장 큰 영향을 미친다.
  - 코드를 얼마나 잘 작성했는지에 따라 화면이 그려지는 속도와 사용자 인터랙션의 자연스러움이 달라진다.
  - 렌더링 성능의 치적화를 위해서는 자신의 서비스의 유형과 프레임워크, 브라우저의 동작 원리 등의 개발 지식이 필요하다.

이 두가지 관점에서 내 프로젝트엔 어떤 최적화를 적용할 수 있는지 알아보고 결과를 살펴보자.

## 성능 측정 도구
최적화를 적용한 전/후로 성능의 변화를 측정하기 위해선 성능 측정 도구가 먼저 필요하다.

1. **크롬 개발자 도구** : 브라우저에서 제공하는 도구로, f12 를 눌러서 열 수 있다. 여러가지 패널을 제공하는데, 그 중 성능 측정과 관련된 패널들이 있다.
  - **Network** : 현재 페이지에서 발생하는 모든 네트워크 트래픽의 상세 정보를 볼 수 있다. 각 리소스의 로딩 시점, 크기 등을 확인할 수 있다.<br>
  ![chrome devtools network panel](/assets/img/captures/1_chrome_devtool_network.png){: width='700' .normal }_네이버 메인페이지_
  - **Performance** : 현재 페이지가 로드될 때 실행되는 모든 작업을 보여준다. 리소스의 로드와 크기 뿐 아니라, 브라우저 메인 쓰레드에서 실행되는 자바스크립트도 차트 형태로 볼 수 있다. 이를 통해 느리게 실행되는 자바스크립트 코드도 확인 가능하다.<br>
  ![chrome devtools performance panel](/assets/img/captures/2_chrome_devtool_performance.png){: width='700' .normal }_네이버 메인페이지_
  - **Lighthouse** : 구글에서 만들어진 툴로 웹 사이트의 성능을 측정하고 개선 방안을 제시해 주며, 원래는 별도로 설치했지만 현재는 개발자 도구의 한 패널로 포함되었다.<br>
  ![chrome devtools lighthouse panel](/assets/img/captures/3_chrome_devtool_lighthouse.png){: width='700' .normal }_네이버 메인페이지_
2. **번들 분석 패키지** : 웹 어플리케이션의 빌드에서 번들링된 파일이 어떤 라이브러리를 포함하는지 시각화 해주는 패키지.
  - **webpack-bundle-analyzer** : 웹팩을 통해 번들링된 프로젝트를 확인
  - **rollup-plugin-visualizer** : 롤업을 통해 번들링된 프로젝트를 확인(vite, sveltekit 등을 지원)

이중에서 가장 많이 사용되는 툴은 Lighthouse 이다. 다른 도구들이 단순히 성능적인 지표들을 보여준다면 Lighthouse 는 어느 부분의 성능이 저조한지, 개선방안은 무엇이 있는지 조언해주는 툴이라고 볼수있다. 아직 최적화 경험이 많지 않기 때문에 나도 이번에 Lighthouse 툴을 사용해 최적화를 진행해보기로 했다.

### Lighthouse
Lighthouse 의 기본적인 사용방법으로 3가지 옵션을 설정해야 한다.

1. Mode : 어떤 방식으로 분석을 진행할 것인가?
  - Navigation : (기본값)초기 페이지 로딩 시 성능 분석
  - Timespan : 사용자 정의 시간 내 성능 분석
  - Snapshot : 현재 상태의 성능 분석
2. Device : 분석 환경으로 데스크탑 / 모바일 둘 중 하나를 선택
3. Categories : 어떤 카테고리의 성능을 분석할 것인가?
  - Performance : 웹 페이지 로딩 과정의 성능 분석
  - Accessibility : 사용자 접근성 분석
  - Best practices : 웹 페이지의 보안, 최신 표준에 중점을 둔 분석
  - SEO : 검색 엔진에 크롤링, 검색 결과

기본적으로 Navigation 모드로 Performance 카테고리를 활용하여 웹 페이지의 종합 성능을 확인할 수 있다. 아래의 스크린샷은 사이드 프로젝트의 메인 페이지를 분석한 결과이다.

#### 분석 결과
![Lighthouse 분석 결과](/assets/img/captures/4_lighthouse_result.png){: width='700' .normal }

먼저 1번으로 체크한 최상단 부분에서는 Categories 에서 활성화한 분석 카테고리들의 평가 점수가 나온다. 2번위치부터는 각각의 카테고리 항목에 대한 성능 지표들과 함께 성능적인 문제가 있는 부분들을 알려준다.

메인 페이지의 Performance 분석 결과를 더 자세히 살펴보자.

#### 성능 지표
![Lighthouse 분석 결과](/assets/img/captures/5_lighthouse_result.png){: width='700' .normal }

Performance 카테고리에서는 가장 먼저 성능 지표(Metrics)로서 웹 바이탈(Web Vitals)라는 지표를 보여준다.

각각의 지표들을 사용해 종합 성능을 계산하는데, 먼저 지표들을 하나씩 살펴보자.

First Contentful Paint(FCP)
: 페이지 로딩 시 브라우저가 DOM 컨텐츠의 첫번째 부분(텍스트, 이미지 등)을 렌더링하기까지 걸리는 시간이다. 따라서 메인 페이지에서 첫번째 컨텐츠를 렌더링하기까지 0.7초가 걸렸다는 뜻이다.

Largest Contentful Paint(LCP)
: 페이지 로딩 시 화면 내의 가장 큰 이미지나 텍스트가 렌더링되기까지 걸리는 시간을 나타낸다. 메인 페이지에서 가장 큰 컨텐츠를 렌더링하는데 1.5초가 걸렸다.

Total Blocking Time(TBT)
: 페이지 내 사용자 입력(클릭, 키보드 입력 등)을 차단한 시간을 총합한 지표이다. 측정은 FCP 와 TTI(*Time to Interactive: 사용자가 페이지와 상호 작용이 가능한 시점까지 걸린 시간. 화면이 보여도 TTI 전까지는 클릭, 입력 등이 안된다.*)사이에 일어나며, 메인 스레드를 독점하여 다른 동작을 방해하는 작업에 걸린 시간을 총합한다. 메인 페이지는 0ms 의 TBT로, 첫번째 컨텐츠 이후부터는 입력이 제한되지 않은 것을 확인할 수 있다.

Cumulative Layout Shift(CLS)
: 페이지 로딩 중 예기치 못한 레이아웃 이동(화면상 요소의 위치나 크기의 순간적 변화. 특정 버튼을 누르려다가 상단에 새로운 요소가 생기면 버튼 위치가 밀려서 다른 버튼이 눌리는 문제가 대표적)을 측정한 지표이다. 0.012 라는 측정값이 나왔는데 네트워크 속도를 제한을 걸고 확인해보니 용량이 큰 폰트가 로딩이 되는 순간 환영 문구의 사이즈가 변해 하단의 input, button 의 위치가 변하는 것이 원인인 것 같다.

Speed Index(SI)
: 페이지 로딩 중 컨텐츠가 시각적으로 표시되는 속도를 나타내는 지표이다. 컨텐츠 전체가 표시되는 시간 뿐 아니라 각각의 일부 요소들이 먼저 뜨는 경우 더 빠른 것으로 판단한다. 메인 페이지는 컨텐츠들이 출력되기까지 0.7초가 걸렸다.

단순히 지표를 계산해서 보여줄 뿐 아니라 성능이 떨어지는 지표를 표시해주고, diagnostics 항목에서는 어떤 부분이 성능을 저하시키는지 진단해주고 대표적인 해결방안도 제안해준다.

메인 페이지의 성능 저하를 유발하는 진단 결과를 살펴보자.

#### 진단 결과
![Lighthouse 진단 결과](/assets/img/captures/6_lighthouse_diagnostics.png){: width='700' .normal }

최상단에서부터 차례대로 우선순위가 높은 문제점들을 알려준다. 첫번째 항목부터 살펴보자.

![Lighthouse 진단 결과](/assets/img/captures/7_lighthouse_diagnostics.png){: width='700' .normal }

메인 페이지 내 존재하는 이미지 파일(로고, 메인 이미지)들을 최신 포맷(WebP, AVIF) 형태로 제공하라는 진단이다. 예상되는 용량 축소는 총 277.8 KiB 중 253 KiB 로 거의 90% 에 가까운 용량을 축소할 수 있다.

내 프로젝트에는 이미지에 최신 포맷을 적용하는 방법과 이미지 사이즈에 맞게 원본 이미지를 리사이징하는 방식을 사용할 것이다.

두번째 항목은 LCP 이미지를 preload 하라는 진단이다. 이미지를 페이지 내 다른 리소스들보다 우선적으로 로딩하는 방식으로, 잘 설명된 링크를 첨부하겠다. [Preload LCP Image](https://speedvitals.com/blog/preload-lcp-image/){: target='_blank' }

세번째 항목은 Javascript 를 축소하라는 진단이다.

![Lighthouse 진단 결과](/assets/img/captures/8_lighthouse_diagnostics.png){: width='700' .normal }

메인 페이지에서 로딩되는 자바스크립트 번들 파일의 용량이 263.3 KiB 인데, 이를 더 줄이라는 진단 결과가 나왔다. 예상되는 축소 용량은 40.9 KiB 라고 한다.

축소를 위해서는 먼저 자바스크립트 번들 파일이이 어떤 코드들로 구성이 되어있는지 확인을 해보아야 한다. **rollup-plugin-visualizer** 를 사용해 어떤 구성인지 확인해보고 결정하자. 또한 필요한 코드들만 불러올 수 있도록 코드 스플리팅 등을 적용해야 할 것 같다.

다음 진단 중 Reduce Unused Javascript 항목은 위 내용과 비슷한 내용으로, 코드의 공백이나 쓸모없는 코드를 제거하는 등의 조치가 가능하다.

마지막으로 Largest Contentful Paint element 이다. 말그대로 렌더링이 제일 오래걸린 요소를 뜻하는데, 메인 페이지의 경우 이미지 파일이 1470ms 가 걸렸다고 한다.

![Lighthouse 진단 결과](/assets/img/captures/9_lighthouse_diagnostics.png){: width='700' .normal }

상세 설명란에는 LCP 의 단계별 소요시간을 비율과 시간 단위로 알려준다.

- **TTFB** : Time to First Byte. 사용자가 페이지의 로딩을 시작하고 HTML 문서 응답의 첫번째 바이트를 받기까지의 시간
- **Load Delay** : 브라우저가 TTFB 시점에서 LCP 리소스의 로딩을 시작하기까지의 시간차
- **Load Time** : LCP 리소스를 로딩하는 시간
- **Render Delay** : LCP 리소스 로딩이 끝난 시점부터 LCP 요소가 완전히 렌더링되기까지 걸린 시간

그 외에는 주황색으로 표시된 워닝들이 있는데, 이미지의 width, hegiht 를 지정하여 레이아웃 쉬프트를 방지하고, font display 속성을 지정하라는 등의 내용이 있다.

## 성능 최적화 진행
먼저 성능 최적화에 가장 시급한 문제는 LCP 이미지와 자바스크립트 번들 파일의 사이즈 최적화이다.

첫번째로 단순히 이미지 파일의 크기를 줄이는것도 도움이 될 수 있겠지만, 좀 더 자세한 분석을 위해 다시한번 LCP 진단 결과를 살펴보자.

![Lighthouse 진단 결과](/assets/img/captures/9_lighthouse_diagnostics.png){: width='700' .normal }

특이한 점은 단순히 리소스가 커서 로딩이 오래걸렸을 거란 예상과는 다르게 Load Time 지표가 아닌 **Load Delay** 지표로 인한 지연이 **78%** 로 가장 큰 비중을 차지했다.

이를 더 자세히 살펴보기 위해 개발자 도구의 performance 탭의 레코딩 기능을 활용해 메인 페이지 로딩 과정을 관찰해보았다.

![Performance 분석 결과](/assets/img/captures/10_performance_result.png){: width='700' .normal }

하단에 표시된 네트워크 부분을 보면 LCP 로 확인된 이미지 파일의 실제 Load Time 자체는 실제 다운로드 시간 외 다른 시간들을 포함해도 10.69ms 에 이루어졌고, 시작 시점부터 10ms 정도를 더하면 237ms 정도로 1470ms 와는 크게 차이가 난다.

왜 lighthouse 에서는 1470ms 나 걸렸는지 궁금해서 설정을 살펴보다 원인을 찾았다.

![Lighthouse 세팅 항목](/assets/img/captures/11_lighthouse_settings.png){: width='500' .normal }

쓰로틀링 항목에서 Lighthouse 자체적으로 시뮬레이팅된 환경이 기본 설정으로 돼어있는데, 개발자 도구의 Performance 탭 등 다른 탭과 통일하기 위해 Devtools throttling 으로 변경하고 다시 Lighthouse 를 돌려보았다.

![Lighthouse 진단 결과](/assets/img/captures/12_lighthouse_diagnostics.png){: width='700' .normal }

이제야 performance 탭과 비슷하게 결과가 나왔다. 리소스 로딩까지 총 260ms, 렌더링까지는 추가 50ms 가 소요됐다.

다음으로 이미지의 최적화를 진행해보자.

먼저 이미지의 최적화를 위해서는 두가지 방식을 사용할 것이다.

1. 이미지 리사이징
  - 이미지의 용도(일반 이미지, 썸네일)별로 사용될 크기의 가로, 세로 최대 2배 사이즈로 축소한다.
2. 이미지 포맷 변경
  - 원본 이미지 포맷(.jpg, .jpeg, .png)과 최신 이미지 포맷(.webp, .avif)을 같이 사용한다.

