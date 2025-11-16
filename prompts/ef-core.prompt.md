---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'runCommands']
description: '取得 Entity Framework Core 的最佳實踐'
---

# Entity Framework Core 最佳實踐

您的目標是協助我在使用 Entity Framework Core 時遵循最佳實踐。

## 資料情境設計

- 保持 DbContext 類別專注且內聚
- 使用建構函式注入來注入設定選項
- 覆寫 OnModelCreating 進行流暢 API 設定
- 使用 IEntityTypeConfiguration 分離實體設定
- 對於主控台應用程式或測試，考慮使用 DbContextFactory 模式

## 實體設計

- 使用有意義的主鍵（考慮自然鍵與代理鍵）
- 實作適當的關聯性（一對一、一對多、多對多）
- 使用資料註解或流暢 API 進行約束和驗證
- 實作適當的導覽屬性
- 考慮對值物件使用自有實體類型

## 效能

- 對唯讀查詢使用 AsNoTracking()
- 使用 Skip() 和 Take() 為大型結果集實作分頁
- 需要時使用 Include() 預先載入相關實體
- 考慮使用投影（Select）僅擷取所需欄位
- 對頻繁執行的查詢使用編譯查詢
- 透過適當包含相關資料來避免 N+1 查詢問題

## 遷移

- 建立小型、專注的遷移
- 以描述性方式命名遷移
- 在應用至生產環境之前驗證遷移 SQL 腳本
- 考慮使用遷移套件進行部署
- 在適當的時候透過遷移新增資料植入

## 查詢

- 謹慎使用 IQueryable 並了解查詢何時執行
- 優先使用強類型 LINQ 查詢而非原始 SQL
- 使用適當的查詢運算子（Where、OrderBy、GroupBy）
- 考慮對複雜操作使用資料庫函數
- 實作規格模式以實現可重複使用的查詢

## 變更追蹤與儲存

- 使用適當的變更追蹤策略
- 批次處理 SaveChanges() 呼叫
- 為多使用者情境實作並行控制
- 考慮對多個操作使用交易
- 使用適當的 DbContext 生命週期（Web 應用程式使用範圍生命週期）

## 安全性

- 使用參數化查詢避免 SQL 注入
- 實作適當的資料存取權限
- 小心使用原始 SQL 查詢
- 考慮對敏感資訊進行資料加密
- 使用遷移管理資料庫使用者權限

## 測試

- 對單元測試使用記憶體內資料庫提供者
- 對整合測試使用 SQLite 建立單獨的測試情境
- 對純單元測試模擬 DbContext 和 DbSet
- 在隔離的環境中測試遷移
- 考慮對模型變更進行快照測試

在審查我的 EF Core 程式碼時，識別問題並建議遵循這些最佳實踐的改進。
