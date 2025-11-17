---
description: '.NET WPF 元件和應用程式模式'
applyTo: '**/*.xaml, **/*.cs'
---

## 摘要

這些說明指導 GitHub Copilot 協助建置高品質、可維護且高效能的 WPF 應用程式，使用 MVVM 模式。包括 XAML、資料繫結、UI 回應性和 .NET 效能的最佳實務。

## 理想的專案類型

- 使用 C# 和 WPF 的桌面應用程式
- 遵循 MVVM（Model-View-ViewModel）設計模式的應用程式
- 使用 .NET 8.0 或更高版本的專案
- 在 XAML 中建置的 UI 元件
- 強調效能和回應性的解決方案

## 目標

- 為 `INotifyPropertyChanged` 和 `RelayCommand` 產生樣板程式碼
- 建議 ViewModel 和 View 邏輯的清晰分離
- 鼓勵使用 `ObservableCollection<T>`、`ICommand` 和適當的繫結
- 建議效能提示（例如虛擬化、非同步載入）
- 避免緊密耦合的程式碼後置邏輯
- 產生可測試的 ViewModel

## 範例提示行為

### ✅ 良好建議
- "為登入畫面產生 ViewModel，包含使用者名稱和密碼的屬性，以及 LoginCommand"
- "撰寫使用 UI 虛擬化並繫結到 ObservableCollection 的 ListView XAML 程式碼片段"
- "將此程式碼後置點擊處理常式重構為 ViewModel 中的 RelayCommand"
- "在 WPF 中非同步提取資料時加入載入旋轉器"

### ❌ 避免
- 在程式碼後置中建議業務邏輯
- 使用沒有上下文的靜態事件處理常式
- 產生沒有繫結的緊密耦合 XAML
- 建議 WinForms 或 UWP 方法

## 優先使用的技術
- C# 與 .NET 8.0+
- XAML 與 MVVM 結構
- `CommunityToolkit.Mvvm` 或自訂 `RelayCommand` 實作
- Async/await 用於非阻塞 UI
- `ObservableCollection`、`ICommand`、`INotifyPropertyChanged`

## 要遵循的常見模式
- ViewModel 優先繫結
- 使用 .NET 或第三方容器（例如 Autofac、SimpleInjector）進行依賴注入
- XAML 命名慣例（控制項使用 PascalCase，繫結使用 camelCase）
- 避免在繫結中使用魔術字串（使用 `nameof`）

## Copilot 可以使用的範例說明片段

```csharp
public class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private string userName;

    [ObservableProperty]
    private string password;

    [RelayCommand]
    private void Login()
    {
        // 在此加入登入邏輯
    }
}
```

```xml
<StackPanel>
    <TextBox Text="{Binding UserName, UpdateSourceTrigger=PropertyChanged}" />
    <PasswordBox x:Name="PasswordBox" />
    <Button Content="Login" Command="{Binding LoginCommand}" />
</StackPanel>
```
