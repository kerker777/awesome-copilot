---
description: 'Comprehensive guide for working with Power Apps Canvas Apps YAML structure based on Microsoft Power Apps YAML schema v3.0. Covers Power Fx formulas, control structures, data types, and source control best practices.'
applyTo: '**/*.{yaml,yml,md,pa.yaml}'
---

# Power Apps Canvas Apps YAML 結構指南

## 概述
本文件基於微軟官方 Power Apps YAML 結構（v3.0）和 Power Fx 文件，提供使用 Power Apps Canvas 應用程式之 YAML 程式碼的全面說明。

**官方結構定義來源**：https://raw.githubusercontent.com/microsoft/PowerApps-Tooling/refs/heads/master/schemas/pa-yaml/v3.0/pa.schema.yaml

## Power Fx 設計原則
Power Fx 是整個 Power Apps Canvas 應用程式中使用的公式語言。它遵循這些核心原則：

### 設計原則
- **簡單**：使用來自 Excel 公式的熟悉概念
- **Excel 相容性**：符合 Excel 公式語法和行為
- **宣告式**：描述您想要什麼，而不是如何達成
- **函數式**：避免副作用；大多數函數都是純函數
- **組合性**：透過組合較簡單的函數來建構複雜邏輯
- **強型別**：型別系統確保資料完整性
- **整合性**：在整個 Power Platform 中無縫運作

### 語言理念
Power Fx 提倡：
- 透過熟悉的類似 Excel 公式進行低程式碼開發
- 當相依性改變時自動重新計算
- 使用編譯時期檢查的型別安全
- 函數式編程模式

## 根結構
每個 Power Apps YAML 檔案都遵循此最上層結構：

```yaml
App:
  Properties:
    # App-level properties and formulas
    StartScreen: =Screen1

Screens:
  # Screen definitions

ComponentDefinitions:
  # Custom component definitions

DataSources:
  # Data source configurations

EditorState:
  # Editor metadata (screen order, etc.)
```

## 1. App 區段
`App` 區段定義應用程式層級的屬性和設定。

```yaml
App:
  Properties:
    StartScreen: =Screen1
    BackEnabled: =false
    # Other app properties with Power Fx formulas
```

### 重點：
- 包含應用程式範圍的設定
- 屬性使用 Power Fx 公式（以 `=` 作為前綴）
- `StartScreen` 屬性通常會被指定

## 2. 畫面區段
將應用程式中的所有畫面定義為無序對應。

```yaml
Screens:
  Screen1:
    Properties:
      # Screen properties
    Children:
      - Label1:
          Control: Label
          Properties:
            Text: ="Hello World"
            X: =10
            Y: =10
      - Button1:
          Control: Button
          Properties:
            Text: ="Click Me"
            X: =10
            Y: =100
```

### 畫面結構：
- **Properties**：畫面層級的屬性和公式
- **Children**：畫面上的控制項陣列（按疊置順序排列）

### 控制項定義格式：
```yaml
ControlName:
  Control: ControlType      # Required: Control type identifier
  Properties:
    PropertyName: =PowerFxFormula
  # Optional properties:
  Group: GroupName          # For organizing controls in Studio
  Variant: VariantName      # Control variant (affects default properties)
  MetadataKey: Key          # Metadata identifier for control
  Layout: LayoutName        # Layout configuration
  IsLocked: true/false      # Whether control is locked in editor
  Children: []              # For container controls (ordered by z-index)
```

### 控制項版本化：
您可以使用 `@` 運算子指定控制項版本：
```yaml
MyButton:
  Control: Button@2.1.0     # Specific version
  Properties:
    Text: ="Click Me"

MyLabel:
  Control: Label            # Uses latest version by default
  Properties:
    Text: ="Hello World"
```

## 3. 控制項類型

### 標準控制項
常見的第一方控制項包括：
- **基本控制項**：`Label`、`Button`、`TextInput`、`HTMLText`
- **輸入控制項**：`Slider`、`Toggle`、`Checkbox`、`Radio`、`Dropdown`、`Combobox`、`DatePicker`、`ListBox`
- **顯示控制項**：`Image`、`Icon`、`Video`、`Audio`、`PDF viewer`、`Barcode scanner`
- **版面配置控制項**：`Container`、`Rectangle`、`Circle`、`Gallery`、`DataTable`、`Form`
- **圖表控制項**：`Column chart`、`Line chart`、`Pie chart`
- **進階控制項**：`Timer`、`Camera`、`Microphone`、`Add picture`、`Import`、`Export`

### 容器和版面配置控制項
容器控制項及其子項需要特別注意：
```yaml
MyContainer:
  Control: Container
  Properties:
    Width: =300
    Height: =200
    Fill: =RGBA(240, 240, 240, 1)
  Children:
    - Label1:
        Control: Label
        Properties:
          Text: ="Inside Container"
          X: =10         # Relative to container
          Y: =10         # Relative to container
    - Button1:
        Control: Button
        Properties:
          Text: ="Container Button"
          X: =10
          Y: =50
```

### 自訂元件
```yaml
MyCustomControl:
  Control: Component
  ComponentName: MyComponent
  Properties:
    X: =10
    Y: =10
    # Custom component properties
```

### 程式碼元件 (PCF)
```yaml
MyPCFControl:
  Control: CodeComponent
  ComponentName: publisherprefix_namespace.classname
  Properties:
    X: =10
    Y: =10
```

## 4. 元件定義
定義可重複使用的自訂元件：

```yaml
ComponentDefinitions:
  MyComponent:
    DefinitionType: CanvasComponent
    Description: "A reusable component"
    AllowCustomization: true
    AccessAppScope: false
    CustomProperties:
      InputText:
        PropertyKind: Input
        DataType: Text
        Description: "Input text property"
        Default: ="Default Value"
      OutputValue:
        PropertyKind: Output
        DataType: Number
        Description: "Output number value"
    Properties:
      Fill: =RGBA(255, 255, 255, 1)
      Height: =100
      Width: =200
    Children:
      - Label1:
          Control: Label
          Properties:
            Text: =Parent.InputText
```

### 自訂屬性類型：
- **Input**：從父元件接收值
- **Output**：向父元件傳送值
- **InputFunction**：由父元件呼叫的函數
- **OutputFunction**：在元件中定義的函數
- **Event**：觸發事件到父元件
- **Action**：具有副作用的函數

### 資料類型：
- `Text`、`Number`、`Boolean`
- `DateAndTime`、`Color`、`Currency`
- `Record`、`Table`、`Image`
- `VideoOrAudio`、`Screen`

## 5. 資料來源
設定資料連線：

```yaml
DataSources:
  MyTable:
    Type: Table
    Parameters:
      TableLogicalName: account

  MyActions:
    Type: Actions
    ConnectorId: shared_office365users
    Parameters:
      # Additional connector parameters
```

### 資料來源類型：
- **Table**：Dataverse 表格或其他表格資料
- **Actions**：連接器動作和流程

## 6. 編輯器狀態
維護編輯器組織：

```yaml
EditorState:
  ScreensOrder:
    - Screen1
    - Screen2
    - Screen3
  ComponentDefinitionsOrder:
    - MyComponent
    - AnotherComponent
```

## Power Fx 公式指南

### 公式語法：
- 所有公式都必須以 `=` 開頭
- 為表達式使用 Power Fx 語法
- 空值可以表示為 `null`（沒有引號）
- 範例：
  ```yaml
  Text: ="Hello World"
  X: =10
  Visible: =Toggle1.Value
  OnSelect: =Navigate(Screen2, ScreenTransition.Fade)
  OptionalProperty: null    # Represents no value
  ```

### 常見公式模式：
```yaml
# Static values
Text: ="Static Text"
X: =50
Visible: =true

# Control references
Text: =TextInput1.Text
Visible: =Toggle1.Value

# Parent references (for controls in containers/galleries)
Width: =Parent.Width - 20
Height: =Parent.TemplateHeight    # In gallery templates

# Functions
OnSelect: =Navigate(NextScreen, ScreenTransition.Slide)
Text: =Concatenate("Hello ", User().FullName)

# Conditional logic
Visible: =If(Toggle1.Value, true, false)
Fill: =If(Button1.Pressed, RGBA(255,0,0,1), RGBA(0,255,0,1))

# Data operations
Items: =Filter(DataSource, Status = "Active")
Text: =LookUp(Users, ID = 123).Name
```

### 疊置順序和控制項排序：
- `Children` 陣列中的控制項按疊置順序排列
- 陣列中的第一個控制項 = 底層（疊置順序 1）
- 陣列中的最後一個控制項 = 頂層（最高疊置順序）
- 所有控制項使用從 1 開始的遞增順序

## 命名慣例

### 實體名稱：
- 畫面名稱：描述性且唯一
- 控制項名稱：型別名稱 + 編號（例如 `Button1`、`Label2`）
- 元件名稱：PascalCase

### 屬性名稱：
- 標準屬性：使用結構定義中的精確大小寫
- 自訂屬性：建議使用 PascalCase

## 最佳做法

### 1. 結構組織：
- 保持畫面邏輯上組織有序
- 使用 `Group` 屬性對相關控制項進行分組
- 為所有實體使用有意義的名稱

### 2. 公式撰寫：
- 保持公式易讀且格式良好
- 在複雜公式中盡可能使用註解
- 避免過度複雜的巢狀表達式

### 3. 元件設計：
- 設計可重複使用的元件
- 為自訂屬性提供清晰的說明
- 使用適當的屬性類型（Input/Output）

### 4. 資料來源管理：
- 為資料來源使用描述性名稱
- 記錄連線需求
- 保持資料來源設定最少化

## 驗證規則

### 必需屬性：
- 所有控制項都必須有 `Control` 屬性
- 元件定義必須有 `DefinitionType`
- 資料來源必須有 `Type`

### 命名模式：
- 實體名稱：最少 1 個字元，英數字
- 控制項型別 ID：遵循模式 `^([A-Z][a-zA-Z0-9]*/)?[A-Z][a-zA-Z0-9]*(@\d+\.\d+\.\d+)?$`
- 程式碼元件名稱：遵循模式 `^([a-z][a-z0-9]{1,7})_([a-zA-Z0-9]\.)+[a-zA-Z0-9]+$`

## 常見問題和解決方案

### 1. 無效的控制項型別：
- 確保控制項型別拼寫正確
- 檢查大小寫是否正確
- 驗證結構定義中支援此控制項型別

### 2. 公式錯誤：
- 所有公式都必須以 `=` 開頭
- 使用適當的 Power Fx 語法
- 檢查屬性參考是否正確

### 3. 結構驗證：
- 保持正確的 YAML 縮排
- 確保存在必需屬性
- 完全遵循結構定義結構

### 4. 自訂元件問題：
- 驗證 `ComponentName` 是否符合定義
- 確保自訂屬性已正確定義
- 檢查屬性型別是否適當
- 如果使用外部元件，驗證元件庫參考

### 5. 效能考量：
- 避免在 YAML 中深度巢狀公式
- 使用有效率的資料來源查詢
- 對大型資料集考慮可委派公式
- 最小化頻繁更新屬性中的複雜計算

## 進階主題

### 1. 元件庫整合：
```yaml
ComponentDefinitions:
  MyLibraryComponent:
    DefinitionType: CanvasComponent
    AllowCustomization: true
    ComponentLibraryUniqueName: "pub_MyComponentLibrary"
    # Component definition details
```

### 2. 回應式設計考量：
- 為回應式調整大小使用 `Parent.Width` 和 `Parent.Height`
- 對複雜 UI 考慮容器型版面配置
- 為動態定位和調整大小使用公式

### 3. 圖庫範本：
```yaml
MyGallery:
  Control: Gallery
  Properties:
    Items: =DataSource
    TemplateSize: =100
  Children:
    - GalleryTemplate:  # Template for each gallery item
        Children:
          - TitleLabel:
              Control: Label
              Properties:
                Text: =ThisItem.Title
                Width: =Parent.TemplateWidth - 20
```

### 4. 表單控制項和資料卡：
```yaml
MyForm:
  Control: Form
  Properties:
    DataSource: =DataSource
    DefaultMode: =FormMode.New
  Children:
    - DataCard1:
        Control: DataCard
        Properties:
          DataField: ="Title"
        Children:
          - DataCardValue1:
              Control: TextInput
              Properties:
                Default: =Parent.Default
```

### 5. 公式中的錯誤處理：
```yaml
Properties:
  Text: =IfError(LookUp(DataSource, ID = 123).Name, "Not Found")
  Visible: =!IsError(DataSource)
  OnSelect: =IfError(
    Navigate(DetailScreen, ScreenTransition.Cover),
    Notify("Navigation failed", NotificationType.Error)
  )
```

## Power Apps 原始碼管理

### 存取原始碼檔案：
Power Apps YAML 檔案可以透過多種方法取得：

1. **Power Platform CLI**：
   ```powershell
   # List canvas apps in environment
   pac canvas list

   # Download and extract YAML files
   pac canvas download --name "MyApp" --extract-to-directory "C:\path\to\destination"
   ```

2. **從 .msapp 手動提取**：
   ```powershell
   # Extract .msapp file using PowerShell
   Expand-Archive -Path "C:\path\to\yourFile.msapp" -DestinationPath "C:\path\to\destination"
   ```

3. **Dataverse Git 整合**：直接存取原始碼檔案，無需 .msapp 檔案

### .msapp 中的檔案結構：
- `\src\App.pa.yaml` - 代表主應用程式設定
- `\src\[ScreenName].pa.yaml` - 每個畫面一個檔案
- `\src\Component\[ComponentName].pa.yaml` - 元件定義

**重要注意事項**：
- 只有 `\src` 資料夾中的檔案用於版本控制
- .pa.yaml 檔案是**唯讀的**，僅供檢閱之用
- 不支援外部編輯、合併和衝突解決
- .msapp 中的 JSON 檔案對版本控制並不穩定

### 結構定義版本演變：
1. **實驗性格式** (*.fx.yaml)：不再開發
2. **早期預覽**：暫時格式，不再使用
3. **原始碼** (*.pa.yaml)：具有版本控制支援的目前作用中格式

## Power Fx 公式參考

### 公式類別：

#### **函數**：接受參數、執行操作、傳回值
```yaml
Properties:
  Text: =Concatenate("Hello ", User().FullName)
  X: =Sum(10, 20, 30)
  Items: =Filter(DataSource, Status = "Active")
```

#### **訊號**：傳回環境資訊（沒有參數）
```yaml
Properties:
  Text: =Location.Latitude & ", " & Location.Longitude
  Visible: =Connection.Connected
  Color: =If(Acceleration.X > 5, Color.Red, Color.Blue)
```

#### **列舉**：預先定義的常數值
```yaml
Properties:
  Fill: =Color.Blue
  Transition: =ScreenTransition.Fade
  Align: =Align.Center
```

#### **具名運算子**：存取容器資訊
```yaml
Properties:
  Text: =ThisItem.Title        # In galleries
  Width: =Parent.Width - 20    # In containers
  Height: =Self.Height / 2     # Self-reference
```

### YAML 的必需 Power Fx 函數：

#### **導覽和應用程式控制**：
```yaml
OnSelect: =Navigate(NextScreen, ScreenTransition.Cover)
OnSelect: =Back()
OnSelect: =Exit()
OnSelect: =Launch("https://example.com")
```

#### **資料操作**：
```yaml
Items: =Filter(DataSource, Category = "Active")
Text: =LookUp(Users, ID = 123).Name
OnSelect: =Patch(DataSource, ThisItem, {Status: "Complete"})
OnSelect: =Collect(LocalCollection, {Name: TextInput1.Text})
```

#### **條件邏輯**：
```yaml
Visible: =If(Toggle1.Value, true, false)
Text: =Switch(Status, "New", "🆕", "Complete", "✅", "❓")
Fill: =If(Value < 0, Color.Red, Color.Green)
```

#### **文字處理**：
```yaml
Text: =Concatenate("Hello ", User().FullName)
Text: =Upper(TextInput1.Text)
Text: =Substitute(Label1.Text, "old", "new")
Text: =Left(Title, 10) & "..."
```

#### **數學操作**：
```yaml
Text: =Sum(Sales[Amount])
Text: =Average(Ratings[Score])
Text: =Round(Calculation, 2)
Text: =Max(Values[Number])
```

#### **日期和時間函數**：
```yaml
Text: =Text(Now(), "mm/dd/yyyy")
Text: =DateDiff(StartDate, EndDate, Days)
Text: =Text(Today(), "dddd, mmmm dd, yyyy")
Visible: =IsToday(DueDate)
```

### 公式語法指南：

#### **基本語法規則**：
- 所有公式以 `=` 開頭
- 沒有前置 `+` 或 `=` 符號（不同於 Excel）
- 文字字串使用雙引號：`="Hello World"`
- 屬性參考：`ControlName.PropertyName`
- 不支援在 YAML 內容中加入註解

#### **公式元素**：
```yaml
# Literal values
Text: ="Static Text"
X: =42
Visible: =true

# Control property references
Text: =TextInput1.Text
Visible: =Checkbox1.Value

# Function calls
Text: =Upper(TextInput1.Text)
Items: =Sort(DataSource, Title)

# Complex expressions
Text: =If(IsBlank(TextInput1.Text), "Enter text", Upper(TextInput1.Text))
```

#### **行為與屬性公式的對比**：
```yaml
# Property formulas (calculate values)
Properties:
  Text: =Concatenate("Hello ", User().FullName)
  Visible: =Toggle1.Value

# Behavior formulas (perform actions - use semicolon for multiple actions)
Properties:
  OnSelect: =Set(MyVar, true); Navigate(NextScreen); Notify("Done!")
```

### 進階公式模式：

#### **使用集合**：
```yaml
Properties:
  Items: =Filter(MyCollection, Status = "Active")
  OnSelect: =ClearCollect(MyCollection, DataSource)
  OnSelect: =Collect(MyCollection, {Name: "New Item", Status: "Active"})
```

#### **錯誤處理**：
```yaml
Properties:
  Text: =IfError(Value(TextInput1.Text), 0)
  OnSelect: =IfError(
    Patch(DataSource, ThisItem, {Field: Value}),
    Notify("Error updating record", NotificationType.Error)
  )
```

#### **動態屬性設定**：
```yaml
Properties:
  Fill: =ColorValue("#" & HexInput.Text)
  Height: =Parent.Height * (Slider1.Value / 100)
  X: =If(Alignment = "Center", (Parent.Width - Self.Width) / 2, 0)
```

## 使用公式的最佳做法

### 公式組織：
- 將複雜公式分解為更小、易讀的部分
- 使用變數儲存中間計算結果
- 使用描述性控制項名稱為複雜邏輯加入註解
- 將相關計算分組在一起

### 效能最佳化：
- 使用可委派函數處理大型資料集
- 避免在頻繁更新的屬性中使用巢狀函數呼叫
- 為複雜資料轉換使用集合
- 最小化外部資料來源的呼叫

## Power Fx 資料類型和操作

### 資料類型類別：

#### **基本類型**：
- **布林值**：`=true`、`=false`
- **數字**：`=123`、`=45.67`
- **文字**：`="Hello World"`
- **日期**：`=Date(2024, 12, 25)`
- **時間**：`=Time(14, 30, 0)`
- **日期時間**：`=Now()`

#### **複雜類型**：
- **色彩**：`=Color.Red`、`=RGBA(255, 128, 0, 1)`
- **記錄**：`={Name: "John", Age: 30}`
- **表格**：`=Table({Name: "John"}, {Name: "Jane"})`
- **GUID**：`=GUID()`

#### **型別轉換**：
```yaml
Properties:
  Text: =Text(123.45, "#,##0.00")        # Number to text
  Text: =Value("123.45")                 # Text to number
  Text: =DateValue("12/25/2024")         # Text to date
  Visible: =Boolean("true")              # Text to boolean
```

#### **型別檢查**：
```yaml
Properties:
  Visible: =Not(IsBlank(OptionalField))
  Visible: =Not(IsError(Value(TextInput1.Text)))
  Visible: =IsNumeric(TextInput1.Text)
```

### 表格操作：

#### **建立表格**：
```yaml
Properties:
  Items: =Table(
    {Name: "Product A", Price: 10.99},
    {Name: "Product B", Price: 15.99}
  )
  Items: =["Option 1", "Option 2", "Option 3"]  # Single-column table
```

#### **篩選和排序**：
```yaml
Properties:
  Items: =Filter(Products, Price > 10)
  Items: =Sort(Products, Name, Ascending)
  Items: =SortByColumns(Products, "Price", Descending, "Name", Ascending)
```

#### **資料轉換**：
```yaml
Properties:
  Items: =AddColumns(Products, "Total", Price * Quantity)
  Items: =RenameColumns(Products, "Price", "Cost")
  Items: =ShowColumns(Products, "Name", "Price")
  Items: =DropColumns(Products, "InternalID")
```

#### **彙總**：
```yaml
Properties:
  Text: =Sum(Products, Price)
  Text: =Average(Products, Rating)
  Text: =Max(Products, Price)
  Text: =CountRows(Products)
```

### 變數和狀態管理：

#### **全域變數**：
```yaml
Properties:
  OnSelect: =Set(MyGlobalVar, "Hello World")
  Text: =MyGlobalVar
```

#### **內容變數**：
```yaml
Properties:
  OnSelect: =UpdateContext({LocalVar: "Screen Specific"})
  OnSelect: =Navigate(NextScreen, None, {PassedValue: 42})
```

#### **集合**：
```yaml
Properties:
  OnSelect: =ClearCollect(MyCollection, DataSource)
  OnSelect: =Collect(MyCollection, {Name: "New Item"})
  Items: =MyCollection
```

## Power Fx 增強連接器和外部資料

### 連接器整合：
```yaml
DataSources:
  SharePointList:
    Type: Table
    Parameters:
      TableLogicalName: "Custom List"

  Office365Users:
    Type: Actions
    ConnectorId: shared_office365users
```

### 使用外部資料：
```yaml
Properties:
  Items: =Filter(SharePointList, Status = "Active")
  OnSelect: =Office365Users.SearchUser({searchTerm: SearchInput.Text})
```

### 委派考量：
```yaml
Properties:
  # Delegable operations (executed server-side)
  Items: =Filter(LargeTable, Status = "Active")    # Efficient

  # Non-delegable operations (may download all records)
  Items: =Filter(LargeTable, Len(Description) > 100)  # Warning issued
```

## 故障排除和常見模式

### 常見錯誤模式：
```yaml
# Handle blank values
Properties:
  Text: =If(IsBlank(OptionalText), "Default", OptionalText)

# Handle errors gracefully
Properties:
  Text: =IfError(RiskyOperation(), "Fallback Value")

# Validate input
Properties:
  Visible: =And(
    Not(IsBlank(NameInput.Text)),
    IsNumeric(AgeInput.Text),
    IsMatch(EmailInput.Text, Email)
  )
```

### 效能最佳化：
```yaml
# Efficient data loading
Properties:
  Items: =Filter(LargeDataSource, Status = "Active")    # Server-side filtering

# Use delegation-friendly operations
Properties:
  Items: =Sort(Filter(DataSource, Active), Name)        # Delegable
  # Avoid: Sort(DataSource, If(Active, Name, ""))       # Not delegable
```

### 記憶體管理：
```yaml
# Clear unused collections
Properties:
  OnSelect: =Clear(TempCollection)

# Limit data retrieval
Properties:
  Items: =FirstN(Filter(DataSource, Status = "Active"), 50)
```

記住：本指南提供 Power Apps Canvas 應用程式 YAML 結構和 Power Fx 公式的全面涵蓋。請務必根據官方結構定義驗證您的 YAML，並在 Power Apps Studio 環境中測試公式。
