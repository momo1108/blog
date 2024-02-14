---
layout: post
title: ASP.NET 알아보기 2장 - MVC, Blazor
date: '2024-02-14 15:32:19 +0900'
category: [ASP.NET, 튜토리얼]
tags: [asp.net]
---

# 앞서서
이번 포스트에서는 실제로 ASP.NET 에서 MVC 구조를 어떻게 사용하는지, Blazor 가 무엇인지에 대해서 간단하게 알아보자.

이번 포스트에서 참조한 .NET 의 공식 샘플 프로젝트 링크도 있으니 참조하자.(https://github.com/dotnet-presentations/ContosoCrafts)

## MVC 란?
관심사를 분리하는 측면의 용어이다.

Model 을 사용해 DB와 소통을 한다. View 는 따로 아는 정보 없이 그저 정보나 데이터를 출력하는 역할만 한다. Controller 는 DB와 View 사이의 일종의 조율자의 역할을 한다.

웹 프로젝트 루트폴더에 Models, Pages, Controllers 폴더를 생성하고 각자 역할에 맞는 코드를 작성한다. 그 외에 애플리케이션 내에서 사용할 커스텀 기능들을 Services 폴더에 작성했다.

먼저 샘플 프로젝트는 products.json 파일 데이터를 DB 대용으로 사용하고 있으며 파일 내의 Ratings 라는 항목에 사용자와의 상호작용을 통해 평점을 추가하는 것이 목표이다.

인덱스 페이지에서 해당 파일의 내용들을 출력해주고, 각 항목의 버튼을 클릭 시 상세 조회와 별점 기능을 제공하는 모달을 띄우도록 만든다.

여기서 Models 폴더에는 products.json 이 가지고 있는 데이터의 구조를 코드로 작성해줘야 하고,

Controllers 폴더에서는 원래 서버와 api 통신을 통해 데이터를 불러오고 수정해야 하지만 로컬 환경이기 때문에 바로 Services에 작성한 코드를 사용하고있다.

Pages(Views) 폴더에서는 url 경로에 따른 뷰를 페이지 단위로 작성해놓았다.

ASP.NET Core 템플릿을 사용해서 프로젝트를 생성하면 View에 해당하는 Pages 폴더는 자동 생성된다. Model이나 Controller는 우리의 필요에 따라 생성하면 된다.

### Model
먼저 우리가 사용할 products.json 파일의 데이터를 ASP.NET Core 프로젝트에서 불러와서 사용하기 위해 해당 데이터를 불러올 Model을 정의해보자.

각 항목은 product가 되는셈이니 `Product.cs` 로 네이밍을 해서 Models 폴더에 생성해보자.

필드 정의는 간단하게 getter와 setter를 가지는 public 필드로 정의했다. 이 부분은 우리 마음대로 설정하면 될 듯 하다.

```cs
using System.Text.Json;
using System.Text.Json.Serialization;

namespace TestProject.WebSite.Models
{
    public class Product
    {
        public string Id { get; set; }
        public string Maker { get; set; }

        [JsonPropertyName("img")]
        public string Image { get; set; }
        public string Url { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        public int[]? Ratings { get; set; }

        public override string ToString() => JsonSerializer.Serialize<Product>(this);
    }
}
```

Java 강의를 들은적이 있어서 그런지 뭔가 굉장히 자바랑 비슷하다.

실제 데이터와 다른 필드명을 매칭해주는 방식도 그렇고, 인스턴스 출력을 위해 오버라이드하는 ToString 메서드가 굉장히 익숙하다.

`[JsonPropertyName("img")]` 코드는 웹 프로젝트 내부에서 사용하는 Image라는 필드명과 실제 데이터에 저장된 img 필드명을 매칭하기 위해서 사용한다.

ToString 메서드에서 사용하는 `JsonSerializer.Serialize<Product>(this);` 코드는 파라미터에 들어온 값을 JSON string으로 변환해주는 역할을 한다.

뭔가 아쉬운 점 한가지는 기능을 import(using) 할 때 왜 하위항목을 따로 불러와야 하는지가 의문이다.

이렇게 모델을 정의해보았다. 물론 실제 데이터를 읽어서 이 Model 구조로 사용하기 위해서는 json 파일을 읽어와서 저장하는 코드가 따로 필요하다.

이와 관련된 코드는 Services 폴더에 코드로 작성할 것이다.

### Service
에디터에서 Services 폴더에 Add - Class 를 통해 파일(`JsonFileProductService.cs`)을 생성하고 코드를 작성했다.

```cs
using System.Text.Json;
using TestProject.WebSite.Models;

namespace TestProject.WebSite.Services
{
    public class JsonFileProductService(IWebHostEnvironment webHostEnvironment)
    {
        public IWebHostEnvironment WebHostEnvironment { get; } = webHostEnvironment;

        private string JsonFileName { get { return Path.Combine(WebHostEnvironment.WebRootPath, "data", "products.json"); } }

        public IEnumerable<Product> GetProducts()
        {
            using(var jsonFileReader = File.OpenText(JsonFileName))
            {
                return JsonSerializer.Deserialize<Product[]>(jsonFileReader.ReadToEnd(), 
                    new JsonSerializerOptions
                    {
                        PropertyNameCaseInsensitive = true
                    }
                );
            }
        }

        public void AddRating(string productId, int rating)
        {
            var products = GetProducts();

            // LINQ
            var query = products.First(p => p.Id == productId);

            if(query.Ratings == null)
            {
                query.Ratings = new int[] { rating };
            } else
            {
                query.Ratings = [.. query.Ratings, rating];
            }

            using(var outputStream = File.OpenWrite(JsonFileName))
            {
                JsonSerializer.Serialize(
                    new Utf8JsonWriter(outputStream, new JsonWriterOptions
                    {
                        SkipValidation = true,
                        Indented = true,
                    }),
                    products
                );
            }
        }
    }
}
```

IWebHostEnvironment 는 애플리케이션이 실행되고있는 웹 호스팅 환경의 정보를 제공해주는 인터페이스이다.

이를 이용해 불러올 데이터 파일이 있는 WebRootPath 를 사용할 수 있고, `using(var jsonFileReader = File.OpenText(경로))` 코드를 통해 파일을 읽어올 수 있다.

문자 데이터를 읽어와서 스트림으로 만들고, 이후 ReadToEnd 메서드를 통해 string으로 변환한 후에 JsonSerializer.Deserialize 메서드를 통해 json string 데이터를 `IEnumerable<Product>` 형태로 변환해 return 한다.

여기서 generic type에 사용된 Product 는 위 Models에 작성한 Product.cs에 있는 클래스를 불러온 것이다.

이후 AddRating 메서드는 사용자가 선택한 Item을 불러온 데이터에서 찾아내(LINQ 활용) 수정 후 다시 products.json 파일에 덮어쓰기 한다.

이로써 서버 내부적으로 필요한 서비스는 모두 작성했다. 작성한 서비스는 Program.cs 에서 `builder.Services.AddTransient<JsonFileProductService>();` 코드를 통해 추가할 수 있다.

추가해주지 않으면 다른 곳에서 코드를 작성할 때 서비스 코드를 찾지 못한다.

### View
현재 프로젝트에 존재하는 Index 페이지에 데이터를 가져와서 View를 업데이트 할 것이다.

이를 위해서는 먼저 페이지의 코드를 살펴보자. 대충 구성은 다음과 비슷하다.

```cshtml
@page
@model IndexModel
@{
    ViewData["Title"] = "Indexx page";
}

<div class="text-center">
    <h1 class="display-4">Test Project2</h1>
    <p>Learn about <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>
```

파일의 확장자가 cshtml 이고 코드를 보면 html과 csharp 코드가 같이 사용되고 있다.

이를 Razor View 파일이라고 부르는데, html에 csharp 코드를 사용하기 위해 `@`(at) 기호를 사용한다.

최상단을 보면 여러가지 디렉티브들이 사용되고 있다. 그 중 `@model` 디렉티브를 보면 `IndexModel` 을 불러오고 있는데, 이는 Index.cshtml과 같은 경로에 위치하는 Index.cshtml.cs 파일에 존재하는 class 이다.

Index.cshtml.cs 파일같이 페이지 뒤의 코드를 페이지 모델이라 부르는데, 여기에 페이지에 출력될 데이터가 어떤건지 정의한다.

인덱스 페이지 모델의 초기 구성은 다음과 같다.

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace TestProject.WebSite.Pages
{
    public class IndexModel(ILogger<IndexModel> logger) : PageModel
    {

    }

    public void onGet()
    {
    
    }
}
```

onGet 메서드가 처음부터 있었는지 기억이 안난다 ㅡ,.ㅡ

어쨌든 코드를 보면 IndexModel이 PageModel을 상속받는걸 볼 수 있다.

또 constructor의 인자로서 단순히 ILogger 를 넘겨주는 것 만으로도 로거 기능을 불러와 Azure, Cloud 등에서 사용될 수 있다.

따로 만든 서비스들도 불러와서 인자로 넣어주면 사용가능하다.

onGet 메서드에는 페이지가 열릴 때 실행될 코드를 넣어주면 된다.

Services 에 구현한 기능들을 불러와 사용한 코드는 다음과 같다.

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using TestProject.WebSite.Models;
using TestProject.WebSite.Services;

namespace TestProject.WebSite.Pages
{
    public class IndexModel(ILogger<IndexModel> logger, JsonFileProductService productService) : PageModel
    {
        public JsonFileProductService ProductService = productService;
        public IEnumerable<Product> Products { get; private set; }

        public void OnGet()
        {
            Products = ProductService.GetProducts();
        }
    }
}
```

이제 Razor View 파일에서 PageModel 에 정의된 Products 필드를 사용할 수 있다.

이를 통해 인덱스 페이지에 출력할 코드를 작성해보자.

```cshtml
@page
@using TestProject.WebSite.Components
@model IndexModel
@{
    ViewData["Title"] = "Indexx page";
}

<div class="text-center">
    <h1 class="display-4">Test Project2</h1>
    <p>Learn about <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

<div class="row row-cols-3 gy-5">
    @foreach (var product in Model.Products)
    {
        <div class="card">
            <div class="card-img" style="background-image: url('@product.Image');"></div>
            <div class="card-body">
                <h5 class="card-title">@product.Title</h5>
            </div>
            <div class="card-footer">
                <small class="text-muted">
                    <button @onclick="(e => SelectProduct(product.Id))"
                        data-bs-toggle="modal"
                        data-bs-target="#productModal"
                        class="btn btn-primary" >
                        More Info</button>
                </small>
            </div>
        </div>
    }
</div>
```

템플릿 내부적으로 bootstrap이 사용되고 있다. Bootstrap의 CSS class들을 이용해 3줄로 구성된 상품 목록 코드를 작성하였다.

강의에서 사용하던 개발 템플릿은 4년전 템플릿이다. 현재 템플릿은 많이 업데이트 됐는데, 템플릿에서 사용하는 bootstrap의 버전이 5 버전으로 넘어가면서 데이터속성의 네이밍에 prefix가 추가됐다.

기존의 `data-toggle`, `data-target`, `data-dismiss` 등의 네이밍에서 `data-bs-toggle`, `data-bs-target`, `data-bs-dismiss` 이런식으로 bootstrap에서 사용하는 데이터 속성이라는 걸 구분짓기 위해 bs(bootstrap)이라는 prefix를 추가한 듯 하다.

코드를 보면 상품의 정보를 출력하는데 `@foreach` 를 사용하였다. 여기에 IndexModel 에서 정의한 Products 를 사용해서 각 상품에 동일한 코드를 적용하였다.

완성된 Razor View 코드를 실제 url 경로와 매핑하기 위해서는 Program.cs 내부에 `builder.Services.AddRazorPages();` 메서드와 `app.MapRazorPages();` 메서드를 사용해주면 된다.(이 두 코드는 기본적으로 포함되어 있다)

### Controller
이번 샘플 프로젝트에서는 Controller를 활용하기보단 Service를 바로 호출하는 방식을 사용했다. 하지만 사용 방법은 알아둬야 하므로 정리해보자.

보통 Controller에서는 API를 구축해서 요청이 오면 해당 로직을 실행하는 코드를 작성한다.

ASP.NET Core의 프로젝트 애플리케이션에는 `Program.cs` 라는 파일이 존재한다. 여기에는 애플리케이션 시작 코드가 포함된다.

`Program.cs`는 다음과 같이 구성된다.

- 앱에서 요구하는 서비스가 구성됩니다.
- 앱의 요청 처리 파이프라인이 일련의 미들웨어 구성 요소로 정의됩니다.

구현에는 간단한 방법과 정석적인 방법이 있다.

먼저 간단한 방법을 알아보자.

`app.MapGet`, `app.MapPost` 등의 메서드를 활용하는 방법이다.

예를 들어 다음의 코드를 살펴보자.

```cs
app.MapGet("/products", (context) =>
{
    var products = app.Services.GetService<JsonFileProductService>().GetProducts();
    var json = JsonSerializer.Serialize<IEnumerable<Product>>(products);
    return context.Response.WriteAsync(json);
});
```

위 코드는 /products 경로에 Get 요청을 매핑하는 코드이다. 보다시피 사용 방식 자체는 간단하지만, 시작 코드를 정의하는 `Program.cs` 에 Controller 코드를 포함시키는 것 자체가 바람직하지 않다.

요청이 많아질수록 더 시작코드가 더러워지기도 한다.

따라서 정석적인 방법을 알아보자.

Controllers 폴더를 생성 후 여기에 Controller 코드를 작성하는 방식이다.

상품에 따른 상호작용이 필요하므로 `ProductsController.cs` 라고 네이밍을 하고 생성해보자.

Visual Studio에서는 새로운 파일을 Add 할 때 기본 템플릿들을 제공하주는데, Controller 템플릿을 선택하면 빠르게 구성할 수 있다.

강의에서 선택한 템플릿은 API - API Controller(Empty) 템플릿이다. 제공되는 코드는 다음과 같다.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace TestProject.WebSite.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ProductsController : ControllerBase
    {
    }
}
```

보다시피 Controller 클래스는 ControllerBase 클래스를 상속받아서 기본적인 기능들을 사용한다.

대괄호(`[]`) 로 묶인 코드들은 Controller가 매핑될 경로와 ApiController 임을 명시한다.

기본적인 경로는 api/ 라는 prefix가 붙는다. 이는 사용자 임의로 삭제나 변경해도 된다. 경로 내부에 대괄호(`[]`)로 감싸진 controller 라는 토큰은 파일명(클래스명?)이 대체하는 것 같다.

따라서 위 샘플 코드의 경우 `서버주소/api/produdcts` 경로로 API 요청을 받는 코드가 된다.

샘플 프로젝트에서 작성한 코드를 살펴보자.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using TestProject.WebSite.Models;
using TestProject.WebSite.Services;

namespace TestProject.WebSite.Controllers
{
    [Route("[controller]")] // 파일이름으로 기본설정 되는듯.
    [ApiController]
    public class ProductsController(JsonFileProductService productService) : ControllerBase // ControllerBase 자식 클래스
    {
        public JsonFileProductService ProductService { get; } = productService;

        [HttpGet]
        public IEnumerable<Product> Get()
        {
            return ProductService.GetProducts();
        }

        [Route("Rate")]
        [HttpGet]
        public ActionResult Get(
            [FromQuery] string productId, 
            [FromQuery] int rating
        )
        {
            ProductService.AddRating(productId, rating);
            return Ok();
        }
    }
}
```

`[HttpGet]` 을 사용해 Get 요청을 매핑할 수 있고, 반환할 데이터를 return 하면 된다.

클래스 내부에도 `[Route("경로")]` 를 다시 사용해 SubRoute를 설정할 수도 있다. 따로 반환할 내용이 없으면 ActionResult 를 return type으로 지정하고 `Ok` 메서드를 사용하면 된다.

또한 Get 메서드의 매개변수에 `[FromQuery]` 를 사용해 QueryString 에서 값을 뽑아 사용할 수 있다.

`[HttpGet]` 말고도 `[HttpPost]`, `[HttpPut]` 등 다양한 요청을 매핑할 수 있고, `[FromBody]` 를 사용해 body의 내용을 사용할 수도 있다.

이렇게 Conroller 코드를 작성 후 Program.cs 에 `builder.Services.AddControllers();` 메서드와 `app.MapControllers();` 메서드를 사용해주면 된다.

## Blazor

새로운 application 모델.

- Blazor 를 전체 apllication 로서 사용해도 되고, 페이지 내부에서만 사용해도 된다. 사용자 맘대로
- 재사용 가능한 컴포넌트 기능 제공
- Visual Studio 에서 Razor Component 템플릿 제공

4년전 강의에서는 Razor Component를 사용하는데 코드가 마음에 들지 않았다.

아래의 코드를 살펴보자.

```razor
@page
@using TestProject.WebSite.Components
@model IndexModel
@{
    ViewData["Title"] = "Indexx page";
}

<div class="text-center">
    <h1 class="display-4">Test Project2</h1>
    <p>Learn about <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

@(await Html.RenderComponentAsync<ProductList>(RenderMode.ServerPrerendered))

<script src="_framework/blazor.server.js"></script>
```

컴포넌트를 사용하기 위해 `@using` 디렉티브를 사용했다. 문제는 맨 밑의 컴포넌트 렌더링 코드이다.

사용하는 함수와 클래스도 난해하고 뭘 하는건지 보기가 힘들다.

4년전 강의에서 사용한 코드이다 보니, 현재는 좀 더 나아지지 않았을까 MS의 도큐먼트를 찾아봤다.

도큐먼트의 코드는 조금 더 간단하긴 했다.

```cshtml
@page
@using TestProject.WebSite.Components
@model IndexModel
@{
    ViewData["Title"] = "Indexx page";
}

<div class="text-center">
    <h1 class="display-4">Test Project2</h1>
    <p>Learn about <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>

<component type="typeof(ProductList)" render-mode="ServerPrerendered" />

<script src="_framework/blazor.server.js"></script>
```

일단 컴포넌트를 렌더링하는 코드가 훨씬 가독성이 좋아졌다.

~~그리고 아래의 스크립트를 불러오는 코드를 제거해보았는데 일단 동작에는 문제가 없어서 빼두었다. 문제가 생기면 다시 넣자.~~(모달이 안열려....)

`@for` 디렉티브 사용 시 주의사항

```cshtml
@for(int star = 1; star < 6; star++)
{
    if (star <= selectedRating)
    {
        <span class="fa-star fa checked" @onmouseover="(e=>SetSelectedRating(star))" @onclick="(e=>SubmitRating(star))" />
    } else
    {
        <span class="fa-star fa" @onmouseover="(e=>SetSelectedRating(star))" @onclick="(e=>SubmitRating(star))" />
    }
}
```

위 코드는 별점 시스템을 위한 코드블럭이었다. 마우스를 올리면 내가 선택한 별점이 세터 메서드를 통해 변경되고, 클릭하면 해당 별점을 업데이트하는 형태이다.

결과만 얘기하면 제대로 동작하지 않았다. 왜 그랬을까? 디버깅을 통해 값을 살펴보니 별점을 1점부터 5점까지 세팅되기를 바랬지만, **어떤 별점을 선택하든 이벤트 핸들러 메서드에 들어온 star 값은 전부 6이었다.**

이는 `@for` 디렉티브를 통한 렌더링된 코드에서 star 변수가 모두 for loop가 끝난 시점의 star를 참조하고 있기 때문이다.

Razor Page가 렌더링 되는 과정이 뭔지는 정확히 알아봐야 겠지만, 적어도 디버깅을 통해 살펴본 결과는 그렇다.

그냥 코드를 짜다보니 사람의 시점에서는 `@for` 디렉티브를 사용해 렌더링되는 하나 하나의 span 태그가 각각 해당 시점의 star 값을 사용하여 렌더링이 될거라고 생각했지만, 서버 입장에서는 모두 같은 star 변수를 참조하며 for loop 가 완료된 후의 값으로 렌더링을 완료한것이다.

이를 해결하기 위해서는 반복되는 코드블럭 내부에서 지역변수를 따로 생성하여 거기에 해당 시점의 star 값을 저장해놓고 사용하는 것이다.

아래의 코드를 살펴보자.

```cshtml
@for(int star = 1; star < 6; star++)
{
    var currentStar = star;
    if (star <= selectedRating)
    {
        <span class="fa-star fa checked" @onmouseover="(e=>SetSelectedRating(currentStar))" @onclick="(e=>SubmitRating(currentStar))" />
    } else
    {
        <span class="fa-star fa" @onmouseover="(e=>SetSelectedRating(currentStar))" @onclick="(e=>SubmitRating(currentStar))" />
    }
}
```

달라진 것은 딱 하나 star 의 값을 currentStar 라는 변수에 저장한 것이다. 이 변수는 지역변수이고, for loop가 진행됨에 따라 각각 새로 초기화 되는 변수이기 때문에 원하는 대로 동작을 이끌어낼 수 있었다.

이외에도 컴포넌트에 리액트처럼 Property를 사용할 수 있나 찾아봤는데 Parameter라는 개념이 있더라. 근데 동작이 내맘대로 안돼서 좀 더 알아봐야할듯.

라이프사이클도 알아야 할 듯 하다.(https://learn.microsoft.com/en-us/aspnet/core/blazor/components/lifecycle?view=aspnetcore-8.0#lifecycle-events)

## 배포
추후 정리할 예정..