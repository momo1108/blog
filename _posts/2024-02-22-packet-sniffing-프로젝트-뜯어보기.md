---
layout: post
title: packet sniffing 프로젝트 뜯어보기
date: '2024-02-22 15:35:31 +0900'
category: [자기계발]
---

# 앞서서
이 포스트는 내가 원하는 기능의 앱을 개발하기 위한 기본 과정을 참조할 수 있는 깃헙 프로젝트를 학습 목적으로 직접 분석해보는 포스트입니다.

실제로 이 프로젝트를 사용하는게 아닙니다.

![감사콩](/assets/img/emoji/감사콩.png){: width='100' .normal }

---

## 프로젝트의 구조
이 프로젝트는 따져보자면 Windows Service 템플릿의 프로젝트이다. 앱의 설정을 담당하는 `App.config` 와 엔트리포인트를 담당하는 `Program.cs` 두 개의 베이스코드를 제외하면 모두 직접 생성한 서비스들이다.

---

## Program.cs
엔트리 포인트인 메인 메서드가 있는 Program.cs 파일에 스태틱 변수나 메서드들이 많이 있다.

네임스페이스 내의 전체 코드는 아래와 같다.

```cs
internal static class Program
{
    public static bool IsConsole = Console.OpenStandardInput(1) != Stream.Null;
    [STAThread]
    static void Main(string[] args)
    {
        Properties.Settings.Default.Providers.Clear();
        Bluegrams.Application.PortableSettingsProvider.SettingsFileName = AppDomain.CurrentDomain.FriendlyName + ".ini";
        Bluegrams.Application.PortableSettingsProvider.ApplyProvider(Properties.Settings.Default);
        if (!AdminRelauncher()) return;
        VersionCompatibility();
        if (!IsConsole) Warning();
        AttemptFirewallPrompt();
        if (!IsConsole)
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new MainWindow());
        }
        else
        {
            var httpBridge = new HttpBridge();
            httpBridge.args = args;
            httpBridge.Start();
        }
        if (File.Exists(Utilities.Logger.fileName) && Properties.Settings.Default.AutoUpload)
        {
            var fileBytes = File.ReadAllBytes(Utilities.Logger.fileName);
            var fileText = File.ReadAllText(Utilities.Logger.fileName);
            if (fileBytes.Length > 100 && fileText.Contains("8|"))
                Utilities.Uploader.UploadLog(fileBytes);
        }
    }
    static void AttemptFirewallPrompt()
    {
        var ipAddress = Dns.GetHostEntry(Dns.GetHostName()).AddressList[0];
        var ipLocalEndPoint = new IPEndPoint(ipAddress, 12345);
        var t = new TcpListener(ipLocalEndPoint);
        t.Start();
        t.Stop();
    }
    static void VersionCompatibility()
    {
        (var region, var installedVersion) = VersionCheck.GetLostArkVersion();
        if (installedVersion == null)
        {
            MessageBox.Show("Launch Lost Ark before launching logger", "Lost Ark Not Running", MessageBoxButtons.OK, MessageBoxIcon.Exclamation);
            Environment.Exit(0);
        }
        else if (region == Region.Unknown)
            MessageBox.Show("DPS Meter is out of date.\nDPS Meter might not work until updated.\nCheck Discord/Github for more info.\nFeel free to add a message in the discord informing shalzuth that it's out of data", "Out of date!", MessageBoxButtons.OK, MessageBoxIcon.Warning);
        else if (Properties.Settings.Default.Region != region)
        {
            Properties.Settings.Default.Region = region;
            Properties.Settings.Default.Save();
        }
    }
    static void Warning()
    {
        if (Properties.Settings.Default.IgnoreWarning) return;
        if (AppDomain.CurrentDomain.FriendlyName.Contains("LostArk"))
        {
            //var tempName = "LostArkDps" + Guid.NewGuid().ToString().Substring(0, 6) + ".exe";
            var tempName = "DpsMeter.exe";
            MessageBox.Show("LostArkLogger.exe is flagged.\nRenaming to " + tempName + " !", "Error!", MessageBoxButtons.OK, MessageBoxIcon.Error);
            File.Copy(AppDomain.CurrentDomain.FriendlyName, tempName);
            Process.Start(tempName);
            Environment.Exit(0);
        }
        else
        {
            Process.GetProcessesByName("LostArkLogger").ToList().ForEach(p => p.Kill());
            if (File.Exists("LostArkLogger.exe")) File.Delete("LostArkLogger.exe");
        }
        var res = MessageBox.Show("The game director has instructed Amazon Game Studios to ban users using a DPS Meter.\n\nAt this time, please refrain from using the DPS Meter.\n\nSelect \"Retry\" to voice your feedback, as this is not a hack nor a solution that should violate TOS", "Warning!", MessageBoxButtons.AbortRetryIgnore, MessageBoxIcon.Warning);
        if (res == DialogResult.Abort) Environment.Exit(0);
        else if (res == DialogResult.Retry)
        {
            Process.Start("https://forums.playlostark.com/t/talk-to-us-already-about-the-dps-meter/370558");
            Environment.Exit(0);
        }
        Properties.Settings.Default.IgnoreWarning = true;
        Properties.Settings.Default.Save();
    }
    private static bool AdminRelauncher()
    {
        if (!new WindowsPrincipal(WindowsIdentity.GetCurrent()).IsInRole(WindowsBuiltInRole.Administrator))
        {
            var startInfo = new ProcessStartInfo
            {
                UseShellExecute = true,
                WorkingDirectory = Environment.CurrentDirectory,
                FileName = Assembly.GetEntryAssembly().CodeBase.Replace(".dll", ".exe"),
                Verb = "runas"
            };
            try { Process.Start(startInfo); }
            catch (Exception ex) { MessageBox.Show("This program must be run as an administrator!", "Error!", MessageBoxButtons.OK, MessageBoxIcon.Error); }
            return false;
        }
        return true;
    }
}
```

---

가장 먼저나온 구문이다.

```cs
public static bool IsConsole = Console.OpenStandardInput(1) != Stream.Null;

[STAThread]
static void Main(string[] args)
{
    ...
}
```

IsConsole 변수는 단순히 bool 값이고, 변수에 해당하는 값은 `Console.OpenStandardInput(1)` 로 받은 1바이트 크기의 인풋 스트림이 Null 이 아닌지의 여부이다.

디버그 모드에서의 IsConsole 결과값은 True 였다.

메인 메서드에는 STAThread 속성이 사용되었다. 툴팁의 설명은 아래와 같다.

*어플리케이션의 COM 쓰레딩 모델이 STA(Single-Threaded Apartment) 임을 명시하는 속성*

여기서 말하는 COM 이란 무엇일까? MS 도큐먼트에는 다음과 같이 정의되어있다.

> COM is a platform-independent, distributed, object-oriented system for creating binary software components that can interact. COM is the foundation technology for Microsoft's OLE (compound documents) and ActiveX (Internet-enabled components) technologies.
{: .prompt-info }

자세한 내용은 해당 [포스트(COM 이란 무엇인가)](/posts/com-이란-무엇인가/){: target='_blank' } 에 정리해놓겠다.

---

다음은 메인 메서드이다.

```cs
Properties.Settings.Default.Providers.Clear();
Bluegrams.Application.PortableSettingsProvider.SettingsFileName = AppDomain.CurrentDomain.FriendlyName + ".ini";
Bluegrams.Application.PortableSettingsProvider.ApplyProvider(Properties.Settings.Default);
```

`Properties.Settings.Default` 은 **Application Settings** 와 관련된 코드이다. **Application Settings** 의 자세한 내용은 [.NET(Dotnet) application settings 관리](/posts/net-dotnet-application-settings-관리/){: target='_blank' } 포스트를 확인하자.

정확힌 모르겠다. `Properties.Settings.Default.Providers` 는 wrapper 클래스에서 사용된 application settings providers 의 collection 을 반환하며, `Clear` 메서드는 해당 collection 의 모든 item 들을 삭제한다.

그 다음 [Bluegrams](https://bluegrams.com/){: target='_blank' } 는 개발을 위한 오픈소스 소프트웨어 툴을 만드는 곳인데, 그 중에 `PortableSettingsProvider` 는 .NET 프레임워크의 application settings 구조 내부의 `SettingsProvider` 클래스를 커스텀해 built-in Settings Provider 와 비슷하게 만든 implementation 이다. App Settings 를 XML 포맷으로 저장할 수 있다.

깃헙에서 소개하는 기능은 아래와 같다.

- Make application settings portable together with the app
- Configure the location and name of the settings file
- Easily plug into existing apps (just one line needed to make existing app settings portable)

기본적으로 .NET 에서 제공하는 Applcation Settings 값들을 그대로 적용해 저장할 파일 이름까지 설정하는 코드인 것 같다.

참고로 Application Settings 관련 설정된 User-Scoped Settings 값들은 Project Designer 또는 App.config 에서 확인 가능하다.

![project designer - settings](/assets/img/captures/1_project-designer.png){: width='600' .normal }

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
 <!-- 다른 설정 내용 생략 -->
 <userSettings>
  <TestProject.Properties.Settings>
   <setting name="IgnoreWarning" serializeAs="String">
    <value>False</value>
   </setting>
   <setting name="Region" serializeAs="String">
    <value>Steam</value>
   </setting>
   <setting name="Npcap" serializeAs="String">
    <value>False</value>
   </setting>
   <setting name="AutoUpload" serializeAs="String">
    <value>False</value>
   </setting>
   <setting name="DisplayNames" serializeAs="String">
    <value>True</value>
   </setting>
  </TestProject.Properties.Settings>
 </userSettings>
</configuration>
```

---

다음 코드를 살펴보자.

```cs
static void Main(string[] args)
{
    // 생략
    if (!AdminRelauncher()) return;
    // 생략
}

private static bool AdminRelauncher()
{
    if (!new WindowsPrincipal(WindowsIdentity.GetCurrent()).IsInRole(WindowsBuiltInRole.Administrator))
    {
        var startInfo = new ProcessStartInfo
        {
            UseShellExecute = true,
            WorkingDirectory = Environment.CurrentDirectory,
            FileName = Assembly.GetEntryAssembly().CodeBase.Replace(".dll", ".exe"),
            Verb = "runas"
        };
        try { Process.Start(startInfo); }
        catch (Exception ex) { MessageBox.Show("This program must be run as an administrator!", "Error!", MessageBoxButtons.OK, MessageBoxIcon.Error); }
        return false;
    }
    return true;
}
```

`WindowsPrincipal` 은 윈도우의 group membership 을 확인할 수 있게 해주는 클래스이다. 해당 클래스의 `IsInRole(WindowsBuiltInRole.역할)` 메서드를 사용해 현재 프로그램이 해당 역할로 실행되었는지 확인할 수 있다. 위 코드의 경우 특정 `WindowsIdentity` 객체를 통해 초기화 하는 방법을 사용했다. `WindowsIdentity` 객체는 윈도우 유저를 나타내며, `GetCurrent` 메서드를 사용해 현재 윈도우 유저의 `WindwosIdentity` 객체를 반환받을 수 있다.

.NET 에서의 Principal, Identity 객체에 대한 정보는 해당 도큐먼트[(Principal and Identity Objects)](https://learn.microsoft.com/en-us/dotnet/standard/security/principal-and-identity-objects){: target='_blank' } 를 확인하자.

admin 권한이 아닌 경우, if문 안의 내용이 실행된다. `Process.Start` 메서드를 이용해 외부 응용 프로그램을 실행하는데, `ProcessStartInfo` 를 설정하여 실행할 프로세스를 지정하는 코드이다.

---

잠깐 `ProcessStartInfo` 에 사용가능한 속성들을 살펴보자.

|Properties|Type|Definition|
|:--|:--|:--|
|UseShellExecute|bool|프로세스를 시작하기 위해 OS Shell 을 사용할 것인가|
|WorkingDirectory|string|`UseShellExecute` 값이 true 인 경우 실행할 프로세스의 실행파일이 포함된 경로를 뜻한다.<br>`UseShellExecute` 값이 false 인 경우 `WorkingDirectory` 속성은 실행파일을 찾는데 사용되는게 아니다. 실행되는 프로세스의 context 내부에서 값 자체로서의 의미를 가진다.|
|FileName|string|시작할 어플리케이션의 이름 혹은 기본 실행 앱이 설정되어있는 파일 타입의 문서 이름|
|Verb|string|FileName 에서 명시한 어플리케이션이나 문서를 열 때 사용할 Verb(action)|


> **Verb 란?**
>
> 각각의 파일 확장자는 자기만의 verb 들이 있다. 이는 `Verbs` 속성을 통해 알 수 있으며, `ProcessStartInfo` 객체를 초기화한 뒤 `인스턴스.Verbs` 에서 foreach 문을 통해 하나씩 확인해볼 수 있다. 그 외의 Verb 는 사용할 수 없다.
>
> 예를 들면, "`print`" verb 는 `FileName` 에 명시된 문서를 출력할 것이다. verb 의 예시로는 "Edit", "Open", "OpenAsReadOnly", "Print", and "Printto" 등이 있다. `Verb` 를 사용할 경우, `FileName` 속성에 꼭 **파일 확장자를 같이 써주어야 한다.**
{: .prompt-info }

추가적인 속성들은 [도큐먼트](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.processstartinfo.filename?view=net-8.0){: target='_blank' } 를 확인하자.

---

다시 돌아와서, 코드를 살펴보자.

```cs
UseShellExecute = true,
WorkingDirectory = Environment.CurrentDirectory,
FileName = Assembly.GetEntryAssembly().CodeBase.Replace(".dll", ".exe"),
Verb = "runas"
```

shell 을 사용하도록 설정하고, 실행파일의 경로는 현재 실행된 경로로 지정한다. `Assembly.GetEntryAssembly` 메서드로 기본 어플리케이션 도메인의 프로세스 실행 파일을 알아내고, `CodeBase` 속성으로 원본 어셈블리 경로를 받아낸다. 그 후 `.dll` 확장자를 `.exe` 로 변환한다. `Verb` 로는 "`RunAs`" 를 사용해서 다른 사용자 계정으로 실행한다. 아마도 어드민 계정이 사용될 때 까지 계속 새 프로세스를 실행하도록 설정하는 것 같다.

그 후 설정한 정보들로 새로운 Process 를 시작하고, 예외가 발생하면 어드민 권한으로 실행하라는 메세지 박스를 출력한다. 그 후 false 를 return 하여 현재의 프로세스는 Main 메서드에서 종료하도록 한다. 프로세스가 시작될 때 Admin 역할인 경우만 true 가 return 되기 때문에, 이 경우만 Main 메서드에서 다음 코드로 넘어갈 수 있게 해놓은 것 같다.

> **Assembly 란?**
>
> 위 코드를 살펴보다가 Assembly 라는 단어가 생소해서 C# 에서 어떤 뜻인지 찾아봤다. Assembly 는 .NET 기반 어플리케이션에서 deployment, version control, reuse, activation scoping, and security permissions 등의 기본 단위이다. Assembly 는 executable(`.exe`) 혹은 dynamic link library(`.dll`) 의 형태를 가지며 .NET 어플리케이션의 기본 요소이다.
>
> 자세한 내용은 [도큐먼트(Assemblies in .NET)](https://learn.microsoft.com/en-us/dotnet/standard/assembly/){: target='_blank' } 참조
{: .prompt-info }

---

다음 코드이다.

```cs
static void Main(string[] args)
{
    // 생략
    VersionCompatibility();
    // 생략
}

static void VersionCompatibility()
{
    (var region, var installedVersion) = VersionCheck.GetLostArkVersion();
    if (installedVersion == null)
    {
        MessageBox.Show("Launch Lost Ark before launching logger", "Lost Ark Not Running", MessageBoxButtons.OK, MessageBoxIcon.Exclamation);
        Environment.Exit(0);
    }
    else if (region == Region.Unknown)
        MessageBox.Show("DPS Meter is out of date.\nDPS Meter might not work until updated.\nCheck Discord/Github for more info.\nFeel free to add a message in the discord informing shalzuth that it's out of data", "Out of date!", MessageBoxButtons.OK, MessageBoxIcon.Warning);
    else if (Properties.Settings.Default.Region != region)
    {
        Properties.Settings.Default.Region = region;
        Properties.Settings.Default.Save();
    }
}
```
{: file='Program.cs' }

```cs
[DllImport("kernel32")] public static extern bool QueryFullProcessImageName([In] IntPtr hProcess, [In] int dwFlags, [Out] StringBuilder lpExeName, ref int lpdwSize);

public static (Region, Version) GetLostArkVersion()
{
    var lostArkProcesses = Process.GetProcessesByName("LOSTARK");
    foreach(var lostArkProcess in lostArkProcesses)
    {
        var sb = new StringBuilder(1024);
        int bufferLength = sb.Capacity + 1;
        QueryFullProcessImageName(lostArkProcess.Handle, 0, sb, ref bufferLength);
        var lostArkExe = sb.ToString();
        var version = new Version(FileVersionInfo.GetVersionInfo(lostArkExe).ProductVersion.Split(' ')[0]);
        if (version == SupportedSteamVersion) return (Region.Steam, version);
        else if (version == SupportedKoreaVersion) return (Region.Korea, version);
        else return (Region.Unknown, version);
    }
    return (Region.Unknown, null);
    var fileName = @"C:\Program Files (x86)\Steam\steamapps\common\Lost Ark\Binaries\Win64\LOSTARK.exe";
    if (!File.Exists(fileName))
    {
        var installLocation = Microsoft.Win32.Registry.LocalMachine.OpenSubKey(@"SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Steam App 1599340")?.GetValue("InstallLocation");
        if (installLocation != null)
        {
            fileName = Path.Combine(installLocation.ToString(), "Binaries", "Win64", "LOSTARK.exe");
        }
    }
    if (File.Exists(fileName)) return (Region.Steam, new Version(FileVersionInfo.GetVersionInfo(fileName).ProductVersion.Split(' ')[0]));
    else return (Region.Steam, new Version("0.0.0.0"));

}
```
{: file='Utilities/VersionCheck.cs' }

Main 에서 VersionCompatibility 라는 스태틱 메서드를 실행하는데, 그 메서드 첫줄에서 `Utilities/VersionCheck.cs` 파일에 정의된 `GetLostArkVersion` 스태틱 메서드를 실행하고 있다. 첫줄부터 심상치 않은 코드가 있다...

---

```cs
[DllImport("kernel32")] public static extern bool QueryFullProcessImageName([In] IntPtr hProcess, [In] int dwFlags, [Out] StringBuilder lpExeName, ref int lpdwSize);
```

첫번째 코드를 살펴보자. `DllImportAttribute` 를 사용하고 있다. [도큐먼트](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.dllimportattribute?view=net-8.0){: target='_blank' }에 따르면 어트리뷰트가 사용된 메서드는 unmanaged DLL(Dynamic-Link Library) 로부터 실행됨을 뜻하는 것 같다.

DLL 은 간단히 말하면 공유 라이브러리(shared library)인데, 관련 자세한 내용은 따로 포스트 [DLL(Dynamic-Link Library)란 무엇인가](/posts/net-dotnet-application-settings-관리/){: target='_blank' } 에 정리해두었다.

