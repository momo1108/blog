---
layout: post
title: 쿠키(Cookie)의 SameSite 설정
date: '2024-01-10 17:47:23 +0900'
category: [자기계발, Cookie]
tags: [cookie, samesite]
---

# Cookie의 SameSite 속성
Loaple 사이드 프로젝트 사이트에서 현재 일간 방문자수를 체크하기 위해 Cookie를 사용하고 있다.

간단하게 설명하면 Loaple 도메인에 접속한 시점에 쿠키가 존재하지 않으면 쿠키를 생성하며 방문자수를 +1 하여 갱신한다.

쿠키의 만료기간을 그 날 자정까지로 설정해서 다음날이 되면 쿠키가 사라지게 만들어놓았다.

그런데 직접 사용하다보니 인덱스 페이지를 접속했을 때 방문자 수가 증가했음에도 그 다음 캐릭터 검색 페이지로 넘어가면 다시 방문자 수가 증가한 것을 확인했다.

원인을 확인하기 위해서 직접 로그를 찍어보았다.

확인 결과 Chrome 브라우저는 문제가 없었으나 Firefox 브라우저에서 이 같은 증상이 발생했다..

정확히는 인덱스 페이지에 처음 접속하면 쿠키가 없는 것을 로그로 확인하고 생성했는데, 캐릭터 생성 페이지로 넘어갔을 때 또 다시 쿠키가 없는 것으로 출력이 됐다.

Firefox의 개발자 도구를 확인해보니 SameSite 속성에 문제가 있다는 듯한 메세지가 보였다.

## 조치
원인에 대해서 구글링 하다보니 찾은 내용이 있다.

Chrome 브라우저의 경우 2020년 2월 출시된 Chrome 80 버전 부터 SameSite 값을 따로 설정하지 않는 경우 `SameSite=Lax` 로 취급한다는 업데이트 내용이 있었다.(<https://developers.google.com/search/blog/2020/01/get-ready-for-new-samesitenone-secure?hl=ko>{:target="_blank"})

Firefox 브라우저의 경우에도 동일한 조치를 한다는 공지가 있긴 하지만, 일부 웹사이트에는 예전 기본값이 적용될 수도 있다고 적혀있다.(<https://hacks.mozilla.org/2020/08/changes-to-samesite-cookie-behavior/>{:target="_blank"})

이 때문에 동작에 차이가 있는 것 같다. Chrome 브라우저에서 동작에 문제가 없었으므로, 해당 설정으로 통일해주도록 하자.

쿠키를 설정하는 부분의 코드 끝에 `SameSite=Lax;` 라고 추가해주면 된다.

빌드 후 동작을 확인해보니 Firefox에서도 제대로 쿠키가 동작했다.

## SameSite
그렇다면 원인은 무엇이었을까?

기존에 사용되던 SameSite 의 값은 `None` 값이었다.

SameSite의 값에 따른 동작방식은 Mozilla에서 긁어와봤다.



Controls whether or not a cookie is sent with cross-site requests, providing some protection against cross-site request forgery attacks (CSRF).

The possible attribute values are:

Strict

    Means that the browser sends the cookie only for same-site requests, that is, requests originating from the same site that set the cookie. If a request originates from a different domain or scheme (even with the same domain), no cookies with the SameSite=Strict attribute are sent.
Lax

    Means that the cookie is not sent on cross-site requests, such as on requests to load images or frames, but is sent when a user is navigating to the origin site from an external site (for example, when following a link). This is the default behavior if the SameSite attribute is not specified.
None

    means that the browser sends the cookie with both cross-site and same-site requests. The Secure attribute must also be set when setting this value, like so SameSite=None; Secure. If Secure is missing an error will be logged:

    Cookie "myCookie" rejected because it has the "SameSite=None" attribute but is missing the "secure" attribute.

    This Set-Cookie was blocked because it had the "SameSite=None" attribute but did not have the "Secure" attribute, which is required in order to use "SameSite=None".

    Note: A Secure cookie is only sent to the server with an encrypted request over the HTTPS protocol. Note that insecure sites (http:) can't set cookies with the Secure directive, and therefore can't use SameSite=None.

