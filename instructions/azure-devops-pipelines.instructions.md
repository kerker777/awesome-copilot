---
description: 'Azure DevOps Pipeline YAML 文件的最佳實踐'
applyTo: '**/azure-pipelines.yml, **/azure-pipelines*.yml, **/*.pipeline.yml'
---

# Azure DevOps Pipeline YAML 最佳實踐

## 一般指南

- 使用一致的 YAML 語法,適當縮排(2個空格)
- 為管道、階段、作業和步驟始終包含有意義的名稱和顯示名稱
- 實作適當的錯誤處理和條件執行
- 使用變數和參數使管道可重用且易於維護
- 遵循服務連接和權限的最小權限原則
- 包含全面的記錄和診斷以進行故障排除

## 管道結構

- 使用階段組織複雜的管道以獲得更好的可視化和控制
- 使用作業對相關步驟進行分組,並在可能時啟用並行執行
- 實作階段和作業之間的適當依賴關係
- 使用範本建立可重用的管道元件
- 保持管道文件專注且模組化 - 將大型管道拆分為多個文件

## 建置最佳實踐

- 使用特定的代理程式池版本和 VM 映像以保持一致性
- 快取依賴項(npm、NuGet、Maven 等)以提升建置效能
- 使用有意義的名稱和保留政策實作適當的成品管理
- 使用建置變數儲存版本號和建置中繼資料
- 包含程式碼品質閘道(linting、測試、安全掃描)
- 確保建置可重現且與環境無關

## 測試整合

- 將單元測試作為建置過程的一部分執行
- 以標準格式(JUnit、VSTest 等)發布測試結果
- 包含程式碼覆蓋率報告和品質閘道
- 在適當的階段實作整合和端對端測試
- 使用測試影響分析(如可用)以最佳化測試執行
- 在測試失敗時快速失敗以提供快速回饋

## 安全考量

- 使用 Azure Key Vault 儲存敏感配置和密鑰
- 使用變數群組實作適當的密鑰管理
- 使用具有最小所需權限的服務連接
- 啟用安全掃描(依賴漏洞、靜態分析)
- 對生產部署實作核准閘道
- 盡可能使用託管身分而不是服務主體

## 部署策略

- 實作適當的環境晉升(dev → staging → production)
- 使用具有適當環境目標的部署作業
- 適時實作藍綠或金絲雀部署策略
- 包含回滾機制和健康檢查
- 使用基礎設施即程式碼(ARM、Bicep、Terraform)進行一致的部署
- 為每個環境實作適當的配置管理

## 變數和參數管理

- 使用變數群組在管道間共享配置
- 實作執行時參數以實現靈活的管道執行
- 根據分支或環境使用條件變數
- 保護敏感變數並將其標記為密鑰
- 記錄變數目的和預期值
- 對複雜的變數邏輯使用變數範本

## 效能最佳化

- 適時使用並行作業和矩陣策略
- 為依賴項和建置輸出實作適當的快取策略
- 當不需要完整歷史記錄時,對 Git 操作使用淺複製
- 使用多階段建置和層快取最佳化 Docker 映像建置
- 監控管道效能並最佳化瓶頸
- 有效使用管道資源觸發器

## 監控和可觀察性

- 在整個管道中包含全面的記錄
- 使用 Azure Monitor 和 Application Insights 進行部署追蹤
- 對失敗和成功實作適當的通知策略
- 包含部署健康檢查和自動回滾觸發器
- 使用管道分析識別改進機會
- 記錄管道行為和故障排除步驟

## 範本和可重用性

- 為常見模式建立管道範本
- 使用擴展範本實現完整的管道繼承
- 實作步驟範本以建立可重用的任務序列
- 對複雜的變數邏輯使用變數範本
- 適當地為範本進行版本控制以保持穩定性
- 記錄範本參數和使用範例

## 分支和觸發策略

- 為不同的分支類型實作適當的觸發器
- 使用路徑篩選器僅在相關文件變更時觸發建置
- 為 main/master 分支配置適當的 CI/CD 觸發器
- 使用拉取請求觸發器進行程式碼驗證
- 實作計劃觸發器以執行維護任務
- 考慮多儲存庫場景的資源觸發器

## 範例結構

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - docs/*
      - README.md

variables:
  - group: shared-variables
  - name: buildConfiguration
    value: 'Release'

stages:
  - stage: Build
    displayName: '建置和測試'
    jobs:
      - job: Build
        displayName: '建置應用程式'
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            displayName: '使用 .NET SDK'
            inputs:
              version: '8.x'

          - task: DotNetCoreCLI@2
            displayName: '還原依賴項'
            inputs:
              command: 'restore'
              projects: '**/*.csproj'

          - task: DotNetCoreCLI@2
            displayName: '建置應用程式'
            inputs:
              command: 'build'
              projects: '**/*.csproj'
              arguments: '--configuration $(buildConfiguration) --no-restore'

  - stage: Deploy
    displayName: '部署到預備環境'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployToStaging
        displayName: '部署到預備環境'
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  displayName: '下載 drop 成品'
                  artifact: drop
                - task: AzureWebApp@1
                  displayName: '部署到 Azure Web App'
                  inputs:
                    azureSubscription: 'staging-service-connection'
                    appType: 'webApp'
                    appName: 'myapp-staging'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
```

## 要避免的常見反模式

- 直接在 YAML 文件中硬編碼敏感值
- 使用過於廣泛的觸發器導致不必要的建置
- 在單一階段中混合建置和部署邏輯
- 未實作適當的錯誤處理和清理
- 使用已棄用的任務版本而沒有升級計劃
- 建立難以維護的單體管道
- 未使用適當的命名慣例以確保清晰
- 忽略管道安全最佳實踐
