---
layout: post
title: C sharp(C#) 알아보기 3장 - LINQ, OOP
date: '2024-02-14 14:56:37 +0900'
category: [C#, 튜토리얼]
tags: [c#]
---

# 앞서서
이번에는 C# 에서 제공하는 아주 독특한 기능인 LINQ 와, 객체에 대해서 알아보자.

## LINQ
LINQ 란 Language Integrated Query 의 약자인데, 대충 직역해보자면 프로그래밍 언어에 통합된 쿼리 이다.

다시말하면 C# 내부적으로 query 기능이 구현되어있다는 것이다. 자세한 내용은 [ms 도큐먼트](https://learn.microsoft.com/en-us/dotnet/csharp/linq/) 참조

```cs
// Specify the data source.
List<int> scores = [97, 92, 81, 60];

// Define the query expression.
IEnumerable<int> scoreQuery =
    from score in scores
    where score > 80
    select score;

// Execute the query.
foreach (var i in scoreQuery) {
    Console.Write(i + " ");
}
```

`scoreQuery` 자체는 정답이 아니라 query 이다. query의 실행은 `foreach` 구문에서 이루어진다.

Syntax Highlighting을 보면 알겠지만, 단순히 SQL 등의 query를 문자열로 쓰는게 아니라, 내부적으로 구현이 되어있다.

### Query Expressions
다양한 작업을 체인시켜서 한번에 사용할 수 있다. (ex. 기존의 Sort를 orderby로 체이닝)

```cs
// Specify the data source.
List<int> scores = [90, 88, 97, 92, 81, 60];

// Define the query expression.
IEnumerable<string> scoreQuery =
    from score in scores
    where score > 80
    orderby score descending
    select $"The score is {score}";

Console.WriteLine(scoreQuery.Count());

// Execute the query.
foreach (string s in scoreQuery) {
    Console.WriteLine(s);
}
```

### Method Syntax
```cs
// Specify the data source.
List<int> scores = [90, 88, 97, 92, 81, 60];

// Define the query expression.
var scoreQuery = scores.Where(s => s > 80).OrderByDescending(s => s);

// Execute the query.
foreach (int score in scoreQuery) {
    Console.WriteLine(score);
}

var result = scoreQuery.ToList();
```

## OOP
Object-oriented Programming. 객체 개념에 대해서 알아보자.

예전 버전의 C#에서는 어플리케이션의 메인 파일의 생김새가 이랬다.(older version of C#. 현재는 이런 폼을 자동 생성해주는 것)

```cs
using System;

namespace MyNamespace {
    public class MyApp {
        public static void Main() {
            Console.WriteLine("Hello");
        }
    }
}
```

namespace 와 class 명이 구분되기 위한 식별자같은 역할을 한다.

class를 선언할 때 보편적인 방법으로 멤버 필드를 설정할 수 있지만, primary constructor를 사용하면 더 깔끔한 코드를 작성할 수 있다.

**보편적인 방법**

```cs
var p1 = new Person("Hyechan", "Bang", new DateOnly(1993, 11, 8));
var p2 = new Person("Peter", "Parker", new DateOnly(1970, 5, 12));

List<Person> people = [p1, p2];

Console.WriteLine(people[0].First);

public class Person {
    public Person(string firstname, string lastname, DateOnly birthday){
        this.firstname = firstname;
        this.lastname = lastname;
        this.birthday = birthday;
    }

    private string firstname;
    private string lastname;
    private DateOnly birthday; // 새로운 타입(DateTime에서 Date만)

    public string First { get { return firstname; } }
    public string Last { get { return lastname; } }
    public DateOnly Birthday { get { return birthday; } }
}
```

**primary constructor를 사용한 방법**

```cs
var p1 = new Person("Hyechan", "Bang", new DateOnly(1993, 11, 8));
var p2 = new Person("Peter", "Parker", new DateOnly(1970, 5, 12));

List<Person> people = [p1, p2];

Console.WriteLine(people[0].First);

public class Person(string firstname, string lastname, DateOnly birthday)
{
    public string First {get;} = firstname; // 이건 한번 정하고 고정하기 위해서 get에서 return 안하고 대입한건가?
    public string Last {get;} = lastname;
    public DateOnly Birthday {get;} = birthday;
}
```

### Derived or Abstract Classes, Overrides
상속과 추상 클래스, 메서드 오버라이딩 등에 대해 알아보자. 상속의 경우 자바랑은 조금 다르다.

```cs
var p1 = new Person("Hyechan", "Bang", new DateOnly(1993, 11, 8));
var p2 = new Person("Peter", "Parker", new DateOnly(1970, 5, 12));

p1.Pets.Add(new Dog("Fred"));
p1.Pets.Add(new Dog("Barney"));

p2.Pets.Add(new Cat("Beyonce"));

List<Person> people = [p1, p2];

foreach (var person in people) {
    Console.WriteLine($"{person}");
    foreach (Pet pet in person.Pets) {
        Console.WriteLine($"{pet}");
    }
}

public class Person(string firstname, string lastname, DateOnly birthday)
{
    public string First {get {return firstname;}}
    public string Last {get {return lastname;}}
    public DateOnly Birthday {get {return birthday;}}

    public List<Pet> Pets {get;} = [];

    public override string ToString()
    {
        return $"{First} {Last}";
    }
}

public abstract class Pet(string firstname) {
    public string First {get;} = firstname;

    public abstract string MakeNoise();

    public override string ToString()
    {
        return $"└ My name is {First} and I'm a {GetType().Name}. I {MakeNoise()}!";
    }
}

// 상속 시 콜론(:) 사용 후 부모 constructor에 바로 넘겨주기 가능.(C# 12버전부터 지원)
public class Cat(string firstname) : Pet {
    public override string MakeNoise(){return "meow";}
}

public class Dog(string firstname) : Pet {
    public override string MakeNoise() => "bark";
}
```

추상 클래스와 오버라이드의 경우 자바와 동일하게 abstract, override 키워드를 붙여주면 된다.

상속의 경우는 마치 TypeScript 처럼 부모 클래스를 콜론(`:`)으로 지정해주고 바로 primary constructor 를 활용해 부모 constructor 에 전달이 가능하다. 코드가 굉장히 간단해졌다.(C# 12 버전부터 지원)

## 마치며
C# 에 대해서 굉장히 간단하게 알아보았다. 자바와 굉장히 비슷한 면이 많은 것 같다.