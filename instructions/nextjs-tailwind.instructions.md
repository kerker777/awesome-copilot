---
description: 'Next.js + Tailwind development standards and instructions'
applyTo: '**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.css'
---

# Next.js + Tailwind 開發說明

高品質 Next.js 應用程式的開發說明，採用 Tailwind CSS 樣式與 TypeScript。

## 專案背景

- 最新版 Next.js（App Router）
- TypeScript 型別安全性
- Tailwind CSS 樣式設計

## 開發標準

### 架構
- App Router 搭配伺服器元件與用戶端元件
- 按功能/網域分組路由
- 實作適當的錯誤邊界
- 預設使用 React 伺服器元件
- 盡可能運用靜態最佳化

### TypeScript
- 啟用嚴格模式
- 清晰的型別定義
- 透過型別守衛進行適當的錯誤處理
- 使用 Zod 進行執行時期的型別驗證

### 樣式設計
- Tailwind CSS 搭配一致的色彩配置
- 響應式設計模式
- 深色模式支援
- 遵循容器查詢的最佳實踐
- 保持語義 HTML 結構

### 狀態管理
- 使用 React 伺服器元件管理伺服器狀態
- 使用 React hooks 管理用戶端狀態
- 適當的載入與錯誤狀態
- 在適當的地方進行樂觀更新

### 資料取得
- 使用伺服器元件直接查詢資料庫
- 使用 React Suspense 處理載入狀態
- 適當的錯誤處理與重試邏輯
- 快取失效策略

### 安全性
- 輸入驗證與淨化
- 適當的身份驗證檢查
- CSRF 保護
- 速率限制實作
- 安全的 API 路由處理

### 效能
- 使用 next/image 最佳化影像
- 使用 next/font 最佳化字型
- 路由預取
- 適當的程式碼分割
- 套件大小最佳化

## 實作流程
1. 規劃元件層級結構
2. 定義型別與介面
3. 實作伺服器端邏輯
4. 構建用戶端元件
5. 新增適當的錯誤處理
6. 實作響應式樣式
7. 新增載入狀態
8. 撰寫測試
