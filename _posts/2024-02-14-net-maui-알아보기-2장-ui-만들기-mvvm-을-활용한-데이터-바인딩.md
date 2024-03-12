---
layout: post
title: ".NET MAUI 알아보기 2장 - UI 만들기, MVVM 을 활용한 데이터 바인딩"
date: '2024-02-14 15:47:33 +0900'
category: [.NET MAUI, 튜토리얼]
tags: [.net maui]
---

# 앞서서
이번 포스트에서는 화면을 구성하는 UI를 만드는 방법과, 화면에 출력될 내용에 데이터를 연결하는 방법(데이터 바인딩)을 알아보자.

데이터 바인딩은 외부 패키지인 MVVM을 Nuget을 활용해서 설치하여 진행할 것이다.

## UI 만들기
UI 를 만들고 각각의 다른 레이아웃과 컨트롤들을 이용해보자.

TodoList 를 만들면서 위의 내용을 수행해보자.

MainPage.xaml 을 수정해서 홈페이지를 변경해보자.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="TestMauiApp.MainPage">

    <Grid RowDefinitions="100, Auto, *"
          ColumnDefinitions=".75*, .25*"
          Padding="10"
          RowSpacing="10"
          ColumnSpacing="10">
        <Image Grid.ColumnSpan="2"
               Source="logo1.png"
               BackgroundColor="Transparent" />

        <Entry Placeholder="Enter task"
               Grid.Row="1" />
        <Button Text="Add"
                Grid.Row="1"
                Grid.Column="1" />
        <CollectionView Grid.Row="2" Grid.ColumnSpan="2">
            <CollectionView.ItemsSource>
                <x:Array Type="{x:Type x:String}">
                    <x:String>Apples</x:String>
                    <x:String>Bananas</x:String>
                    <x:String>Oranges</x:String>
                </x:Array>
            </CollectionView.ItemsSource>
            <!--단순 텍스트를 예쁘게 출력하기 위해 템플릿 생성 후 데이터바인딩-->
            <CollectionView.ItemTemplate> 
                <DataTemplate>
                    <SwipeView>
                        <SwipeView.RightItems> <!--left, top, bottom 도 가능-->
                            <SwipeItems>
                                <SwipeItem Text="Delete"
                                           BackgroundColor="Red"/>
                            </SwipeItems>
                        </SwipeView.RightItems>
                        <Grid Padding="0,5">
                            <Frame>
                                <Label Text="{Binding .}"
                                   FontSize="20" />
                            </Frame>
                        </Grid>
                    </SwipeView>
                </DataTemplate>
                
            </CollectionView.ItemTemplate>
        </CollectionView>
    </Grid>
</ContentPage>
```
{: file='MainPage.xaml' }

일단 TodoList를 출력할 세번째 행인 `CollectionView` 도 스크롤을 지원하므로 `ScrollView` 를 삭제하고 새로운 화면을 만들었다. 

전체적인 구조를 Grid로써 생성했고, 그리드의 속성을 설정했다.

- `RowDefinitions` : 행의 개수, 크기에 대한 정의
  - `숫자`는 해상도에 따라 설정되는 듯 하다.
  - `Auto` 는 필요한 만큼의 크기로만 자동으로 설정되는 듯 하다.
  - `*` 는 기본적으로 비율에 사용된다
    - `".75*, .25*"` : 75% 와 25%, `".75*, *"` : 75% 와 100%
    - 딱 하나만 사용되는 경우에는 남은 공간 전체를 채우는 역할을 하는 듯 하다.(flex: auto; 역할과 비슷한듯)
- `ColumnDefinitions` : 열의 개수, 크기에 대한 정의
- `Padding` : Grid 내부 padding
- `RowSpacing` : 행간 여백 설정
- `ColumnSpacing` : 열간 여백 설정

Grid 내부 요소로 3개의 행을 설정했는데, 첫 행은 로고 이미지, 두번째 행은 Todo Item 입력창과 추가버튼, 마지막 행은 Todo Item List 이다.

열을 2개로 설정해 놓았기 때문에, 하나의 요소로 전체 열을 채워 사용하고 싶은 경우 `Grid.ColumnSpan` 속성을 사용해 병합할 열 개수를 지정한다.

또한 행과 열 개수가 정해진 Grid의 경우 행과 열 기준 첫번째 요소 다음 부터는 몇번째 요소인지 순서를 직접 지정해줘야 한다. 그렇지않으면 모두 첫번째로 지정된다.

행과 열 순서 지정은 `Grid.Row`, `Grid.Column` 을 활용한다. 순서는 index 와 같이 0부터 시작한다.

TodoList 출력을 위해 `CollectionView` 를 사용했고, 이 역시 행 순서 지정과 열 병합을 지정한다.

`CollectionView` 에 출력에 사용할 데이터인 `ItemSource` 와 출력될 형태를 나타내는 `ItemTemplate` 을 설정한다.

`ItemTemplate` 에 `DataTemplate` 을 사용해 데이터 바인딩을 진행할 수 있게 한다.

Swipe를 통해 삭제버튼을 보이게 하기 위해서 `SwipeView` 를 사용하고, 기본적인 형태는 `Grid` 로 설정 후 스와이프 시 오른쪽에 버튼을 출력하기 위해 `SwipeView.RightItems` 를 사용한다.

여기까지 기본적인 UI를 구성했다. 튜토리얼에서는 XAML을 활용해 UI를 구성했지만, C# 을 활용해 UI를 구성하는 방법도 있다고 하니 나중에 필요해지면 알아보도록 하자.

## MVVM 과 XAML 을 활용한 Data Binding
이제 MVVM(Model-View-ViewModel)[^1]과 XAML을 활용하여 responsive, reactive 한 어플리케이션으로 만들어보자.

이 구조는 XAML로 어플리케이션을 개발할 때 많이 사용되는 구조이다.

Data Binding 을 지원해 UI와 code behind 소통함으로써 control 과 data 의 flow 를 관리할 수 있다.

`View`는 버튼, 라벨, 엔트리 등을 사용해 데이터를 어떻게 디스플레이 할까에 사용된다.

`View Model`은 완전히 decoupled 된 code behind 라고 생각하면 된다. View Model은 **무엇을 디스플레이 할지** 를 나타낸다고 보면 된다. 객체나 문자열들, 버튼이 클릭되면 발생할 이벤트, 라벨에 무엇을 출력할지 등을 포함한다.

.NET MAUI 내부의 Binding System 은 이것들을 모두 합쳐 우리의 UI 와 code behind 가 서로 업데이트 할 수 있게 해준다.

단순히 값들 뿐 아니라 이벤트 핸들러 같은 개념도 포함된다.

이제 어플리케이션에 MVVM을 적용해보자.

일단 Todo Item 입력창에 데이터 바인딩을 하기위한 방법을 살펴보자.

ViewModel 폴더를 만들고 `MainViewModel.cs` 파일을 생성했다.

```cs
using System.ComponentModel;

namespace TestMauiApp.ViewModel;

public class MainViewModel : INotifyPropertyChanged
{
    string text;
    public string Text
    {
        get => text;
        set 
        { 
            text = value;
            OnPropertyChanged(nameof(Text));
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;

    void OnPropertyChanged(string name) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

어플리케이션에 MVVM을 구현하는 전통적인 방법은 INotifyPropertyChanged 인터페이스를 상속받아서 이벤트핸들러를 구현하는 방법이다.

인터페이스의 PropertyChangedEventHandler 는 .NET MAUI 가 자동으로 참조하며, 우리는 UI 업데이트를 원하는 시점에 .NET MAUI에게 notify 할 수 있다.

커스텀 메서드에 보다시피 직접 해당 이벤트를 실행하는 방식이다.

그 다음은 데이터바인딩을 위한 Text 필드를 만들고 세터에 OnPropertyChagned 메서드를 사용하면 UI와 code behind 에서 Text를 세팅할 때 마다 PropertyChanged 알림을 통해 Entry 가 자동으로 업데이트되도록 한다.

여기까지가 기본적인 방식이나, 커뮤니티가 MS가 만든 좋은 패키지들이 있다.

Nuget을 이용해 패키지를 추가해보자.

Solution Explorer 에서 프로젝트를 우클릭하고 Manage Nuget Packages 를 선택하자.

![우클릭 스샷](/assets/img/captures/3_solutionexplorer.png)

Browse 메뉴에서 CommunityToolkit.Mvvm 을 검색하자.

강의에서는 8.0.0 버전을 사용했다.

이 패키지는 .NET 어플리케이션에 사용 가능하다. 또 훌륭한 소스코드 생성기능을 제공해준다. 설치를 완료하고 View Model 코드를 수정해보자.

```cs
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;
using System.Collections.ObjectModel;
using System.ComponentModel;

namespace TestMauiApp.ViewModel;

public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    string? text;

    [ObservableProperty]
    ObservableCollection<string> items = [];

    [RelayCommand]
    void Add()
    {
        // 입력값이 없거나 whitespace면 return 한다.
        if (string.IsNullOrWhiteSpace(Text)) return;
        
        // TodoList에 추가 후 입력값을 초기화 한다.
        Items.Add(Text);
        Text = string.Empty;
    }

    [RelayCommand]
    void Delete(string s)
    {
        if(Items.Contains(s))
        {
            Items.Remove(s);
        }
    }
}
```

string text 필드를 제외한 모든 코드를 삭제 후, CommunityToolkit.Mvvm.ComponentModel 에서 제공하는 ObservableObject를 상속받자.

ObservableObject 엔 내부적으로 이벤트 관련 기능이 포함되어 있다.

이제 입력창에 바인딩할 필드인 text 에 소스 제너레이터를 사용해보자. `[ObservableProperty]` 를 사용해 이전에 직접 지정했었던 getter 와 setter, event invoke 까지 모두 자동으로 소스코드가 생성된다.

관련 코드들은 솔루션 익스플로러에서 dependency에서 analyzer 쪽에 생성되는 걸 확인할 수 있다.

마찬가지로 Todo Item들을 저장할 필드도 생성하자. System.Collections.ObjectModel 에서 제공하는 ObservableCollection 을 사용한다.

이제 추가를 위한 이벤트 핸들러를 만들자. 마찬가지로 소스 제너레이터 `[RelayCommand]` 를 통해 우리는 기능과 관련된 코드만 작성하면 된다.

Add 와 Delete 메서드 모두 ObservableProperty 인 Items 에 변화를 일으키므로, 실행될 때 마다 UI에 변화가 있을 것이다.

이제 MainPage.xaml 에 변경된 내용에 맞게 코드를 수정해보자.

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="TestMauiApp.MainPage"
             xmlns:viewmodel="clr-namespace:TestMauiApp.ViewModel"
             x:DataType="viewmodel:MainViewModel">
```
{: file='MainPage.xaml' }

ContentPage 에 ViewModel 컨텍스트를 추가하고, x:DataType 을 통해 ContentPage 와 ViewModel 을 연결한다.

TodoList에 해당되는 CollectionView 에 데이터를 바인딩한다.

```xml
<CollectionView Grid.Row="2"
                Grid.ColumnSpan="2"
                ItemsSource="{Binding Items}">
```
{: file='MainPage.xaml' }

CollectionView.ItemsSource 을 삭제하고 대신 속성값으로 바인딩을 한다.

CollectionView.ItemsSource 내부에 설정했었던 Type 을 이제 템플릿에 사용되는 DataTemplate 에 지정해야 한다.

DataTemplate은 MainViewModel 이 아닌 string 이 bound 돼있기 때문이다.

```xml
<DataTemplate x:DataType="{x:Type x:String}">
```
{: file='MainPage.xaml' }

이제 Entry 와 Button 에 각각 text 필드와 Add 이벤트핸들러를 바인딩하자. 각각 `Text` 속성과 `Command` 속성을 사용하면 된다.

```xml
<Entry Placeholder="Enter task"
        Grid.Row="1"
        Text="{Binding Text}"/>
<Button Text="Add"
        Command="{Binding AddCommand}"
        Grid.Row="1"
        Grid.Column="1" />
```
{: file='MainPage.xaml' }

마지막으로 SwipeItem 에 Delete 이벤트핸들러를 바인딩하는데, 해당 요소가 CollectionView, DataTemplate 의 하위 요소이므로 Binding 시에 상위 요소들의 바인딩값(Items)나 타입(x:Type x:String)이 아닌 MainViewModel 의 Delete 이벤트핸들러를 바인딩하도록 명시해줘야 한다.

또 이벤트핸들러에 전달될 값을 `CommandParameter` 속성에 명시해준다.

```xml
<SwipeItem Text="Delete"
    BackgroundColor="Red"
    Command="{Binding Source={RelativeSource AncestorType={x:Type viewmodel:MainViewModel}}, Path=DeleteCommand}"
    CommandParameter="{Binding .}"/>
```
{: file='MainPage.xaml' }

이제 MainPage 의 code behind 에서 Context 를 바인딩한다.

```cs
using TestMauiApp.ViewModel;

namespace TestMauiApp
{
    public partial class MainPage : ContentPage
    {
        public MainPage(MainViewModel vm)
        {
            InitializeComponent();
            BindingContext = vm;
        }
    }
}
```

최종적으로 `MauiProgram.cs` 에 dependency service를 사용해 system을 등록하자.

```cs
using Microsoft.Extensions.Logging;
using TestMauiApp.ViewModel;

namespace TestMauiApp
{
    public static class MauiProgram
    {
        public static MauiApp CreateMauiApp()
        {
            var builder = MauiApp.CreateBuilder();
            builder
                .UseMauiApp<App>()
                .ConfigureFonts(fonts =>
                {
                    fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                    fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
                });

            builder.Services.AddSingleton<MainPage>();
            builder.Services.AddSingleton<MainViewModel>();

#if DEBUG
    		builder.Logging.AddDebug();
#endif

            return builder.Build();
        }
    }
}
```

Singleton 서비스는 한번만 생성되는 global static 같은 개념이다.

Transient 서비스는 매번 생성되는 개념이다.(나중에 사용한다)

MainPage 와 MainViewModel 을 등록했다.

이로써 MVVM이 결합된 어플리케이션을 만들었다.

---

## Resources
- .NET MAUI 도큐먼트 : <https://learn.microsoft.com/ko-kr/dotnet/maui/?view=net-maui-8.0>{:target="_blank"}
- 좋은 .NET MAUI 라이브러리, 리소스가 정리된 깃헙(awesome-dotnet-maui) : <https://github.com/jsuarezruiz/awesome-dotnet-maui>{:target="_blank"}

---

[^1]: UI와 UI가 아닌 코드를 분리하기 위한 아키텍처 디자인 패턴이다. https://learn.microsoft.com/ko-kr/windows/uwp/data-binding/data-binding-and-mvvm