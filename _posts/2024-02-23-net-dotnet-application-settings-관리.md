---
layout: post
title: ".NET(Dotnet) application settings 관리"
date: '2024-02-23 17:35:54 +0900'
category: [C#]
tags: [application settings]
---

# 개요
Application Settings 에 프로젝트에 설정할 수 있는 세팅값들을 저장해놓고, 변경할 수 있다.

이러한 세팅값들은 우리가 어플리케이션에 대한 정보를 동적으로 저장할 수 있게 해준다. 세팅을 사용함으로써 어플리케이션의 런타임 코드에 들어가면 안되는 정보들을 클라이언트의 컴퓨터를 통해 저장할 수 있게 할 수 있다. 이런 정보들의 대표적인 예로 Connection Strings, User Preferences 등이 있다.

정보를 설정할 때 각 정보들은 다른 이름(`Name`)을 가져야 한다. 문자, 숫자, 밑줄의 조합으로 만들 수 있고 숫자로 시작하거나 공백을 포함해선 안된다.

데이터 타입또한 설정하는데, `ToString` / `FromString` 을 implement한 `TypeConverter`[^fn1] 클래스 혹은 XML 로 Serialized[^fn2] 되는 타입이면 모두 사용 가능하다.

정보에 해당하는 값(`Value`)은 데이터 타입에 맞는 값을 설정할 수 있다.

이러한 Application Settings 는 scope 를 기준으로 두 가지로 나뉘며, 프로젝트 시스템에서 두 파일로 나뉘어 저장된다.

Application-Scoped Settings
: 웹 서비스의 URL, DB Connetion String 등의 정보를 나타내는데 사용가능. 어플리케이션과 연결되어 있기 때문에, 사용자가 런타임에 수정이 불가능하다.<br>`app.config` 파일 = Design time 에 생성되며, 첫번째 application setting 을 생성할 때 만들어진다.

User-Scoped Settings
: Form 의 위치나 폰트 설정 등을 유지하는데 사용가능. 사용자가 런타임에 수정이 가능하다.<br>`user.config` 파일 = runtime 에 생성되며, 어플리케이션을 실행한 사용자가 user setting 값을 변경하는 순간 만들어진다.(단, 유저가 값을 변경해도 어플리케이션에서 저장하는 메서드가 실행이 되어야 실제 디스크에 저장된다.)

---

## Design time 에 Application Settings 만들기
방법은 2가지가 있다.

- `Project Designer`(Solution Explorer 에서 프로젝트 우클릭 - Properties 선택) 의 `Settings` 페이지를 사용한다.
- form 이나 control 에 `Properties 윈도우`(F4) 를 사용해 property 에 setting 을 바인딩할 수 있다.

Application-Scoped Settings(ex. db connetion string, 서버 리소스 reference) 를 생성하면, app.config 파일 내부의 `<appliactionSettings>` 태그에 저장된다.(Connection strings 는 `<connectionStrings>` 태그도 사용해 저장해야 한다.)

User-Scoped Settings(ex. 기본 폰트, 메인페이지, 창 크기) 를 생성하면, app.config 파일 내부의 `<userSettings>` 태그에 저장된다.

---

## runtime 에 application settings 에 접근, 수정하는 법
아래 예시처럼 `Settings` 클래스에 직접 접근해야 한다.

```cs
Properties.Settings.Default.FirstUserSetting = "abc";
```

사용자의 세팅값을 유지하기 위해서는 이 wrapper class 의 `Save` 메서드를 실행해주어야 한다. 해당 메서드는 아래의 예시 코드처럼 실행하면 된다.

```cs
Properties.Settings.Default.Save();
```

---

## 마치며
`Settings` 클래스로 appliaction settings 에 접근하고 싶다면 도큐먼트 [Manage application settings (.NET)](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/advanced/application-settings-overview?view=netframeworkdesktop-4.8){: target='_blank' } 를 참조.

---

[^fn1]: <https://learn.microsoft.com/ko-kr/dotnet/api/system.componentmodel.typeconverter?view=net-8.0>{: target='_blank' } 참조
[^fn2]: <https://learn.microsoft.com/en-us/sql/relational-databases/xml/define-the-serialization-of-xml-data?view=sql-server-ver16>{: target='_blank' } 참조