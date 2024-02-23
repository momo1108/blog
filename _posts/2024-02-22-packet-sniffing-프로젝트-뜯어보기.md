---
layout: post
title: packet sniffing 프로젝트 뜯어보기
date: '2024-02-22 15:35:31 +0900'
category: [자기계발]
---

# 앞서서
이 포스트는 내가 원하는 기능의 앱을 개발하기 위한 기본 과정을 참조할 수 있는 깃헙 프로젝트를 직접 분석해보는 포스트입니다.

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
```

IsConsole 변수는 단순히 bool 값이고, 변수에 해당하는 값은 `Console.OpenStandardInput(1)` 로 받은 1바이트 크기의 인풋 스트림이 Null 이 아닌지의 여부이다.

디버그 모드에서의 IsConsole 결과값은 True 였다.

다음은 메인 메서드의 첫 코드이다.

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

참고로 Application Settings 관련 User-Scoped Settings 값들은 App.config 에 아래처럼 저장되어있다.

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
