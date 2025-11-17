---
description: '根據 Microsoft 指導的全面 Power BI DAX 最佳實踐與模式，用於建立有效率、易於維護且效能優良的 DAX 公式。'
applyTo: '**/*.{pbix,dax,md,txt}'
---

# Power BI DAX 最佳實踐

## 概述
本文件根據 Microsoft 官方指導與最佳實踐，提供在 Power BI 中撰寫有效率、易於維護且效能優良的 DAX (資料分析運算式) 公式的全面指引。

## DAX 核心原則

### 1. 公式結構與變數
始終使用變數以提升效能、可讀性與偵錯能力：

```dax
// ✅ PREFERRED: Using variables for clarity and performance
Sales YoY Growth % =
VAR CurrentSales = [Total Sales]
VAR PreviousYearSales = 
    CALCULATE(
        [Total Sales],
        SAMEPERIODLASTYEAR('Date'[Date])
    )
RETURN
    DIVIDE(CurrentSales - PreviousYearSales, PreviousYearSales)

// ❌ AVOID: Repeated calculations without variables  
Sales YoY Growth % =
DIVIDE(
    [Total Sales] - CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date])),
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
)
```

**變數的主要優點：**
- **效能**：計算只評估一次並快取
- **可讀性**：複雜公式變得自我說明
- **偵錯**：可暫時傳回變數值進行測試
- **可維護性**：修改只需在一個地方進行

### 2. 正確的參考語法
遵循 Microsoft 推薦的欄位與量值參考模式：

```dax
// ✅ ALWAYS fully qualify column references
Customer Count = 
DISTINCTCOUNT(Sales[CustomerID])

Profit Margin = 
DIVIDE(
    SUM(Sales[Profit]),
    SUM(Sales[Revenue])
)

// ✅ NEVER fully qualify measure references
YTD Sales Growth = 
DIVIDE([YTD Sales] - [YTD Sales PY], [YTD Sales PY])

// ❌ AVOID: Unqualified column references
Customer Count = DISTINCTCOUNT([CustomerID])  // Ambiguous

// ❌ AVOID: Fully qualified measure references
Growth Rate = DIVIDE(Sales[Total Sales] - Sales[Total Sales PY], Sales[Total Sales PY])  // Breaks if measure moves
```

### 3. 錯誤處理策略
使用適當的模式實現穩健的錯誤處理：

```dax
// ✅ PREFERRED: Use DIVIDE function for safe division
Profit Margin = 
DIVIDE([Total Profit], [Total Revenue])

// ✅ PREFERRED: Use defensive strategies in model design
Average Order Value = 
VAR TotalOrders = COUNTROWS(Orders)
VAR TotalRevenue = SUM(Orders[Amount])
RETURN
    IF(TotalOrders > 0, DIVIDE(TotalRevenue, TotalOrders))

// ❌ AVOID: ISERROR and IFERROR functions (performance impact)
Profit Margin = 
IFERROR([Total Profit] / [Total Revenue], BLANK())

// ❌ AVOID: Complex error handling that could be prevented
Unsafe Calculation = 
IF(
    OR(
        ISBLANK([Revenue]),
        [Revenue] = 0
    ),
    BLANK(),
    [Profit] / [Revenue]
)
```

## DAX 函式類別與最佳實踐

### 聚合函式
```dax
// Use appropriate aggregation functions for performance
Customer Count = DISTINCTCOUNT(Sales[CustomerID])  // ✅ For unique counts
Order Count = COUNTROWS(Orders)                    // ✅ For row counts
Average Deal Size = AVERAGE(Sales[DealValue])      // ✅ For averages

// Avoid COUNT when COUNTROWS is more appropriate
// ❌ COUNT(Sales[OrderID]) - slower for counting rows
// ✅ COUNTROWS(Sales) - faster and more explicit
```

### 篩選與內容函式
```dax
// Efficient use of CALCULATE with multiple filters
High Value Customers = 
CALCULATE(
    DISTINCTCOUNT(Sales[CustomerID]),
    Sales[OrderValue] > 1000,
    Sales[OrderDate] >= DATE(2024,1,1)
)

// Proper context modification patterns
Same Period Last Year = 
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR('Date'[Date])
)

// Using FILTER appropriately (avoid as filter argument)
// ✅ PREFERRED: Direct filter expression
High Value Orders = 
CALCULATE(
    [Total Sales],
    Sales[OrderValue] > 1000
)

// ❌ AVOID: FILTER as filter argument (unless table manipulation needed)
High Value Orders = 
CALCULATE(
    [Total Sales],
    FILTER(Sales, Sales[OrderValue] > 1000)
)
```

### 時間智能模式
```dax
// Standard time intelligence measures
YTD Sales = 
CALCULATE(
    [Total Sales],
    DATESYTD('Date'[Date])
)

MTD Sales = 
CALCULATE(
    [Total Sales],
    DATESMTD('Date'[Date])
)

// Moving averages with proper date handling
3-Month Moving Average = 
VAR CurrentDate = MAX('Date'[Date])
VAR StartDate = EDATE(CurrentDate, -2)
RETURN
    CALCULATE(
        DIVIDE([Total Sales], 3),
        DATESBETWEEN(
            'Date'[Date],
            StartDate,
            CurrentDate
        )
    )

// Quarter over quarter growth
QoQ Growth = 
VAR CurrentQuarter = [Total Sales]
VAR PreviousQuarter = 
    CALCULATE(
        [Total Sales],
        DATEADD('Date'[Date], -1, QUARTER)
    )
RETURN
    DIVIDE(CurrentQuarter - PreviousQuarter, PreviousQuarter)
```

### 進階 DAX 模式
```dax
// Ranking with proper context
Product Rank = 
RANKX(
    ALL(Product[ProductName]),
    [Total Sales],
    ,
    DESC,
    DENSE
)

// Running totals
Running Total = 
CALCULATE(
    [Total Sales],
    FILTER(
        ALL('Date'[Date]),
        'Date'[Date] <= MAX('Date'[Date])
    )
)

// ABC Analysis (Pareto)
ABC Classification = 
VAR CurrentProductSales = [Total Sales]
VAR TotalSales = CALCULATE([Total Sales], ALL(Product))
VAR RunningTotal = 
    CALCULATE(
        [Total Sales],
        FILTER(
            ALL(Product),
            [Total Sales] >= CurrentProductSales
        )
    )
VAR PercentageOfTotal = DIVIDE(RunningTotal, TotalSales)
RETURN
    SWITCH(
        TRUE(),
        PercentageOfTotal <= 0.8, "A",
        PercentageOfTotal <= 0.95, "B",
        "C"
    )
```

## 效能最佳化技巧

### 1. 有效率的變數使用
```dax
// ✅ Store expensive calculations in variables
Complex Measure = 
VAR BaseCalculation = 
    CALCULATE(
        SUM(Sales[Amount]),
        FILTER(
            Product,
            Product[Category] = "Electronics"
        )
    )
VAR PreviousYear = 
    CALCULATE(
        BaseCalculation,
        SAMEPERIODLASTYEAR('Date'[Date])
    )
RETURN
    DIVIDE(BaseCalculation - PreviousYear, PreviousYear)
```

### 2. 內容轉換最佳化
```dax
// ✅ Minimize context transitions in iterator functions
Total Product Profit =
SUMX(
    Product,
    Product[UnitPrice] - Product[UnitCost]
)

// ❌ Avoid unnecessary calculated columns in large tables
// Create in Power Query instead when possible
```

### 3. 有效率的篩選模式
```dax
// ✅ Use table expressions efficiently
Top 10 Customers = 
CALCULATE(
    [Total Sales],
    TOPN(
        10,
        ALL(Customer[CustomerName]),
        [Total Sales]
    )
)

// ✅ Leverage relationship filtering
Sales with Valid Customers = 
CALCULATE(
    [Total Sales],
    FILTER(
        Customer,
        NOT(ISBLANK(Customer[CustomerName]))
    )
)
```

## 常見要避免的 DAX 反模式

### 1. 效能反模式
```dax
// ❌ AVOID: Nested CALCULATE functions
Inefficient Nested = 
CALCULATE(
    CALCULATE(
        [Total Sales],
        Product[Category] = "Electronics"
    ),
    'Date'[Year] = 2024
)

// ✅ PREFERRED: Single CALCULATE with multiple filters
Efficient Single = 
CALCULATE(
    [Total Sales],
    Product[Category] = "Electronics",
    'Date'[Year] = 2024
)

// ❌ AVOID: Converting BLANK to zero unnecessarily
Sales with Zero = 
IF(ISBLANK([Total Sales]), 0, [Total Sales])

// ✅ PREFERRED: Keep BLANK as BLANK for better visual behavior
Sales = SUM(Sales[Amount])
```

### 2. 可讀性反模式
```dax
// ❌ AVOID: Complex nested expressions without variables
Complex Without Variables = 
DIVIDE(
    CALCULATE(SUM(Sales[Revenue]), Sales[Date] >= DATE(2024,1,1)) - 
    CALCULATE(SUM(Sales[Revenue]), Sales[Date] >= DATE(2023,1,1), Sales[Date] < DATE(2024,1,1)),
    CALCULATE(SUM(Sales[Revenue]), Sales[Date] >= DATE(2023,1,1), Sales[Date] < DATE(2024,1,1))
)

// ✅ PREFERRED: Clear variable-based structure
Year Over Year Growth = 
VAR CurrentYear = 
    CALCULATE(
        SUM(Sales[Revenue]),
        Sales[Date] >= DATE(2024,1,1)
    )
VAR PreviousYear = 
    CALCULATE(
        SUM(Sales[Revenue]),
        Sales[Date] >= DATE(2023,1,1),
        Sales[Date] < DATE(2024,1,1)
    )
RETURN
    DIVIDE(CurrentYear - PreviousYear, PreviousYear)
```

## DAX 偵錯與測試策略

### 1. 基於變數的偵錯
```dax
// Use this pattern for step-by-step debugging
Debug Measure = 
VAR Step1 = CALCULATE([Sales], 'Date'[Year] = 2024)
VAR Step2 = CALCULATE([Sales], 'Date'[Year] = 2023)  
VAR Step3 = Step1 - Step2
VAR Step4 = DIVIDE(Step3, Step2)
RETURN
    -- Return different variables for testing:
    -- Step1  -- Test current year sales
    -- Step2  -- Test previous year sales  
    -- Step3  -- Test difference calculation
    Step4     -- Final result
```

### 2. 測試模式
```dax
// Include data validation in measures
Validated Measure =
VAR Result = [Complex Calculation]
VAR IsValid =
    Result >= 0 &&
    Result <= 1 &&
    NOT(ISBLANK(Result))
RETURN
    IF(IsValid, Result, BLANK())
```

## 量值組織與命名

### 1. 命名慣例
```dax
// Use descriptive, consistent naming
Total Sales = SUM(Sales[Amount])
Total Sales YTD = CALCULATE([Total Sales], DATESYTD('Date'[Date]))
Total Sales PY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
Sales Growth % = DIVIDE([Total Sales] - [Total Sales PY], [Total Sales PY])

// Prefix for measure categories
KPI - Revenue Growth = [Sales Growth %]
Calc - Days Since Last Order = DATEDIFF(MAX(Orders[OrderDate]), TODAY(), DAY)
Base - Order Count = COUNTROWS(Orders)
```

### 2. 量值相依性
```dax
// Build measures hierarchically for reusability
// Base measures
Revenue = SUM(Sales[Revenue])
Cost = SUM(Sales[Cost])

// Derived measures
Profit = [Revenue] - [Cost]
Margin % = DIVIDE([Profit], [Revenue])

// Advanced measures
Profit YTD = CALCULATE([Profit], DATESYTD('Date'[Date]))
Margin Trend = [Margin %] - CALCULATE([Margin %], PREVIOUSMONTH('Date'[Date]))
```

## 模型整合最佳實踐

### 1. 星狀結構的使用
```dax
// Leverage proper relationships
Sales by Category = 
CALCULATE(
    [Total Sales],
    Product[Category] = "Electronics"
)

// Use dimension tables for filtering
Regional Sales = 
CALCULATE(
    [Total Sales],
    Geography[Region] = "North America"
)
```

### 2. 處理缺失的關聯
```dax
// When direct relationships don't exist
Cross Table Analysis =
VAR CustomerList = VALUES(Customer[CustomerID])
RETURN
    CALCULATE(
        [Total Sales],
        FILTER(
            Sales,
            Sales[CustomerID] IN CustomerList
        )
    )
```

## 進階 DAX 概念

### 1. 列內容與篩選內容
```dax
// Understanding context differences
Row Context Example = 
SUMX(
    Sales,
    Sales[Quantity] * Sales[UnitPrice]  // Row context
)

Filter Context Example = 
CALCULATE(
    [Total Sales],  // Filter context
    Product[Category] = "Electronics"
)
```

### 2. 內容轉換
```dax
// When row context becomes filter context
Sales Per Product =
SUMX(
    Product,
    CALCULATE([Total Sales])  // Context transition happens here
)
```

### 3. 擴展欄位與計算表格
```dax
// Use for complex analytical scenarios
Product Analysis = 
ADDCOLUMNS(
    Product,
    "Total Sales", CALCULATE([Total Sales]),
    "Rank", RANKX(ALL(Product), CALCULATE([Total Sales])),
    "Category Share", DIVIDE(
        CALCULATE([Total Sales]),
        CALCULATE([Total Sales], ALL(Product[ProductName]))
    )
)
```

### 4. 進階時間智能模式
```dax
// Multi-period comparisons with calculation groups
// Example showing how to create dynamic time calculations
Dynamic Period Comparison = 
VAR CurrentPeriodValue = 
    CALCULATE(
        [Sales],
        'Time Intelligence'[Time Calculation] = "Current"
    )
VAR PreviousPeriodValue = 
    CALCULATE(
        [Sales],
        'Time Intelligence'[Time Calculation] = "PY"
    )
VAR MTDCurrent = 
    CALCULATE(
        [Sales],
        'Time Intelligence'[Time Calculation] = "MTD"
    )
VAR MTDPrevious = 
    CALCULATE(
        [Sales],
        'Time Intelligence'[Time Calculation] = "PY MTD"
    )
RETURN
    DIVIDE(MTDCurrent - MTDPrevious, MTDPrevious)

// Working with fiscal years and custom calendars
Fiscal YTD Sales = 
VAR FiscalYearStart = 
    DATE(
        IF(MONTH(MAX('Date'[Date])) >= 7, YEAR(MAX('Date'[Date])), YEAR(MAX('Date'[Date])) - 1),
        7,
        1
    )
VAR FiscalYearEnd = MAX('Date'[Date])
RETURN
    CALCULATE(
        [Total Sales],
        DATESBETWEEN(
            'Date'[Date],
            FiscalYearStart,
            FiscalYearEnd
        )
    )
```

### 5. 進階效能最佳化技巧
```dax
// Optimized running totals
Running Total Optimized = 
VAR CurrentDate = MAX('Date'[Date])
RETURN
    CALCULATE(
        [Total Sales],
        FILTER(
            ALL('Date'[Date]),
            'Date'[Date] <= CurrentDate
        )
    )

// Efficient ABC Analysis using RANKX
ABC Classification Advanced = 
VAR ProductRank = 
    RANKX(
        ALL(Product[ProductName]),
        [Total Sales],
        ,
        DESC,
        DENSE
    )
VAR TotalProducts = COUNTROWS(ALL(Product[ProductName]))
VAR ClassAThreshold = TotalProducts * 0.2
VAR ClassBThreshold = TotalProducts * 0.5
RETURN
    SWITCH(
        TRUE(),
        ProductRank <= ClassAThreshold, "A",
        ProductRank <= ClassBThreshold, "B",
        "C"
    )

// Efficient Top N with ties handling
Top N Products with Ties = 
VAR TopNValue = 10
VAR MinTopNSales = 
    CALCULATE(
        MIN([Total Sales]),
        TOPN(
            TopNValue,
            ALL(Product[ProductName]),
            [Total Sales]
        )
    )
RETURN
    IF(
        [Total Sales] >= MinTopNSales,
        [Total Sales],
        BLANK()
    )
```

### 6. 複雜分析情景
```dax
// Customer cohort analysis
Cohort Retention Rate = 
VAR CohortMonth = 
    CALCULATE(
        MIN('Date'[Date]),
        ALLEXCEPT(Sales, Sales[CustomerID])
    )
VAR CurrentMonth = MAX('Date'[Date])
VAR MonthsFromCohort = 
    DATEDIFF(CohortMonth, CurrentMonth, MONTH)
VAR CohortCustomers = 
    CALCULATE(
        DISTINCTCOUNT(Sales[CustomerID]),
        'Date'[Date] = CohortMonth
    )
VAR ActiveCustomersInMonth = 
    CALCULATE(
        DISTINCTCOUNT(Sales[CustomerID]),
        'Date'[Date] = CurrentMonth,
        FILTER(
            Sales,
            CALCULATE(
                MIN('Date'[Date]),
                ALLEXCEPT(Sales, Sales[CustomerID])
            ) = CohortMonth
        )
    )
RETURN
    DIVIDE(ActiveCustomersInMonth, CohortCustomers)

// Market basket analysis
Product Affinity Score = 
VAR CurrentProduct = SELECTEDVALUE(Product[ProductName])
VAR RelatedProduct = SELECTEDVALUE('Related Product'[ProductName])
VAR TransactionsWithBoth = 
    CALCULATE(
        DISTINCTCOUNT(Sales[TransactionID]),
        Sales[ProductName] = CurrentProduct
    ) +
    CALCULATE(
        DISTINCTCOUNT(Sales[TransactionID]),
        Sales[ProductName] = RelatedProduct
    ) -
    CALCULATE(
        DISTINCTCOUNT(Sales[TransactionID]),
        Sales[ProductName] = CurrentProduct,
        CALCULATE(
            COUNTROWS(Sales),
            Sales[ProductName] = RelatedProduct,
            Sales[TransactionID] = EARLIER(Sales[TransactionID])
        ) > 0
    )
VAR TotalTransactions = DISTINCTCOUNT(Sales[TransactionID])
RETURN
    DIVIDE(TransactionsWithBoth, TotalTransactions)
```

### 7. 進階偵錯與分析
```dax
// Debug measure with detailed variable inspection
Complex Measure Debug = 
VAR Step1_FilteredSales = 
    CALCULATE(
        [Sales],
        Product[Category] = "Electronics",
        'Date'[Year] = 2024
    )
VAR Step2_PreviousYear = 
    CALCULATE(
        [Sales],
        Product[Category] = "Electronics",
        'Date'[Year] = 2023
    )
VAR Step3_GrowthAbsolute = Step1_FilteredSales - Step2_PreviousYear
VAR Step4_GrowthPercentage = DIVIDE(Step3_GrowthAbsolute, Step2_PreviousYear)
VAR DebugInfo = 
    "Current: " & FORMAT(Step1_FilteredSales, "#,0") & 
    " | Previous: " & FORMAT(Step2_PreviousYear, "#,0") &
    " | Growth: " & FORMAT(Step4_GrowthPercentage, "0.00%")
RETURN
    -- Switch between these for debugging:
    -- Step1_FilteredSales    -- Test current year
    -- Step2_PreviousYear     -- Test previous year
    -- Step3_GrowthAbsolute   -- Test absolute growth
    -- DebugInfo              -- Show debug information
    Step4_GrowthPercentage    -- Final result

// Performance monitoring measure
Query Performance Monitor = 
VAR StartTime = NOW()
VAR Result = [Complex Calculation]
VAR EndTime = NOW()
VAR ExecutionTime = DATEDIFF(StartTime, EndTime, SECOND)
VAR WarningThreshold = 5 // seconds
RETURN
    IF(
        ExecutionTime > WarningThreshold,
        "⚠️ Slow: " & ExecutionTime & "s - " & Result,
        Result
    )
```

### 8. 使用複雜資料型別
```dax
// JSON parsing and manipulation
Extract JSON Value = 
VAR JSONString = SELECTEDVALUE(Data[JSONColumn])
VAR ParsedValue = 
    IF(
        NOT(ISBLANK(JSONString)),
        PATHCONTAINS(JSONString, "$.analytics.revenue"),
        BLANK()
    )
RETURN
    ParsedValue

// Dynamic measure selection
Dynamic Measure Selector = 
VAR SelectedMeasure = SELECTEDVALUE('Measure Selector'[MeasureName])
RETURN
    SWITCH(
        SelectedMeasure,
        "Revenue", [Total Revenue],
        "Profit", [Total Profit],
        "Units", [Total Units],
        "Margin", [Profit Margin %],
        BLANK()
    )
```

## DAX 公式文件

### 1. 備註最佳實踐
```dax
/* 
Business Rule: Calculate customer lifetime value based on:
- Average order value over customer lifetime
- Purchase frequency (orders per year)  
- Customer lifespan (years since first order)
- Retention probability based on last order date
*/
Customer Lifetime Value = 
VAR AvgOrderValue = 
    DIVIDE(
        CALCULATE(SUM(Sales[Amount])),
        CALCULATE(DISTINCTCOUNT(Sales[OrderID]))
    )
VAR OrdersPerYear = 
    DIVIDE(
        CALCULATE(DISTINCTCOUNT(Sales[OrderID])),
        DATEDIFF(
            CALCULATE(MIN(Sales[OrderDate])),
            CALCULATE(MAX(Sales[OrderDate])),
            YEAR
        ) + 1  -- Add 1 to avoid division by zero for customers with orders in single year
    )
VAR CustomerLifespanYears = 3  -- Business assumption: average 3-year relationship
RETURN
    AvgOrderValue * OrdersPerYear * CustomerLifespanYears
```

### 2. 版本控制與變更管理
```dax
// Include version history in measure descriptions
/*
Version History:
v1.0 - Initial implementation (2024-01-15)
v1.1 - Added null checking for edge cases (2024-02-01)  
v1.2 - Optimized performance using variables (2024-02-15)
v2.0 - Changed business logic per stakeholder feedback (2024-03-01)

Business Logic:
- Excludes returns and cancelled orders
- Uses ship date for revenue recognition
- Applies regional tax calculations
*/
```

## 測試與驗證架構

### 1. 單元測試模式
```dax
// Create test measures for validation
Test - Sales Sum = 
VAR DirectSum = SUM(Sales[Amount])
VAR MeasureResult = [Total Sales]
VAR Difference = ABS(DirectSum - MeasureResult)
RETURN
    IF(Difference < 0.01, "PASS", "FAIL: " & Difference)
```

### 2. 效能測試
```dax
// Monitor execution time for complex measures
Performance Monitor =
VAR StartTime = NOW()
VAR Result = [Complex Calculation]
VAR EndTime = NOW()
VAR Duration = DATEDIFF(StartTime, EndTime, SECOND)
RETURN
    "Result: " & Result & " | Duration: " & Duration & "s"
```

記住：請務必與業務使用者驗證 DAX 公式，確保計算符合業務需求與期望。使用 Power BI 的效能分析工具與 DAX Studio 進行效能最佳化與偵錯。