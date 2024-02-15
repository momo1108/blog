---
layout: post
title: ".NET MAUI 알아보기 3장 - Navigation, Native API"
date: '2024-02-14 15:52:04 +0900'
category: [.NET MAUI, 튜토리얼]
tags: [.net maui]
---

# 앞서서
이번 포스트에서는 다른 페이지로 넘어가는 Navigation 방법과, 플랫폼에서 제공하는 Native API 를 사용하는 법에 대해서 알아보자.

## Navigation
네비게이션을 통해 다른 페이지로 이동해보자. 페이지 이동 과정에 데이터도 전달할 수 있다.

.NET MAUI 에는 몇가지의 네비게이션 방법이 존재한다. 그중에서도 URI Shell-based Navigation 을 중점적으로 알아보자.

Shell 이란 것은 우리 어플리케이션의 구조를 담당한다. 또한 Shell은 constructor injection, dependency injection 뿐 아니라 URI-based Navigation 을 제공한다.

웹과 비슷하게 `/주소` 형태로 자세한 경로들을 설정 가능하고, 간단한 데이터 타입이나 복잡한 객체도 query parameter 도 전달할 수 있다.

새로운 페이지를 만들어보자. TodoList의 아이템을 클릭하면 DetailPage 로 이동하게 만들고 싶다. 

솔루션 익스플로러에서 프로젝트 우클릭 후 Add - New Items 에서 템플릿 중 .NET MAUI 템플릿의 XAML Content Page를 선택해 DetailPage.xaml 을 만들어보자.

이제 DetailPage 의 ViewModel을 생성한 후 mvvm toolkit에 맞게 코드를 수정해주고, DetailPage 의 code behind 에도 BindingContext 를 설정해준다.

DetailPage 와 DetailViewModel 또한 서비스에 추가해주어야 하는데, 메인페이지는 한번 글로벌로 생성해놓고 메모리에 보관하며 사용하지만 디테일페이지는 매번 새로 생성하고 싶다.

따라서 이 경우에는 `AddTransient` 를 사용해준다.

다음으론 AppShell에 기본페이지인 Main 말고 Detail 페이지를 추가하자.

먼저 code behind 에 새로운 Route 를 추가해보자.

```cs
namespace TestMauiApp
{
    public partial class AppShell : Shell
    {
        public AppShell()
        {
            InitializeComponent();
            Routing.RegisterRoute(nameof(DetailPage), typeof(DetailPage));
        }
    }
}
```

RegisterRoute 함수를 통해 페이지의 경로를 등록한다.

이제 네비게이션을 위한 이벤트 핸들링이 필요한데, 메인페이지에서 TodoList 를 디스플레이하는 CollectionView 의 내부 속성이 존재한다.(SelectionChanged, SelectionMode 등)

하지만 우리의 경우 아이템을 선택하는 개념보다는 그냥 탭을 통해 이동하는게 목적이기 때문에, CollectionView 의 Selection 은 사용하지 않도록 설정한다.

대신 Label 을 감싸는 Frame 자체에 클릭을 인식하는 built-in 기능(Frame.GestureRecognizers)을 추가해주자. 이 기능은 다양한 제스쳐를 지원한다.(Tap, Swipe, Pinch, Pan, Drop, Drag, ...)

```xml
<Frame>
    <Frame.GestureRecognizers>
        <TapGestureRecognizer Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodel:MainViewModel}}, Path=TapCommand}"
                              CommandParameter="{Binding .}"/>
    </Frame.GestureRecognizers>
    <Label Text="{Binding .}"
        FontSize="20" />
</Frame>
```
{: file='MainPage.xaml' }

이벤트핸들러는 Delete와 똑같은 방법으로 등록한다. 이제 MainViewModel 에서 라우팅을 담당할 Tap 메서드를 만들어보자.

```cs
[RelayCommand]
async Task Tap(string s)
{
    await Shell.Current.GoToAsync($"{nameof(DetailPage)}?Text={s}",
        new Dictionary<string, object>
        {
            {"hello", new object()} 
        });
}
```

라우팅이 완전히 될때까지 기다리게 하도록 async / await 를 사용하고, Shell.Current.GoToAsync 함수를 통해 실제 라우팅이 진행된다.

이때, query parameter 에 간단한 데이터는 직접 `Key=Value` 형태로 전달이 가능하고, 복잡한 객체같은 경우 두번째 파라미터에 Dictionary 형태로 전달이 가능하다.

전달한 query 값은 DetailPage 의 code behind 혹은 DetailViewModel 에서 직접 사용이 가능하다.

```cs
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace TestMauiApp.ViewModel;

[QueryProperty("Todo", "Text")]
public partial class DetailViewModel: ObservableObject
{
    [ObservableProperty]
    string todo;

    [RelayCommand]
    async Task GoBack()
    {
        await Shell.Current.GoToAsync("../");
    }
}
```

DetailViewModel 에 간단하게 `[QueryProperty("사용할이름", "전달받은이름")]` 코드 제너레이터를 사용해 전달받을 수 있다.

또한 메인으로 돌아가기 위한 방법으로 같은 함수를 사용하지만, 직접 메인 경로를 입력할 필요 없이 페이지들이 스택에 쌓여있는걸 활용할 수 있다.

`../` 는 이전 페이지, `../../` 는 2 페이지 전 이런 식이다.

마찬가지로 DetailPage 에 ViewModel 설정을 하고 바인딩을하자.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="TestMauiApp.DetailPage"
             xmlns:viewmodel="clr-namespace:TestMauiApp.ViewModel"
             x:DataType="viewmodel:DetailViewModel"
             Title="DetailPage">
    <VerticalStackLayout>
        <Label 
            Text="{Binding Todo}"
            VerticalOptions="Center" 
            HorizontalOptions="Center" />
        <Button Text="GoBack" Command="{Binding GoBackCommand}" />
    </VerticalStackLayout>
</ContentPage>
```
{: file='DetailPage.xaml' }

## Native API 기능
Native API 는 말하자면 각 플랫폼(Windows, Mac, Android, iOS)에 특화된 것들이라고 할 수 있다.

.NET MAUI 에서는 C# 안에서 Native API 에 접근이 가능하다.(iOS, Android for .NET, Windows App SDK, UI3 시스템 덕분에)

또한 UI 를 개발할 때와 마찬가지로, 공통적인 플랫폼 API(ex. Geolocation, Sensor, connectivity, etc..) 들을 개발자가 사용할 수 있도록 하나의 API로 만들어놓았다.

이제 우리의 어플리케이션에 거의 모든 어플리케이션에 흔히 사용되는 connectivity 를 결합해보자.

TodoList 에 아이템을 추가할 때, 인터넷에 연결되지 않으면 안되게 해보자.

Microsoft.Maui 네임스페이스에는 ApplicationModel, Authentication, Controls, Devices 등 다양한 기능들을 가진 namespace 들이 있다.

네트워크를 체크할 때 `Connectivity.NetworkAccess` 이렇게 클래스를 직접 사용할 수도 있는데, 강의에서는 뷰모델을 테스트가 가능하게 만들기 위해 인터페이스인 `IConnectivity` 를 필드로 사용했다.(무슨말인지 잘 모르겠네)

MainViewModel의 constructor 에 인터페이스 IConnectivity 를 추가하고 필드를 설정한다.(강의에서는 직접 constuctor를 호출하고 안에서 초기화했다. 어떻게 constuctor 가 동작해서 저게 설정되는지는 잘 모르겠다.)

```cs
public partial class MainViewModel(IConnectivity connectivity) : ObservableObject
{
    private readonly IConnectivity connectivity = connectivity;
    ...
}
```

그 후 Add 메서드에서 네트워크 연결을 체크하는 코드를 작성해보자.

```cs
[RelayCommand]
async Task Add()
{
    if (string.IsNullOrWhiteSpace(Text)) return;

    if(connectivity.NetworkAccess != NetworkAccess.Internet)
    {
        await Shell.Current.DisplayAlert("Error!", "No Internet has connected.", "Got it");
        return;
    }
        
    Items.Add(Text);
    Text = string.Empty;
}
```

`connectivity.NetworkAccess` 는 현재 네트워크 연결 상태를 나타낸다. `NetworkAccess.Internet` 는 정상 네트워크 연결시의 enum 값이며, NetworkAccess 의 enum 값들은 다음과 같다.

```cs
public enum NetworkAccess
{
	/// <summary>The state of the connectivity is not known.</summary>
	Unknown = 0,

	/// <summary>No connectivity.</summary>
	None = 1,

	/// <summary>Local network access only.</summary>
	Local = 2,

	/// <summary>Limited internet access.</summary>
	ConstrainedInternet = 3,

	/// <summary>Local and Internet access.</summary>
	Internet = 4
}
```

정상적인 네트워크 연결상태라면, Internet 이라는 enum 값을 가지는 것이다. 그렇지 않은 경우 다른 값을 가지므로 두 값을 비교해서 판단한다.

네트워크 연결을 체크하고 알람 기능을 사용하는데, 이는 async 함수이므로 Add 메서드도 이에 맞게 async 키워드와 Task 를 return 하도록 변경했다.

여기까지 동작관련 코드들은 변경이 완료됐다.

이제 `MauiProgram.cs` 에서 메인 어플리케이션에 Connectivity 서비스를 등록해야 한다.

```cs
using Microsoft.Extensions.Logging;
using TestMauiApp.ViewModel;

namespace TestMauiApp
{
    public static class MauiProgram
    {
        public static MauiApp CreateMauiApp()
        {
            // 다른 코드들...

            builder.Services.AddSingleton<IConnectivity>(Connectivity.Current);

            // 다른 코드들...
        }
    }
}
```

IConnectivity 서비스를 등록할 때 `Connectivity.Current`(Connectivity 클래스가 static 으로 사용될 때 기본 구현체를 제공해줌) 를 사용하게 한다.

이로써 서비스 등록을 마쳤고 네트워크 연결 상태에 따라 동작을 하게 만들었다. 위 코드로 크로스 플랫폼에서 동작이 가능하다.

디버깅은 내부적으로 네트워크 연결에 의존하는 부분이 있기 때문에, 테스트를 할 때는 Debugging을 사용하지 않고 실행하는게 좋다.(Ctrl + F5)

---

## Resources
- .NET MAUI 도큐먼트 : <https://learn.microsoft.com/ko-kr/dotnet/maui/?view=net-maui-8.0>{:target="_blank"}
- 좋은 .NET MAUI 라이브러리, 리소스가 정리된 깃헙(awesome-dotnet-maui) : <https://github.com/jsuarezruiz/awesome-dotnet-maui>{:target="_blank"}