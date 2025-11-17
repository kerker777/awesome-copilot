---
description: '從 .NET MAUI 9 版本升級到版本 10 的說明，包括重大變更、已棄用的 API，以及從 ListView 遷移至 CollectionView 的策略。'
applyTo: '**/*.csproj, **/*.cs, **/*.xaml'
---

# 從 .NET MAUI 9 升級到 .NET MAUI 10

本指南協助您將 .NET MAUI 應用程式從 .NET 9 升級到 .NET 10，重點關注需要程式碼更新的關鍵重大變更和過時的 API。

---

## 目錄

1. [快速開始](#快速開始)
2. [更新目標框架](#更新目標框架)
3. [重大變更（P0 - 必須修復）](#重大變更p0---必須修復)
   - [MessagingCenter 設為內部](#messagingcenter-設為內部)
   - [ListView 和 TableView 已棄用](#listview-和-tableview-已棄用)
4. [已棄用的 API（P1 - 儘快修復）](#已棄用的-apip1---儘快修復)
5. [建議的變更（P2）](#建議的變更p2)
6. [大量遷移工具](#大量遷移工具)
7. [測試您的升級](#測試您的升級)
8. [疑難排解](#疑難排解)

---

## 快速開始

**五步驟升級流程：**

1. **更新 TargetFramework** 為 `net10.0`
2. **更新 CommunityToolkit.Maui** 至 12.3.0+（如果您使用它）- 必要
3. **修復重大變更** - MessagingCenter（P0）
4. **將 ListView/TableView 遷移至 CollectionView**（P0 - 重要）
5. **修復已棄用的 API** - 動畫方法、DisplayAlert、IsBusy（P1）

> ⚠️ **主要重大變更**：
> - CommunityToolkit.Maui **必須**是 12.3.0 或更高版本
> - ListView 和 TableView 現已過時（最重要的遷移工作）

---

## 更新目標框架

### 單一平台

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>
</Project>
```

### 多平台

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net10.0-android;net10.0-ios;net10.0-maccatalyst;net10.0-windows10.0.19041.0</TargetFrameworks>
  </PropertyGroup>
</Project>
```

### 選用：Linux 相容性（GitHub Copilot、WSL 等）

> 💡 **Linux 開發**：如果您在 Linux 上建置（例如 GitHub Codespaces、WSL 或使用 GitHub Copilot），您可以透過條件式排除 iOS/Mac Catalyst 目標來讓專案在 Linux 上編譯：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <!-- 從 Android 開始（永遠支援） -->
    <TargetFrameworks>net10.0-android</TargetFrameworks>

    <!-- 僅在非 Linux 時加入 iOS/Mac Catalyst -->
    <TargetFrameworks Condition="!$([MSBuild]::IsOSPlatform('linux'))">$(TargetFrameworks);net10.0-ios;net10.0-maccatalyst</TargetFrameworks>

    <!-- 僅在 Windows 時加入 Windows -->
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net10.0-windows10.0.19041.0</TargetFrameworks>
  </PropertyGroup>
</Project>
```

**優點：**
- ✅ 在 Linux 上成功編譯（不需要 iOS/Mac 工具）
- ✅ 與 GitHub Codespaces 和 Copilot 相容
- ✅ 根據建置作業系統自動包含正確的目標
- ✅ 在作業系統環境之間切換時不需要變更

**參考：**[dotnet/maui#32186](https://github.com/dotnet/maui/pull/32186)

### 更新必要的 NuGet 套件

> ⚠️ **重要**：如果您使用 CommunityToolkit.Maui，您**必須**更新至 12.3.0 或更高版本。較早的版本與 .NET 10 不相容，將導致編譯錯誤。

```bash
# 更新 CommunityToolkit.Maui（如果您使用它）
dotnet add package CommunityToolkit.Maui --version 12.3.0

# 更新其他常見套件至 .NET 10 相容版本
dotnet add package Microsoft.Maui.Controls --version 10.0.0
```

**檢查所有 NuGet 套件：**
```bash
# 列出所有套件並檢查更新
dotnet list package --outdated

# 將所有套件更新至最新相容版本
dotnet list package --outdated | grep ">" | cut -d '>' -f 1 | xargs -I {} dotnet add package {}
```

---

## 重大變更（P0 - 必須修復）

### MessagingCenter 設為內部

**狀態：** 🚨 **重大** - `MessagingCenter` 現在是 `internal`，無法存取。

**您會看到的錯誤：**
```
error CS0122: 'MessagingCenter' is inaccessible due to its protection level
```

**需要遷移：**

#### 步驟 1：安裝 CommunityToolkit.Mvvm

```bash
dotnet add package CommunityToolkit.Mvvm --version 8.3.0
```

#### 步驟 2：定義訊息類別

```csharp
// 舊：不需要訊息類別
MessagingCenter.Send(this, "UserLoggedIn", userData);

// 新：建立訊息類別
public class UserLoggedInMessage
{
    public UserData Data { get; set; }

    public UserLoggedInMessage(UserData data)
    {
        Data = data;
    }
}
```

#### 步驟 3：更新傳送呼叫

```csharp
// ❌ 舊（.NET 10 已損壞）
using Microsoft.Maui.Controls;

MessagingCenter.Send(this, "UserLoggedIn", userData);
MessagingCenter.Send<App, string>(this, "StatusChanged", "Active");

// ✅ 新（必要）
using CommunityToolkit.Mvvm.Messaging;

WeakReferenceMessenger.Default.Send(new UserLoggedInMessage(userData));
WeakReferenceMessenger.Default.Send(new StatusChangedMessage("Active"));
```

#### 步驟 4：更新訂閱呼叫

```csharp
// ❌ 舊（.NET 10 已損壞）
MessagingCenter.Subscribe<App, UserData>(this, "UserLoggedIn", (sender, data) =>
{
    // 處理訊息
    CurrentUser = data;
});

// ✅ 新（必要）
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (recipient, message) =>
{
    // 處理訊息
    CurrentUser = message.Data;
});
```

#### ⚠️ 重要的行為差異：重複訂閱

**WeakReferenceMessenger** 如果您嘗試在同一個接收者上多次註冊相同的訊息型別，會擲回 `InvalidOperationException`（MessagingCenter 允許這樣做）：

```csharp
// ❌ 這會在 WeakReferenceMessenger 中擲回 InvalidOperationException
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) => Handler1(m));
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) => Handler2(m)); // ❌ 擲回！

// ✅ 解決方案 1：重新註冊前先取消註冊
WeakReferenceMessenger.Default.Unregister<UserLoggedInMessage>(this);
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) => Handler1(m));

// ✅ 解決方案 2：在一個註冊中處理多個動作
WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (r, m) =>
{
    Handler1(m);
    Handler2(m);
});
```

**為什麼這很重要：**如果您的程式碼在多個地方訂閱相同的訊息（例如在頁面建構函式和 `OnAppearing` 中），您會遇到執行階段錯誤。

#### 步驟 5：完成時取消註冊

```csharp
// ❌ 舊
MessagingCenter.Unsubscribe<App, UserData>(this, "UserLoggedIn");

// ✅ 新（重要 - 防止記憶體洩漏）
WeakReferenceMessenger.Default.Unregister<UserLoggedInMessage>(this);

// 或取消註冊此接收者的所有訊息
WeakReferenceMessenger.Default.UnregisterAll(this);
```

#### 完整的前後範例

**之前（.NET 9）：**
```csharp
// 傳送者
public class LoginViewModel
{
    public async Task LoginAsync()
    {
        var user = await AuthService.LoginAsync(username, password);
        MessagingCenter.Send(this, "UserLoggedIn", user);
    }
}

// 接收者
public partial class MainPage : ContentPage
{
    public MainPage()
    {
        InitializeComponent();

        MessagingCenter.Subscribe<LoginViewModel, User>(this, "UserLoggedIn", (sender, user) =>
        {
            WelcomeLabel.Text = $"Welcome, {user.Name}!";
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        MessagingCenter.Unsubscribe<LoginViewModel, User>(this, "UserLoggedIn");
    }
}
```

**之後（.NET 10）：**
```csharp
// 1. 定義訊息
public class UserLoggedInMessage
{
    public User User { get; }

    public UserLoggedInMessage(User user)
    {
        User = user;
    }
}

// 2. 傳送者
public class LoginViewModel
{
    public async Task LoginAsync()
    {
        var user = await AuthService.LoginAsync(username, password);
        WeakReferenceMessenger.Default.Send(new UserLoggedInMessage(user));
    }
}

// 3. 接收者
public partial class MainPage : ContentPage
{
    public MainPage()
    {
        InitializeComponent();

        WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, (recipient, message) =>
        {
            WelcomeLabel.Text = $"Welcome, {message.User.Name}!";
        });
    }

    protected override void OnDisappearing()
    {
        base.OnDisappearing();
        WeakReferenceMessenger.Default.UnregisterAll(this);
    }
}
```

**關鍵差異：**
- ✅ 型別安全的訊息類別
- ✅ 沒有魔術字串
- ✅ 更好的 IntelliSense 支援
- ✅ 更容易重構
- ⚠️ **必須記得取消註冊！**

---

### ListView 和 TableView 已棄用

**狀態：** 🚨 **已棄用（P0）** - `ListView`、`TableView` 和所有 Cell 型別現已過時。遷移至 `CollectionView`。

**您會看到的警告：**
```
warning CS0618: 'ListView' is obsolete: 'ListView is deprecated. Please use CollectionView instead.'
warning CS0618: 'TableView' is obsolete: 'Please use CollectionView instead.'
warning CS0618: 'TextCell' is obsolete: 'The controls which use TextCell (ListView and TableView) are obsolete. Please use CollectionView instead.'
```

**過時的型別：**
- `ListView` → `CollectionView`
- `TableView` → `CollectionView`（用於設定頁面，考慮使用帶 BindableLayout 的垂直 StackLayout）
- `TextCell` → 帶 Label 的自訂 DataTemplate
- `ImageCell` → 帶 Image + Label 的自訂 DataTemplate
- `EntryCell` → 帶 Entry 的自訂 DataTemplate
- `SwitchCell` → 帶 Switch 的自訂 DataTemplate
- `ViewCell` → DataTemplate

**影響：**這是一個**重大**的變更。ListView 和 TableView 是 MAUI 應用程式中最常用的控制項。

#### 為什麼這需要時間

將 ListView/TableView 轉換為 CollectionView 不是簡單的尋找/取代：

1. **不同的事件模型** - `ItemSelected` → `SelectionChanged`，參數不同
2. **不同的分組** - GroupDisplayBinding 不再存在
3. **內容動作** - 必須轉換為 SwipeView
4. **項目大小** - `HasUnevenRows` 處理方式不同
5. **平台特定程式碼** - iOS/Android ListView 平台配置需要移除
6. **需要測試** - CollectionView 虛擬化方式不同，可能影響效能

#### 遷移策略

**步驟 1：盤點您的 ListView**

```bash
# 找出所有 ListView/TableView 使用
grep -r "ListView\|TableView" --include="*.xaml" --include="*.cs" .
```

**步驟 2：基本 ListView → CollectionView**

**之前（ListView）：**
```xaml
<ListView ItemsSource="{Binding Items}"
          ItemSelected="OnItemSelected"
          HasUnevenRows="True">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextCell Text="{Binding Title}"
                     Detail="{Binding Description}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**之後（CollectionView）：**
```xaml
<CollectionView ItemsSource="{Binding Items}"
                SelectionMode="Single"
                SelectionChanged="OnSelectionChanged">
    <CollectionView.ItemTemplate>
        <DataTemplate>
            <VerticalStackLayout Padding="10">
                <Label Text="{Binding Title}"
                       FontAttributes="Bold" />
                <Label Text="{Binding Description}"
                       FontSize="12"
                       TextColor="{StaticResource Gray600}" />
            </VerticalStackLayout>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

> ⚠️ **注意：**CollectionView 預設 `SelectionMode="None"`（選取已停用）。您必須明確設定 `SelectionMode="Single"` 或 `SelectionMode="Multiple"` 以啟用選取。

**程式碼後置變更：**
```csharp
// ❌ 舊（ListView）
void OnItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    if (e.SelectedItem == null)
        return;

    var item = (MyItem)e.SelectedItem;
    // 處理選取

    // 取消選取
    ((ListView)sender).SelectedItem = null;
}

// ✅ 新（CollectionView）
void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (e.CurrentSelection.Count == 0)
        return;

    var item = (MyItem)e.CurrentSelection.FirstOrDefault();
    // 處理選取

    // 取消選取（選用）
    ((CollectionView)sender).SelectedItem = null;
}
```

**步驟 3：分組 ListView → 分組 CollectionView**

**之前（分組 ListView）：**
```xaml
<ListView ItemsSource="{Binding GroupedItems}"
          IsGroupingEnabled="True"
          GroupDisplayBinding="{Binding Key}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <TextCell Text="{Binding Name}" />
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

**之後（分組 CollectionView）：**
```xaml
<CollectionView ItemsSource="{Binding GroupedItems}"
                IsGrouped="true">
    <CollectionView.GroupHeaderTemplate>
        <DataTemplate>
            <Label Text="{Binding Key}"
                   FontAttributes="Bold"
                   BackgroundColor="{StaticResource Gray100}"
                   Padding="10,5" />
        </DataTemplate>
    </CollectionView.GroupHeaderTemplate>

    <CollectionView.ItemTemplate>
        <DataTemplate>
            <VerticalStackLayout Padding="20,10">
                <Label Text="{Binding Name}" />
            </VerticalStackLayout>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

**步驟 4：內容動作 → SwipeView**

> ⚠️ **平台注意事項：**SwipeView 需要觸控輸入。在 Windows 桌面上，它僅適用於觸控螢幕，不適用於滑鼠/觸控板。考慮為桌面場景提供替代 UI（例如按鈕、右鍵選單）。

**之前（帶 ContextActions 的 ListView）：**
```xaml
<ListView.ItemTemplate>
    <DataTemplate>
        <ViewCell>
            <ViewCell.ContextActions>
                <MenuItem Text="Delete"
                         IsDestructive="True"
                         Command="{Binding Source={RelativeSource AncestorType={x:Type local:MyPage}}, Path=DeleteCommand}"
                         CommandParameter="{Binding .}" />
            </ViewCell.ContextActions>

            <Label Text="{Binding Title}" Padding="10" />
        </ViewCell>
    </DataTemplate>
</ListView.ItemTemplate>
```

**之後（帶 SwipeView 的 CollectionView）：**
```xaml
<CollectionView.ItemTemplate>
    <DataTemplate>
        <SwipeView>
            <SwipeView.RightItems>
                <SwipeItems>
                    <SwipeItem Text="Delete"
                              BackgroundColor="Red"
                              Command="{Binding Source={RelativeSource AncestorType={x:Type local:MyPage}}, Path=DeleteCommand}"
                              CommandParameter="{Binding .}" />
                </SwipeItems>
            </SwipeView.RightItems>

            <VerticalStackLayout Padding="10">
                <Label Text="{Binding Title}" />
            </VerticalStackLayout>
        </SwipeView>
    </DataTemplate>
</CollectionView.ItemTemplate>
```

**步驟 5：設定用的 TableView → 替代方法**

TableView 常用於設定頁面。以下是現代的替代方案：

**選項 1：帶分組資料的 CollectionView**
```xaml
<CollectionView ItemsSource="{Binding SettingGroups}"
                IsGrouped="true"
                SelectionMode="None">
    <CollectionView.GroupHeaderTemplate>
        <DataTemplate>
            <Label Text="{Binding Title}"
                   FontAttributes="Bold"
                   Margin="10,15,10,5" />
        </DataTemplate>
    </CollectionView.GroupHeaderTemplate>

    <CollectionView.ItemTemplate>
        <DataTemplate>
            <Grid Padding="15,10" ColumnDefinitions="*,Auto">
                <Label Text="{Binding Title}"
                       VerticalOptions="Center" />
                <Switch Grid.Column="1"
                        IsToggled="{Binding IsEnabled}"
                        IsVisible="{Binding ShowSwitch}" />
            </Grid>
        </DataTemplate>
    </CollectionView.ItemTemplate>
</CollectionView>
```

**選項 2：垂直 StackLayout（用於小型設定清單）**
```xaml
<ScrollView>
    <VerticalStackLayout BindableLayout.ItemsSource="{Binding Settings}"
                        Spacing="10"
                        Padding="15">
        <BindableLayout.ItemTemplate>
            <DataTemplate>
                <Border StrokeThickness="0"
                       BackgroundColor="{StaticResource Gray100}"
                       Padding="15,10">
                    <Grid ColumnDefinitions="*,Auto">
                        <Label Text="{Binding Title}"
                              VerticalOptions="Center" />
                        <Switch Grid.Column="1"
                               IsToggled="{Binding IsEnabled}" />
                    </Grid>
                </Border>
            </DataTemplate>
        </BindableLayout.ItemTemplate>
    </VerticalStackLayout>
</ScrollView>
```

**步驟 6：移除平台特定的 ListView 程式碼**

如果您使用了平台特定的 ListView 功能，請移除它們：

```csharp
// ❌ 舊 - 移除這些 using 陳述式（.NET 10 中現已過時）
using Microsoft.Maui.Controls.PlatformConfiguration;
using Microsoft.Maui.Controls.PlatformConfiguration.iOSSpecific;
using Microsoft.Maui.Controls.PlatformConfiguration.AndroidSpecific;

// ❌ 舊 - 移除 ListView 平台配置（.NET 10 中現已過時）
myListView.On<iOS>().SetSeparatorStyle(SeparatorStyle.FullWidth);
myListView.On<Android>().IsFastScrollEnabled();

// ❌ 舊 - 移除 Cell 平台配置（.NET 10 中現已過時）
viewCell.On<iOS>().SetDefaultBackgroundColor(Colors.White);
viewCell.On<Android>().SetIsContextActionsLegacyModeEnabled(false);
```

**遷移：**CollectionView 沒有相同方式的平台特定配置。如果您需要平台特定的樣式：

```csharp
// ✅ 新 - 使用條件式編譯
#if IOS
var backgroundColor = Colors.White;
#elif ANDROID
var backgroundColor = Colors.Transparent;
#endif

var grid = new Grid
{
    BackgroundColor = backgroundColor,
    // ... 其餘的儲存格內容
};
```

或在 XAML 中：
```xaml
<CollectionView.ItemTemplate>
    <DataTemplate>
        <Grid>
            <Grid.BackgroundColor>
                <OnPlatform x:TypeArguments="Color">
                    <On Platform="iOS" Value="White" />
                    <On Platform="Android" Value="Transparent" />
                </OnPlatform>
            </Grid.BackgroundColor>
            <!-- 儲存格內容 -->
        </Grid>
    </DataTemplate>
</CollectionView.ItemTemplate>
```

#### 常見模式與陷阱

**1. 空白檢視**
```xaml
<!-- CollectionView 有內建的 EmptyView 支援 -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.EmptyView>
        <ContentView>
            <VerticalStackLayout Padding="50" VerticalOptions="Center">
                <Label Text="找不到項目"
                       HorizontalTextAlignment="Center" />
            </VerticalStackLayout>
        </ContentView>
    </CollectionView.EmptyView>
    <!-- ... -->
</CollectionView>
```

**2. 下拉重新整理**
```xaml
<RefreshView IsRefreshing="{Binding IsRefreshing}"
             Command="{Binding RefreshCommand}">
    <CollectionView ItemsSource="{Binding Items}">
        <!-- ... -->
    </CollectionView>
</RefreshView>
```

**3. 項目間距**
```xaml
<!-- 使用 ItemsLayout 設定間距 -->
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.ItemsLayout>
        <LinearItemsLayout Orientation="Vertical"
                          ItemSpacing="10" />
    </CollectionView.ItemsLayout>
    <!-- ... -->
</CollectionView>
```

**4. 頁首和頁尾**
```xaml
<CollectionView ItemsSource="{Binding Items}">
    <CollectionView.Header>
        <Label Text="我的清單"
               FontSize="24"
               Padding="10" />
    </CollectionView.Header>

    <CollectionView.Footer>
        <Label Text="清單結尾"
               Padding="10"
               HorizontalTextAlignment="Center" />
    </CollectionView.Footer>

    <!-- ItemTemplate -->
</CollectionView>
```

**5. 載入更多 / 無限捲動**
```xaml
<CollectionView ItemsSource="{Binding Items}"
                RemainingItemsThreshold="5"
                RemainingItemsThresholdReachedCommand="{Binding LoadMoreCommand}">
    <!-- ItemTemplate -->
</CollectionView>
```

**6. 項目大小最佳化**

CollectionView 使用 `ItemSizingStrategy` 來控制項目測量：

```xaml
<!-- 預設：每個項目個別測量（像 HasUnevenRows="True"） -->
<CollectionView ItemSizingStrategy="MeasureAllItems">
    <!-- ... -->
</CollectionView>

<!-- 效能：僅測量第一個項目，其餘使用相同高度 -->
<CollectionView ItemSizingStrategy="MeasureFirstItem">
    <!-- 當所有項目高度相似時使用 -->
</CollectionView>
```

> 💡 **效能提示：**如果您的清單項目高度一致，使用 `ItemSizingStrategy="MeasureFirstItem"` 可以在大型清單上獲得更好的效能。

#### .NET 10 處理常式變更（iOS/Mac Catalyst）

> ℹ️ **.NET 10 預設在 iOS 和 Mac Catalyst 上使用新的最佳化 CollectionView 和 CarouselView 處理常式**，提供改進的效能和穩定性。

**如果您在 .NET 9 中選擇使用新的處理常式**，您現在應該**移除**此程式碼：

```csharp
// ❌ 在 .NET 10 中移除此程式碼（這些處理常式現在是預設的）
#if IOS || MACCATALYST
builder.ConfigureMauiHandlers(handlers =>
{
    handlers.AddHandler<CollectionView, CollectionViewHandler2>();
    handlers.AddHandler<CarouselView, CarouselViewHandler2>();
});
#endif
```

在 .NET 10 中會自動使用最佳化的處理常式 - 不需要配置！

**只有在遇到問題時**，您可以還原到舊版處理常式：

```csharp
// 在 MauiProgram.cs 中 - 僅在需要時
#if IOS || MACCATALYST
builder.ConfigureMauiHandlers(handlers =>
{
    handlers.AddHandler<Microsoft.Maui.Controls.CollectionView,
                        Microsoft.Maui.Controls.Handlers.Items.CollectionViewHandler>();
});
#endif
```

然而，Microsoft 建議使用新的預設處理常式以獲得最佳結果。

#### 測試檢查清單

遷移後，測試這些場景：

- [ ] **項目選取**正常運作
- [ ] **分組清單**以適當的標題顯示
- [ ] **滑動動作**（如果使用）在 iOS 和 Android 上都能運作
- [ ] **空白檢視**在清單為空時出現
- [ ] **下拉重新整理**正常運作（如果使用）
- [ ] **捲動效能**可接受（特別是大型清單）
- [ ] **項目大小**正確（CollectionView 預設自動調整大小）
- [ ] **選取視覺狀態**正確顯示/隱藏
- [ ] **資料繫結**正確更新清單
- [ ] **從清單項目導覽**正常運作

#### 遷移複雜度因素

ListView 到 CollectionView 的遷移很複雜，因為：
- 每個 ListView 可能有獨特的行為
- 平台特定程式碼需要更新
- 需要大量測試
- 內容動作需要 SwipeView 轉換
- 分組清單需要範本更新
- 可能需要變更 ViewModel

#### 快速參考：ListView vs CollectionView

| 功能 | ListView | CollectionView |
|---------|----------|----------------|
| **選取事件** | `ItemSelected` | `SelectionChanged` |
| **選取參數** | `SelectedItemChangedEventArgs` | `SelectionChangedEventArgs` |
| **取得已選取** | `e.SelectedItem` | `e.CurrentSelection.FirstOrDefault()` |
| **內容選單** | `ContextActions` | `SwipeView` |
| **分組** | `IsGroupingEnabled="True"` | `IsGrouped="true"` |
| **群組標題** | `GroupDisplayBinding` | `GroupHeaderTemplate` |
| **偶數列** | `HasUnevenRows="False"` | 自動調整大小（預設） |
| **空白狀態** | 手動 | `EmptyView` 屬性 |
| **儲存格** | TextCell、ImageCell 等 | 自訂 DataTemplate |

---

## 已棄用的 API（P1 - 儘快修復）

這些 API 在 .NET 10 中仍然可用，但會顯示編譯器警告。它們將在未來版本中移除。

### 1. 動畫方法

**狀態：** ⚠️ **已棄用** - 所有同步動畫方法替換為非同步版本。

**您會看到的警告：**
```
warning CS0618: 'ViewExtensions.FadeTo(VisualElement, double, uint, Easing)' is obsolete: 'Please use FadeToAsync instead.'
```

**遷移表：**

| 舊方法 | 新方法 | 範例 |
|-----------|-----------|---------|
| `FadeTo()` | `FadeToAsync()` | `await view.FadeToAsync(0, 500);` |
| `ScaleTo()` | `ScaleToAsync()` | `await view.ScaleToAsync(1.5, 300);` |
| `TranslateTo()` | `TranslateToAsync()` | `await view.TranslateToAsync(100, 100, 250);` |
| `RotateTo()` | `RotateToAsync()` | `await view.RotateToAsync(360, 500);` |
| `RotateXTo()` | `RotateXToAsync()` | `await view.RotateXToAsync(45, 300);` |
| `RotateYTo()` | `RotateYToAsync()` | `await view.RotateYToAsync(45, 300);` |
| `ScaleXTo()` | `ScaleXToAsync()` | `await view.ScaleXToAsync(2.0, 300);` |
| `ScaleYTo()` | `ScaleYToAsync()` | `await view.ScaleYToAsync(2.0, 300);` |
| `RelRotateTo()` | `RelRotateToAsync()` | `await view.RelRotateToAsync(90, 300);` |
| `RelScaleTo()` | `RelScaleToAsync()` | `await view.RelScaleToAsync(0.5, 300);` |
| `LayoutTo()` | `LayoutToAsync()` | 請參閱下方特別說明 |

#### 遷移範例

**簡單動畫：**
```csharp
// ❌ 舊（已棄用）
await myButton.FadeTo(0, 500);
await myButton.ScaleTo(1.5, 300);
await myButton.TranslateTo(100, 100, 250);

// ✅ 新（必要）
await myButton.FadeToAsync(0, 500);
await myButton.ScaleToAsync(1.5, 300);
await myButton.TranslateToAsync(100, 100, 250);
```

**循序動畫：**
```csharp
// ❌ 舊
await image.FadeTo(0, 300);
await image.ScaleTo(0.5, 300);
await image.FadeTo(1, 300);

// ✅ 新
await image.FadeToAsync(0, 300);
await image.ScaleToAsync(0.5, 300);
await image.FadeToAsync(1, 300);
```

**平行動畫：**
```csharp
// ❌ 舊
await Task.WhenAll(
    image.FadeTo(0, 300),
    image.ScaleTo(0.5, 300),
    image.RotateTo(360, 300)
);

// ✅ 新
await Task.WhenAll(
    image.FadeToAsync(0, 300),
    image.ScaleToAsync(0.5, 300),
    image.RotateToAsync(360, 300)
);
```

**帶取消：**
```csharp
// 新：非同步方法支援取消
CancellationTokenSource cts = new();

try
{
    await view.FadeToAsync(0, 2000);
}
catch (TaskCanceledException)
{
    // 動畫已取消
}

// 從其他地方取消
cts.Cancel();
```

#### 特殊情況：LayoutTo

`LayoutToAsync()` 已棄用，附帶特別訊息：「使用 Translation 來動畫版面配置變更。」

```csharp
// ❌ 舊（已棄用）
await view.LayoutToAsync(new Rect(100, 100, 200, 200), 250);

// ✅ 新（改用 TranslateToAsync）
await view.TranslateToAsync(100, 100, 250);

// 或直接動畫 Translation 屬性
var animation = new Animation(v => view.TranslationX = v, 0, 100);
animation.Commit(view, "MoveX", length: 250);
```

---

### 2. DisplayAlert 和 DisplayActionSheet

**狀態：** ⚠️ **已棄用** - 同步方法替換為非同步版本。

**您會看到的警告：**
```
warning CS0618: 'Page.DisplayAlert(string, string, string)' is obsolete: 'Use DisplayAlertAsync instead'
```

#### 遷移範例

**DisplayAlert：**
```csharp
// ❌ 舊（已棄用）
await DisplayAlert("成功", "資料已成功儲存", "確定");
await DisplayAlert("錯誤", "儲存失敗", "取消");
bool result = await DisplayAlert("確認", "刪除此項目？", "是", "否");

// ✅ 新（必要）
await DisplayAlertAsync("成功", "資料已成功儲存", "確定");
await DisplayAlertAsync("錯誤", "儲存失敗", "取消");
bool result = await DisplayAlertAsync("確認", "刪除此項目？", "是", "否");
```

**DisplayActionSheet：**
```csharp
// ❌ 舊（已棄用）
string action = await DisplayActionSheet(
    "選擇動作",
    "取消",
    "刪除",
    "編輯", "分享", "複製"
);

// ✅ 新（必要）
string action = await DisplayActionSheetAsync(
    "選擇動作",
    "取消",
    "刪除",
    "編輯", "分享", "複製"
);
```

**在 ViewModel 中（使用 IDispatcher）：**
```csharp
// 如果從 ViewModel 呼叫，您需要存取 Page
public class MyViewModel
{
    private readonly IDispatcher _dispatcher;
    private readonly Page _page;

    public MyViewModel(IDispatcher dispatcher, Page page)
    {
        _dispatcher = dispatcher;
        _page = page;
    }

    public async Task ShowAlertAsync()
    {
        await _dispatcher.DispatchAsync(async () =>
        {
            await _page.DisplayAlertAsync("資訊", "來自 ViewModel 的訊息", "確定");
        });
    }
}
```

---

### 3. Page.IsBusy

**狀態：** ⚠️ **已棄用** - 屬性將在 .NET 11 中移除。

**您會看到的警告：**
```
warning CS0618: 'Page.IsBusy' is obsolete: 'Page.IsBusy has been deprecated and will be removed in .NET 11'
```

**為什麼棄用：**
- 跨平台行為不一致
- 自訂選項有限
- 與現代 MVVM 模式配合不佳

#### 遷移範例

**簡單頁面：**
```xaml
<!-- ❌ 舊（已棄用） -->
<ContentPage IsBusy="{Binding IsLoading}">
    <StackLayout>
        <Label Text="這裡是內容" />
    </StackLayout>
</ContentPage>

<!-- ✅ 新（建議） -->
<ContentPage>
    <Grid>
        <!-- 主要內容 -->
        <StackLayout>
            <Label Text="這裡是內容" />
        </StackLayout>

        <!-- 載入指示器覆蓋層 -->
        <ActivityIndicator IsRunning="{Binding IsLoading}"
                          IsVisible="{Binding IsLoading}"
                          Color="{StaticResource Primary}"
                          VerticalOptions="Center"
                          HorizontalOptions="Center" />
    </Grid>
</ContentPage>
```

**帶載入覆蓋層：**
```xaml
<!-- ✅ 更好：自訂載入覆蓋層 -->
<ContentPage>
    <Grid>
        <!-- 主要內容 -->
        <ScrollView>
            <VerticalStackLayout Padding="20">
                <Label Text="您的內容在這裡" />
            </VerticalStackLayout>
        </ScrollView>

        <!-- 載入覆蓋層 -->
        <Grid IsVisible="{Binding IsLoading}"
              BackgroundColor="#80000000">
            <VerticalStackLayout VerticalOptions="Center"
                               HorizontalOptions="Center"
                               Spacing="10">
                <ActivityIndicator IsRunning="True"
                                 Color="White" />
                <Label Text="載入中..."
                       TextColor="White" />
            </VerticalStackLayout>
        </Grid>
    </Grid>
</ContentPage>
```

**在程式碼後置中：**
```csharp
// ❌ 舊（已棄用）
public partial class MyPage : ContentPage
{
    async Task LoadDataAsync()
    {
        IsBusy = true;
        try
        {
            await LoadDataFromServerAsync();
        }
        finally
        {
            IsBusy = false;
        }
    }
}

// ✅ 新（建議）
public partial class MyPage : ContentPage
{
    async Task LoadDataAsync()
    {
        LoadingIndicator.IsVisible = true;
        LoadingIndicator.IsRunning = true;
        try
        {
            await LoadDataFromServerAsync();
        }
        finally
        {
            LoadingIndicator.IsVisible = false;
            LoadingIndicator.IsRunning = false;
        }
    }
}
```

**在 ViewModel 中：**
```csharp
public class MyViewModel : INotifyPropertyChanged
{
    private bool _isLoading;
    public bool IsLoading
    {
        get => _isLoading;
        set
        {
            _isLoading = value;
            OnPropertyChanged();
        }
    }

    public async Task LoadDataAsync()
    {
        IsLoading = true;
        try
        {
            await LoadDataFromServerAsync();
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

---

## 建議的變更（P2）

這些變更是建議的，但不是立即必要的。考慮在下次重構週期中進行遷移。

### Application.MainPage

**狀態：** ⚠️ **已棄用** - 屬性將在未來版本中移除。

**您會看到的警告：**
```
warning CS0618: 'Application.MainPage' is obsolete: 'This property is deprecated. Initialize your application by overriding Application.CreateWindow...'
```

#### 遷移範例

```csharp
// ❌ 舊（已棄用）
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
        MainPage = new AppShell();
    }

    // 稍後變更頁面
    public void SwitchToLoginPage()
    {
        MainPage = new LoginPage();
    }
}

// ✅ 新（建議）
public partial class App : Application
{
    public App()
    {
        InitializeComponent();
    }

    protected override Window CreateWindow(IActivationState? activationState)
    {
        return new Window(new AppShell());
    }

    // 稍後變更頁面
    public void SwitchToLoginPage()
    {
        if (Windows.Count > 0)
        {
            Windows[0].Page = new LoginPage();
        }
    }
}
```

**CreateWindow 的優點：**
- 更好的多視窗支援
- 更明確的初始化
- 更清晰的關注點分離
- 與 Shell 配合更好

---

## 大量遷移工具

使用這些尋找/取代模式來快速更新您的程式碼庫。

### Visual Studio / VS Code

**正規表示式模式 - 尋找/取代**

#### 動畫方法

```regex
尋找:    \.FadeTo\(
取代: .FadeToAsync(

尋找:    \.ScaleTo\(
取代: .ScaleToAsync(

尋找:    \.TranslateTo\(
取代: .TranslateToAsync(

尋找:    \.RotateTo\(
取代: .RotateToAsync(

尋找:    \.RotateXTo\(
取代: .RotateXToAsync(

尋找:    \.RotateYTo\(
取代: .RotateYToAsync(

尋找:    \.ScaleXTo\(
取代: .ScaleXToAsync(

尋找:    \.ScaleYTo\(
取代: .ScaleYToAsync(

尋找:    \.RelRotateTo\(
取代: .RelRotateToAsync(

尋找:    \.RelScaleTo\(
取代: .RelScaleToAsync(
```

#### 顯示方法

```regex
尋找:    DisplayAlert\(
取代: DisplayAlertAsync(

尋找:    DisplayActionSheet\(
取代: DisplayActionSheetAsync(
```

#### ListView/TableView 偵測（需要手動遷移）

**⚠️ 注意：**ListView/TableView 遷移**無法**自動化。使用這些搜尋來找出實例：

```bash
# 在 XAML 中找出所有 ListView 使用
grep -r "<ListView" --include="*.xaml" .

# 在 XAML 中找出所有 TableView 使用
grep -r "<TableView" --include="*.xaml" .

# 在 C# 程式碼中找出 ListView
grep -r "new ListView\|ListView " --include="*.cs" .

# 在 XAML 中找出 Cell 型別
grep -r "TextCell\|ImageCell\|EntryCell\|SwitchCell\|ViewCell" --include="*.xaml" .

# 找出 ItemSelected 處理常式（需要改為 SelectionChanged）
grep -r "ItemSelected=" --include="*.xaml" .
grep -r "ItemSelected\s*\+=" --include="*.cs" .

# 找出 ContextActions（需要改為 SwipeView）
grep -r "ContextActions" --include="*.xaml" .

# 找出平台特定的 ListView 程式碼（需要移除）
grep -r "PlatformConfiguration.*ListView" --include="*.cs" .
```

**建立遷移清單：**
```bash
# 產生所有 ListView/TableView 實例的報告
echo "=== ListView/TableView 遷移清單 ===" > migration-report.txt
echo "" >> migration-report.txt
echo "XAML ListView 實例:" >> migration-report.txt
grep -rn "<ListView" --include="*.xaml" . >> migration-report.txt
echo "" >> migration-report.txt
echo "XAML TableView 實例:" >> migration-report.txt
grep -rn "<TableView" --include="*.xaml" . >> migration-report.txt
echo "" >> migration-report.txt
echo "ItemSelected 處理常式:" >> migration-report.txt
grep -rn "ItemSelected" --include="*.xaml" --include="*.cs" . >> migration-report.txt
echo "" >> migration-report.txt
cat migration-report.txt
```

### PowerShell 腳本

```powershell
# 在所有 .cs 檔案中取代動畫方法
Get-ChildItem -Path . -Recurse -Filter *.cs | ForEach-Object {
    $content = Get-Content $_.FullName -Raw

    # 動畫方法
    $content = $content -replace '\.FadeTo\(', '.FadeToAsync('
    $content = $content -replace '\.ScaleTo\(', '.ScaleToAsync('
    $content = $content -replace '\.TranslateTo\(', '.TranslateToAsync('
    $content = $content -replace '\.RotateTo\(', '.RotateToAsync('
    $content = $content -replace '\.RotateXTo\(', '.RotateXToAsync('
    $content = $content -replace '\.RotateYTo\(', '.RotateYToAsync('
    $content = $content -replace '\.ScaleXTo\(', '.ScaleXToAsync('
    $content = $content -replace '\.ScaleYTo\(', '.ScaleYToAsync('
    $content = $content -replace '\.RelRotateTo\(', '.RelRotateToAsync('
    $content = $content -replace '\.RelScaleTo\(', '.RelScaleToAsync('

    # 顯示方法
    $content = $content -replace 'DisplayAlert\(', 'DisplayAlertAsync('
    $content = $content -replace 'DisplayActionSheet\(', 'DisplayActionSheetAsync('

    Set-Content $_.FullName $content
}

Write-Host "✅ 遷移完成！"
```

---

## 測試您的升級

### 建置驗證

```bash
# 清理方案
dotnet clean

# 還原套件
dotnet restore

# 為每個平台建置
dotnet build -f net10.0-android -c Release
dotnet build -f net10.0-ios -c Release
dotnet build -f net10.0-maccatalyst -c Release
dotnet build -f net10.0-windows -c Release

# 檢查警告
dotnet build --no-incremental 2>&1 | grep -i "warning CS0618"
```

### 啟用警告為錯誤（暫時）

```xml
<!-- 加入到您的 .csproj 以捕捉所有過時的 API 使用 -->
<PropertyGroup>
  <WarningsAsErrors>CS0618</WarningsAsErrors>
</PropertyGroup>
```

### 測試檢查清單

- [ ] 應用程式在所有平台上成功啟動
- [ ] 所有動畫正常運作
- [ ] 對話框（警示/動作表）正確顯示
- [ ] 載入指示器正常運作（如果您使用了 IsBusy）
- [ ] 元件間通訊正常運作（MessagingCenter 替換）
- [ ] 建置輸出中沒有 CS0618 警告
- [ ] 沒有與過時 API 相關的執行階段例外

---

## 疑難排解

### 錯誤：'MessagingCenter' is inaccessible due to its protection level

**原因：**MessagingCenter 在 .NET 10 中現在是內部的。

**解決方案：**
1. 安裝 `CommunityToolkit.Mvvm` 套件
2. 使用 `WeakReferenceMessenger` 替換（請參閱 [MessagingCenter 區段](#messagingcenter-設為內部)）
3. 為每個訊息型別建立訊息類別
4. 別忘了取消註冊！

---

### 警告：動畫方法已過時

**原因：**使用同步動畫方法（`FadeTo`、`ScaleTo` 等）

**快速修復：**
```bash
# 使用大量遷移工具區段中的 PowerShell 腳本
# 或使用尋找/取代模式
```

**手動修復：**
在每個動畫方法呼叫結尾加上 `Async`：
- `FadeTo` → `FadeToAsync`
- `ScaleTo` → `ScaleToAsync`
- 等等

---

### Page.IsBusy 不再運作

**原因：**IsBusy 仍然可用但已棄用。

**解決方案：**使用明確的 ActivityIndicator 替換（請參閱 [IsBusy 區段](#3-pageisbusy)）

---

### 建置失敗，出現「找不到目標框架 'net10.0'」

**原因：**未安裝 .NET 10 SDK 或不是最新版本。

**解決方案：**
```bash
# 檢查 SDK 版本
dotnet --version  # 應該是 10.0.100 或更高

# 從以下位置安裝 .NET 10 SDK：
# https://dotnet.microsoft.com/download/dotnet/10.0

# 更新工作負載
dotnet workload update
```

---

### MessagingCenter 遷移中斷現有程式碼

**常見問題：**

1. **忘記取消註冊：**
   ```csharp
   // ⚠️ 如果不取消註冊會導致記憶體洩漏
   protected override void OnDisappearing()
   {
       base.OnDisappearing();
       WeakReferenceMessenger.Default.UnregisterAll(this);
   }
   ```

2. **錯誤的訊息型別：**
   ```csharp
   // ❌ 錯誤
   WeakReferenceMessenger.Default.Register<UserLoggedIn>(this, handler);
   WeakReferenceMessenger.Default.Send(new UserData());  // 錯誤的型別！

   // ✅ 正確
   WeakReferenceMessenger.Default.Register<UserLoggedInMessage>(this, handler);
   WeakReferenceMessenger.Default.Send(new UserLoggedInMessage(userData));
   ```

3. **接收者參數混淆：**
   ```csharp
   // 接收者參數是註冊的物件（this）
   WeakReferenceMessenger.Default.Register<MyMessage>(this, (recipient, message) =>
   {
       // recipient == this
       // message == 傳送的訊息
   });
   ```

---

### 遷移後動畫未完成

**原因：**忘記 `await` 關鍵字。

```csharp
// ❌ 錯誤 - 動畫執行但程式碼立即繼續
view.FadeToAsync(0, 500);
DoSomethingElse();

// ✅ 正確 - 等待動畫完成
await view.FadeToAsync(0, 500);
DoSomethingElse();
```

---

### 警告：ListView/TableView/TextCell 已過時

**原因：**使用已棄用的 ListView、TableView 或 Cell 型別。

**解決方案：**遷移至 CollectionView（請參閱 [ListView 和 TableView 區段](#listview-和-tableview-已棄用)）

**快速決策指南：**
- **簡單清單** → 帶自訂 DataTemplate 的 CollectionView
- **少於20項的設定頁面** → 帶 BindableLayout 的 VerticalStackLayout
- **20項以上的設定頁面** → 分組的 CollectionView
- **分組資料清單** → 帶 `IsGrouped="True"` 的 CollectionView

---

### CollectionView 沒有 SelectedItem 事件

**原因：**CollectionView 使用 `SelectionChanged` 而不是 `ItemSelected`。

**解決方案：**
```csharp
// ❌ 舊（ListView）
void OnItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    var item = e.SelectedItem as MyItem;
}

// ✅ 新（CollectionView）
void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    var item = e.CurrentSelection.FirstOrDefault() as MyItem;
}
```

---

### 平台特定的 ListView 配置已過時

**原因：**使用 `Microsoft.Maui.Controls.PlatformConfiguration.*Specific.ListView` 擴充功能。

**錯誤：**
```
warning CS0618: 'ListView' is obsolete: 'With the deprecation of ListView, this class is obsolete. Please use CollectionView instead.'
```

**解決方案：**
1. 移除平台特定的 ListView using 陳述式：
   ```csharp
   // ❌ 移除這些
   using Microsoft.Maui.Controls.PlatformConfiguration;
   using Microsoft.Maui.Controls.PlatformConfiguration.iOSSpecific;
   using Microsoft.Maui.Controls.PlatformConfiguration.AndroidSpecific;
   ```

2. 移除平台特定的 ListView 呼叫：
   ```csharp
   // ❌ 移除這些
   myListView.On<iOS>().SetSeparatorStyle(SeparatorStyle.FullWidth);
   myListView.On<Android>().IsFastScrollEnabled();
   viewCell.On<iOS>().SetDefaultBackgroundColor(Colors.White);
   ```

3. CollectionView 有不同的平台自訂選項 - 請參閱 CollectionView 文件以取得替代方案。

---

### ListView 遷移後 CollectionView 效能問題

**常見原因：**

1. **未使用 DataTemplate 快取：**
   ```xaml
   <!-- ❌ 效能不佳 -->
   <CollectionView.ItemTemplate>
       <DataTemplate>
           <ComplexView />
       </DataTemplate>
   </CollectionView.ItemTemplate>

   <!-- ✅ 更好 - 使用更簡單的範本 -->
   <CollectionView.ItemTemplate>
       <DataTemplate>
           <VerticalStackLayout Padding="10">
               <Label Text="{Binding Title}" />
           </VerticalStackLayout>
       </DataTemplate>
   </CollectionView.ItemTemplate>
   ```

2. **複雜的巢狀版面配置：**
   - 避免在 ItemTemplate 中使用深度巢狀的版面配置
   - 盡可能使用 Grid 而不是 StackLayout
   - 考慮使用 FlexLayout 處理複雜的版面配置

3. **圖片未快取：**
   ```xaml
   <Image Source="{Binding ImageUrl}"
          Aspect="AspectFill"
          HeightRequest="80"
          WidthRequest="80">
       <Image.Behaviors>
           <!-- 如需要請加入快取行為 -->
       </Image.Behaviors>
   </Image>
   ```

---

## 快速參考卡

### 優先順序檢查清單

**必須修復（P0 - 重大/重要）：**
- [ ] 使用 `WeakReferenceMessenger` 替換 `MessagingCenter`
- [ ] 將 `ListView` 遷移至 `CollectionView`
- [ ] 將 `TableView` 遷移至 `CollectionView` 或 `BindableLayout`
- [ ] 使用自訂 DataTemplate 替換 `TextCell`、`ImageCell` 等
- [ ] 將 `ContextActions` 轉換為 `SwipeView`
- [ ] 移除平台特定的 ListView 配置

**應該修復（P1 - 已棄用）：**
- [ ] 更新動畫方法：加上 `Async` 後綴
- [ ] 更新 `DisplayAlert` → `DisplayAlertAsync`
- [ ] 更新 `DisplayActionSheet` → `DisplayActionSheetAsync`
- [ ] 使用 `ActivityIndicator` 替換 `Page.IsBusy`

**建議修復（P2）：**
- [ ] 將 `Application.MainPage` 遷移至 `CreateWindow`

### 常見模式

```csharp
// 動畫
await view.FadeToAsync(0, 500);

// 警示
await DisplayAlertAsync("標題", "訊息", "確定");

// 訊息傳遞
WeakReferenceMessenger.Default.Send(new MyMessage());
WeakReferenceMessenger.Default.Register<MyMessage>(this, (r, m) => { });
WeakReferenceMessenger.Default.UnregisterAll(this);

// 載入
IsLoading = true;
try { await LoadAsync(); }
finally { IsLoading = false; }
```

---

## 其他資源

- **官方文件：**https://learn.microsoft.com/dotnet/maui/
- **遷移指南：**https://learn.microsoft.com/dotnet/maui/migration/
- **GitHub 問題：**https://github.com/dotnet/maui/issues
- **CommunityToolkit.Mvvm：**https://learn.microsoft.com/dotnet/communitytoolkit/mvvm/

---

**文件版本：**2.0
**最後更新：**2025 年 11 月
**適用於：**.NET MAUI 10.0.100 及更高版本
