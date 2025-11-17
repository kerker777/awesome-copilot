---
description: 'Comprehensive Power BI data modeling best practices based on Microsoft guidance for creating efficient, scalable, and maintainable semantic models using star schema principles.'
applyTo: '**/*.{pbix,md,json,txt}'
---

# Power BI 資料建模最佳實踐

## 概述
本文件提供了設計高效、可擴展且易於維護的 Power BI 語義模型的全面指南，遵循 Microsoft 官方指導和維度建模最佳實踐。

## 星狀綱要設計原則

### 1. 基本表格類型
**維度表** - 儲存描述性的商務實體：
- 產品、客戶、地理位置、時間、員工
- 包含唯一的主鍵欄位（最好是代理鍵）
- 行數相對較少
- 用於篩選、分組和提供內容
- 支援階層向下鑽研情景

**事實表** - 儲存可測量的商務事件：
- 銷售交易、網站點擊、製造事件
- 包含指向維度表的外鍵
- 用於彙總的數值量值
- 行數較多（通常隨時間增長）
- 代表特定的粒度/詳細等級

```
範例星狀綱要結構：

DimProduct (維度)          FactSales (事實)           DimCustomer (維度)
├── ProductKey (PK)        ├── SalesKey (PK)          ├── CustomerKey (PK)
├── ProductName            ├── ProductKey (FK)        ├── CustomerName
├── Category               ├── CustomerKey (FK)       ├── CustomerType
├── SubCategory            ├── DateKey (FK)           ├── Region
└── UnitPrice              ├── SalesAmount            └── RegistrationDate
                           ├── Quantity
DimDate (維度)             └── DiscountAmount
├── DateKey (PK)
├── Date
├── Year
├── Quarter
├── Month
└── DayOfWeek
```

### 2. 表格設計最佳實踐

#### 維度表設計
```
✅ 應該做：
- 使用代理鍵（自動遞增整數）作為主鍵
- 包含用於整合目的的商務鍵
- 建立階層式屬性（類別 > 子類別 > 產品）
- 使用描述性名稱和適當的資料類型
- 為遺失的維度資料包含「未知」記錄
- 保持維度表相對較窄（聚焦屬性）

❌ 不應該做：
- 在大型模型中使用自然商務鍵作為主鍵
- 將事實和維度特性混在同一個表格中
- 建立不必要的寬維度表
- 留下未妥善處理的遺失值
```

#### 事實表設計
```
✅ 應該做：
- 在所需的最細粒度等級儲存資料
- 使用與維度表鍵相符的外鍵
- 僅包含數值、可測量的欄位
- 在所有事實表列保持一致的粒度
- 使用適當的資料類型（貨幣使用十進位、計數使用整數）

❌ 不應該做：
- 包含描述性文字欄位（這些應該在維度中）
- 混合同一個事實表中的不同粒度
- 儲存可在查詢時計算的計算值
- 當代理鍵會更簡單時使用複合鍵
```

## 關係設計與管理

### 1. 關係類型與最佳實踐

#### 一對多關係（標準模式）
```
設定：
- 從維度（一方）到事實（多方）
- 單向篩選（維度篩選事實）
- 標記為「假設參考完整性」以提高 DirectQuery 效能

範例：
DimProduct (1) ← ProductKey → (*) FactSales
DimCustomer (1) ← CustomerKey → (*) FactSales
DimDate (1) ← DateKey → (*) FactSales
```

#### 多對多關係（謹慎使用）
```
使用時機：
✅ 真正的多對多商務關係
✅ 當橋接表模式不可行時
✅ 用於進階分析情景

最佳實踐：
- 盡可能建立明確的橋接表
- 使用低基數的關係欄位
- 仔細監控效能影響
- 清楚地記錄商務規則

使用橋接表的範例：
DimCustomer (1) ← CustomerKey → (*) BridgeCustomerAccount (*) ← AccountKey → (1) DimAccount
```

#### 一對一關係（罕見）
```
使用時機：
- 使用額外屬性擴展維度表
- 退化維度情景
- 將個人識別資訊與營運資料分開

實作方式：
- 考慮如果可能就合併為單一表格
- 用於安全性/隱私分離
- 維護參考完整性
```

### 2. 關係設定指南
```
篩選方向：
✅ 單向：預設選擇，效能最佳
✅ 雙向：僅在商務邏輯需要交叉篩選時
❌ 避免：迴圈關係路徑

交叉篩選方向：
- 維度到事實：始終單向
- 事實到事實：避免直接關係，使用共享維度
- 維度到維度：僅當商務邏輯需要時

參考完整性：
✅ 對於資料品質有保證的 DirectQuery 來源啟用
✅ 透過使用 INNER JOIN 改進查詢效能
❌ 如果來源資料有孤立記錄，不要啟用
```

## 儲存模式最佳化

### 1. 匯入模式最佳實踐
```
何時使用匯入模式：
✅ 資料大小在容量限制範圍內
✅ 需要複雜的分析計算
✅ 穩定資料集的歷史資料分析
✅ 需要最佳查詢效能

最佳化策略：
- 移除不必要的欄位和列
- 使用適當的資料類型
- 盡可能預先彙總資料
- 為大型資料集實作漸進式重新整理
- 最佳化 Power Query 轉換
```

#### 匯入資料縮減技術
```
垂直篩選（欄位縮減）：
✅ 移除未在報表或關係中使用的欄位
✅ 移除可以用 DAX 計算的計算欄位
✅ 移除只在 Power Query 中使用的中間欄位
✅ 最佳化資料類型（整數 vs. 十進位、日期 vs. 日期時間）

水平篩選（列縮減）：
✅ 篩選到相關的時間期間（例如過去 3 年的資料）
✅ 篩選到相關的商務實體（作用中客戶、特定地區）
✅ 移除測試、無效或已取消的交易
✅ 實施適當的資料封存策略

資料類型最佳化：
文字 → 數值：盡可能將代碼轉換為整數
日期時間 → 日期：不需要時間時使用日期類型
十進位 → 整數：整數量值使用整數
高精度 → 低精度：符合商務需求
```

### 2. DirectQuery 模式最佳實踐
```
何時使用 DirectQuery 模式：
✅ 資料超過匯入容量限制
✅ 需要即時資料
✅ 安全性/合規要求資料保留在來源
✅ 與營運系統整合

最佳化要求：
- 最佳化來源資料庫效能
- 在來源表上建立適當的索引
- 最小化複雜的 DAX 計算
- 使用簡單的量值和彙總
- 限制每個報表頁面的視覺效果數量
- 實作查詢縮減技術
```

#### DirectQuery 效能最佳化
```
資料庫最佳化：
✅ 在經常篩選的欄位上建立索引
✅ 在關係主鍵欄位上建立索引
✅ 為複雜聯結使用具體化檢視
✅ 實施適當的資料庫維護
✅ 考慮用於分析工作負載的資料行存放區索引

DirectQuery 模型設計：
✅ 保持 DAX 量值簡單
✅ 避免在大型表上建立計算欄位
✅ 嚴格遵循星狀綱要設計
✅ 最小化跨表操作
✅ 在來源預先彙總資料時

查詢效能：
✅ 在報表設計中盡早套用篩選
✅ 使用適當的視覺效果類型
✅ 限制高基數篩選
✅ 監控並最佳化緩慢查詢
```

### 3. 複合模型設計
```
何時使用複合模型：
✅ 結合歷史資料（匯入）與即時資料（DirectQuery）
✅ 使用額外資料來源擴展現有模型
✅ 平衡效能和資料新鮮度要求
✅ 整合多個 DirectQuery 來源

儲存模式選擇：
匯入：小型維度表、歷史彙總事實
DirectQuery：大型事實表、即時營運資料
雙重：需要與匯入和 DirectQuery 事實搭配使用的維度表
混合：結合歷史資料（匯入）和最近資料（DirectQuery）的事實表
```

#### 雙重儲存模式策略
```
雙重模式的使用時機：
✅ 與匯入和 DirectQuery 事實相關的維度表
✅ 小型、緩變的參考表
✅ 需要靈活查詢的查閱表

設定：
- 將維度表設定為雙重模式
- Power BI 自動選擇最佳查詢路徑
- 維護維度資料的單一副本
- 實現跨來源關係的高效率
```

## 進階建模模式

### 1. 日期表設計
```
必要的日期表屬性：
✅ 連續日期範圍（無間隙）
✅ 在 Power BI 中標記為日期表
✅ 包含標準階層（年 > 季 > 月 > 日）
✅ 加入商務特定欄位（會計年度、工作日、假日）
✅ 為日期欄位使用日期資料類型

日期表實作：
DateKey (整數)：20240315 (YYYYMMDD 格式)
Date (日期)：2024-03-15
Year (整數)：2024
Quarter (文字)：Q1 2024
Month (文字)：March 2024
MonthNumber (整數)：3
DayOfWeek (文字)：Friday
IsWorkingDay (布林值)：TRUE
FiscalYear (整數)：2024
FiscalQuarter (文字)：FY2024 Q3
```

### 2. 緩變維度 (SCD)
```
第 1 類 SCD（覆寫）：
- 使用新值更新現有記錄
- 遺失歷史內容
- 簡單實作和維護
- 用於非關鍵屬性變更

第 2 類 SCD（保留歷史）：
- 為變更建立新記錄
- 維護完整的歷史記錄
- 包含生效日期範圍
- 使用代理鍵進行唯一識別

實作模式：
CustomerKey (代理)：1, 2, 3, 4
CustomerID (商務)：101, 101, 102, 103
CustomerName：「John Doe」、「John Smith」、「Jane Doe」、「Bob Johnson」
EffectiveDate：2023-01-01、2024-01-01、2023-01-01、2023-01-01
ExpirationDate：2023-12-31、9999-12-31、9999-12-31、9999-12-31
IsCurrent：FALSE、TRUE、TRUE、TRUE
```

### 3. 角色扮演維度
```
情景：日期表用於訂單日期、出貨日期、交付日期

實作選項：

選項 1：多重關係（推薦）
- 單一日期表與事實多重關係
- 一個作用中關係（訂單日期）
- 非作用中關係用於出貨日期和交付日期
- 在 DAX 量值中使用 USERELATIONSHIP

選項 2：多個日期表
- 分開表格：OrderDate、ShipDate、DeliveryDate
- 每一個有專屬關係
- 對報表作者來說更直覺
- 由於複製造成模型大小較大

DAX 實作：
Sales by Order Date = [Total Sales]  // 使用作用中關係
Sales by Ship Date = CALCULATE([Total Sales], USERELATIONSHIP(FactSales[ShipDate], DimDate[Date]))
Sales by Delivery Date = CALCULATE([Total Sales], USERELATIONSHIP(FactSales[DeliveryDate], DimDate[Date]))
```

### 4. 多對多的橋接表
```
情景：學生可以在多個課程中，課程可以有多個學生

橋接表設計：
DimStudent (1) ← StudentKey → (*) BridgeStudentCourse (*) ← CourseKey → (1) DimCourse

橋接表結構：
StudentCourseKey (PK)：代理鍵
StudentKey (FK)：DimStudent 的參考
CourseKey (FK)：DimCourse 的參考
EnrollmentDate：額外內容
Grade：額外內容
Status：作用中、已完成、已退出

關係設定：
- DimStudent 到 BridgeStudentCourse：一對多
- BridgeStudentCourse 到 DimCourse：多對一
- 將一個關係設定為雙向以進行篩選傳播
- 隱藏報表檢視中的橋接表
```

## 效能最佳化策略

### 1. 模型大小最佳化
```
欄位最佳化：
✅ 完全移除未使用的欄位
✅ 使用最小的適當資料類型
✅ 將高基數文字轉換為整數並使用查閱表
✅ 移除冗餘的計算欄位

列最佳化：
✅ 篩選到商務相關的時間期間
✅ 移除無效、測試或已取消的交易
✅ 適當地封存歷史資料
✅ 為成長資料集使用漸進式重新整理

彙總策略：
✅ 預先計算常見彙總
✅ 用於高層級報告的摘要表
✅ 在 Premium 中實作自動彙總
✅ 考慮用於複雜分析要求的 OLAP 立方體
```

### 2. 關係效能
```
鍵選擇：
✅ 整數鍵優於文字鍵
✅ 傾向代理鍵而非自然鍵
✅ 確保來源資料的參考完整性
✅ 在主鍵欄位上建立適當的索引

基數最佳化：
✅ 設定正確的關係基數
✅ 在適當時使用「假設參考完整性」
✅ 最小化雙向關係
✅ 盡可能避免多對多關係

交叉篩選策略：
✅ 使用單向篩選作為預設
✅ 僅在需要時啟用雙向
✅ 測試交叉篩選的效能影響
✅ 記錄雙向關係的商務原因
```

### 3. 查詢效能模式
```
高效模型模式：
✅ 適當的星狀綱要實作
✅ 標準化維度表
✅ 非標準化事實表
✅ 相關表的一致粒度
✅ 計算表和欄位的適當使用

查詢最佳化：
✅ 預先篩選大型資料集
✅ 為資料使用適當的視覺效果類型
✅ 最小化報表中複雜的 DAX
✅ 有效利用模型關係
✅ 考慮用於大型即時資料集的 DirectQuery
```

## 安全性與治理

### 1. 列級安全性 (RLS)
```
實作模式：

使用者型安全性：
[UserEmail] = USERPRINCIPALNAME()

角色型安全性：
VAR UserRole =
    LOOKUPVALUE(
        UserRoles[Role],
        UserRoles[Email],
        USERPRINCIPALNAME()
    )
RETURN
    Customers[Region] = UserRole

動態安全性：
LOOKUPVALUE(
    UserRegions[Region],
    UserRegions[Email],
    USERPRINCIPALNAME()
) = Customers[Region]

最佳實踐：
✅ 使用不同的使用者帳戶進行測試
✅ 保持安全邏輯簡單高效
✅ 清楚地記錄安全要求
✅ 使用安全角色，而非個別使用者篩選
✅ 考慮複雜 RLS 的效能影響
```

### 2. 資料治理
```
文件要求：
✅ 所有量值的商務定義
✅ 資料譜系和來源系統對應
✅ 重新整理排程和相依性
✅ 安全性和存取控制文件
✅ 變更管理程序

資料品質：
✅ 實作資料驗證規則
✅ 監控資料完整性
✅ 適當地處理遺失值
✅ 驗證商務規則實作
✅ 定期資料品質評估

版本控制：
✅ Power BI 檔案的原始檔控制
✅ 環境晉升程序
✅ 變更追蹤和核准程序
✅ 備份和復原程序
```

## 測試與驗證架構

### 1. 模型測試檢查清單
```
功能測試：
□ 所有關係正確運作
□ 量值計算預期值
□ 篩選適當傳播
□ 安全規則如設計般運作
□ 資料重新整理成功完成

效能測試：
□ 模型在可接受時間內載入
□ 查詢在 SLA 要求內執行
□ 視覺效果互動有回應
□ 記憶體使用在容量限制內
□ 已完成併發使用者負載測試

資料品質測試：
□ 沒有遺失外鍵關係
□ 量值總計與來源系統相符
□ 日期範圍完整且連續
□ 安全篩選產生正確結果
□ 商務規則已正確實作
```

### 2. 驗證程序
```
商務驗證：
✅ 與來源系統比較報表總計
✅ 使用商務使用者驗證複雜計算
✅ 測試邊界狀況和邊界條件
✅ 確認商務邏輯實作
✅ 驗證報表在不同篩選中的準確性

技術驗證：
✅ 使用實際資料量進行效能測試
✅ 併發使用者測試
✅ 使用不同使用者角色進行安全測試
✅ 資料重新整理測試和監控
✅ 災害復原測試
```

## 要避免的常見反面模式

### 1. 綱要反面模式
```
❌ 雪花綱要（除非必要）：
- 多個標準化維度表
- 複雜的關係鏈
- 降低查詢效能
- 對商務使用者更複雜

❌ 單一大型表格：
- 混合事實和維度
- 極端非標準化
- 難以維護和擴展
- 分析查詢效能不佳

❌ 多個事實表具有直接關係：
- 事實之間的多對多
- 複雜的篩選傳播
- 難以維護一致性
- 更好的做法是使用共享維度
```

### 2. 關係反面模式
```
❌ 到處使用雙向關係：
- 效能影響
- 不可預測的篩選行為
- 維護複雜性
- 應該是例外，不是規則

❌ 沒有商務正當理由的多對多關係：
- 通常表示遺失維度
- 可能隱藏資料品質問題
- 複雜的除錯和維護
- 橋接表通常是更好的解決方案

❌ 迴圈關係：
- 模糊的篩選路徑
- 不可預測的結果
- 困難的除錯
- 總是透過適當的設計避免
```

## 進階資料建模模式

### 1. 緩變維度實作
```powerquery
// 第 1 類 SCD：用於基於雜湊的變更偵測的 Power Query 實作
let
    Source = Source,

    #"Added custom" = Table.TransformColumnTypes(
        Table.AddColumn(Source, "Hash", each Binary.ToText(
            Text.ToBinary(
                Text.Combine(
                    List.Transform({[FirstName],[LastName],[Region]}, each if _ = null then "" else _),
                "|")),
            BinaryEncoding.Hex)
        ),
        {{"Hash", type text}}
    ),

    #"Marked key columns" = Table.AddKey(#"Added custom", {"Hash"}, false),

    #"Merged queries" = Table.NestedJoin(
        #"Marked key columns",
        {"Hash"},
        ExistingDimRecords,
        {"Hash"},
        "ExistingDimRecords",
        JoinKind.LeftOuter
    ),

    #"Expanded ExistingDimRecords" = Table.ExpandTableColumn(
        #"Merged queries",
        "ExistingDimRecords",
        {"Count"},
        {"Count"}
    ),

    #"Filtered rows" = Table.SelectRows(#"Expanded ExistingDimRecords", each ([Count] = null)),

    #"Removed columns" = Table.RemoveColumns(#"Filtered rows", {"Count"})
in
    #"Removed columns"
```

### 2. 具有查詢折疊的漸進式重新整理
```powerquery
// 最佳化的漸進式重新整理模式
let
  Source = Sql.Database("server","database"),
  Data  = Source{[Schema="dbo",Item="FactInternetSales"]}[Data],
  FilteredByStart = Table.SelectRows(Data, each [OrderDateKey] >= Int32.From(DateTime.ToText(RangeStart,[Format="yyyyMMdd"]))),
  FilteredByEnd = Table.SelectRows(FilteredByStart, each [OrderDateKey] < Int32.From(DateTime.ToText(RangeEnd,[Format="yyyyMMdd"])))
in
  FilteredByEnd
```

### 3. 語義連結整合
```python
# 在 Python 中使用 Power BI 語義模型
import sempy.fabric as fabric
from sempy.relationships import plot_relationship_metadata

relationships = fabric.list_relationships("my_dataset")
plot_relationship_metadata(relationships)
```

### 4. 進階分區策略
```json
// 具有時間篩選的 TMSL 分區
"partition": {
      "name": "Sales2019",
      "mode": "import",
      "source": {
        "type": "m",
        "expression": [
          "let",
          "    Source = SqlDatabase,",
          "    dbo_Sales = Source{[Schema=\"dbo\",Item=\"Sales\"]}[Data],",
          "    FilteredRows = Table.SelectRows(dbo_Sales, each [OrderDateKey] >= 20190101 and [OrderDateKey] <= 20191231)",
          "in",
          "    FilteredRows"
        ]
      }
}
```

記住：始終與商務使用者驗證模型設計，並使用實際的資料量和使用模式進行測試。使用 Power BI 的內建工具（如效能分析器和 DAX Studio）進行最佳化和除錯。
